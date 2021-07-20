---
title: 从零开始开发一个App（1）- Scrapy爬虫
date: 2016-08-01 18:18:42
tags:
categories: 技术杂谈
---

### 前言
最近我体验了一次全栈（伪）开发App的经历，获益良多，我想把过程记录一下，一是回顾与巩固，二是抛砖引玉，如有谬误以求大神指点。

首先，我们需要明确我们最终的目标是什么。
比如现在我要做一个简单的游戏评测资讯的App。
那么我首先需要【数据来源】然后需要一个提供数据接口的【服务端】，我将先完成这二者，然后才开始App的开发。

因此我将分为三步[Python爬虫]->[Ruby服务端]->[iOS客户端]来完成这个App。

而爬虫技术，将作为先遣部队，为我们攻下第一个数据堡垒。

### 开始
许多语言都有成熟的爬虫框架，我选择的是使用Python语言的Scrapy框架，这是一个非常完善而且功能强大的爬虫框架，我们只需要用到最基础的功能。Scrapy拥有非常棒的[中文文档](http://scrapy-chs.readthedocs.io/zh_CN/0.24/index.html) 安装和入门教程文档一应俱全，我就不赘述了。

安装好之后，我们打开终端，开始创建令人激动的爬虫

```
scrapy startproject yxReview

cd yxReview 

scrapy genspider yx_review www.ali213.net/news/pingce
```

完成后结构大致如图
![k](http://upload-images.jianshu.io/upload_images/25038-3289227b3c31677b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就这样，基础的框架和代码已经被生成了，接着我们用编辑器打开yxReview目录，首先在`items.py`文件下新建一个item。
我们可以先打开网页[游侠评测](http://www.ali213.net/news/pingce/)分析一下

![游侠评测](http://upload-images.jianshu.io/upload_images/25038-c322d36580a526ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


简单起见，我们目前只抓取4个属性，创建代码：

```python
class ArticleItem(scrapy.Item):
    title = scrapy.Field()
    cover_image = scrapy.Field()
    summary = scrapy.Field()
    score = scrapy.Field()
```

然后我们进入目录下的spiders文件夹，打开`yx_review.py`
如下编辑


```python
# -*- coding: utf-8 -*-
import scrapy

from yxReview.items import ArticleItem

class YxReviewSpider(scrapy.Spider):
    name = "yx_review"
    allowed_domains = ["www.ali213.net"]
    start_urls = (
        'http://www.ali213.net/news/pingce/',
    )

    def parse(self, response):
        items = []
        for sel in response.xpath('//div[@class="t3_l_one_l"]'):
            item = ArticleItem()
            item["cover_image"] = sel.xpath("div[@class='one_l_pic']/a/img/@src").extract()
            item["title"] =   sel.xpath("div[@class='one_l_con']/div[@class='one_l_con_tit']/a/text()").extract()
            item["summary"] = sel.xpath("div[@class='one_l_con']/div[@class='one_l_con_con']/text()").extract()
            items.append(item)

        index = 0
        for scoreSel in response.xpath('//div[@class = "t3_l_one_r"]'):
            item = items[index]
            item["score"] = scoreSel.xpath("div/span/text()").extract()
            index = index + 1
            yield item

        print items
```
这里主要是parse方法，返回请求到的HTML然后解析出我们需要的数据装进ArticleItem里，然后将items传输到`pipeline`中。

### 传输管道
在爬虫scrapy中，pipeline是一个重要的概念，它相当于一个“加工器”，可以连接多个自定义的pipeline，完成数据的后续处理工作，比如进行筛选分类，或者持久化到本地等等，按优先级串连。

在本例中，为了简便，我将创建一个管道将数据简单处理并保存到本地文件中。

打开`pipelines.py`，编辑如下

```python
# -*- coding: utf-8 -*-

from scrapy import signals
from scrapy.contrib.exporter import JsonItemExporter

class YxreviewPipeline(object):

    @classmethod
    def from_crawler(cls, crawler):
         pipeline = cls()
         crawler.signals.connect(pipeline.spider_opened, signals.spider_opened)
         crawler.signals.connect(pipeline.spider_closed, signals.spider_closed)
         return pipeline

    def spider_opened(self, spider):
        self.file = open('items.json', 'wb')
        self.exporter = JsonItemExporter(self.file)
        self.exporter.start_exporting()

    def spider_closed(self, spider):
        self.exporter.finish_exporting()
        self.file.close()

    def process_item(self, item, spider):
        self.checkData(item, "title")
        self.checkData(item, "summary")
        self.checkData(item, "cover_image")
        self.checkData(item, "score")

        self.exporter.export_item(item)

        return item

    def checkData(self ,item, field):
        if len(item[field]) > 0:
            newText = item[field][0].encode("utf-8")
            item[field] = newText.strip()
        else:
            item[field] = ""

```

前面三个方法，相当于给当前的pipeline提供了一个JsonItemExporter的插件，用于将所有爬取的item导出为JSON格式的文件之中。

另外需要说明的是，这里自定义了一个`checkData`方法，作为一个简单的数据类型验证，以及将之前解析的内容转换成字符串并且进行了utf-8编码的转码，保障中文内容的正确显示。

完成后，打开工程目录下的`items.json`文件，可以看到数据已经以JSON格式保存了下来。

![items.json](http://upload-images.jianshu.io/upload_images/25038-5309230be55509fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 告一段落

至此，爬虫的任务可以告一段落，当然，在实际应用中还需要解决更多的问题，比如分页爬取，反爬虫的应对等等，迫于文章篇幅暂且不表，算作扩展阅读吧 ：）

下一篇，我们将使用ruby on rails编写服务端，提供移动端的REST API接口。

---

**系列链接**

{% post_link 从零开始开发一个App（1）-Scrapy爬虫 %}
{% post_link 从零开始开发一个App（2）-简易REST-API服务端 %}
{% post_link 从零开始开发一个App（3）-iOS客户端 %}
