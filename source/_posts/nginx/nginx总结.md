---
title: nginx总结
top: false
author: zhaohuan
date: 2023-08-14 15:17:31
tags:
- 总结
- nginx
categories:
- nginx
---



# Nginx 
## Nginx 常用命令
梳理部分常用的命令

```sh
/sbin/nginx -t                   # 检查配置文件nginx.conf是否正确
/sbin/nginx                      # 启动nginx
/sbin/nginx -s reload            # 重新载入配置文件
/sbin/nginx -s reopen            # 重启 Nginx
/sbin/nginx -s stop              # 停止 Nginx
/sbin/nginx -c conf_file         # 指定配置文件
```



## nginx安装

### yum安装

1. 远端下载

   ```sh
   yum install -y nginx
   ```

2. 启动nginx

   ```sh
   systemctl enable nginx.service #启动
   systemctl enable nginx.service #开机启动
   ```

3. 在浏览器验证nginx是否启动成功

#### 卸载nginx

```sh
# 方式一
yum remove nginx
#方式二
1. 查找nginx文件 whereis nginx 或者 find / -name nginx
2. 依次删除 rm -rf /usr/local/nginx
```

### 编译安装

到 nginx 官网下载软件: http://nginx.org/

#### 1. 安装 pcre 

##### 方式一

1. 获取pcre压缩文件

```sh
wget https://jaist.dl.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.gz
tar –xvf pcre-8.37.tar.gz

```

2. 进入 pcre-8.37.tar.gz解压后的目录，执行 `./configure `

3. 执行完成后，再执行 `make && make install` 命令，完成pcre的安装

#####  方式二

通过yum安装

```sh
yum -y install pcre
pcre-config --version
```

#### 2. 安装依赖

安装 openssl 、 zlib 、 gcc 依赖

```sh
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

#### 3. 安装nginx

1. 去官网下载nginx http://nginx.org/en/download.html

   ```sh
   或者 wget http://nginx.org/download/nginx-1.18.0.tar.gz
   # 最新版 http://nginx.org/download/nginx-1.22.1.tar.gz
   ```

2. 解压到指定目录

   ```sh
   tar –zxvf nginx-1.12.2.tar.gz
   ```

   进入 nginx.xx.tar.gz 解压后的目录，执行` ./configure`来进行检查

3. 执行`make && make install `命令，完成nginx的安装

   ```sh
   # 完整命令示例：
   wget http://nginx.org/download/nginx-1.18.0.tar.gz
   tar -zxvf nginx-1.18.0.tar.gz
   cd nginx-1.18.0
   
   ./configure --sbin-path=/usr/local/nginx/nginx \
   --conf-path=/usr/local/nginx/nginx.conf \
   --pid-path=/usr/local/nginx/nginx.pid \
   --with-http_gzip_static_module \
   --with-http_stub_status_module \
   --with-file-aio \
   --with-http_realip_module \
   --with-http_ssl_module \
   --with-pcre=/usr/local/src/pcre-8.44 \
   --with-zlib=/usr/local/src/zlib-1.2.11 \
   --with-openssl=/usr/local/src/openssl-1.1.1g
   # configure配置简洁版
   ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
   # config 报错 执行 yum -y install pcre-devel
   
   make && make install
   
   #
   ```

   

4. 可查看安装完成后的文件

   ```sh
   rpm -ql nginx # -q 代表询问模式，-l 代表返回列表
   ```

   在安装目录的sbin目录下执行`./nginx`来启动nginx

5. 浏览器访问nginx，查看效果。

#### 4. 目录结构

-  **src** 目录：存放 Nginx 的源代码。
-  **man** 目录：存放 Nginx 的帮助文档 。
-  **html** 目 录 ：存放默认网站文件 。
-  **contrib** 目录：存放其他机构或组织贡献的文档资料。
-  **conf** 目录：存放 Nginx 服务器的配置文件。
-  **auto** 目录：存放大量的脚本文件， 和 configure 脚本程序相关。
-  **configure** 文件： Nginx 自动安装脚本，用于检查环境，生成编译代码需要的 makefile 文件。





## 文件结构
整体结构为嵌套结构：

![nginx.conf结构](nginx.conf配置.jpg)

- **全局块：**配置影响全局，包括运行 nginx 的用户组，进程存放，日志，配置文件等
- **events：**配置影响 nginx 服务器与客户端的网络连接，包括进程最大连接数，数据驱动模型，序列化等
- **http：**配置代理，缓存，日志，第三方模块等，可嵌套多个 server
  - **server：**配置虚拟主机的参数
    - **location：**配置请求路由，页面处理

如：

```sh
#全局配置------------------------------------------------------------------------
...              

