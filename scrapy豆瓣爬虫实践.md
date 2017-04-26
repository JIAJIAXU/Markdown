---
title: scrapy豆瓣爬虫实践
tags: python, scrapy, 豆瓣
grammar_cjkRuby: true
---


# scrapy爬取豆瓣TOP250电影、豆瓣电影标签下的所有电影、豆瓣书籍标签下的所有书籍
## 实战1——豆瓣电影TOP250
```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider
from scrapy.http import Request
from douban_top_movie.items import DoubanTopMovieItem
import urllib


class DoubanMovieTop250Spider(CrawlSpider):
    name = 'douban_top_movie'  
    allowed_domians = ['douban.com']
    # start_urls = ['http://movie.douban.com/top250']
    # # headers = {
    #     'User_Agent': "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    # }

    def start_requests(self):
        headers = {
            # 'USER_AGENT': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36',
            'referer': 'https://www.baidu.com/link?url=L93TvYnXlGkFnh3k2g_tfUm4x5OgGg7kLHmLWNigJ-1dt0iTliqwxNLKpk60A2Bc&wd=&eqid=da2e1ccc0019dc020000000358fb3f59'
        }
        # 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        yield Request("http://movie.douban.com/top250", headers=headers)

    def parse(self, response):
        next_page = response.xpath(
            '//*[contains(@class,"next")]//@href').extract()
        for url in next_page:
            yield Request(urllib.parse.urljoin(response.url, url))
        # parse也是parse中Request的默认回调函数
        # parse是默认的start_requests/start_url的回调（callback）函数
        # 使用urllib的方法将下一页的相对链接转为绝对链接与使用scrapy自带的response.urljoin()的作用一样
        # 此处应该判断一下下一页链接是否为None
        # for next in next_page:
            # if next is not None:
            # next = response.urljoin(next)
            # yield Request(next,callback=self.parse)
			# 如果知道所有的页数，也可以在start_url中建立所有的url列表
        selector_item = response.xpath(
            '//*[contains(@class,"hd")]//@href').extract()
        for url in selector_item:
            yield Request(urllib.parse.urljoin(response.url, url), callback=self.parse_item)
        # for url in selector_item:
        #   url = response.urljoin(url)
        #   yield Request(url,callback = self.parse_item)

    def parse_item(self, response):
        movie = DoubanTopMovieItem()
        movie['title'] = response.xpath(
            '//*[@id="content"]/h1/span[1]/text()').extract()
        movie['star'] = response.xpath(
            '//*[@id="interest_sectl"]/div[1]/div[2]/strong/text()').extract()
        movie['score'] = response.xpath(
            '//*[@id="interest_sectl"]/div[1]/div[2]/div/div[2]/a/span/text()').extract()
        movie['ranking'] = response.xpath(
            '//*[@id="content"]/div[1]/span[1]/text()').extract()
        print(response.request.headers['User-Agent'])
        return movie
```
**这是典型的双向爬虫**：爬取下一页（水平爬虫）、爬取列表页的详情页（垂直爬虫），__总共请求数=25*10=250__
## 实战2——豆瓣电影TOP250
```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider
from scrapy.http import Request
from douban_movie.items import doubanmovie
import urllib


class DoubanMovieTop250Spider(CrawlSpider):
    name = 'douban_movie'  # 此处name必须与项目名字一致
    allowed_domians = ['douban.com']

    def start_requests(self):
        # , headers={'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        yield Request("http://movie.douban.com/top250")

    def parse(self, response):
        next_page = response.xpath(
            '//*[contains(@class,"next")]//@href').extract()
        for url in next_page:
            yield Request(urllib.parse.urljoin(response.url, url))

        selector_items = response.xpath(
            '//div[@class="item"]')
        for item in selector_items:
            yield self.parse_item(item, response)

    def parse_item(self, selector, response):
        movie = doubanmovie()
        movie['title'] = selector.xpath(
            './/*[@class="title"]/text()').extract()[0]
        movie['star'] = selector.xpath(
            './/*[@class="rating_num"]/text()').extract()
        movie['score'] = selector.xpath(
            './/*[@class="star"]/span[4]/text()').extract()
        movie['ranking'] = selector.xpath(
            './/*[@class="pic"]/em/text()').extract()
        return movie
```