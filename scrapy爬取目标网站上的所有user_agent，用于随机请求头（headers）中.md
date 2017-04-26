---
title: scrapy爬取目标网站上的所有user_agent，用于随机请求头（headers）中
tags: python,scrapy,user_agent
grammar_cjkRuby: true
---
## 方法1——直接获取列表页的数目，生成所有起始请求url的列表

```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider, Rule
# from scrapy.linkextractors import LinkExtractor
from scrapy.http import Request
from mobile_ua.items import MobileUaItem
# import urllib


class UA(CrawlSpider):
    name = 'mobile_ua'  # 此处name必须与项目名字一致
    allowed_domians = ['fynas.com']
    # start_urls = ['http://movie.douban.com/top250']
    # # headers = {
    #     'User_Agent': "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    # }

    def start_requests(self):
        headers = {
            'Referer': 'https://www.baidu.com/link?url=X06jI-gYq2kDwTxom7kf_Ys-40WNT-JPrNrpNrzWtmbBwKPxCXgzj-QRHocd57HH&wd=&eqid=fdc6d559001957d10000000358fb4e9c',
            # 'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'
        }
		# 观察发现目标网站的列表页共有273页，因此生成一个起始请求的url列表
        for i in range(273):
            urls = "http://www.fynas.com/ua/search?d=&b=&k=&page=" + str(i)
            yield Request(urls, headers=headers)
# 获取所有index页的URL

    def parse(self, response):
        # next_page = response.xpath(
        #     '//*[contains(@class,"paginationControl")]/a[10]//@href').extract()
        # for url in next_page:
        #     yield Request(urllib.parse.urljoin(response.url, url))
        selector_items = response.xpath(
            # '//div[@class="item"]')
            '//table[@class="table table-bordered"]/tr[position()>1]')  # 下面xpath的根路径
        for item in selector_items:
            yield self.parse_item(item, response)

# 抓取每个index页上的表格信息

    def parse_item(self, selector, response):
        MUA = MobileUaItem()
        MUA['equipment'] = selector.xpath(  # 此处的xpath使用.//开头，表示的是相对于上面的xpath('//table[@class="table table-bordered"]/tr[position()>1]')开始匹配
            './/td[1]/a/text()').extract()
        MUA['system'] = selector.xpath(
            './/td[2]/text()').extract()
        MUA['browser'] = selector.xpath(
            './/td[3]/text()').extract()
        MUA['ua'] = selector.xpath(
            './/td[4]/text()').extract()
        print(response.request.headers['User-Agent'])
        return MUA
```
**总请求数=273,在列表页中有所有需要爬取的信息的话，建议直接在列表页中爬取，详情页的爬取会增加爬虫请求次数，造成爬取速度降低，对网站负担大**
## 方法2——获取'下一页'按钮的url来进行翻页操作
```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider  # , Rule
# from scrapy.linkextractors import LinkExtractor
from scrapy.http import Request
from mobie_ua1.items import MobieUa1Item
import urllib


class UA(CrawlSpider):
    name = 'mobie_ua1'  # 此处name必须与项目名字一致
    allowed_domians = ['fynas.com']
    # start_urls = ['http://movie.douban.com/top250']
    # # headers = {
    #     'User_Agent': "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    # }

    def start_requests(self):
        # 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        yield Request("http://www.fynas.com/ua/search?b=&k=", headers={'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})

    def parse(self, response):
        up_page = response.xpath(  # xpath返回的是一个[]字符串数组
            '//*[contains(@class,"paginationControl")]/a[1]/text()').extract()

        if '上页' in up_page[0]:  # < 上页
            next_page = response.xpath(
                '//*[contains(@class,"paginationControl")]/a[11]//@href').extract()
        else:
            next_page = response.xpath(
                '//*[contains(@class,"paginationControl")]/a[10]//@href').extract()
        for url in next_page:
            yield Request(urllib.parse.urljoin(response.url, url))

        selector_items = response.xpath(
            # '//div[@class="item"]')
            '//table[@class="table table-bordered"]/tr[position()>1]')  # 下面xpath的根路径
        for item in selector_items:
            yield self.parse_item(item, response)

# 抓取每个index页上的表格信息

    def parse_item(self, selector, response):
        MUA = MobieUa1Item()
        MUA['equipment'] = selector.xpath(  # 此处的xpath使用.//开头，表示的是相对于上面的xpath('//table[@class="table table-bordered"]/tr[position()>1]')开始匹配
            './/td[1]/a/text()').extract()
        MUA['system'] = selector.xpath(
            './/td[2]/text()').extract()
        MUA['browser'] = selector.xpath(
            './/td[3]/text()').extract()
        MUA['ua'] = selector.xpath(
            './/td[4]/text()').extract()
        print(response.request.headers['User-Agent'])
        return MUA
```
**注意：**xpath返回的是[]字符串元组，对其进行操作或比较时，要指定索引位置如：
```python
if '上页' in up_page[0]
```
如果写成
```python
if '上页' in up_page
# 就会报错
```
另外scrapy抓取的结果最后是返回到一个列表里（list）如：
```python
movie={'title':[''], 'star':[''], 'ranking':['']}
```