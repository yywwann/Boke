---
title: Python Scrapy爬虫简单教程
date: 2020-01-31 23:25:10
tags:
- 爬虫
categories:
- 爬虫
---

# 项目仓库
[地址](https://github.com/yywwann/WallhavenSpider)

# 目标确定

框架选择Scrapy.
爬取网页:[https://www.ygdy8.net/html/gndy/china/index.html](https://www.ygdy8.net/html/gndy/china/index.html)

<!--more-->
# 页面分析

打开页面,[https://www.ygdy8.net/html/gndy/china/index.html](https://www.ygdy8.net/html/gndy/china/index.html)

进入开发者工具查看元素,我们需要电影名与电影分类.这个页面可以满足要求.

![](https://img-blog.csdnimg.cn/20190719183525108.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM3OTAw,size_16,color_FFFFFF,t_70)

我们还需要图片,这个页面里并没有,我们打开这个链接,发现了图片,页面分析结束.

![](https://img-blog.csdnimg.cn/20190719183423561.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM3OTAw,size_16,color_FFFFFF,t_70)

# scrapy shell

下面将会使用 Scrapy 提供的 shell 调试工具来抓取该页面中的信息。使用如下命令来开启 shell 调试：


```shell
scrapy shell https://www.ygdy8.net/html/gndy/china/index.html
```

运行上面命令，将会看到 Scrapy 并未抓取到页面数据，但有些运用反爬虫的页面会返回了 403 错误，不允许使用 Scrapy“爬取”数据。为了解决这个问题，我们需要让 Scrapy 伪装成浏览器。
我们可以加上浏览器头来伪装浏览器访问.


```shell
scrapy shell -s USER_AGENT='Mozilla/5.0' https://www.ygdy8.net/html/gndy/china/index.html
```

接下来就可以使用 XPath 或 css 选择器来提取我们感兴趣的信息了。
Xpath是一个简单方便的选择器.这里简单的介绍一下它的语法.


| 表达式 | 作用 |
| :-: | :-: |
| nodename | 匹配此节点的所有内容 |
| / | 匹配根节点 |
| // | 匹配任意位置的节点 |
| . | 匹配当前节点 |
| .. | 匹配父节点 |
| @ | 匹配属性 |

典型的，比如可以使用 //div 来匹配页面中任意位置处的 <div…/> 元素，也可以使用 //div/span 来匹配页面中任意位置处的 <div…> 元素内的 <span…/> 子元素。

XPath 还支持“谓词”，就是在节点后增加一个方括号，在方括号内放一个限制表达式对该节点进行限制。

典型的，我们可以使用 //div[@class］来匹配页面中任意位置处、有 class 属性的 <div…/> 元素，也可以使用 //div/span[1] 来匹配页面中任意位置处的 <div…/> 元素内的第一个 <span…/> 子元素；使用 //div/span[last()] 来匹配页面中任意位置处的 <div…/> 元素内的最后一个 <span…/> 子元素；使用 //div/span[last()-1] 来匹配页面中任意位置处的 <div…/> 元素内的倒数第二个 <span…/> 子元素.
string-length(text())!=6
以页面为例

![](https://img-blog.csdnimg.cn/20190719203449746.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM3OTAw,size_16,color_FFFFFF,t_70)

我们发现每个标签内容都在table中 , 所以开头为:`//table[@class=“tbspan”]
`然后依次向下写`//table[@class=“tbspan”]/tr/td/b/a
`到这里我们发现了两个 `<a>`标签我们需要第二个
所以后面写 `//table[@class=“tbspan”]/tr/td/b/a[2] `即可获取标签对象
我们可以通过`text()`方法获取便签內的文本.也可以通过`@href`获取它的链接地址
输入


```shell
response.xpath ('(//table[@class="tbspan"]/tr/td/b/a[2]/text())').extract()
```

![](https://img-blog.csdnimg.cn/20190721132205886.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM3OTAw,size_16,color_FFFFFF,t_70)

输入

```shell
response.xpath ('(//table[@class="tbspan"]/tr/td/b/a[2]/@href)').extract()
```
![](https://img-blog.csdnimg.cn/20190721132233635.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM3OTAw,size_16,color_FFFFFF,t_70)


# 开始建立scrapy项目


```shell
scrapy startproject MovieSpider
```

在上面命令中，scrapy 是Scrapy 框架提供的命令；startproject 是 scrapy 的子命令，专门用于创建项目；MovieSpider 就是要创建的项目名。
运行结果：

![](https://img-blog.csdnimg.cn/20190721133534502.)

此时在当前目录下就可以看到一个 MovieSpider 目录，该目录就代表 MovieSpider 项目。

查看 MovieSpider 项目，可以看到如下文件结构：


```shell
MovieSpider
  ├── MovieSpider
  │   ├── __init__.py
  │   ├── items.py
  │   ├── middlewares.py
  │   ├── pipelines.py
  │   ├── __pycache__
  │   ├── settings.py
  │   └── spiders
  │       ├── __init__.py
  │       └── __pycache__
  └── scrapy.cfg
```

下面大致介绍这些目录和文件的作用：

* scrapy.cfg：项目的总配置文件，通常无须修改。
* MovieSpider：项目的 Python 模块，程序将从此处导入 Python 代码。
* MovieSpider/items.py：用于定义项目用到的 Item 类。Item 类就是一个 DTO（数据传输对象），通常就是定义 N 个属性，该类需要由开发者来定义。
* MovieSpider/pipelines.py：项目的管道文件，它负责处理爬取到的信息。该文件需要由开发者编写。
* MovieSpider/settings.py：项目的配置文件，在该文件中进行项目相关配置。
* MovieSpider/spiders：在该目录下存放项目所需的蜘蛛，蜘蛛负责抓取项目感兴趣的信息。

通过前面的 Scrapy shell 调试，已经演示了使用 XPath 从 HTML 文档中提取信息的方法，只要将这些调试的测试代码放在 Spider 中，即可实现真正的 Scrapy 爬虫。
基于 Scrapy 项目开发爬虫大致需要如下几个步骤：

## 定义 Item 类。该类仅仅用于定义项目需要爬取的 N 个属性。

编辑 `items.py`


```python
import scrapy

class MoviespiderItem(scrapy.Item):
    #电影名
    name = scrapy.Field()
    #url
    url = scrapy.Field()
```

上面程序中，第 2 行代码表明所有的 Item 类都需要继承 scrapy.Item 类，接下来就为所有需要爬取的信息定义对应的属性，每个属性都是一个 scrapy.Field 对象。

该 Item 类只是一个作为数据传输对象（DTO）的类，因此定义该类非常简单。

## 编写 Spider 类

应该将该 Spider 类文件放在 spiders 目录下。这一步是爬虫开发的关键，需要使用 XPath 或 CSS 选择器来提取 HTML 页面中感兴趣的信息。

Scrapy 为创建 Spider 提供了 scrapy genspider 命令，该命令的语法格式如下：


```shell
scrapy genspider [options] <name> <domain>
```

在命令行窗口中进入 ZhipinSpider 目录下，然后执行如下命令即可创建一个 Spider：


```shell
scrapy genspider job_position "ygdy8.net"
```

运行上面命令，即可在 MovieSpider 项目的 MovieSpider/spider 目录下找到一个 job_position.py 文件
编辑job_position.py


```python
import scrapy
from MovieSpider.items import MoviespiderItem

class JobPositionSpider(scrapy.Spider):
    name = 'job_position'
    allowed_domains = ['ygdy8.net']
    #这里写上你要爬取的页面
    start_urls = ['https://www.ygdy8.net/html/gndy/china/index.html']

	#爬取的方法
    def parse(self, response):
        #注意在上面导入MoviespiderItem包
        item =  MoviespiderItem()
        #匹配
        for jobs_primary in response.xpath('//table[@class="tbspan"]'):
            item['name'] = jobs_primary.xpath ('./tr/td/b/a[2]/text()').extract()
            item['url']  = jobs_primary.xpath ('./tr/td/b/a[2]/@href').extract()
            #不能使用return 
            yield item
```

## 修改pipeline类
这个类是对爬取的文件最后的处理,一般为负责将所爬取的数据写入文件或数据库中.
这里我们将它输出到控制台.


```python

class  MoviespiderPipeline(object):
    def process_item(self, item, spider):
        print("name:",item['name'])
        print("url:",item['url'])
```

## 修改settings类

```python
BOT_NAME = 'MovieSpider'
 SPIDER_MODULES = ['MovieSpider.spiders']
 NEWSPIDER_MODULE = 'MovieSpider.spiders'
 ROBOTSTXT_OBEY = True
# 配置默认的请求头
DEFAULT_REQUEST_HEADERS = {
    "User-Agent" : "Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:61.0) Gecko/20100101 Firefox/61.0",
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'
}
# 配置使用Pipeline
ITEM_PIPELINES = {
     'MovieSpider.pipelines.MoviespiderPipeline': 300,
}
```

一个 Scarpy项目的简单架构就完成了我们可以运行一下试试.
在MovieSpider目录下执行


```shell
scrapy crawl job_position
```

可以看到运行结果

![](https://img-blog.csdnimg.cn/20190722150226832.)

但只有 一页的内容 , 我们可以解析下一页 .
将以下代码加到 job_position.py 的 parse 方法中


```python
       new_links = response.xpath('//a[text()="下一页"]/@href').extract()
        if new_links and len(new_links) > 0:
                # 获取下一页的链接
                new_link = new_links[0]
                # 再次发送请求获取下一页数据
                yield scrapy.Request("https://www.ygdy8.net/html/gndy/china/" + new_link, callback=self.parse)
```

job_position.py最终内容


```python
# -*- coding: utf-8 -*-
import scrapy
from MovieSpider.items import MoviespiderItem

class JobPositionSpider(scrapy.Spider):
    name = 'job_position'
    allowed_domains = ['ygdy8.net']
    start_urls = ['https://www.ygdy8.net/html/gndy/china/index.html']

    def parse(self, response):
        item =  MoviespiderItem()
        for jobs_primary in response.xpath('//table[@class="tbspan"]'):
            item['name'] = jobs_primary.xpath ('./tr/td/b/a[2]/text()').extract()
            item['uri']  = jobs_primary.xpath ('./tr/td/b/a[2]/@href').extract()
            yield item

        new_links = response.xpath('//a[text()="下一页"]/@href').extract()
        if new_links and len(new_links) > 0:
                # 获取下一页的链接
                new_link = new_links[0]
                # 再次发送请求获取下一页数据
                yield scrapy.Request("https://www.ygdy8.net/html/gndy/china/" + new_link, callback=self.parse)
                
```

再次执行 , 就会一页一页的爬取 .

# 保存数据

我们可以将爬取到的内入保存到本地.这里使用python的`pandas`模块
我们只需修改`pipeline.py`的内容即可


```python
class MoviespiderPipeline(object):
    def __init__(self):# 定义构造器，初始化要写入的文件
        self.mypd =  pd.DataFrame(columns=['name','uri'])

    def close_spider(self, spider):#重写close_spider回调方法
        self.mypd.to_csv("movie.csv")

    def process_item(self, item, spider):#添加数据到pandas中
        self.mypd = self.mypd.append({'name':item['name'][0],'uri':item['uri'][0]},ignore_index=True)
        print(len(self.mypd))

```
这样就把数据爬取到文件中了, 同理把数据爬取到数据库什么的都是类似的 .

# 爬取图片
我们在上面获得了电影名和电影详情的路径 , 接下来只需要将每个详情页面的图片爬取出来.
分析页面:

![](https://img-blog.csdnimg.cn/2019072215375437.?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODM3OTAw,size_16,color_FFFFFF,t_70)

我们可以很简单的得到图片的路径
我们可以选择保存图片路径用其他下载器下载也可以用自带的图片文件选择器, 我这里使用自带的下载器.
建立名为 `ImageSpider` 的Scrapy项目 , 使用自带的下载器需要修改一些内容, 所以我们先修改`settings.py`
settings. py

```python
BOT_NAME = 'ImageSpider'

SPIDER_MODULES = ['ImageSpider.spiders']
NEWSPIDER_MODULE = 'ImageSpider.spiders'

ROBOTSTXT_OBEY = True
ITEM_PIPELINES = {
    'scrapy.pipelines.images.ImagesPipeline': 1,
    'scrapy.pipelines.files.FilesPipeline': 2,
}
FILES_STORE = 'D:'  # 文件存储路径
IMAGES_STORE = 'D:' # 图片存储路径

# 避免下载最近90天已经下载过的文件内容
FILES_EXPIRES = 90
# 避免下载最近90天已经下载过的图像内容
IMAGES_EXPIRES = 30

# 设置图片缩略图

# 图片过滤器，最小高度和宽度，低于此尺寸不下载
IMAGES_MIN_HEIGHT = 110
IMAGES_MIN_WIDTH = 110
# 浏览器头
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
    'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36",
}


IMAGES_STORE='D:\\jiandan'

ITEM_PIPELINES = {
    'ImageSpider.pipelines.ImagespiderPipeline': 300,
}
```

items. py

```python
import scrapy


class ImagespiderItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    image_urls = scrapy.Field()#图片的链接
    images = scrapy.Field()
```

job_position. py


```python
# -*- coding: utf-8 -*-
import scrapy
import pandas as pd

from ImageSpider.items import ImagespiderItem

class JobPositionSpider(scrapy.Spider):
    name = 'job_position'
    allowed_domains = ['ygdy8.net']
    start_urls = ['https://www.ygdy8.net/html/gndy/dyzz/20190713/58833.html']
   

    def parse(self, response):
        item = ImagespiderItem()
        item['image_urls'] = response.xpath ('//p/img[1]/@src').extract()
        yield item
```

pipelines. py

```python
import scrapy
from scrapy.exceptions import DropItem
from scrapy.pipelines.images import ImagesPipeline   #内置的图片管道

class ImagespiderPipeline(ImagesPipeline):
    def get_media_requests(self, item, info):
        for image_url in item['image_urls']:
            print("图片连接:",image_url)
            yield scrapy.Request(image_url)
    def item_completed(self, results, item, info):
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem("Item contains no images")
        return item
```

* get_media_requests(item, info)
在工作流程中，管道会得到图片的URL并从项目中下载。为了这么做，你需要重写 get_media_requests() 方法，并对各个图片URL返回一个Request

这些请求将被管道处理，当它们完成下载后，结果将以2元素的元组列表形式传送到 item_completed() 方法: 每个元组包含 (success, file_info_or_error):
success 是一个布尔值，当图片成功下载时为 True ，因为某个原因下载失败为False
file_info_or_error 是一个包含下列关键字的字典（如果成功为 True ）或者出问题时为 Twisted Failure 。
url - 文件下载的url。这是从 get_media_requests() 方法返回请求的url。
path - 图片存储的路径（类似 IMAGES_STORE）
checksum - 图片内容的 MD5 hash
item_completed() 接收的元组列表需要保证与 get_media_requests() 方法返回请求的顺序相一致。下面是 results 参数的一个典型值:
> [(True,
{‘checksum’: ‘2b00042f7481c7b056c4b410d28f33cf’,
‘path’: ‘full/0a79c461a4062ac383dc4fade7bc09f1384a3910.jpg’,
‘url’: ‘http://www.example.com/files/product1.jpg’}),
(False,
Failure(…))]
该方法 必须返回每一个图片的URL。

* item_completed(results, items, info)
当一个单独项目中的所有图片请求完成时,例如，item里面一共有10个URL，那么当这10个URL全部下载完成以后，ImagesPipeline.item_completed() 方法将被调用。默认情况下， item_completed() 方法返回item。.

总之 , 一个图片爬虫写好了 我们可以运行一下试一试 , 可以在图片保存路径发现图片 .

但这只是一张图片 我们 需要的是刚刚怕下来的`movie.csv`中全部的图片 ,那只需要修改 job_position.py


```python
import scrapy
import pandas as pd

from ImageSpider.items import ImagespiderItem

class JobPositionSpider(scrapy.Spider):
    name = 'job_position'
    allowed_domains = ['ygdy8.net']
    start_urls = ['https://www.ygdy8.net/html/gndy/dyzz/20190713/58833.html']#这句话可写可不写 , 没有影响
    def start_requests(self):#重写这个方法
        url = pd.read_csv("movie.csv",usecols=["uri"])#读取csv文件,改为你的路径
        url = url.head(5)#只取前5条数据
        urls=[x  for x in url["uri"]]
        for url in urls:
            url='https://www.ygdy8.net/'+url
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        item = ImagespiderItem()
        item['image_urls'] = response.xpath ('//p/img[1]/@src').extract()
        yield item
```

这样 就完成了整个项目 .


