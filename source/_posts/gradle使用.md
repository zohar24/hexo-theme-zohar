---
title: Gradle 安装
top: false
author: zhaohuan
date: 2022-08-14 15:27:31
tags:
- 总结
- gradle
categories:
- gradle
---

## Gradle 安装前的准备

Gradle 可以安装在 Linux，macOS，Windows 等主流操作系统，唯一的要求就是操作系统上已经安装了 Java JDK 7 及以上版本。可以通过 java -version 验证是否满足条件，以下是我的例子：

```bash
> java -version

java version "1.8.0_152"
Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)
```

## 第一种安装方式

### 1. [访问官网](https://link.ld246.com/forward?goto=https%3A%2F%2Fgradle.org%2Freleases)下载最新版安装包

官方提供以下 2 种压缩包可供下载

- binary-only (如果不需要源码、文档，选择下载这个压缩包就够了)
- complete (包含文档及源码)

### 2. 解压

以下操作以 gradle 4.9 版本为例

#### 2-1. Linux & macOS 用户

将下载的压缩包解压到任意你想要存放的位置，如：

```bash
> mkdir /opt/gradle
> unzip -d /opt/gradle gradle-4.9-bin.zip
> ls /opt/gradle/gradle-4.9

LICENSE  NOTICE  bin  getting-started.html  init.d  lib  media
```

#### 2-2. Windows 用户

将下载的压缩包解压到任意你想要存放的位置，如：

> 在 c 盘新建一个 Gradle 目录
>
> 将下载回来的压缩包解压到 c:\Gradle 下
>
> 最终路径格式 c:\Gradle\gradle-4.9

### 3. 配置系统环境变量

为了更方便的使用 gradle 命令，我们需要将 gradle 安装目录下的 bin 文件夹路径加入到 path 环境变量

#### 3-1. Linux & macOS 用户

```bash
export PATH=$PATH:/opt/gradle/gradle-4.9/bin
```

#### 3-2. Windows 用户

> 右击 我的电脑 → 属性 → 高级系统配置 → 环境变量 → 系统变量
>
> 找到 Path 变量 → 编辑
>
> 将 C:\Gradle\gradle-4.9\bin 添加进去
>
> 保存，退出

至此，Gradle 已经成功安装到自己的机器，并可以方便的调用 gradle 命令执行各种操作了

下面👇介绍另外一种更方便、高效的安装方法，只需执行一条命令就能完成 Gradle 的安装

## 第二种安装方式

通过包管理软件快速安装 gradle

### 1. macOS 用户

#### 通过 [Homebrew](https://link.ld246.com/forward?goto=http%3A%2F%2Fbrew.sh%2F) 安装

```bash
brew install gradle
```

### 2. Windows 用户

#### 2-1. 通过 [Scoop](https://link.ld246.com/forward?goto=http%3A%2F%2Fscoop.sh%2F) 安装

一款灵感来自 Homebrew 的 Windows 系统包管理软件

```bash
scoop install gradle
```

#### 2-2. 通过 [Chocolatey](https://link.ld246.com/forward?goto=https%3A%2F%2Fchocolatey.org%2F) 安装

另一款 Windows 系统包管理软件

```bash
choco install gradle
```

### 3. Linux 用户

#### 3-1. 通过 [SDKMAN!](https://link.ld246.com/forward?goto=http%3A%2F%2Fsdkman.io%2F) 安装

```bash
skd install gradle
```

## 验证 Gradle 是否安装成功

打开命令行工具输入以下命令

```bash
gradle -v
```

应该看到类似输出，以我在 macOS 系统为例：

```bash
------------------------------------------------------------
Gradle 4.9
------------------------------------------------------------
 
Build time:   2018-07-16 08:14:03 UTC
Revision:     efcf8c1cf533b03c70f394f270f46a174c738efc

Groovy:       2.4.12
Ant:          Apache Ant(TM) version 1.9.11 compiled on March 23 2018
JVM:          1.8.0_152 (Oracle Corporation 25.152-b16)
OS:           Mac OS X 10.13.4 x86_64
```

详见 Gradle 官方 [安装文档](https://link.ld246.com/forward?goto=https%3A%2F%2Fdocs.gradle.org%2Fcurrent%2Fuserguide%2Finstallation.html)