#events 配置--------------------------------------------------------------------
events {
   ...
}
#http 配置----------------------------------------------------------------------
http
{
	#http 全局配置
    ...
    #server 全局配置
    server
    { 
    	#server全局配置
        ...       
        #location配置
        location [PATTERN]   
        {
            ...
        }
    }
}
```

### 全局配置

```sh
#全局配置-------------------------------------------------------------
#指定nginx运行的用户及用户组,默认为nobody
#user  nobody nobody;

#开启线程数，最大值可设逻辑CPU核数
#worker_processes  1; 

#定位全局错误日志文件，级别以notice显示，还有debug,info,warn,error,crit模式，debug输出最多，crir输出最少，根据实际环境而定
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#指定进程id的存储文件位置
#pid        logs/nginx.pid;

#指定一个nginx进程打开的最多文件描述符数目，受系统进程的最大打开文件数量限制
#worker_rlimit_nofile 65535

#envents 配置----------------------------------------------------------
events {
    ...
}

#http 配置-------------------------------------------------------------
http {
    ...
}
```



### events 配置

```sh
events {
    #设置工作模式为epoll,除此之外还有select,poll,kqueue,rtsig和/dev/poll模式
    use epoll;
    #定义每个进程的最大连接数,受系统进程的最大打开文件数量限制
    worker_connections  1024;
}
```



### http 配置

```sh
http {
    #主模块指令，实现对配置文件所包含的文件的设定，可以减少主配置文件的复杂度
    include       mime.types;
    
    #核心模块指令，默认设置为二进制流，也就是当文件类型未定义时使用这种方式
    default_type  application/octet-stream;
    
    #下面代码为日志格式的设定，main为日志格式的名称，可自行设置，后面引用
	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	                  '$status $body_bytes_sent "$http_referer" '
	                  '"$http_user_agent" "$http_x_forwarded_for"';
	#引用日志main格式
	access_log  logs/access.log  main;

    #设置允许客户端请求的最大的单个文件字节数
    client_max_body_size 20M;
    #指定来自客户端请求头的headebuffer大小
    client_header_buffer_size  32k;
    #指定连接请求试图写入缓存文件的目录路径
    client_body_temp_path /dev/shm/client_body_temp;
    #指定客户端请求中较大的消息头的缓存最大数量和大小，目前设置为4个32KB
    large client_header_buffers 4 32k;

    #开启高效文件传输模式
    sendfile        on;
    #开启防止网络阻塞
    tcp_nopush     on;
    #开启防止网络阻塞
    tcp_nodelay    on;

    #设置客户端连接保存活动的超时时间
    #keepalive_timeout  0; # 无限时间
    keepalive_timeout  65;

    #设置客户端请求读取header超时时间
    client_header_timeout 10;
    #设置客户端请求body读取超时时间
    client_body_timeout 10;

    #HttpGZip模块配置
    #开启gzip压缩
    gzip  on;
    #设置允许压缩的页面最小字节数
    gzip_min_length 1k;
    #申请4个单位为16K的内存作为压缩结果流缓存
    gzip_buffers 4 16k;
    #设置识别http协议的版本，默认为1.1
    gzip_http_version 1.1;
    #指定gzip压缩比，1-9数字越小，压缩比越小，速度越快
    gzip_comp_level 2;
    #指定压缩的类型
    gzip_types text/plain application/x-javascript text/css application/xml;
    #让前端的缓存服务器进过gzip压缩的页面
    gzip_vary on; 
    
    # server配置
    server {
        
    }    
}
```

### server 配置

```sh
server {
    #单连接请求上限次数
    keepalive_requests 120; 
    #监听端口
    listen       88;
    #监听地址，可以是ip，最好是域名
    server_name  111.222.333.123;
    #server_name  www.123.com;
    #设置访问的语言编码
    charset utf-8;
    #设置虚拟主机访问日志的存放路径及日志的格式为main
    access_log  /www/wwwlogs/111.222.333.123.log main; #响应日志
    error_log  /www/wwwlogs/111.222.333.123.log main; #错误日志
    
    #PHP-INFO-START  PHP引用配置，可以注释或修改
    include enable-php-74.conf;
    #PHP-INFO-END
    
    #REWRITE-START URL重写规则引用
    include /www/server/panel/vhost/rewrite/111.222.333.123.conf;
    #REWRITE-END
    
    #设置主机基本信息
    #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
    location  ~*^.+$ {
    	#根目录
        root html;  
        #设置默认页
        index  index.html index.htm;
        #拒绝的ip,黑名单
        deny 127.0.0.1;  
        #允许的ip，白名单
        allow 172.18.5.54; 
    } 
    
    #禁止访问的文件或目录
    location ~ ^/(\.user.ini|\.htaccess|\.git|\.svn|\.project|LICENSE|README.md)
    {
        return 404;
    }
    
    #SSL证书验证目录相关设置
    location ~ \.well-known{
        allow all;
    }
    
	#图片资源配置
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires      30d;
        error_log /dev/null;
        access_log off;
    }
    
    #网站js与css资源配置
    location ~ .*\.(js|css)?$
    {
        expires      12h;
        error_log /dev/null;
        access_log off; 
    }
    
    #访问异常页面配置
    error_page  404              /404.html;
    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
		root   html;
	}
}
```

## 代理

### 正向代理

正向代理，任何可以连接到该代理服务器的软件，就可以通过代理访问任何的其他服务器，然后把数据返回给客户端，这里代理服务器只对**客户端**负责。

正向代理时，由客户端发送对某一个目标服务器的请求，代理服务器在中间将请求转发给该目标服务器，目标服务器将结果返回给代理服务器，代理服务器再将结果返回给客户端。

使用正向代理时，客户端是需要配置代理服务的地址、端口、账号密码（如有）等才可使用的。

![正向代理流程](正向代理流程.png)

通过上图可以看到，客户端并没有直接与服务器相连。正向代理隐藏了真实的客户端地址。可以很好地保护客户端的安全性。



#### 正向代理的适用场景

- **访问被禁止的资源**（让客户端访问原本不能访问的服务器。可能是由于路由的原因，或者策略配置的原因，客户端不能直接访问某些服务器。为了访问这些服务器，可通过代理服务器来访问）

- - 突破网络审查（比如谷歌、youtube…）

![正向代理](正向代理应用.png)

- - 再比如客户端IP被服务器封禁，可以绕过IP封禁
  - 也可以突破网站的区域限制

- **隐藏客户端的地址**（对于被请求的服务器而言，代理服务器代表了客户端，所以在服务器或者网络拓扑上，看不到原始客户端）

- **进行客户访问控制**

- - 可以集中部署策略，控制客户端的访问行为（访问认证等）
  - 记录用户访问记录（上网行为管理）
  - 内部资源的控制（公司、教育网等）

- **加速访问资源**

- - 使用缓冲特性减少网络使用率（代理服务器设置一个较大的缓冲区，当有外界的信息通过时，同时也将其保存到缓冲区中，当其他用户再访问相同的信息时， 则直接由缓冲区中取出信息，传给用户，以提高访问速度。）

- **过滤内容**（可以通过代理服务器统一过滤一些危险的指令/统一加密一些内容、防御代理服务器两端的一些攻击性行为）

### 反向代理

服务器根据客户端的请求，从其关系的一组或多组后端服务器（如Web服务器）上获取资源，然后再将这些资源返回给客户端，客户端只会得知代理服务器的IP地址，而不知道在代理服务器后面的服务器集群的存在。

![img](反向代理流程.png)

反向代理整个流程：由客户端发起对代理服务器的请求，代理服务器在中间将请求转发给某一个服务器，服务器将结果返回给代理服务器，代理服务器再将结果返回给客户端。

#### **反向代理的适用场景**

- **负载均衡**

- - 如果服务器集群中有负荷较高者，反向代理通过URL重写，根据连线请求从负荷较低者获取与所需相同的资源或备援。可以有效降低服务器压力，增加服务器稳定性

- **提升服务器安全性**

- - 可以对客户端隐藏服务器的IP地址
  - 也可以作为应用层防火墙，为网站提供对基于Web的攻击行为（例如DoS/DDoS）的防护，更容易排查恶意软件等

- **加密/SSL加速：**将SSL加密工作交由配备了SSL硬件加速器的反向代理来完成

- **提供缓存服务**，加速客户端访问

- - 对于静态内容及短时间内有大量访问请求的动态内容提供缓存服务

- **数据统一压缩**

- - 节约带宽
  - 为网络带宽不好的网络提供服务

- **统一的访问权限控制**

- **统一的访问控制**

- **突破互联网的封锁**

- - 突破谷歌访问封锁

![img](反向代理应用.png)

- - - 也就是说，不需要客户端进行代理，我们通过谷歌代理网站（该代理服务器可以访问谷歌，而我们可以访问该公开的代理服务器），也可以突破封锁。

- **为在私有网络下**（如局域网）的服务器集群提供NAT穿透及外网发布服务
- **上传下载减速控制**

### 区别

- 正向代理为客户端服务。
- 反向代理为服务器端服务。

## Rewrite

### 地址重写，地址转发，重定向

**地址重写：**为了标准化网址，比如输入baidu.com和www.baidu.com，都会被重写到www.baidu.com，而且我们在浏览器看到的也会是 www.baidu.com

**地址转发：**指在网络数据传输过程中数据分组到达路由器或桥接器后，该设备通过检查分组地址并将数据转发到最近的局域网的过程。

不同点：

- 地址重写会改变浏览器中的地址，使之变成重写成浏览器最新的地址。而地址转发他是不会改变浏览器的地址的。
- 地址重写会产生两次请求，而地址转发只会有一次请求。
- 地址转发一般发生在同一站点项目内部，而地址重写且不受限制。
- 地址转发的速度比地址重定向快。

### URL 重写

在 Nginx 中通过在 server 或 location 中配置 rewrite 指令实现

#### 语法

```sh
rewrite regex replacement [flag];
```

- **rewrite**：该指令是实现URL重写的指令
- **regex**：用于匹配URI的正则表达式
- **replacement**：将regex正则匹配到的内容替换成 replacement。
- **flag**:标记
  - **last:** 本条规则匹配完成后，继续向下匹配新的location URI 规则。(不常用)
  - **break:** 本条规则匹配完成即终止，不再匹配后面的任何规则(不常用)。
  - **redirect:** 返回302临时重定向，浏览器地址会显示跳转新的URL地址。
  - **permanent:** 返回301永久重定向。浏览器地址会显示跳转新的URL地址。

#### 使用

```sh
rewrite ^/(.*) http://www.baidu.com/$1 permanent;
```

- **rewrite** ：指令。
- **regex**：正则表达式，匹配完整的域名和后面的路径地址。
- **replacement**：$1是取regex部分()里面的内容。如果匹配成功后跳转到的URL。
- **flag；**permanent，代表永久重定向的含义，即跳转到 http://www.baidu.com/$1 地址上。

| **字符**  | **描述**                                                     |
| --------- | ------------------------------------------------------------ |
| \         | 转义字符标记，如 `\n`匹配一个换行符，而`\$`则匹配`$`         |
| ^         | 匹配输入字符串的起始位置                                     |
| $         | 匹配输入字符串的结束位置                                     |
| *         | 匹配前面的字符零次或多次。如`ol*`能匹配`o`及`ol`、`oll`      |
| +         | 匹配前面的字符一次或多次。如`ol+`能匹配`ol`及`oll`、`oll`，但不能匹配`o` |
| ?         | 匹配前面的字符零次或一次，例如`do(es)?`能匹配`do`或者`does`，`?`等效于`{0,1}` |
| .         | 匹配除“`\n`之外的任何单个字符，若要匹配包括“\n”在内的任意字符，请使用诸如`[.\n]`之类的模式。 |
| (pattern) | 匹配括号内pattern并可以在后面获取对应的匹配，常用`$0...$9`属性获取小括号中的匹配内容，要匹配圆括号字符需要`\(Content\)` |

### if 指令使用

语法：

```shell
if (condition) {
  // ....
}
```

Rewrite 可以使用的全局变量

- **$args**: 该变量中存放了请求URL中的请求指令。比如 `http://127.0.0.1:3001?arg1=value1&arg2=value2` 中的 `arg1=value1&arg2=value2`

