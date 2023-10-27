---
title: NFS服务器
top: false
author: zhaohuan
date: 2023-04-10 15:18:31
tags:
- NFS
categories:
- Linux
---


# NFS服务器

## NFS介绍

​	NFS 就是 Network FileSystem 的缩写，最早之前是由 Sun 这家公司所发展出来的。 它最大的功能就是可以透过网络，让不同的机器、不同的操作系统、可以彼此分享个别的档案 (share files)。所以，你也可以简单的将他看做是一个文件服务器 (file server) 呢！这个 NFS 服务器可以让你的 PC 来将网络远程的 NFS 服务器分享的目录，挂载到本地端的机器当中， 在本地端的机器看起来，那个远程主机的目录就好像是自己的一个磁盘分区槽一样 (partition)！使用上面相当的便利！

​	当我们的 NFS 服务器设定好了分享出来的 /home/sharefile 这个目录后，其他的 NFS 客户端就可以将这个目录挂载到自己系统上面的某个挂载点 (挂载点可以自定义)，例如 NFS client 1 与 NFS client 2 挂载的目录就不相同。我只要在 NFS client 1 系统中进入 /home/data/sharefile 内，就可以看到 NFS 服务器系统内的 /home/sharefile 目录下的所有数据了 (当然，权限要足够！这个 /home/data/sharefile 就好像 NFS client 1 自己机器里面的一个 partition 喔！只要权限对了，那么你可以使用 cp, cd, mv, rm... 等等磁盘或档案相关的指令！

​	NFS 只提供了基本的文件处理功能，而不提供任何 TCP/IP 数据传输功能。它需要借助 RPC 协议才能实现 TCP/IP 数据传输功能



## RPC

​	RPC（Remote Procedure Call，远程过程调用）是一种网络程序的编程方法，它定义了一种进程间通过网络进行交互通信的机制。

​	正常情况下，如果客户端要和服务器相互通信，那么客户端和服务器就必须利用系统的套接口函数，编写出一套完整的网络通信协议才能实现。而如果使用了 RPC，那么服务器和客户端之间只需要相互调用对方提供的 RPC 接口函数（俗称客户桩）即可实现相互通信。函数的参数和返回值就是要传递的信息。而 RPC 将全权负责网络通信。这样一来，网络程序的设计就变得简单许多，跟编写本地程序一样简单，通信效率也会有所提高。当调用 RPC 的服务器程序启动时，相关的进程就会向 RPC 发起注册。RPC 就会开启特定的端口来为客户端提供服务。

​	由于端口号可能是不固定的，因此 RPC 服务器必须提供一种查询服务端口的方法。RPC 服务器上有一个叫端口映射器的东西，它固定监听 UDP 111 端口。RPC 客户端通过访问这个端口就可以查询到对应服务所使用的端口。

​	使用 RPC 协议的网络程序在服务器和客户端上都必须借助 RPC 服务才可相互通信。



## 安装nfs

### 查看是否安装过nfs

```shell
rpm -qa | grep nfs 
rpm -qa | grep rpcbind 
```

### 安装nfs

```shell
 yum install nfs-utils
```



### 编辑配置文件

```shell
vim /etc/exports

/tmp 192.168.100.0/24(ro) localhost(rw) *.ev.ncku.edu.tw(ro,sync)
```

* 我想将 /tmp 分享出去给大家使用，由于这个目录本来就是大家都可以读写的，因此想让所有的人都可以存取。此外，我要让 root 写入的档案还是具有 root 的权限，那如何设计配置文件

```shell
/tmp  *(rw,no_root_squash)
```

* 





配置说明

| 参数值                     | 内容说明                                                     |
| -------------------------- | ------------------------------------------------------------ |
| rw ro                      | 该目录分享的权限是可擦写 (read-write) 或只读 (read-only)，但最终能不能读写，还是与文件系统的 rwx 及身份有关。 |
| sync async                 | sync 代表数据会同步写入到内存与硬盘中，async 则代表数据会先暂存于内存当中，而非直接写入硬盘！ |
| no_root_squash root_squash | 客户端使用 NFS 文件系统的账号若为 root 时，系统该如何判断这个账号的身份?<br/>预设的情况下，客户端 root 的身份会由 root_squash 的设定压缩成 nfsnobody， 如此对服务器的系统会较有保障。但如果你想要开放客户端使用 root 身份来操作服务器的文件系统，那么这里就得要开 no_root_squash 才行！ |
| all_squash                 | 不论登入 NFS 的使用者身份为何， 他的身份都会被压缩成为匿名用户，通常也就是 nobody(nfsnobody) 啦！ |
| anonuid anongid            | anon 意指 anonymous (匿名者) 前面关于 *_squash 提到的匿名用户的 UID 设定值，通常为 nobody(nfsnobody)，但是你可以自行设定这个 UID 的值！当然，这个 UID 必需要存在于你的 /etc/passwd 当中！ anonuid 指的是 UID 而 anongid 则是群组的 GID 啰。 |

### 启动nfs

```shell
/etc/init.d/rpcbind start
/etc/init.d/nfs start
# 重启
systemctl reload nfs

```

### 查看nfs启动状态

```shell
netstat -tulnp| grep -E '(rpc|nfs)'


showmount [-ae] [hostname|IP]
选项与参数：
-a ：显示目前主机与客户端的 NFS 联机分享的状态；
-e ：显示某部主机的 /etc/exports 所分享的目录数据。
```

## 挂载

命令用法

```python
mount.nfs <服务器 IP 或主机名>:<共享路径> <挂载点> [-o <选项>]
或者
mount -t nfs NFS_SERVER:/PATH/TO/SOME_EXPORT /PATH/TO/SOMEWHRERE
```



### 取消挂载

```shell
umount /home/nfs/public
```



## 常见问题处理

#### Failed to start nfs.service: Unit nfs.service not found

ubuntu 10.0开启配置nfs 服务service nfs start时出现:

> Failed to start nfs.service: Unit nfs.service not found.

原因是ubuntu 10.0以上的版本取消了`service nfs start`。改成了`sudo service nfs-server start` 。这样就完成启动了


#### System Error: No route to host.

​	由于nfs服务需要开启 `mountd,nfs,nlockmgr,portmapper,rquotad`这5个服务，需要将这5个服务的端口加到iptables里面
而nfs 和 portmapper两个服务是固定端口的，nfs为2049，portmapper为111。其他的3个服务是用的随机端口，那就需要
先把这3个服务的端口设置成固定的。

查看当前这5个服务的端口并记录下来

``` shell
rpcinfo -p
```

​	固定`nlockmgr,portmapper,rquotad`三个的端口	

```shell
mountd  976/tcp
mountd  976/udp
rquotad  966/tcp
rquotad  966/udp
nlockmgr 33993/tcp
nlockmgr 33993/udp
```

​	重启nfs

```shell
 service nfs restart
```

防火墙中开放这5个端口

   编辑iptables配置文件 

 ```shell
  vim /etc/sysconfig/iptables
 ```

   添加如下行：

```shell
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p tcp --dport 111 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p tcp --dport 976 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p tcp --dport 2049 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p tcp --dport 966 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p tcp --dport 33993 -j ACCEPT

-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p udp --dport 111 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p udp --dport 976 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p udp --dport 2049 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p udp --dport 966 -j ACCEPT
-A RH-Firewall-1-INPUT -s 192.168.0.0/24 -m state --state NEW -p udp --dport 33993 -j ACCEPT
```

保存退出并重启iptables

```shell
service iptables restart
```

### 防火墙相关

```shell
# 查看防火墙状态
service iptables status
systemctl status firewalld.service
# 停止防火墙
service iptables stop
systemctl stop firewalld.service
# 启动防火墙
service iptables start
systemctl start firewalld.service
# 重启防火墙
service iptables restart 
systemctl restart firewalld.service
# 永久关闭防火墙
chkconfig iptables off  
# 永久关闭后重启
chkconfig iptables on　
# 禁用防火墙
systemctl disable firewalld.service
```



