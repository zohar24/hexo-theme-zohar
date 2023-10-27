---
title: Mysql安装
top: false
author: zhaohuan
date: 2023-03-10 13:17:31
tags:
- mysql
categories:
- 数据库
---


## 1. 下载MySQL

https://dev.mysql.com/downloads/

https://downloads.mysql.com/archives/community/

如果是centos版本，则MySQL选择Red Hat版本



## 2. 安装

### 2.1 卸载mariadb

linux系统会自动携带一个数据库mariadb，需要把它给卸载掉

```sh
# 查看是否有mariadb
rpm -qa | grep mariadb

# 若有，则卸载
yum remove mariadb-libs-5.5.52-1.el7.x86_64 -y

# 查看是否卸载成功
rpm -qa | grep mariadb

```



### 2.2 安装MySQL

上传MySQL捆绑包到服务器上

解压MySQL安装包

```sh
mkdir mysql-8.0.33
tar -xvf mysql-community-server-8.0.33-1.el8.x86_64.rpm -C mysql-8.0.33
```

按顺序安装rpm

```sh
#5.7
mysql-community-common -> mysql-community-libs -> mysql-community-client -> mysql-community-server
#8.0
rpm -ivh mysql-community-common-8.0.33-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.33-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.33-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.33-1.el7.x86_64.rpm
rpm -ivh mysql-community-icu-data-files-8.0.33-1.el7.x86_64.rpm
# yum install -y net-tools 如果安装server失败再安装net-tools
rpm -ivh mysql-community-server-8.0.33-1.el7.x86_64.rpm

```

验证是否安装成功

```
mysql --version
```

## 3 启动MySQL

```sh
#启动mysql服务器
service mysqld start
# 查看mysql是否启动
service mysqld status
# 每次开机都要手动启动mysql
systemctl start mysqld
# 开机时自动开启mysql 
systemctl enable mysqld
# 停止mysql服务器
service mysqld stop
```

若启动时遇到如下的报错，需删除数据库存放的数据

>  [InnoDB] Unsupported redo log format (v0). The redo log was created before MySQL 5.7.9
>
> [MY-010020] [Server] Data Dictionary initialization failed.


```sh
# 删除数据库下的所有数据
rm -rf /var/lib/mysql/*
# 启动MySQL
service mysqld start 或 systemctl start mysqld
```



### 3.1 登录MySQL 修改密码

第一次登录mysql需要使用mysql的临时密码，该密码存放在mysql日志文件中

```sh
vi /var/log/mysqld.log

# 查看某一行日志为：A temporary password is generated for root@localhost: k9r0TzkLR4//
# 则k9r0TzkLR4// 为首次登录密码
```

#### 3.1.1 命令登录MySQL

```sh
mysql -uroot -p
```

输入刚才获取到的临时密码



#### 3.1.2 修改密码

```sh
# 想设置密码简单需要先修改成复杂密码，然后降低密码校验的复杂度
alter user root@localhost identified by '6789@jklJKL';
# 修改密码校验配置
# mysql8.0
set global validate_password.policy = 0;
set global validate_password.length = 4;
# mysql5.7
set global validate_password.policy = 0;
set global validate_password.length = 4;
# 重新设置简单密码
alter user root@localhost identified by '123456';

# 查看密码策略
SHOW VARIABLES LIKE 'validate_password%';
0/LOW：只验证长度；
1/MEDIUM：验证长度、数字、大小写、特殊字符；
2/STRONG：验证长度、数字、大小写、特殊字符、字典文件；

```



### 3.2 配置远程登录

此时root用户只能用于本机访问，不能用于远程访问，否则会报错误。因此如果navicat想远程连接，是无法连接的！

授予root用户远程访问权限

```sh
update mysql.user set host='%' where user='root';
flush privileges;
```



### 3.3 配置防火墙

使用本地使用管理工具如navicat，尝试连接mysql数据库。如果拒绝访问，可能是防火墙没设置

