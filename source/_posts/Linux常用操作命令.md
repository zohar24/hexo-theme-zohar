---
title: Linux常用操作命令
author: zohar
top: false
cover: false
toc: true
mathjax: true
date: 2022-08-17 15:27:31
tags:
- 基础命令
- Linux
categories:
- Linux
---

# 操作系统概述
操作系统 Operating System 简称 OS，通俗讲就是一款软件，不过和一般的软件不同，操作系统是管理和控制计算机硬件与软件资源的计算机程序，是直接运行在“裸机”上的最基本的系统软件，任何其他的软件都必须在操作系统的支持下才能运行。


## Linux 文件系统
* **/var：**包含在正常操作中被改变的文件、假脱机文件、记录文件、加锁文件、临时文件和页格式化文件等。

* **/home：** 包含用户的文件：参数设置文件、个性化文件、文档、数据、EMALL、缓存数据等，每增加一个用户，系统就会根据其用户名在 home 目录下新建和其他用户同名的文件夹，用于保存其用户配置。

* **/proc：**包含虚幻的文件，他们实际上并不存在于磁盘上，也不占用任何空间（用 ls-l 可以显示它们的大小）当查看这些文件时，实际上是在访问存在内存中的信息，这些信息用于访问系统。

* **/bin：**包含系统启动时需要的执行文件（二进制），这些文件可以被普通用户使用。

* **/etc：**为操作系统的配置文件目录（防火墙、启动项）

* **/root：**为系统管理员（也叫超级用户或根用户）的 Home 目录。

* **/dev：**为设备目录，Linux下设备被当成文件，这样一来硬件被抽象化、便于读写、网络共享以及需要临时装载到文件系统中，正常情况下，设备会有一个独立的子目录，这些设备的内容会出现在独立的子目录下。

# 空间

## df

* df -h ：查看硬盘剩余容量

  ```shell
  df（英文全拼：disk free） 命令用于显示目前在 Linux 系统上的文件系统磁盘使用情况统计。
  ```

## du 

* du -sh * ，du -h --max-depth=1 ：查看当前目录下每项大小

  ```shell
  du(disk usage)命令用于显示目录或文件所占用的磁盘空间。
  -h或--human-readable 以K，M，G为单位，提高信息的可读性。
  -s或--summarize 仅显示总计。
  --max-depth=<目录层数> 超过指定层数的目录后，予以忽略。
  ```

## rm

* rm -rf 文件夹 ：删除文件夹
* rm edu_* ：删除edu开头的所有文件

## free

* free -m 内存



# 进程

* netstat -tunlp|grep [端口号]  根据端口查进程号
* ps -ef|grep [进程号] 根据进程号查相关进程信息

# 文件

## 概览

* find [搜索范围路径] -name [文件名/文件夹名] 可用*来模糊查询

* rm -rf [文件名/文件夹名] 删除文件 注意：绝对不要用`/*`删除文件

* tail -200f XXX.log 实时查看

* tar zxvf xx.tar.gz  解压文件

## 复制和移动

 ### 文件复制命令cp

**命令格式：**

> cp [-adfilprsu] 源文件(source) 目标文件(destination)
>
> cp [option] source1 source2 source3 ...  directory
>
> 即： cp  [options] sourcedir  destdir

**参数说明：**

> **-a**  :  是指archive的意思，也说是指复制所有的目录
>
> **-d**  :  若源文件为连接文件(link file)，则复制连接文件属性而非文件本身
>
> **-f**  :  强制(force)，若有重复或其它疑问时，不会询问用户，而强制复制
>
> **-i**  :  若目标文件(destination)已存在，在覆盖时会先询问是否真的操作
>
> **-l**  :  建立硬连接(hard link)的连接文件，而非复制文件本身
>
> **-p**  :  与文件的属性一起复制，而非使用默认属性
>
> **-r**  :  递归复制，用于目录的复制操作
>
> **-s**  :  复制成符号连接文件(symbolic link)，即“快捷方式”文件
>
> **-u**  :  若目标文件比源文件旧，更新目标文件

