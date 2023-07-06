+++
title = "Linux备忘录"
tags = ["Linux", "SSH"]
date = "2023-07-06"
update = "2023-07-06"
enableGitalk = true
+++

## 前言
在 [Linux下使用Let's Encrypt配置Nginx SSL证书](/posts/lets-encrypt-linux/) 这篇文章中，粗略的讲解了一下在linux中如何使用acme.sh来实现ssl证书的签发及配置

尽管上述文章只是很简单的进行了介绍我认为以后会看的懂，但是随着时间的流逝和恰逢大变，明显感觉到记忆力的减弱和身心俱疲以及对技术的痴迷与日俱减，所以我选择了以本文来重构一下的我学习的linux以及需要的记忆的地方

{{< notice tip >}}
本文的编写以及本人所使用的linux系统为 

Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)

不能保证所有的命令能被其他版本所用，也可能会有错误
{{< /notice >}}

- - -

## SSH
先引用一下来自维基百科的说法
{{< image openssh.png >}}

至于如何通过SSH来链接，据我所知有两种工具可以实现，分别为：
- [Xshell 7](https://www.xshell.com/zh/)
- [Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH)

如果是以前的话，我会推荐使用Xshell 7，但是现在，我发现**Win32-OpenSSH**更胜一筹

那么来尝试使用一下**Win32-OpenSSH**吧，首先打开cmd，输入
```
ssh root@127.0.0.1
```

其中root为账户名，127.0.0.1为ip地址，当然您也可以使用在**.ssh**中设置的别名

```
Host dream
	HostName 127.0.0.1
	User root
	ServerAliveInterval 60
	IdentityFile C:\Users\Administrator.DESKTOP-1A2DGNK\.ssh\dream
```

- **dream**	别名
- **User**	账户
- **ServerAliveInterval**	心跳间隔
- **IdentityFile**	密钥文件


经过上述配置后，您可以使用以下的命令来进行链接

```
ssh dream
```

{{< image dream.png >}}

{{< notice note >}}
如果要尝试使用以上命令，则必须在环境变量中配置ssh否则无法直接使用！
{{</ notice >}}

{{< notice tip >}}
至于如何使用sftp来进行文件的上传和下载这里就不过多赘述了，请使用**Xftp 7**来完成
{{< /notice >}}


- - -


## SSH安全性

{{< notice warning >}}
如果仅设置了密码登录，存在被爆破的风险，而且root用户无法远程登录

之前开了个虚拟机采用账号密码都是root，结果一晚上过去我就再也连不上我的服务器了...
{{</ notice >}}

这里采用**Win32-OpenSSH**为例，介绍如何使用密钥登录的方式来防范风险

* 首先打开Win32-OpenSSH的文件
{{< image sshkey.png >}}

*然后打开ssh-keygen.exe
{{< image keygen.png >}}

一共需要输入3次，分别为 
- **密钥的文件名**
- **密钥的密码**
- **密码确认密码**

输入完成后会在根目录下生成文件
{{< image sshpub.png >}}

其中后缀名为.pub需要放入在服务器上，打开ssh远程链接
```
#进入.ssh目录
cd .ssh

#创建authorized_keys文件，如果存在则直接编辑即可
touch authorized_keys >> 您的pub文件内的内容
```

pub文件的内容是这种形式 ssh-rsa xxxxxx

***没有后缀名的文件为登录私钥，一定要妥善保存，否则后续可能会无法链接到服务器！！！！***

* 然后打开ssh配置项
```
cd /etc/ssh
 
vi sshd_config
```

修改以下配置

#### 允许密钥登录
PubkeyAuthentication yes

#### 允许root登录（此项可能存在风险）
PermitRootLogin yes

#### 存放pub密钥的文件路径
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2

#### 关闭密码登录（即仅允许密钥方式的登录）
PasswordAuthentication no


以上配置保存完毕后，输入 systemctl restart sshd.service 重启ssh服务，即可完成上述配置！


- - -

## 包管理器
linux中的包管理器真的是非常的好用，仅需要几个字母便可以直接安装/卸载/更新软件

```
#安装
apt install xxx

#重新安装
apt reinstall xxx

#移除
apt remove xxx

#包源更新
apt update

#更新
apt upgrade
```

由于国内特殊的网络原因，导致访问这些软件时下载更新很慢，鉴于此可以参考使用[**清华大学镜像源**](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)来解决此问题

使用vi编辑/etc/apt/sources.list,修改如下

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

使用包管理器安装的部分软件可能版本过老，如果遇到此情况的话请参考下列官方文档教程处理（其他软件大同小异不做过多赘述）
- [**Nginx**](https://nginx.org/en/linux_packages.html#Ubuntu)
- [**Redis**](https://redis.io/docs/getting-started/installation/install-redis-on-linux/)
- [**DotNET**](https://learn.microsoft.com/zh-cn/dotnet/core/install/linux-ubuntu#register-the-microsoft-package-repository)

- - -

## 系统代理
打开/etc/environment文件

在文件中增加如下内容，然后:wq保存后reboot机器即可正常使用
```
http_proxy="http://ip:port/"
https_proxy="http://ip:port/"
```
- - -

## 结语
这些仅是linux系统中非常少的一些常规用法，无奈由于时间和精力不足，无法对其中的内容做更多的处理

感谢micoya，fox等大佬的帮助和教导让我踏上了新世界的大门，行则将至道阻且长，本文会持续更新下去

```Linux
echo "Hello Linux"
```

- - -
### 参考资料
- [OpenSSH - Wiki](https://zh.wikipedia.org/wiki/OpenSSH)