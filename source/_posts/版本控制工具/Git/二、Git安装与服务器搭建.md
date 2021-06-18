---
title: 二、Git安装与服务器搭建
date: 2021-04-29 10:39:00
tags: 
  - 环境搭建与配置
categories:
  - 版本控制工具
  - Git
---

# 安装Git

## Windows安装

下载最新版Git安装包：https://git-scm.com/download/win

打开安装包后，**可一直使用默认选项，点下一步**，下面是各步骤选项详述

1. 设置安装路径

![安装路径](二、Git安装与服务器搭建\安装路径.jpg)

2. 选择安装组件

![安装组件](二、Git安装与服务器搭建\安装组件.jpg)

3. 选择默认编辑器

   ![image-20210427134320574](二、Git安装与服务器搭建\image-20210427134320574.png)

4. 修改系统的环境变量

![环境变量](二、Git安装与服务器搭建\环境变量.jpg)

4. SSL的证书的选择

![SSH证书](二、Git安装与服务器搭建\SSH证书.jpg)

> **https：（全称：Hyper Text Transfer Protocol over Secure Socket Layer）**
>
> 简单讲是HTTP的安全版。即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同[http](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2Fhttp):体系。用于安全的HTTP数据传输。
>
> [参考链接：百科](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2Fhttps%2F285356%3Ffr%3Daladdin)

5. 配置行尾结束符

![配置行尾结束符](二、Git安装与服务器搭建\配置行尾结束符.jpg)

**配置行尾结束符**

| 系统    | 换行符 |
| ------- | ------ |
| windows | \n\r   |
| unix    | \n     |
| mac     | \r     |

6. 配置终端仿真

![配置终端仿真](二、Git安装与服务器搭建\配置终端仿真.jpg)

> 大多数其他Cygwin/MSYS终端一样，MinTTY也是基于pseudo终端("pty")设备的。但是MinTTY并不能完全替代windows的[命令提示符](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6)。windows上自带简单的[文本输出](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E6%96%87%E6%9C%AC%E8%BE%93%E5%87%BA)的原生态的[命令提示符](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6)通常可以很好的工作，但交互性更好的诸如MinTTY这样的应用程序却可能出现故障——虽然通常都有应对方案。这就是为什么MinTTY不能完全替代windows自带的[命令提示符](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6)。
>
> [参考链接：百科](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FMinTTY%2F9579126%3Ffr%3Daladdin)

7. 其他的配置

![其他配置](二、Git安装与服务器搭建\其他配置.jpg)

