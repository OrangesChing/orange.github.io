---
title: JDK安装与部署
date: 2021-04-28 08:59:00
tags: 
  - 环境搭建与配置
categories:
  - Java
---

# CentOS安装

## 下载JDK

可在Windows上下载linux tar.gz结尾的包，在再上传到服务器

```
wget https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html
```

## 解压

```
# 创建文件夹
cd /usr/local
mkdir java
cd java
# 解压
tar -zxvf
cd jdk （解压后的文件夹名）
# 查看JAVA_HOME路径
pwd
```

## 配置

```
# 编辑配置，配置环境变量
vi /etc/profile
	i
	# 添加以下内容
	export JAVA_HOME = jdk的主目录路径(上述pwd输出的日志)
	export PATH = $JAVA_HOME/bin:$PATH
	export CLASSPATH = .:$JAVA_HOME/lib
	# 内容结束
	:wq
# 配置生效
source /etc/profile
```

## 验证安装

```
java -version
```

# Windows安装