- **$content_length**: 该变量中存放了请求头中的Content-length字段
- **$content_type**: 该变量中存放了请求头中的 Content-type字段
- **$document_root**: 该变量中存放了针对当前请求的根路径

- **$document_uri**: 该变量中存放了请求的当前URI, 但是不包括请求指令。比如 `http://xxx.abc.com/home/1?arg1=value1& arg2=value2;` 中的 `/home/1`

- **$host:** 变量中存放了请求的URL中的主机部分字段，比如`http://xxx.abc.com:8080/home`中的 `xxx.abc.com`
- **$httphost:**  该变量与host唯一区别带有端口号：比如上面的是 `xxx.abc.com:8080`
- **$http_user_agent**: 变量中存放客户端的代理信息
- **$http_cookie**, 该变量中存放客户端的cookie信息
- **$remote_addr** 该变量中存放客户端的地址
- **$remote_port** 该变量中存放了客户端与服务器建立连接的端口号
- **$remote_user** 变量中存放客户端的用户名
- **$request_body_file** 变量中存放了发给后端服务器的本地文件资源的名称

- **$request_method** 变量中存放了客户端的请求方式，比如 ‘GET’、'POST’等
- **$request_filename** 变量中存放了当前请求的资源文件的路径名
- **$request_uri** 变量中存放了当前请求的URI，并且带请求指令
- **$querystring**  和变量args一样