> 认证管理器：[参考链接](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FMicrosoft%2FGit-Credential-Manager-for-Windows)就是Github的账号等认证机制
>
> 符号链接：[参考](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fgit-for-windows%2Fgit%2Fwiki%2FSymbolic-Links)官方介绍[参考博文](https://link.jianshu.com?t=http%3A%2F%2Fblog.csdn.net%2Fu013777351%2Farticle%2Fdetails%2F50557260)

## CentOS安装

### 仓库安装

使用yum源仓库安装，安装方便，但缺点在于安装的版本不是最新版本且无法安装历史版本，可使用下面语句查看源仓库的版本

```
yum info git
```

![仓库版本](二、Git安装与服务器搭建\image-20210427095620314.png)

使用下述语句安装

```
yum -y install git
```

安装的git在`/usr/bin/git`下

### 源码安装

#### 源码安装依赖的依赖库安装

```shell
yum remove git -y
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker -y
```

#### 下载并解压源码

https://mirrors.edge.kernel.org/pub/software/scm/git/找到最新版本的链接

```shell
cd /usr/local/src/
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
tar zxvf git-2.9.5.tar.gz
```

#### 配置安装路径，开始编译安装

安装路径为`/user/local/git`

```shell
cd git-2.9.5
./configure --prefix=/usr/local/git/
make
make install
```

#### 添加到环境变量

```shell
vim /etc/profile
   i
   文件末尾添加内容:export PATH="/usr/local/git/bin:$PATH"
   :wq
source /etc/profile
```

#### 查看是否安装成功（查看版本号）

```shell
git --version  
```

#### 将Git设置为默认路径，不然后面克隆时会报错

```shell
ln -sf /usr/local/git/bin/git-upload-pack /bin/git-upload-pack
ln -sf /usr/local/git/bin/git-receive-pack /bin/git-receive-pack
```

# Git服务器搭建



## 创建用户

```shell
# 添加git账户
$ adduser git

# 修改git的密码
$ passwd git
# 然后两次输入git的密码确认后

# 查看git是否安装成功
$ cd /home && ls -al
# 如果已经有了git，那么表示成，参考如下：
# drwxr-xr-x.  5 root root 4096 Apr  4 15:03 .
# dr-xr-xr-x. 19 root root 4096 Apr  4 15:05 ..
# drwx------  10 git  git  4096 Apr  4 00:26 git
# 默认还给我们分配一个名字叫git的组。
```

## 创建空仓库

创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。在仓库想放的文件夹下执行以下命令：

```
su git
git init --bare learngit.git  
# 若使用root执行上述命令，则使用以下命令把owner改为git
chown git:git learngit.git  
```

## 权限管理

### 手动管理

#### 密码验证

以上步骤完成后，就可以在客户端使用密码拉取服务器代码了，在客户端执行：

```
git clone git@IP地址:仓库路径
如： git clone git@196.1.1.2:~/git/test.git
```

当你第一次使用Git的clone或者push命令连接GitHub时，会得到一个警告：

这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入yes回车即可。

![image-20210427150347039](二、Git安装与服务器搭建/image-20210427150347039.png)

Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：

`Warning: Permanently added 'github.com' (RSA) to the list of known hosts.`

这个警告只会出现一次，后面的操作就不会有任何警告了。
如果你实在担心有人冒充GitHub服务器，输入yes前可以对照GitHub的RSA Key的指纹信息是否与SSH连接给出的一致

#### SSH公钥验证

##### 服务器打开RSA认证
首先需要将`/etc/ssh/sshd_config`中将RSA认证打开，即：

```
RSAAuthentication yes     
PubkeyAuthentication yes     
AuthorizedKeysFile  .ssh/authorized_keys
```

重启sshd服务

```
systemctl restart sshd.service
```

这里我们可以看到公钥存放在`.ssh/authorized_keys`文件中。所以我们在`/home/git`下创建.ssh目录，然后创建`authorized_keys`文件，用于存公钥

```shell
# 切换到git账号，注意一定要切到这个用户，不然创建的文件要手动授权给这个用户
su git

# 进入 git账户的主目录
cd /home/git

# 创建.ssh的配置，如果此文件夹已经存在请忽略此步。
mkdir .ssh

# 进入刚创建的.ssh目录并创建authorized_keys文件,此文件存放客户端远程访问的 ssh的公钥。
cd /home/git/.ssh
touch authorized_keys

# 设置权限，此步骤不能省略，而且权限值也不要改，不然会报错。
chmod 700 /home/git/.ssh/
chmod 600 /home/git/.ssh/authorized_keys
```

##### 客户端创建ssh公钥私钥

**查看是否有私钥**
检查是否已经拥有ssh公钥和私钥，进入用户的主目录：

- Windows系统：C:\Users\用户名
- Linux系统：/home/用户名
- Mac系统：/Users/用户名

查看是否有`.ssh`文件夹，此文件夹下是否有如下几个文件：`id_rsa`、`id_rsa.pub`，`id_rsa.pub`即为需要用的私钥。若有则无需创建私钥，跳下一步

**创建私钥**

打开Shell（Windows下打开Git Bash），创建SSH Key：

```
ssh-keygen -t rsa -C "youremail@example.com"  
```

一路三个回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码

创建成功后再次查看是否有私钥

##### 上传公钥

```shell
# 切换到git账户
su git
cd /home/git/.ssh
# 上传公钥，选择公钥文件上传
rz
# 查看一下.ssh目录是否有authorized_keys和刚上传的*.pub文件
ls -al

# 如果有，那么进行下面的把*.pub文件中的内容添加到authorized_keys中. 
# >> 是在文件后面追加的意思，主要如果用其他编辑器，每个ssh的pub要单独一行，建议用cat命令方便简单。
cat id_rsa.pub >> authorized_keys
```

然后再次clone的时候，或者是之后push的时候，就不需要再输入密码了：

![image-20210427155719466](二、Git安装与服务器搭建/image-20210427155719466.png)

##### 禁用git用户的shell登陆
出于安全考虑，第二步创建的git用户不允许登录shell

1. 创建`git-shell-commands`目录

给 `/home/git` 下面创建`git-shell-commands`目录，并把目录的拥有者设置为git账户。可以直接用git账号登录服务器终端操作。

```
$ su git
$ mkdir /home/git/git-shell-commands
```

> 此文件夹是git-shell用到的目录，需要我们手动创建，不然报错：fatal: Interactive git shell is not enabled. hint: ~/git-shell-commands should exist and have read and execute access.

2. 修改`/etc/passwd`文件，修改

```shell
vim /etc/passwd

# 可以通过 vim的正则搜索快速定位到这行，  命名模式下  :/git:x
# 找到这句, 注意1000可能是别的数字
git:x:1000:1000::/home/git:/bin/bash
# 改为：
git:x:1000:1000::/home/git:/usr/local/git/bin/git-shell
# 注意修改后的路径为 git安装路径/bin/git-shell，若创建软链接可使用软链接
# 这句其实就是把git用户登录使用的shell从普通shell变更为git-shell
# 最好不要直接改，可以先复制一行，然后注释掉一行，修改一行，保留原始的，这就是经验
# vim快捷键： 命令模式下：yy复制行， p 粘贴  0光标到行首 $到行尾 x删除一个字符  i进入插入模式 
# 修改完后退出保存：  esc进入命令模式， 输入：:wq!   保存退出。
```

修改成功测试：

ssh登录，跳转到git-shell，不可执行shell指令

![image-20210427165832161](二、Git安装与服务器搭建/image-20210427165832161.png)

git clone

![image-20210427165954350](二、Git安装与服务器搭建/image-20210427165954350.png)

此时我们就不用担心客户端通过shell登录执行一些乱七八糟的操作了，只允许使用git-shell进行管理git的仓库

如果有其他小伙伴要连接git服务器，仅需要把他的公钥也添加到authorized_keys即可

### 自动管理

以下配置承接自本文：客户端创建SSH公钥私钥部分

#### 添加`gitolite`依赖包

依赖`perl`包

```shell
yum install 'perl(Data::Dumper)' 
```

#### 清空服务器端配置的ssh的公钥

确保：`~/.ssh/authorized_keys`文件是空的，或者不存在。如果已经存在，建议你把他改名即可，比如：authorized_keys.bak

#### 上传管理员客户端的ssh公钥

把你管理员电脑的ssh的`id_rsa.pub`文件拷贝到服务器的：` $HOME/YourName.pub`

YourName可以自定义，最好根据不同伙伴的名字命名。

#### 安装配置gitolite

```shell
# 切换到git账号
su git
# 进入git主目录
cd /home/git
# 下载gitolite的仓库
git clone https://github.com/sitaramc/gitolite
# 创建bin文件夹，必须！！！
mkdir -p $HOME/bin
# 用下载下来的仓库中的insall执行安装操作，指向的目录就是上一命令行创建的目录
./gitolite二进制/install -to $HOME/bin

# 把上传到服务器的 管理员的公钥setup到gitolite中，注意：YourName.pub改成你自己的文件名。
~/bin/gitolite setup -pk ~/YourName.pub
# 此时安装配完成后，查看git主目录
ls /home/git
# 以下为输出结果
# drwxr-xr-x   7 git  git  4096 Apr  3 23:50 bin               # 我们创建的存放gitolite二进制
# drwxrwxr-x   6 git  git  4096 Apr  3 23:40 gitolite
# drwx------   6 git  git  4096 Apr  3 23:52 .gitolite
# -rw-------   1 git  git  7130 Apr  3 23:52 .gitolite.rc
# -rw-------   1 git  git   398 Apr  3 23:39 malun.pub         # 管理员的公钥
# drwxrw----   3 git  git  4096 Apr  3 23:40 .pki
# -rw-------   1 git  git    19 Apr  4 00:26 projects.list     # 仓库列表（gitolite自动创建）
# drwx------   5 git  git  4096 Apr  4 00:26 repositories      # 存放所有仓库文件夹
# drwx------   2 git  git  4096 Apr  4 15:50 .ssh

# repositories目录下已经有了两个git仓库了。
# .
# |-- gitolite-admin.git    # 管理配置权限的仓库
# `-- testing.git           # 测试仓库
```

好了，到此位置，管理员就可以直接把默认的远程管理的仓库gitolite-admin直接clone到本地进行管理git服务了。

#### 管理员在本地管理和配置服务器端仓库

下载服务器端的远程管理仓库

```shell
# 下载远程管理仓库, 请把aicoder.com换成你自己服务器的域名或者ip
$ git clone git@aicoder.com:gitolite-admin
$ cd gitolite-admin
# 目录结构如下：
# .
# ├── conf                # 配置文件夹
# │   └── gitolite.conf   # 配置权限的文件
# └── keydir              # 客户端的公钥文件夹，所有伙伴的公钥要放到此目录下
#     └── malun.pub
```

####  gitolite的权限配置

1. 添加其他开发的小伙伴
   
   把小伙伴的公钥发给管理员。管理员添加到`gitolite-admin`仓库的`keydir`目录下,注意  文件名字格式为`username.pub`,username就是配置权限时的用户名。
   
2. 配置用户对仓库的读写权限
   
   直接修改conf文件夹下的，gitolite.conf文件。简单解释下几个用法：
   
   - `repo`代表仓库的意思，如果新添加一个repo，代表服务端新建一个空仓库，仓库push到服务端后会自动创建。
- `RW` 代表可读可写
   - `@all` 代表所有人。
   - `master`和 `dev`代表分支
   
   参考：
   
   ```
   @admin = malun  
   @om = malun bcd  
     
   repo gitolite-admin  
       RW+     =   malun 
     
   repo testing  
       RW+     =   @all  
     
   repo om  
       RW+     =   @admin  
       RW+ master = @admin  
       RW+ dev  =   @om  
   ```

3. 应用修改到服务器端
   
   做好配置后，由管理员把修改push到服务器端，会自动处理
   
   ```shell
   git add conf
   git add keydir
   git commit -m "added foo, gave access to alice, bob, carol"
   git push
   ```

参考：

https://www.cnblogs.com/fly_dragon/p/8718614.html

https://blog.csdn.net/wave_1102/article/details/47779401?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.control