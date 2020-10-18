---
title: ubantu系统安装pytorch GPU版本
copyright: true
top: 1
cover: false
toc: true
mathjax: true
date: 2020-08-21 15:50:16
password:
english_title: ubantu系统安装pytorch-GPU版本
tags:
- pytorch
- python
categories:
- 技术
---

# **0 准备工作**

用conda安装Pytorch过程中会连接失败，这是因为Anaconda.org的服务器在国外，需要切换到国内镜像源：

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/ 
```

设置搜索时显示通道地址

```bash
conda config --set show_channel_urls yes
```

创建虚拟环境并激活之，在该环境下安装下面的包：

```bash
conda create -n pytorch_gpu python=3.6
source activate pytorch_gpu
```



# 1 安装显卡驱动

> 先用nvidia-smi进行查询，如果可以查到显卡驱动就不需要安装了。

### **1.1. 查看显卡硬件型号**

在终端输入：`ubuntu-drivers devices`，可以看到如下界面：

![20191205157552667542415.png](http://image.nysdy.com/20191205157552667542415.png)

从上图可以看出，我的显卡是：`[GeForce GTX 1080 Ti]`，所以推荐安装的版本号是`nvidia-driver-435 - distro non-free recommended`。

### **1.2. 开始安装**

- 如果同意安装推荐版本，那我们只需要终端输入：`sudo ubuntu-drivers autoinstall` 就可以自动安装了。
- 当然我们也可以使用 apt 命令安装自己想要安装的版本，比如我想安装 `340` 这个版本号的版本，终端输入：`sudo apt install nvidia-340` 就自动安装了。
- 安装过程中按照提示操作，除非你知道每个提示的真实含义，否则所有的提示都选择默认就可以了，安装完成后重启系统，NVIDIA 显卡就可以正常工作了。安装完成后你可以参照 `https://linuxconfig.org/benchmark-your-graphics-card-on-linux` 上的介绍测试你的显卡。
- 最后`reboot`重启就可以了

## 1.3. 查看NVIDIA驱动版本

输入`nvidia-smi`：显示如下：

![](http://image.nysdy.com/20200821155338.png)



# 2 安装CUDA

## 0 CUDA对应的NVIDIA驱动版本对照表

一般而言，不同版本的CUDA要求不同的NVIDIA驱动版本,同时显卡驱动版本要不低于CUDA的安装版本，具体的对照关系如下：[官网地址](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)

![](http://image.nysdy.com/20200821155720.png)

## **1 安装CUDA**

按照上述对应表，找到要按照的CUDA版本，比如按照上图来说应该安装9.2版本

```bash
conda install cudatoolkit=9.2 -n pytorch_gpu -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/linux-64/
```

## **2 安装cuDNN**

所安装的cuDNN版本注意和CUDA对应，可以在[CUDA官网](https://developer.nvidia.com/rdp/cudnn-archive)找到版本对应关系：

```bash
conda install cudnn=7.6.5 -n pytorch_gpu -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/linux-64/
```

## 3 **找到对应pytorch版本**

先到官网找到在你的操作系统、包、CUDA版本、语言版本下对应的安装脚本，[官网地址](https://pytorch.org/get-started/locally/)，直接根据你的实际情况选择Pytorch安装包版本，然后复制页面自动生成的脚本进行安装。

![](http://image.nysdy.com/20200821164952.png)

运行：

```shell
conda install pytorch torchvision cudatoolkit=9.2 -c pytorch
```

如果显示如下错误:

```shell
CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://conda.anaconda.org/pytorch/linux-64/torchvision-0.7.0-py36_cu92.tar.bz2>
Elapsed: -

An HTTP error occurred when trying to retrieve this URL.
HTTP errors are often intermittent, and a simple retry will get you on your way.

CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://conda.anaconda.org/pytorch/linux-64/pytorch-1.6.0-py3.6_cuda9.2.148_cudnn7.6.3_0.tar.bz2>
Elapsed: -

An HTTP error occurred when trying to retrieve this URL.
HTTP errors are often intermittent, and a simple retry will get you on your way.
```

可以换成下面的方式

```shell
conda install pytorch torchvision cudatoolkit=9.2 -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
```



## # **3 测试**

```python
import torch
flag = torch.cuda.is_available()
print(flag)

ngpu= 1
# Decide which device we want to run on
device = torch.device("cuda:0" if (torch.cuda.is_available() and ngpu > 0) else "cpu")
print(device)
print(torch.cuda.get_device_name(0))
print(torch.rand(3,3).cuda())
```

结果：

```python
True
cuda:0
GeForce GTX 1080
tensor([[0.9530, 0.4746, 0.9819],
        [0.7192, 0.9427, 0.6768],
        [0.8594, 0.9490, 0.6551]], device='cuda:0')
```



# 参考链接

1. [Anaconda环境安装GPU版本Pytorchanaconda pytorch gpu](<https://blog.csdn.net/baidu_26646129/article/details/88380598>)
2. [CUDA对应的NVIDIA驱动版本对照表_zhw864680355的博客-CSDN博客_cuda对应的驱动版本](<https://blog.csdn.net/zhw864680355/article/details/90411288>)
3. [pytorch：测试GPU是否可用_明月几时有，把酒问青天-CSDN博客_torch gpu](<https://blog.csdn.net/weixin_35576881/article/details/89709116?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.channel_param>)