**cp命令案例：**

> **cp  /etc/sys.conf  /home/**
>
> ​    将/etc/sys.conf文件复制到home目录下
>
> **cp /test1/file1 /test3/file2**
>
> ​    将/test1目录下的file1复制到/test3目录，并将文件名改为file2
>
> **cp -r test/ /home/**
>
> ​	将当前目录"test/"以及其所有文件复制到home目录下
>
> **cp -r test/ nettest**
>
> ​	将当前目录"test/"下的所有文件复制到新目录“newtest”下
>
> **cp -a /etc/ /home**
>
> ​    将"/etc/"目录以及所有文件和子目录以及延伸的（保留链接、文件属性）复制到/home目录下

### 文件移动命令mv

**命令格式：**

> mv [-fiv] source destination

**参数说明：**

> **-f**  :  force，强制直接移动而不询问
>
> **-i**  :  若目标文件(destination)已经存在，就会询问是否覆盖
>
> **-u**  :  若目标文件已经存在，且源文件比较新，才会更新

**mv命令案例：**

> mv /test1/file1 /test3/file2
>
> ​    表示将test1目录下的file1复制到test3 目录，并将文件名改为file2
>
> mv * ../
>
> ​     表示Linux当前目录所有文件移动到上一级目录

### 文件删除命令rm

**命令格式：**

> rm [fir] 文件或目录

**参数说明：**

> **-f**  :  强制删除
>
> **-i**  :  交互模式，在删除前询问用户是否操作
>
> **-r**  :  递归删除，常用在目录的删除

#### rm命令案例：

> **rm -i /test/file1**
>
> 表示删除/test目录下的file1文件



 ## 远程复制

### 复制目录

```shell
scp -r local_folder remote_username@remote_ip:remote_folder 
```

### 复制文件

```shell
scp local_file remote_username@remote_ip:remote_folder
scp local_file remote_username@remote_ip:remote_file
```



## 压缩

### tar(Tape Archive，磁带归档的缩写）

用于归档多个文件或目录到单个归档文件中，并且归档文件可以进一步使用 gzip 或者 bzip2 等技术进行压缩。

### 语法格式

```sh
tar <选项> <文件或目录>
```

### 常用选项

```sh
--delete : 从tar包中删除某个文件
-r, --append : 将文件追加到tar归档文件中
-t, --list : 列出tar归档文件中包含的文件或目录
-u, --update : 将已更新的文件追加到tar归档文件中
-x, --extract, --get : 释放tar归档文件中文件及目录
-C, --directory=DIR : 执行归档动作前变更工作目录到 目标DIR
-f, --file=ARCHIVE : 指定 (将要创建或已存在的) 归档文件名
-j, --bip2 : 对归档文件使用 bzip2 压缩
-J, --xz : 对归档文件使用 xz 压缩
-p, --preserve-permissions : 保留原文件的访问权限
-v, --verbose : 显示命令整个执行过程
-z, gzip : 对归档文件使用 gzip 压缩
```

### 示例

```sh
tar -cvf mytest.tar /etc/ /root/anaconda-ks.cfg  #创建一个 tar 文件，将 /etc/ 目录和 /root/anaconda-ks.cfg 文件打包进去
tar -tvf mytest.tar  # 列出归档文件中的内容
tar -rvf mytest.tar original-ks.cfg# 追加某文件到归档(tar)文件中
tar -xvf mytest.tar #释放tar归档至当前所在目录
tar -xvf mytest.tar -C testdir02 #释放 tar 文件到指定目录
tar -zcpvf myarchive.tar.gz /etc/ /opt/ #注-zcpvf顺序不能变 创建并压缩归档文件
tar -zxpvf myarchive.tgz -C /tmp/ #解压 .tar.gz
```

# 防火墙
* service iptables status 、firewall-cmd --state或service firewalld status 防火墙状态
```sh
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