- **$scheme** 变量中存放了客户端请求使用的协议，比如 ‘http’, 'https’等
- **$server_protocol** 变量中存放了客户端请求协议的版本, 比如 ‘HTTP/1.0’、‘HTTP/1.1’ 等

#### 正则表达式：

##### 1、变量匹配

- `~`：表示匹配过程中对大小写敏感
- `~*`：表示匹配过程中对大小写不敏感
- `!~` ：如果`~`匹配失败时，那么该条件就为true
- `!~*'`：如果 `~*` 匹配失败时，那么该条件就为true

举个栗子：

```shell
if ($http_user_agent ~ MSIE) {
	...
}
```

含义：$http_user_agent值中是否含有 MSIE 字符串，如果包含为true，否则为false

##### 2、判断请求的文件是否存在

```shell
if (-f $request_filename) {
  # 判断请求的文件是否存在
}

if (!-f $request_filename) {
  # 判断请求的文件是否不存在
}
```

其他指令：

- -f和!-f用来判断请求文件是否存在
- -d和!-d用来判断请求目录是否存在
- -e和!-e用来判断是请求的文件或者目录否存在
- -x和!-x用来判断请求的文件是否可执行

##### 3、判断手机访问

```shell
if ( $http_user_agent ~* "(Android)|(iPhone)|(Mobile)|(WAP)|(UCWEB)" ){
  rewrite ^/$  http://www.cnblogs.com  permanent；
}
```

