---
title: scrapy豆瓣爬虫实践
tags: python, scrapy, 豆瓣
grammar_cjkRuby: true
---


# scrapy爬取豆瓣TOP250电影、豆瓣电影标签下的所有电影
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
		# 获取'下一页'的链接，使用for循环请求，urljoin转为绝对链接
        next_page = response.xpath(
            '//*[contains(@class,"next")]//@href').extract()
        for url in next_page:  # 理应判断'下一页'链接是否为None
            yield Request(urllib.parse.urljoin(response.url, url))
		# 在当前列表页中抓取包含所有待抓取的item（每页25个item）
		# 通过for循环将这些item逐个传递给解析函数parse_item
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
**使用这种方法与上述的抓取结果一致，但有两个优点：**
* 方法2的总请求数=25*1，减少了请求数，因此抓取速度更快，不容易被服务器封掉爬虫
* 在方法1抓取的过程中会有5个详情链接总是抓取失败，而方法2回避了这个问题
## 实战3——豆瓣电影TOP250
```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor
from scrapy.http import Request
from douban_movie250.items import DoubanMovie250Item
import urllib


class DoubanMovieTop250Spider(CrawlSpider):
    name = 'douban_movie250'  # 此处name必须与项目名字一致
    allowed_domians = ['douban.com']
    # start_urls = ['http://movie.douban.com/top250']
    # # headers = {
    #     'User_Agent': "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    # }

    def start_requests(self):
        # 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        yield Request("http://movie.douban.com/top250", headers={'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})

    rules = (
        Rule(LinkExtractor(restrict_xpaths='//*[contains(@class,"next")]')),
        Rule(LinkExtractor(restrict_xpaths='//*[contains(@class,"hd")]'),
             callback='parse_item'))
	# 使用CrawlSpider中的rules规则来限定需要爬取的url，并且在指定的相应的回调函数
    # def parse(self, response):
    #     next_page = response.xpath(
    #         '//*[contains(@class,"next")]//@href').extract()
    #     for url in next_page:
    #         yield Request(urllib.parse.urljoin(response.url, url))

    #     selector_item = response.xpath(
    #         '//*[contains(@class,"hd")]//@href').extract()
    #     for url in selector_item:
    #         yield Request(urllib.parse.urljoin(response.url, url), callback=self.parse_item)

    def parse_item(self, response):
        movie = DoubanMovie250Item()
        movie['title'] = response.xpath(
            '//*[@id="content"]/h1/span[1]/text()').extract()
        movie['star'] = response.xpath(
            '//*[@id="interest_sectl"]/div[1]/div[2]/strong/text()').extract()
        movie['score'] = response.xpath(
            '//*[@id="interest_sectl"]/div[1]/div[2]/div/div[2]/a/span/text()').extract()
        movie['ranking'] = response.xpath(
            '//*[@id="content"]/div[1]/span[1]/text()').extract()
        return movie
```
**第3种方法与第一种方法爬取的结果与效果一致**，但是使用爬虫规则使得爬虫代码更简洁
## 实战4——豆瓣电影标签下所有电影
```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider
from scrapy.http import Request
from douban.items import DoubanItem
import urllib


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
        title1 = selector.xpath(
            './/*[@class="pl2"]/a/text()').extract()
        title2 = selector.xpath(
            './/*[@class="pl2"]/a/span/text()').extract()
        if title2 == []:
            for i in range(len(title1)):
                t1 = title1[i]
            t1 = t1.strip()
            movie['title'] = [t1]
        else:
            movie['title'] = title2
        # movie['title'] = selector.xpath(
        #     './/*[@class="pl2"]/a/span/text()').extract()
        # title2 = movie['title']
        # if title2 == []:
        #     movie['title'] = title1
        movie['star'] = selector.xpath(
            './/*[@class="rating_nums"]/text()').extract()
        movie['score'] = selector.xpath(
            './/span[@class="pl"]/text()').extract()
        return movie
```
**综合方法1和方法2对豆瓣电影标签下的所有电影进行爬取**