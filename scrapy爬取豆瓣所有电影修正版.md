---
title: scrapy爬取豆瓣所有电影修正版 
tags: python, scrapy,豆瓣电影
grammar_cjkRuby: true
---


## 修改了豆瓣电影的名字以及利用正则表达式只留下评分人数中的数字
```python 
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider
from scrapy.http import Request
from douban.items import DoubanItem
import urllib
import re


class DoubanMovieTop250Spider(CrawlSpider):
    name = 'douban'  # 此处name必须与项目名字一致
    allowed_domians = ['douban.com']

    def start_requests(self):
        # , headers={'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        headers = {
            # 'USER_AGENT': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36',
            'referer': 'https://www.baidu.com/link?url=7u5fi9zxZ7UhcghOj540fjWpmi_nT2hX2EEpXKItLyv78gYcl3zlgHfVo15ZkzTY&wd=&eqid=d2ac1cbe001b4b8c0000000358fb62bf'
        }
        yield Request("https://movie.douban.com/tag/", headers=headers, callback=self.tag_page)

    def tag_page(self, response):
        alltag_url = response.xpath(
            '//*[contains(@class,"tagCol")]//@href').extract()
        for url in alltag_url:
            yield Request(urllib.parse.urljoin(response.url, url), callback=self.index_page)

    def index_page(self, response):
        selector_nextpage = response.xpath(
            '//*[contains(@class,"next")]/a/@href').extract()
        # //*[contains(@class,"next")]/a/@href与//*[contains(@class,"next")]//@href效果一样
        for url in selector_nextpage:
            yield Request(urllib.parse.urljoin(response.url, url), callback=self.index_page)

        selector_items = response.xpath(
            '//tr[@class="item"]')
        for item in selector_items:
            yield self.parse_item(item, response)

    def parse_item(self, selector, response):
        movie = DoubanItem()
        # title1 = selector.xpath(
        #     './/*[@class="pl2"]/a/text()').extract()
        # title2 = selector.xpath(
        #     './/*[@class="pl2"]/a/span/text()').extract()
        # if title2 == []:
        #     for i in range(len(title1)):
        #         t1 = title1[i]
        #     t1 = t1.strip()
        #     movie['title'] = [t1]
        # if title2 == []:
        #     t1 = title1[0].strip()
        #     t1 = t1.strip()
        #     movie['title'] = [t1]
        # else:
        #     movie['title'] = title2
        t = selector.xpath(
            './/*[@class="pl2"]/a/text()').extract()[0].replace('/', '').strip()
		# 将xpath获取的标题有tuple转为字符串，再将字符串末尾中的'/'替换为空格，最后使用strip函数去掉字符串首尾的空格和换行符
        movie['title'] = [t]  # 将处理好的字符转换成tuple赋值给movie['title']
        # movie['title'] = selector.xpath(
        #     './/*[@class="pl2"]/a/text()').extract()[0]  # .replace('/', '').strip()
        movie['star'] = selector.xpath(
            './/*[@class="rating_nums"]/text()').extract()
        num1 = selector.xpath(
            './/span[@class="pl"]/text()').extract()[0]
        num2 = re.findall(r"\d+", num1)  # 正则xpath提取的评分人数，只获取数值
        movie['score']=num2
	  #  for _, num in enumerate(num2):
       #   num = num.group()
        #  movie['score'] = [str(num)]
        # movie['score']= selector.xpath(
        #     './/span[@class="pl"]/text()').extract()
        return movie
```