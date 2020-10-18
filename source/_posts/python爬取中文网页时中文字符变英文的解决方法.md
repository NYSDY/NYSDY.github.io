---
title: python爬取中文网页时中文字符变英文的解决方法
date: 2018-10-25 15:01:56
tags: 
- 爬虫
- python
- scrapy
english_title: solution_of_python_for_Chinese_characters_to_become_English_when_crawling_Chinese_web_pages
categories:
- 爬虫
---

> 使用python的scrapy爬取网页时，源代码中的中文字符在爬取下来后变成了英文字符。

<!-- more -->

### 问题举例

例如，原网页为：

![](https://i.loli.net/2018/10/25/5bd16544a16a3.jpg)

爬取结果为：

![](https://i.loli.net/2018/10/25/5bd165a84336a.jpg)

### 解决方法

**修改请求头**：在`settings.py`文件中找到下属代码：

```python
# Override the default request headers:
#DEFAULT_REQUEST_HEADERS = {
#   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
#   'Accept-Language': 'en',
#}
```

改为：

```python
# Override the default request headers:
DEFAULT_REQUEST_HEADERS = {
   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
   'Accept-Language': 'zh-CN',
}
```

修改结果展示：

![](https://i.loli.net/2018/10/25/5bd168d7293ce.jpg)

### 参考链接

- https://blog.csdn.net/wuqili_1025/article/details/79690103

