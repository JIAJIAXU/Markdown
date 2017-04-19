---
title: scrapy爬取的索引页中含有多个要提取的数据
tags: python,scrapy,爬虫,索引页
grammar_cjkRuby: true
---
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
        # 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        for i in range(273):
            urls = "http://www.fynas.com/ua/search?d=&b=&k=&page=" + str(i)
            yield Request(urls, headers={'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
# 获取所有index页的URL,总共273页

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
        return MUA
```
**遇到的问题**
```markdown
* 1.最初使用chrome的开发者工具直接copy得到目标元素的xpath表达式，用于scrapy的xpath提取页面元素时，发现提取不出目标元素
** 解决办法：在确定爬取页面的内容不是js加载时，如果获取不到目标元素，那么很有可能就是xpath出现问题，chrome中能够识别//table/tbody/tr，而在scrapy中则只能识别//table/tr
* 2.注意xpath表达式//与.//的区别，本例中先提取每个index页上的所有数据的xpath结构，然后在parse_item中解析该xpath结构，相应的就要使用.//来选择相对上级的xpath