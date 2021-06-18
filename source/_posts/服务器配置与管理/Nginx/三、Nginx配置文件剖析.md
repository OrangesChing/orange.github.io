---
title: 三、Nginx配置文件剖析
date: 2021-05-26 10:59:00
categories:
  - 服务器配置与管理
  - Nginx
---

# Nginx配置文件结构

**nginx 文件结构**

```
...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

- **全局块**：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等
- **events块**：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等
- **http块**：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等
- **server块**：配置虚拟主机的相关参数，一个http中可以有多个server
- **location块**：配置请求的路由，以及各种页面的处理情况

# 全局块

从配置文件开始到 events 块之间的内容，主要会设置一些影响nginx 服务器整体运行的配置指令

主要包括配 置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以 及配置文件的引入等。

```
# 配置运行 nginx 服务器用户
user nobody nobody;

# 配置允许生成的 worker process 数
worker_processes auto;
worker_processes 4;
# 这个数字跟电脑 CPU 核数要保持一致这个指令能查看CPU核数：grep ^proces /proc/cpuinfo | wc -l

# 配置 nginx 进程 PID 存放路径
pid logs/nginx.pid;
# 这个文件保存的就是一个数字，nginx master 进程的进程号

# 配置错误日志的存放路径
error_log logs/error.log;
error_log logs/error.log error;

