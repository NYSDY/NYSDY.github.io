---
title: 安装Ubuntu18.04
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-14 17:24:34
password:
english_title: install Ubuntu18.04
tags:
- ubantu
- ssh
- 显卡
categories:
- 技术
---



# t630安装Ubuntu18.04

1. 刻录u盘必须用过主引导分区格式。可以用[制作启动盘工具](https://www.balena.io)
2. 系统尽量选择sever版本，不需要图形界面。
3. 采用UEFI方式安装，dell一般只支持这一种方式安装。重启服务器按F2，在boot中设置为UEFI格式。

![](http://image.nysdy.com/20200913223801.png)

![](http://image.nysdy.com/20200914115538.png)

![](http://image.nysdy.com/20200914115604.png)

![](http://image.nysdy.com/20200914115626.png)

4. 完成后回退到到初始界面，按F11，设置U盘启动，安装系统。![](http://image.nysdy.com/20200914115720.png)

   ![](http://image.nysdy.com/20200914115740.png)

   ![](http://image.nysdy.com/20200914115803.png)

   这时候等待安装即可。

   ![](http://image.nysdy.com/20200914115845.png)

   到这个界面，拔下U盘，点击回车。![](http://image.nysdy.com/20200914115958.png)

其中：安装全部默认配置就行，IP地址不需要静态的，直接用动态的就好。

4. 安装过程可以参考[ [Ubuntu 18.04 Server 版安装过程图文详解](https://blog.csdn.net/zhengchaooo/article/details/80145744) ]（不看也行，全部默认操作即可。不用配置ip，网络选择默认DHCP。安装磁盘选择200G的系统盘即可）



# ssh

安装过程中有让选择是否安装ssh，这步直接跳过就行，反正也没有外网，无法通过GitHub等地方下载ssh。

## 安装步骤

### 0 判断咱们的机器是否安装ssh服务，可以使用如下命令：

```
ssh localhost
```

如果显示：

```
ssh: connect to host localhost port 22: Connection refused
```

这个就表示没有还没有安装SSH

### 1 安装openssh-client

```
sudo apt-get install openssh-client
```

### 2 安装openssh-server

```
sudo apt-get install openssh-service
```

### 3 启动ssh服务

```
sudo service ssh start
```

然后可以查看ssh服务是否开启成功。

```
sudo ps -e | grep ssh
```

然后要远程登陆，可以查看用ipconfig查看地址，如果出现command not found，则说明没有安装这个包。

```
sudo apt install net-tools
```

apt-get install与apt install的区别前者是老版的用法，后者是新版的用法。apt与apt-get有一些类似的命令选项，但是apt并不能向下兼容apt-get。

## 花絮踩坑记

### 1 第一个坑：ubuntu 18.04: Unable to locate package openssh-service

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get update
```

如果上述方法不行，则更换镜像源，见下一节。

参考链接：

- https://www.cnblogs.com/duanxz/p/5532932.html

- https://www.centos.bz/2019/01/%E8%A7%A3%E5%86%B3ssh%E6%97%A0%E6%B3%95%E8%BF%9E%E6%8E%A5%E8%BF%9C%E7%A8%8Bubuntu%EF%BC%8Cuuntu%E5%AE%89%E8%A3%85ssh-server/
- https://blog.csdn.net/coolljp21/article/details/104090258



## 替换清华源

因为只有内网，可以用当前服务器去连接其他服务器进行sources.list文件替换，省得还要自己敲。

## 防火墙知识

```
- 安装：sudo apt-get install ufw
- 查看状态：sudo ufw status
- 开启/关闭：sudo ufw enable | disable
- 默认允许/禁止：sudo ufw default allow | deny
- 允许/禁止：sudo ufw allow|deny 服务 | port，如：sudo ufw deny 22
- 移除规则：sudo ufw delete deny 22
- 允许范围端口：sudo ufw allow 5901:5999/tcp
```

## ubuntu — apt-get install的时候提示unable to locate package

> 由于ubuntu镜像一般默认自带的都是us的官方源http://us.archive.ubuntu.com，和http://security.ubuntu.com。这些镜像源的地址在中国大多数难以连接，因此需要换国内的源，国内的源有网易源、阿里源、科大源等等，替换阿里源。

```
sudo su     #输入密码
cd /etc/apt  #切换到apt源文件
mv  sources.list sources.list_bak  #备份源文件
vim sources.list  #新建一个，然后将下面的内容copy进去
```

- 替换全部为一下内容

  ```
  # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
  # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
  # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
  # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
  deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
  # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
  
  # 预发布软件源，不建议启用
  # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
  # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
  ```

- 保存，然后执行

```
sudo apt-get update
```

**参考链接：**

1. https://www.codenong.com/cs106225795/

2. https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/

    

## 解决Host key verification failed

发生问题：

```shell
$ ssh root@108.61.163.242
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:HDjXJvu0VYXWF+SKMZjSGn4FQmg/+w6eV9ljJvIXpx0.
Please contact your system administrator.
Add correct host key in /Users/wangdong/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/wangdong/.ssh/known_hosts:46
ECDSA host key for 108.61.163.242 has changed and you have requested strict checking.
Host key verification failed.
```

一般这个问题，是你重置过你的服务器后。你再次想访问会出现这个问题。

解决问题也很简单：

```shell
ssh-keygen -R 你要访问的IP地址
```

**参考链接：**

- https://blog.csdn.net/wd2014610/article/details/85639741

# [安装显卡驱动](https://zhuanlan.zhihu.com/p/59618999)

## **1. 使用 Ubuntu 软件仓库中的稳定版本安装**

### **1.1. 查看显卡硬件型号**

在终端输入：`ubuntu-drivers devices`，可以看到如下界面：

![20191205157552667542415.png](http://image.nysdy.com/20191205157552667542415.png)

从上图可以看出，我的显卡是：`[GeForce GTX 1080 Ti]`，所以推荐安装的版本号是`nvidia-driver-435 - distro non-free recommended`。

### **1.2. 开始安装**

- 如果同意安装推荐版本，那我们只需要终端输入：`sudo ubuntu-drivers autoinstall` 就可以自动安装了。
- 当然我们也可以使用 apt 命令安装自己想要安装的版本，比如我想安装 `340` 这个版本号的版本，终端输入：`sudo apt install nvidia-340` 就自动安装了。
- 安装过程中按照提示操作，除非你知道每个提示的真实含义，否则所有的提示都选择默认就可以了，安装完成后重启系统，NVIDIA 显卡就可以正常工作了。安装完成后你可以参照 `https://linuxconfig.org/benchmark-your-graphics-card-on-linux` 上的介绍测试你的显卡。

- 最后`reboot`重启就可以了

## 其他方法

见[链接](https://zhuanlan.zhihu.com/p/59618999)

## 命令使用

### 查看NVIDIA显卡的驱动版本

```shell
cat /proc/driver/nvidia/version
```

### 查看显卡名称以及驱动版本

```shell
nvidia-smi
```

# 安装anaconda3

### Linux

1. 先去anaconda官网下好linux版本的anaconda，然后上传至服务器。
2. 为了使所有用户都使用 Anaconda 自带的 Python，不能把 Anaconda 安装到默认的当前用户的 Home 目录。推荐安装到 `/opt` 目录。

```
bash anaconda-Linux-x86_64.sh -p /opt/anaconda3
```

接下来需要修改全局的环境变量，添加`export PATH=/opt/anaconda3/bin:$PATH`到`/etc/profile`

## 

