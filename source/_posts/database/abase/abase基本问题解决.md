---
title: Abase基本问题解决
author: zohar
top: false
cover: false
toc: true
date: 2023-01-30 18:27:31
tags:
- abase
- postgre
categories:
- 数据库
---
# Abase基本问题解决

## 一、数据库启停

数据库一般安装在thunisoft用户，在/home/thunisoft目录下存在启停数据库的脚本

### 1.1 启动数据库

```
su - thunisoft
./startup_abase1.sh
```

### 1.2 停止数据库

```
su - thunisoft
./stop_abase1.sh
```

### 1.3 查看数据库是否启动

```
ps -ef|grep "postgres -D"
```

## 二、数据库连接数满

目前遇到的数据库连接满的情况，都是由于JDBC驱动未更新导致的。可登录数据库服务器，判断哪个客户端造成的。

### 2.1 查看数据库连接

```
ps -ef|grep postgres
```

### 2.2 定位应用

#### 2.2.1 查看应用服务器连接数据库进程

通过2.1命令定位连接最多应用服务器，登录该服务器定位应用进程号

```
netstat -np|grep 172.16.32.240
```

```
tcp        0      0 ::ffff:172.16.33.84:41654   ::ffff:172.16.32.240:5432   ESTABLISHED 21238/java
tcp        0      0 ::ffff:172.16.33.84:39908   ::ffff:172.16.32.240:5432   ESTABLISHED 19468/java
tcp        0      0 ::ffff:172.16.33.84:39906   ::ffff:172.16.32.240:5432   ESTABLISHED 19468/java
tcp        0      0 ::ffff:172.16.33.84:41662   ::ffff:172.16.32.240:5432   ESTABLISHED 21238/java
tcp        0      0 ::ffff:172.16.33.84:41655   ::ffff:172.16.32.240:5432   ESTABLISHED 21238/java
tcp        0      0 ::ffff:172.16.33.84:41660   ::ffff:172.16.32.240:5432   ESTABLISHED 21238/java
tcp        0      0 ::ffff:172.16.33.84:41658   ::ffff:172.16.32.240:5432   ESTABLISHED 21238/java
```

#### 2.2.2 查看具体应用进程

通过2.2.1命令执行结果，定位连接数据库应用进程号21238、19468。通过如下命令定位应用

```
ps -ef|grep 21238
```

#### 2.2.3 kill应用

先kill该上述应用，可释放连接。

```
kill -9 21238
```

#### 2.2.4 更新驱动

最后跟新JDBC驱动，重启应用。