##### 4、其他

现在我们使用if指令来对nginx加一些判断；比如说我们访问http://xxx.abc.com:8080/home时候，如果$host = ‘xxx.abc.com’ 的时候，就做重定向跳转，nginx配置代码如下：

```shell
server {
  listen 8088;
  server_name xxx.abc.com;
  location / {
    proxy_pass http://127.0.0.1:3001;
    if ($host = 'xxx.abc.com') {
      rewrite ^/(.*) http://www.cnblogs.com redirect;
    }
  }
}
```

### 防盗链

https://www.cnblogs.com/tugenhua0707/p/10798762.html

**什么是防盗链？**

盗链可以理解盗图链接，也就是说把别人的图片偷过来用在自己的服务器上，那么防盗链可以理解为防止其他人把我的图片盗取过去。

**防盗链的实现原理：**

客户端向服务器端请求资源时，为了减少网络带宽，提高响应时间，服务器一般不会一次将所有资源完整地传回客户端。比如请求一个网页时，首先会传回该网页的文本内容，当客户端浏览器在解析文本的过程中发现有图片存在时，会再次向服务器发起对该图片资源的请求，服务器将存储的图片资源再发送给客户端。但是如果这个图片是链接到其他站点的服务器上去了呢，比如在我项目中，我引用了的是淘宝中的一张图片的话，那么当我们网站重新加载的时候，就会请求淘宝的服务器，那么这就很有可能造成淘宝服务器负担。因此这个就是盗链行为。因此我们要实现防盗链。

**实现防盗链：**

使用http协议中请求头部的Referer头域来判断当前访问的网页或文件的源地址。通过该头域的值，我们可以检测访问目标资源的源地址。如果目标源地址不是我们自己站内的URL的话，那么这种情况下，我们采取阻止措施，实现防盗链。但是注意的是：Referer头域中的值是可以被更改的。因此该方法也不能完全安全阻止防盗链。

**使用Nginx服务器的Rewrite功能实现防盗链。**