```sh
#确认防火墙使用的是iptables还是firewalld
service iptables status
service firewalld status
# 哪个命令好使，说明使用的是那个防火墙

# 方案1：简单粗暴，关闭防火墙
systemctl stop iptables 或者 systemctl stop firewalld.service

# 方案2：添加指定端口
/sbin/iptables -I INPUT -p tcp --dport 3306 -j
firewall-cmd --zone=public --add-port=3306/tcp --permanent
# 查看开放的端口
firewall-cmd --list-all
```



### 3.4 配置编码

为了防止以后出现乱码问题，我们需要把mysql的编码修改为`utf8`

```sh
# 查看编码
show variables like '%char%';
# 编辑配置文件
vim /etc/my.cnf
# 1、在 [mysqld] 标签下加上以下内容：
collation-server=utf8_general_ci
character-set-server=utf8

# 2、在 [mysql] 标签下加上一行
default-character-set=utf8
# 3、在 [mysql.server]标签下加上一行
default-character-set=utf8
# 4、在 [mysqld_safe]标签下加上一行
default-character-set=utf8
# 5、在 [client]标签下加上一行
default-character-set=utf8

```



# 二、 配置ssl

可参考 [mysql官网](https://dev.mysql.com/doc/refman/8.0/en/using-encrypted-connections.html) 进行配置。

## 1. 创建一个ssl的用户

### 1.1 创建用户并赋权

```sql
--mysql5.7
--配置权限
grant all privileges on *.* to 'ssltest'@'%'identified by '123456' with grant option;
--创建用户并开启ssl登录
GRANT ALL PRIVILEGES ON *.* TO 'ssltest'@'%' IDENTIFIED BY '123456' REQUIRE SSL;

--mysql8.0
--新版本的 MySQL 8.x 版本已经将创建账户和赋权的方式分开会导致上面的命令在 MySQL 8.x 上执行报语法错误。
-- 创建用户
CREATE USER 'ssltest'@'%' IDENTIFIED BY '123456';
-- 赋权
GRANT ALL ON *.* TO 'ssltest'@'%';
-- 刷新
FLUSH PRIVILEGES;

```

### 1.2 开启ssl登录

```sql
--MySQL8版本中新增了一个system_user帐户类型，如果root用户没有SYSTEM_USER权限，会导致新增的用户开启ssl失败
grant system_user on *.* to 'root';
--开启ssl
ALTER USER 'ssltest'@'%' REQUIRE SSL;
FLUSH PRIVILEGES;
--查看用户情况
SELECT ssl_type From mysql.user Where user="ssltest"

-- 取消ssl验证:
alter user 'xxx'@'%' require none;
```



## 2. 制作证书

### 方式1：使用mysql8默认证书

mysql8自带了ssl证书，可直接使用，一般在data目录里(/var/lib/mysql/)

```sh
mysql-certs:
├── public_key.pem
├── ca.pem 				#ca证书
├── client-cert.pem     #客户端证书
├── server-cert.pem		#服务端证书
├── ca-key.pem			#ca私钥
├── client-key.pem		#客户端私钥
├── private_key.pem		#私钥
├── server-key.pem		#服务端私钥
```

### 方式2：使用mysql_ssl_rsa_setup

mysql_ssl_rsa_setup一般在mysql的bin目录下，Linux在`/var/lib/mysql/`下

```sh
mysql_ssl_rsa_setup -d /opt/mysql/
# -d 指定生成ssl证书的路径
```

### 方式3：使用openssl生成

```sh
# Create clean environment
rm -rf newcerts
mkdir newcerts && cd newcerts

# Create CA certificate
openssl genrsa 2048 > ca-key.pem
openssl req -new -x509 -nodes -days 3600 -key ca-key.pem -out ca.pem

# Create server certificate, remove passphrase, and sign it
# server-cert.pem = public key, server-key.pem = private key
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout server-key.pem -out server-req.pem
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem

# Create client certificate, remove passphrase, and sign it
# client-cert.pem = public key, client-key.pem = private key
openssl req -newkey rsa:2048 -days 3600 -nodes -keyout client-key.pem -out client-req.pem
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3600 -CA ca.pem -CAkey ca-key.pem -set_serial 01 -out client-cert.pem

# 验证
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
# 查看证书
openssl x509 -text -in ca.pem
openssl x509 -text -in server-cert.pem
openssl x509 -text -in client-cert.pem

```

#### openssl相关说明

1. 生成根证书私钥

   ```sh
   openssl genrsa -aes256 -out private/cakey.pem 2048
   命令含义如下：
   genrsa——使用RSA算法产生私钥
   -aes256——使用256位密钥的AES算法对私钥进行加密
   -out——输出文件的路径
   2048——指定私钥长度
   ```

2. 生成证书请求

   ```sh
   openssl req -new -key private/cakey.pem -out private/ca.csr -subj “/C=CN/ST=ZHEJIANG/L=HANGZHOU/O=TEST/OU=mygroup/CN=TEST”
   该命令含义如下：
   req——执行证书签发命令
   -new——新证书签发请求
   -key——指定私钥路径
   -out——输出的csr文件的路径
   -subj——证书相关的用户信息(subject的缩写)
   
   备注：这里需要输入私钥密码；
   ```

3. 检查证书请求信息

   ```sh
   openssl req -text -in ca.csr -noout
   ```

4. 自签发根证书

   ```sh
   openssl x509 -req -days 365 -sha1 -extensions v3_ca -signkey private/cakey.pem -in private/ca.csr -out certs/ca.cer
   
   #该命令的含义如下：
   x509——生成x509格式证书
   -req——输入csr文件
   -days——证书的有效期（天）
   -sha1——证书摘要采用sha1算法
   -extensions——按照openssl.cnf文件中配置的v3_ca项添加扩展
   -signkey——签发证书的私钥
   -in——要输入的csr文件
   -out——输出的cer证书文件
   
   备注：
   （1）创建非根证书指定 -extensions v3_req，表示，在openssl.cnf中v3_req扩展属性为：basicConstraints = CA:FALSE。
   （2）创建根证书时，指定了-extensions v3_ca，basicConstraints = critical,CA:true。
   ```

5. 检查证书

   ```sh
   openssl x509 -text -in ca.cer -noout
   ```

   

## 3. 配置证书

编辑mysql的配置文件，Linux下为my.cnf，windows下为mysql.ini。添加下列配置：

```sh
ssl_ca=ca.pem
ssl_cert=server-cert.pem
ssl_key=server-key.pem

#指定是否要求客户端使用加密连接。默认值为 OFF，如果 ON，则表示客户端必须使用加密连接，如果客户端关闭 ssl ，则连接会报错。
require_secure_transport=ON

# 注意证书目录的所属权限用户为mysql，否则会导致mysql启动时读取证书失败
chown -R mysql.mysql /opt/mysql/
```

 配置完后重启mysql

```sh
service mysqld restart
```



## 4. 验证

```sh
show variables like '%ssl%';

+---------------+--------------------------------+
| Variable_name | Value                          |
+---------------+--------------------------------+
| have_openssl  | YES                            |
| have_ssl      | YES                            |
| ssl_ca        | /etc/mariadb/pki/ca-bundle.crt |
| ssl_capath    |                                |
| ssl_cert      | /etc/mariadb/pki/server.crt    |
| ssl_cipher    |                                |
| ssl_crl       |                                |
| ssl_crlpath   |                                |
| ssl_key       | /etc/mariadb/pki/server.key    |
+---------------+--------------------------------+

当 have_ssl 为 YES 时, 表示此时 MySQL 服务已经支持 SSL 了. 如果是 DESABLE, 则表示当前环境不支持启用ssl.
```

 **查看ssl_cipher**

```sql
--通过ssl登录
mysql -u ssltest -p --ssl-ca=/opt/mysql/mysql-8.0.33/ssl/ca.pem
--登录mysql后
show status like 'ssl_cipher';

--返回结果
+---------------+-----------------------------+
| Variable_name | Value                       |
+---------------+-----------------------------+
| Ssl_cipher    | ECDHE-RSA-AES128-GCM-SHA256 |
+---------------+-----------------------------+
```

当使用ssl登录时，才会有结果，若不通过ssl登录，则显示为空。

## 5. TLS版本

```sql
-- 查看版本
SHOW GLOBAL VARIABLES LIKE 'tls_version';

--配置版本 在mysql配置文件
tls_version=TLSv1.2,TLSv1.3
```



## 6. 附录

#### MySQL Server TLS Protocol Support

| MySQL Server Release              | TLS Protocols Supported                                      |
| :-------------------------------- | :----------------------------------------------------------- |
| MySQL 8.0.15 and below            | TLSv1, TLSv1.1, TLSv1.2                                      |
| MySQL 8.0.16 and MySQL 8.0.17     | TLSv1, TLSv1.1, TLSv1.2, TLSv1.3 (except Group Replication)  |
| MySQL 8.0.18 through MySQL 8.0.25 | TLSv1, TLSv1.1, TLSv1.2, TLSv1.3 (including Group Replication) |
| MySQL 8.0.26 and MySQL 8.0.27     | TLSv1 (deprecated), TLSv1.1 (deprecated), TLSv1.2, TLSv1.3   |
| MySQL 8.0.28 and above            | TLSv1.2, TLSv1.3                                             |

#### MySQL Server TLS Protocol Default Settings

| MySQL Server Release              | `tls_version` Default Setting                                |
| :-------------------------------- | :----------------------------------------------------------- |
| MySQL 8.0.15 and below            | `TLSv1,TLSv1.1,TLSv1.2`                                      |
| MySQL 8.0.16 and MySQL 8.0.17     | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3 (with OpenSSL 1.1.1)``TLSv1,TLSv1.1,TLSv1.2 (otherwise)`Group Replication does not support TLSv1.3 |
| MySQL 8.0.18 through MySQL 8.0.25 | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3 (with OpenSSL 1.1.1)``TLSv1,TLSv1.1,TLSv1.2 (otherwise)`Group Replication supports TLSv1.3 |
| MySQL 8.0.26 and MySQL 8.0.27     | `TLSv1,TLSv1.1,TLSv1.2,TLSv1.3 (with OpenSSL 1.1.1)``TLSv1,TLSv1.1,TLSv1.2 (otherwise)`TLSv1 and TLSv1.1 are deprecated |
| MySQL 8.0.28 and above            | `TLSv1.2,TLSv1.3`                                            |



# 三、卸载mysql

如果之前linux系统上存在mysql想更新版本卸载或出现问题重装的话，需要卸载mysql

## 查看安装情况

```sh
rpm -qa|grep -i mysql
# 会出现如下安装包
mysql-community-client-5.7.28-1.el7.x86_64
mysql-community-server-5.7.28-1.el7.x86_64
mysql-community-common-5.7.28-1.el7.x86_64
mysql-community-libs-5.7.28-1.el7.x86_64
# mysql8
mysql-community-client-8.0.33-1.el7.x86_64
mysql-community-common-8.0.33-1.el7.x86_64
mysql-community-client-plugins-8.0.33-1.el7.x86_64
mysql-community-server-8.0.33-1.el7.x86_64
mysql-community-libs-8.0.33-1.el7.x86_64
mysql-community-icu-data-files-8.0.33-1.el7.x86_64
```

## 删除安装包

```sh
rpm -ev mysql-community-server-5.7.28-1.el7.x86_64
……

# 或者执行以下语句全部卸载
rpm -e --nodeps `rpm -qa|grep mysql`
```

## 删除mysql相关文件

```sh
 # 查询mysql相关文件
 find / -name mysql
 
# 根据查找后的结果进行删除
rm -rf /etc/selinux/targeted/active/modules/100/mysql
……#删除所有

# 删除mysql的my.cnf
rm -rf /etc/my.cnf
```

