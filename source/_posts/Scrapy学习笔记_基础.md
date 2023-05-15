---
title: Linux命令记录
date: 2021-11-20 12:31:01
tags: scrapy
categories: scrapy
keywords: scrapy
description: scrapy学习
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# Scrapy学习笔记_基础

## 1.Scrapy介绍

Scrapy 是一种快速的高级 [web crawling](https://en.wikipedia.org/wiki/Web_crawler) 和 [web scraping](https://en.wikipedia.org/wiki/Web_scraping) 框架，用于对网站进行爬网并从其页面提取结构化数据。它可以用于广泛的用途，从数据挖掘到监控和自动化测试。

Scrapy的官方网址：https://scrapy.org

## 2.安装

如果你在用 [Anaconda](https://docs.anaconda.com/anaconda/) 或 [Miniconda](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html) ，您可以从 [conda-forge](https://conda-forge.org/) 频道，它有针对Linux、Windows和macOS的最新软件包。

使用 `conda` 安装 Scrapy,运行：

```
conda install -c conda-forge scrapy
```

或者，如果您已经熟悉了python包的安装，那么可以从pypi安装scrappy及其依赖项，方法是：

```
pip install Scrapy
```

建议在一个虚拟环境中安装Scrapy，这样避免和其他的系统包起冲突

==注意：==由于操作系统的不同，平台可能会存在一些零碎的依赖项的编译问题，详见[平台特定安装说明](https://www.osgeo.cn/scrapy/intro/install.html#intro-install-platform-notes)

## 3.虚拟环境安装Scrapy

1. 查看已有的虚拟环境

```
# conda在命令行输入以下命令
conda info --envs
# 或者
conda env list

# python在命令行输入以下命令
```

2. 创建新的虚拟环境

```
# conda在命令行输入如下命令
conda create --name newName python=3.7

# python创建虚拟环境
python3 -m venv tutorial-env
```

3. 切换虚拟环境

```
# 在命令行中切换到想要的虚拟环境，我这里切换到python38
conda activate python38

# python-（unix|MacOS）激活虚拟环境
source tutorial-env/bin/activate

# python-windows激活虚拟环境
tutorial-env\Scripts\activate.bat
```



## 4.Scrapy简单使用

以爬取[Scrape | Movie](https://ssr1.scrape.center/)这个网站为例。

1. 创建一个Scrapy项目

```
scrapy startproject spider_quotes
```

以下是项目spider_quotes的目录层级

```
./
├── scrapy.cfg
└── spider_quotes/
    ├── __init__.py
    ├── items.py
    ├── middlewares.py
    ├── pipelines.py
    ├── settings.py
    └── spiders/
        └── __init__.py
```

在spider_quotes/spider/下新建一个quotes_spider.py文件，目录层级如下

```
./
├── scrapy.cfg
└── spider_quotes/
    ├── __init__.py
    ├── items.py
    ├── middlewares.py
    ├── pipelines.py
    ├── settings.py
    └── spiders/
        ├── __init__.py
        └── quotes_spider.py
```

2. 编写一个爬虫

```python
from pathlib import Path
import scrapy


class QuotesSpider(scrapy.Spider):

    name = 'quotes'

    def start_requests(self):
        urls = [
            'https://ssr1.scrape.center/page/1',
            'https://ssr1.scrape.center/page/2'
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response, **kwargs):
        page = response.url.split("/")[-1]
        filename = f'quotes-{page}.html'
        Path(filename).write_bytes(response.body)
        pass
```

3. 运行爬虫

```
# quotes是在文件里取的别名

scrapy crawl quotes
```

4. 执行过程

在QuotesSpider类中，start_requests和parse是重写了Spider这个父类里面的方法，在通过第三步执行这个爬虫时，会先执行start_requests→scrapy.Request→parse（解析方法），默认是先执行start_request

5. start_requests的快捷方式

```python
from pathlib import Path

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'https://ssr1.scrape.center/page/1',
            'https://ssr1.scrape.center/page/2'
    ]

    def parse(self, response):
        page = response.url.split("/")[-1]
        filename = f'quotes-{page}.html'
        Path(filename).write_bytes(response.body)
```

将调用parse()方法来处理每个 对这些 URL 的请求，即使我们没有明确告诉 Scrapy 这样做。发生这种情况是因为parse()是 Scrapy 的默认回调方法，用于没有显式请求的请求 分配的回调。

6. 存储抓取到的数据

存储抓取数据的最简单办法是导出，导出类型：[Feed exports — Scrapy 2.8.0 documentation](https://docs.scrapy.org/en/latest/topics/feed-exports.html#)

```
# 命令行输入，导出为json数据
scrapy crawl quotes -O quotes.json

# 以下是追加写入到json的命令行命令
scrapy crawl quotes -o quotes.jsonl
```

这将会生成一个包含所有抓取项目的文件，以json格式序列化。

这只是适用于小型项目，像一些复杂性的操作是实现不了的，这个时候就需要引用spider_quotes/pipelines.py这个文件了

7. 获取到下一页链接

如这次爬取的案例，爬取的页面是一个很简单的页面，现在，可以通过xpath或者css选择器去获取到href

```
<li class="number">
	<a href="/page/3">3</a>
</li>
```

下面的代码是，只取了第三页的值。response.follow是

```
import scrapy


class QuotesSpider(scrapy.Spider):
    name = 'quotes'

    def start_requests(self):
        urls = [
            'https://ssr1.scrape.center/page/1',
            # 'https://ssr1.scrape.center/page/2',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response, **kwargs):
        page = response.url.split("/")[-1]
        # 通过css选择器去获取到里面的信息
        next_page_css = response.css('li.number.active a::attr(href)').get()
        print(next_page_css)
        # 通过xpath语法去获取
        next_page_xpath = response.xpath('//*[@id="index"]/div[2]/div/div/div/ul/li[3]/a/@href').get()
        print(next_page_xpath)
        # 将地址拼接
        if next_page_xpath is not None:
            next_page = response.urljoin(next_page_xpath)
            yield scrapy.Request(next_page, callback=self.parse)
        # 或者通过快捷的方式去请求下一页的地址
        if next_page_xpath is not None:
            yield response.follow(next_page_xpath, callback=self.parse)
        pass
```

