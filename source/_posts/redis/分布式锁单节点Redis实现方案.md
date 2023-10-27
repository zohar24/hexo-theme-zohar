---
title: 分布式锁单节点Redis实现方案
top: false
author: zhaohuan
date: 2023-02-10 18:17:31
tags:
- 分布式锁
- redis
categories:
- redis
---

加锁：使用SETNX进行加锁，当该指令返回1时，说明成功获得锁
解锁：当得到锁的线程执行完任务之后，使用DEL命令释放锁，以便其他线程可以继续执行setnx命令来获得锁。

1. **死锁问题**：假设线程获取了锁之后，在执行任务的过程中挂掉，来不及显示地执行DEL命令释放锁，那么竞争该锁的线程都会执行不了，产生死锁的情况。
   解决方案：设置锁超时时间。setnx的key必须设置一个超时时间，以保证即使没有被显式释放，这把锁也要在一定时间后自动释放。可以使用expire命令设置锁超时时间

2. **非原子性问题**：setnx和expire不是原子性的操作，假设某个线程执行setnx命令，成功获得了锁，但是还没来得及执行expire命令，服务器就挂掉了，这样一来，这把锁就没有设置过期时间了，变成了死锁，别的线程再也没有办法获得锁了。
   解决方案：redis的set命令支持在获取锁的同时设置key的过期时间set <lock.key> <lock.value> nx ex <expireTime>

3. **锁超时误删问题**：假如线程A成功得到了锁，并且设置的超时时间是30秒。如果某些原因导致线程A30秒都没执行完，锁过期自动释放，线程B得到了锁。随后线程A执行完任务，执行DEL指令来释放锁。这时候线程B还没执行完，线程A实际上删除的是线程B加的锁。
   解决方案：可以在DEL释放锁之前做一个判断，验证当前的锁是不是自己加的锁。在加锁的时候设置UUID当做value，并在删除之前验证key对应的value是不是UUID。

4. **锁校验非原子性问题**：get操作、判断UUID和释放锁是两个独立操作，不是原子性。
   解决方案：对于非原子性的问题，我们可以使用Lua脚本来确保操作的原子性。具体参照代码。

5. **多线程访问同代码块**：虽然避免了线程A误删掉key的情况，但是同一时间有A，B两个线程在访问代码块，仍然是不完美的。
   解决方案：可以让获得锁的线程开启一个守护线程，用来给快要过期的锁“续期”。假设线程A执行了29秒后还没执行完，这时候守护线程会执行expire指令，为这把锁续期20秒。守护线程从第29秒开始执行，每20秒执行一次。当线程A执行完任务，会显式关掉守护线程。如果服务器忽然断电，由于线程A和守护线程在同一个进程，守护线程也会停下。这把锁到了超时的时候，没人给它续命，也就自动释放了。

6. **重入锁问题**：A方法调用B方法，A和B都有加锁的操作，这时就会有死锁的问题。
   解决方案：需要将锁修改为Hash结构，这时加锁解锁都需要使用LUA脚本实现，因为Hash没有SETNX类似的命令能判断不存在才添加。并且删除锁的时候要判断锁重入层级。

   

代码举例：

```java
package com.thunisoft.pcglrpt.controller;

import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;

import java.util.Arrays;
import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/**
 * Redis 实现分布式锁实现类（由Lua + Redis Hash保证）
 * 锁类型：非公平，可重入，自动续期
 */
public class DistributedRedisLock implements Lock {
    /**
     * Redis 调用者
     */
    private StringRedisTemplate stringRedisTemplate;
    /**
     * 锁名称（锁唯一使用）
     */
    private String lockName;
    /**
     * 锁ID（可重入使用）
     */
    private String uuid;

    /**
     * 过期时间
     */
    private long expire = 30;

    public DistributedRedisLock(StringRedisTemplate stringRedisTemplate, String lockName) {
        this.stringRedisTemplate = stringRedisTemplate;
        this.lockName = lockName;
        this.uuid = "uuid:" + Thread.currentThread().getId();
    }

    @Override
    public void lock() {
        tryLock();
    }


    @Override
    public boolean tryLock() {
        try {
            return this.tryLock(-1L, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return false;
    }

    /**
     * 加锁方法
     *
     * @param time 超时时间数字
     * @param unit 时间类型
     * @return 加锁成功与否
     * @throws InterruptedException
     */
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        // 判断过期时间，如果没传自定义的，就使用默认值30，单位为秒
        if (time != -1) this.expire = unit.toSeconds(time);
        // 设置加锁LUA脚本
        String luaLockScript = "" +
                "if " +
                "   redis.call('exists',KEYS[1]) == 0 or redis.call('hexists', KEYS[1], ARGV[1]) == 1 " +       // 判断锁不存在或者锁存在且为当前线程
                "then " +
                "   redis.call('hincrby',KEYS[1],ARGV[1],1) " +                                                        // 重入自增一操作
                "   redis.call('expire',KEYS[1],ARGV[2]) return 1 " +                                                 // 设置过期时间
                "else " +
                "   return 0 " +
                "end";
        // 尝试加锁并循环
        while (!stringRedisTemplate.execute(new DefaultRedisScript<>(luaLockScript, Boolean.class), Arrays.asList(lockName), uuid, String.valueOf(expire))) {
            // 如果未获取到锁，50毫秒后自动重试
            Thread.sleep(50);
        }
        // 获取锁成功后，过期时间自动续期
        renewExpire();
        return true;
    }

    /**
     * 解锁方法
     */
    @Override
    public void unlock() {
        // 设置解锁LUA脚本
        String luaUnLockScript = "" +
                "if " +
                "   redis.call('hexists', KEYS[1], ARGV[1]) == 0 " +                  // 如果不存在锁，返回空
                "then " +
                "   return nil " +
                "elseif " +
                "   redis.call('hincrby', KEYS[1], ARGV[1], -1) == 0 " +             // 如果存在锁并且重入等级降为0，删除锁
                "then " +
                "   return redis.call('del', KEYS[1]) " +                              // 删除锁
                "else " +
                "   return 0 " +
                "end";
        // 执行解锁LUA脚本
        Long flag = stringRedisTemplate.execute(new DefaultRedisScript<>(luaUnLockScript, Long.class), Arrays.asList(lockName), uuid);
        // 如果返回的是nil，flag就为null，抛出异常（锁不属于你或者锁不存在）
        if (flag == null) {
            throw new IllegalMonitorStateException("this lock doesn't belong to you!");
        }
    }

    /**
     * 守护线程锁续期
     */
    public void renewExpire() {
        String lua = "" +
                "if " +
                "   redis.call('hexists',KEYS[1],ARGV[1]) == 1 " +              // 判断如果锁存在，就设置过期时间
                "then " +
                "   return redis.call('expire',KEYS[1],ARGV[2]) " +
                "else " +
                "   return 0 " +
                "end";
        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                if (stringRedisTemplate.execute(new DefaultRedisScript<>(lua, Boolean.class), Arrays.asList(lockName), uuid, String.valueOf(expire))) {
                    renewExpire();
                }
            }
        }, expire * 1000 / 3);       // 三分之一的过期时间执行一次
    }

    /**
     * 无需实现的方法，忽略
     */
    @Override
    public Condition newCondition() {
        return null;
    }

    /**
     * 无需实现的方法，忽略
     */
    @Override
    public void lockInterruptibly() throws InterruptedException {
    }
}

```