Nginx中有一个指令 valid_referers. 该指令可以用来获取 Referer 头域中的值，并且根据该值的情况给 Nginx全局变量 invalidreferer赋值。如果Referer头域中没有符合validreferers指令的值的话，invalidreferer赋值。如果Referer头域中没有符合validreferers指令的值的话，invalid_referer变量将会赋值为

valid_referers 指令基本语法如下：

```
valid_referers none | blocked | server_names | string
```

**none:** 检测Referer头域不存在的情况。

**blocked：** 检测Referer头域的值被防火墙或者代理服务器删除或伪装的情况。那么在这种情况下，该头域的值不以"http://" 或 “https://” 开头。

**server_names:** 设置一个或多个URL，检测Referer头域的值是否是URL中的某个。

因此我们有了 valid_referers指令和$invalid_referer变量的话，我们就可以通过 Rewrite功能来实现防盗链。
下面我们介绍两种方案：

1. 根据请求资源的类型。
2. 根据请求目录。

#### 根据请求文件类型实现防盗链配置实列如下：

```shell
server {
  listen 8080;
  server_name xxx.abc.com
  location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip)$ {
    valid_referers none blocked www.xxx.com www.yyy.com *.baidu.com  *.tabobao.com;
    if ($invalid_referer) {
      rewrite ^/ http://www.xxx.com/images/forbidden.png;
    }
  }
}
```

如上基本配置，当有网络连接对以 gif、jpg、png为后缀的图片资源时候、当有以swf、flv为后缀的媒体资源时、或以 rar、zip为后缀的压缩资源发起请求时，如果检测到Referer头域中没有符合 valid_referers指令的话，那么说明不是本站的资源请求。

location ~* ^.+.(gif|jpg|png|swf|flv|rar|zip)$ 该配置的含义是 设置防盗链的文件类型。

valid_referers none blocked www.xxx.com www.yyy.com *.baidu.com *.tabobao.com; 可以理解为白名单，允许文件链出的域名白名单，如果请求的资源文件不是以这些域名开头的话，就说明请求的资源文件不是该域下的请求，因此可以判断它是盗链。因此如果不是该域下的请求，就会使用 Rewrite进行重定向到 http://www.xxx.com/images/forbidden.png 这个图片，比如这张图片是一个x或其他的标识，然后其他的网站就访问不了你这个图片哦。

#### 根据请求目录实现防盗链的配置实列如下

```shell
server {
  listen 8080;
  server_name xxx.abc.com
  location /file/ {
    root /server/file/;
    valid_referers none blocked www.xxx.com www.yyy.com *.baidu.com  *.tabobao.com;
    if ($invalid_referer) {
      rewrite ^/ http://www.xxx.com/images/forbidden.png;
    }
  }
}
```

其他例子

```shell
例子一（域名跳转）:
    server {
            listen 80;
            server_name   abc.com;
            rewrite   ^/(.*)     http://www.ab c.com/$1 permanent;  # 跳转到www.abc.com网址上
        }
例子二：
  server {
            listen 80;
            server_name   www.myweb.com www.web.info
            if($host ~ myweb\.info){                        #"."需要使用“\”转义，这里是匹配到www.web.info时
                     rewrite ^(.*)  http://www.myweb.com/&1 permanent;   #永久重定向到http://www.myweb.com网址上&1是匹配的uri
            }
        }
例子三(防盗链)：
location ~* \.(gif|jpg|png|swf|flv)$ {
    valid_referers none blocked www.vison.com www.wsvison.com;  #这里表示Referer头域中的值是none或者blocked或者后面这些网址才会返回去正常的gif|jpg|png|swf|flv文件，否则执行下面if块代码
    if ($invalid_referer) {  #上面没有匹配成功，$invalid_referer值为1，否则为0
        return 404;
    } //防盗链
}       
其他例子：    
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
} //如果UA包含"MSIE"，rewrite请求到/msid/目录下

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
 } //如果cookie匹配正则，设置变量$id等于正则引用部分

if ($request_method = POST) {
    return 405;
} //如果提交方法为POST，则返回状态405（Method not allowed）。return不能返回301,302

if ($slow) {
    limit_rate 10k;
} //限速，$slow可以通过 set 指令设置

if (!-f $request_filename){
    break;
    proxy_pass  http://127.0.0.1; 
} //如果请求的文件名不存在，则反向代理到localhost 。这里的break也是停止rewrite检查

if ($args ~ post=140){
    rewrite ^ http://example.com/ permanent;
} //如果query string中包含"post=140"，永久重定向到example.com
```