# 配置文件的引入
include mime.types; include fastcgi_params; include ../../conf/*.conf;

```

# Event块

Event块涉及的指令主要影响 Nginx 服务器与用户的网络连接

常用的设置包括：

- 是否开启对多 work process 下的网络连接进行序列化
- 是否 允许同时接收多个网络连接
- 选取哪种事件驱动模型来处理连接请求
- 每个 word process 可以同时支持的最大连接数等

这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

```
# 设置网络连接的序列化
accept_mutex on;
# 对多个 nginx 进程接收连接进行序列化，防止多个进程对连接的争抢（惊群现象）

# 设置是否允许同时接收多个网络连接
multi_accept off;

# 事件驱动模型的选择
use select|poll|kqueue|epoll|rtsig|/dev/poll|eventport

# 配置最大连接数
worker_connections 512;
```

> 惊群现象：一个网路连接到来，多个睡眠的进程被同时叫醒，但只有一个进程能获得链接，这样会影响系统性能

# HTTP块

`Nginx` 服务器配置中最频繁的部分，<u>代理、缓存和日志</u>定义等绝大多数功能和<u>第三方模块</u>的配置都在这里。`http`块包括 `http全局块`、`server块`

## HTTP全局块

http全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等

```
# 定义 MIME-Type
include mime.types; default_type application/octet-stream;

# 自定义服务日志
access_log logs/access.log main; 
access_log off;

# 配置允许 sendfile 方式传输文件
sendfile off;
sendfile on; 
sendfile_max_chunk 128k;
# nginx 每个 worker process 每次调用 sendfile() 传输的数据量的最大值

# 配置连接超时时间
keepalive_timeout 75s 65s;
# 与用户建立连接后，nginx 可以保持这些连接一段时间，默认 75s 下面的 65s 可以被Mozilla/Konqueror 识别，是发给用户端的头部信息Keep-Alive值

# 单连接请求数上限
keepalive_requests 100
# 和用户端建立连接后，用户通过此连接发送请求；这条指令用于设置请求的上限数
```

## Server块（多个）

这块和虚拟主机有密切关系
每个`http 块`可以包括多个` server 块`，而每个 `server 块`就相当于一个虚拟主机。
而每个`server 块`也分为全局 `server 块`，以及可以同时包含多个 `locaton 块`

当客户端向 Nginx 服务器发送请求时，Nginx首先会根据 **IP地址和端口（listen 属性）** 对server服务器进行配置；如果IP地址匹配不成功，会对 **域名（server_name属性）** 进行匹配；如果域名也匹配不成功，则会**默认匹配第一个server服务器**

> **虚拟主机**（英语：virtual hosting）或称 **共享主机**（shared web hosting），又称**虚拟服务器**，是一种在单一主机或主机群上，实现多网域服务的方法，可以运行多个[网站](https://baike.baidu.com/item/网站)或服务的技术。虚拟主机之间完全独立，并可由用户自行管理，虚拟并非指不存在，而是指空间是由实体的服务器延伸而来，其[硬件](https://baike.baidu.com/item/硬件)系统可以是基于服务器群，或者单个服务器。
>
> 虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了 节省互联网服务器硬件成本

### Server全局块

最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或IP配置

####  基于IP地址和端口的虚拟主机配置

```
listen *:80 | *:8000; # 监听所有的 80 和 8000 端口

listen 192.168.1.10:8000; 
listen 192.168.1.10; 
listen 8000; 
# 等同于 listen *:8000; 
listen 192.168.1.10 default_server backlog=511; 
# 该 ip 的连接请求默认由此虚拟主机处理；最多允许 1024 个网络连接同时处于挂起状态
```

#### 基于名称的虚拟主机配置

```
server_name myserver.com www.myserver.com;

server_name *.myserver.com www.myserver.* myserver2.*; # 使用通配符

# 不允许的情况：server_name www.ab*d.com; *只允许出现在 www 和 com 的位置

server_name ~^www\d+.myserver.com$; # 使用正则

# nginx 的配置中，可以用正则的地方，都以`~`开头

# 从nginx~0.7.40 开始，server_name 中的正则支持 字符串捕获功能（capture）

server_name ~^www.(.+).com$; 
# 当请求通过 www.myserver.com 请求时， myserver 就被记录到`$1`中，在本 server 的上下文中就可以使用
```

如果一个名称 被多个虚拟主机的 server_name 匹配成功，则按以下优先级匹配：

1. 准确匹配到 server_name
2. 通配符在开始时匹配到 server_name
3. 通配符在结尾时匹配到 server_name
4. 正则表达式匹配 server_name
5. 先到先得

### Location块（多个）

`Location块`的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 `server_name/uri-string`），对虚拟主机名称 （也可以是IP 别名）之外的字符串（例如 前面的` /uri-string`）进行匹配，对特定的请求进行处理。即<u>匹配URL</u>， <u>地址定向、数据缓 存和应答控制</u>等功能，还有许多<u>第三方模块</u>的配置也在这里进行

**Location配置语法规则：`location` `[匹配方式]` `url` `{命令序列}`**

这里内容分 2 块，匹配方式和 `url` ， 其中 `url` 又分为 标准 `url`(如：127.0.0.1/home) 和正则 `url` 

#### 匹配方式

**所有匹配方式：**

- 不带匹配方式：必须以指定模式开始
- `=`:  用于`标准 uri`前，必须与指定的模式精确匹配，成功则立即处理，精确匹配，不在子路径生效，优先级最高
- `^~`: 用于`标准 uri`前，类似于不带匹配方式的行为，也是以指定模式开始，不同的是，如果模式匹配，
  那么就停止搜索其他模式了
- `~`:  用于`正则 uri`前，url包含的正则表达式要区分大小写
- `~*`: 用于`正则 uri`前， url包含的正则表达式不区分大小写

**匹配顺序：**

1：带有`=`的精确匹配优先
2：没有修饰符的精确匹配
3：正则表达式按照他们在配置文件中定义的顺序
4：带有`^~`修饰符的，开头匹配
5：带有`~` 或`~*` 修饰符的，如果正则表达式与URI匹配
6：没有修饰符的，如果指定字符串与URI开头匹配

#### 命令序列

##### alias

别名配置，用于访问文件系统，在匹配到location配置的URL路径后，指向【alias】配置的路径。如：

```
location /test/ 
{ 
	alias /home/orange/img/; 
}
# 请求/test/1.jpg（省略了协议与域名），将会返回文件/home/orange/img/1.jpg
```

##### root

根路径配置，用于访问文件系统，在匹配到location配置的URL路径后，指向【root】配置的路径，并把location配置路径附加到其后。如：

```
location /test/
{ 
	root /home/orange/img/; 
}
# 请求/test/1.jpg，将会返回文件/home/orange/img/test/1.jpg
# 相较于alias，使用root会把/test/附加到根目录之后。
```

> root和alias的区别：
>
> root是追加URL
> alias是改变URL
>
> 具体可见例子

##### proxy_pass

反向代理配置，<u>用于代理请求，适用于前后端负载分离或多台机器、服务器负载分离的场景</u>

在匹配到location配置的URL路径后，转发请求到【proxy_pass】配置的URL

是否会附加location配置路径与【proxy_pass】配置的路径后是否有`/`有关，<u>有`/`则不附加</u>，如配置为：

```
# 配法1：带/
location /test/ 
{ 
	proxy_pass http://127.0.0.1:8080/; 
}
# 请求/test/1.jpg将会被转发到http://127.0.0.1:8080/1.jpg

# 配法2：不带/
location /test/ 
{ 
	proxy_pass http://127.0.0.1:8080; 
}
# 请求/test/1.jpg将会被转发到http://127.0.0.1:8080/test/1.jpg
```

参考：

https://blog.csdn.net/qq_40036754/article/details/102463099?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162199566116780265480863%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162199566116780265480863&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-102463099.pc_search_result_no_baidu_js&utm_term=nginx&spm=1018.2226.3001.4187

https://blog.csdn.net/wangjun5159/article/details/109339922