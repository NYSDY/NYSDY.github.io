---
title: python动态网页爬取之安装docker和splash
date: 2018-10-25 10:19:22
tags:
- python
- 爬虫
- splash
- docker
english_title: Python_dynamic_web_crawler_installed_docker_and_splash
categories: 
- 爬虫
---

> 利用python进行动态网页爬取时，在安装docker和splash时踩过的坑，记录了一下自己的安装过程。用的系统是mac os。

<!-- more -->

### 安装scrapy-splash

- 利用pip安装scrapy-splash库：
  `$ pip install scrapy-splash`

### 安装Docker

==下面👇这样安不下去了==

- 如果是Mac的话需要使用brew安装，如下：`brew install docker`  

  报错：

  ```shell
  Error: Failure while executing; `git config --local --replace-all homebrew.private true` exited with 1.
  ```

  解决方法：

  ```sh
  xcode-select --install 
  ```

  然后在执行：

  ```shell
  brew install docker
  ```

  再继续：

  `service docker start`

  报错：

  `-bash: service: command not found`上网上查一堆乱七八糟的解决方式，该路径啥的，真的不想改路径，怕把其他的改崩了。最后放弃这种方式，如果有兴趣也可以尝试解决。

==尝试如下安装DOCKER方法==

1. [去官网下载](https://www.docker.com/get-started)![](https://i.loli.net/2018/10/23/5bcf2ef4c4994.jpg)这种方法下载docker客户端需要从服务器下载，自己电脑下载12k/s，简直慢死了。

2. 拉取镜像(pull the image)：`docker pull scrapinghub/splash`

   ![](https://i.loli.net/2018/10/23/5bcf34a4265a2.jpg)

3. 用docker运行scrapinghub/splash：

   `docker run -p 8050:8050 scrapinghub/splash`

   ![](https://i.loli.net/2018/10/23/5bcf3578aa04a.jpg)

   在浏览器中输入`localhost:8050`![](https://i.loli.net/2018/10/23/5bcf363bc08d8.jpg)

   ==安装成功==

### 参考链接

- http://www.morecoder.com/article/1001249.html
- https://www.jianshu.com/p/e54a407c8a0a