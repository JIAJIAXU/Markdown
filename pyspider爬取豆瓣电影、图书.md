---
title: pyspider爬取豆瓣电影、图书 
tags: python, pyspider, 豆瓣
grammar_cjkRuby: true
---


## 实战1——pyspider爬取豆瓣电影top250（使用翻页操作）
```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2017-03-23 20:42:41
# Project: douban_TOP250_movie

from pyspider.libs.base_handler import *
import re


class Handler(BaseHandler):
    headers = {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        "Accept-Encoding": "gzip, deflate, sdch",
        "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4",
        "Cache-Control": "max-age=0",
        "Connection": "keep-alive",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    }
    crawl_config = {
        "headers": headers,
        "itag": 'v3',
        #        "timeout" : 100,
        #         "proxy": "218.56.132.157:8080"
    }

    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('http://movie.douban.com/top250', callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        # 此处获得‘下一页’按钮的URL
        for each in response.doc('.next > a').items():
            self.crawl(each.attr.href, callback=self.index_page, priority=3)  # 局域内priority为3，之后返回下一页
        # 此处应该还是要用正则匹配一下，只爬取电影链接
        for each in response.doc('a[href^="http"]').items():
            if re.match("https://movie.douban.com/subject/\d+", each.attr.href, re.U):
                self.crawl(each.attr.href,
                           callback=self.detail_page, priority=9)  # 局域内priority为9，优先返回详情页

    @config(priority=2)  # 全局priority为2，优先解析item
    def detail_page(self, response):
        return {
            "title": response.doc('h1 > span').text(),
            "rating": response.doc('.rating_num').text()
        }
```
## 实战2——pyspider爬取豆瓣电影top250（生成起始请求url列表）
```python
from pyspider.libs.base_handler import *
# import re


class Handler(BaseHandler):
    crawl_config = {
        "headers": {
            "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
            "Accept-Encoding": "gzip, deflate, sdch",
            "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4",
            "Cache-Control": "max-age=0",
            "Connection": "keep-alive",
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
        }
    }

    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('http://movie.douban.com/top250', callback=self.index_page)
        for i in range(1, 10):  # 直接获取所有页的链接
            url2 = 'movie.douban.com/top250?start=' + str(25 * i)
            self.crawl(url2, callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):  # 获取详情页的链接，进行爬取
        for each in response.doc('html > body > #wrapper > #content > .clearfix > .article > .grid_view > li > div a[href^="http"]').items():
            self.crawl(each.attr.href, callback=self.detail_page)

    @config(priority=2)
    def detail_page(self, response):
        return {
            "title": response.doc('html > body > #wrapper > #content > h1 > span').text(),
            "rating": response.doc('html > body > #wrapper > #content > .clearfix > .article > .clearfix > .subjectwrap > #interest_sectl > .clearbox > .rating_self > .rating_num').text()
        }
```
## 实战3——pyspider爬取豆瓣电影（翻页操作+多级网页爬取）
```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2017-04-26 10:42:01
# Project: douban_movie

# import re
from pyspider.libs.base_handler import *


class Handler(BaseHandler):
    headers = {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        "Accept-Encoding": "gzip, deflate, sdch",
        "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4",
        "Cache-Control": "max-age=0",
        "Connection": "keep-alive",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    }
    crawl_config = {
        "headers": headers,
        "itag": 'v3',
        #        "timeout" : 100,
        #         "proxy": "218.56.132.157:8080"
    }

    @every(minutes=24 * 60)
    def on_start(self):  # 返回tag索引页
        self.crawl('http://movie.douban.com/tag/', callback=self.index_page)

    @config(age=24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('a[href^="http"]').items():
            if 'tag' in each.attr.href:  # 获取每个tag对应的列表页
                self.crawl(each.attr.href, callback=self.list_page)

    @config(age=10 * 24 * 60 * 60, priority=2)
    def list_page(self, response):  # 获取每个列表页中每个目标链接的详情页
        for each in response.doc('HTML>BODY>DIV#wrapper>DIV#content>DIV.grid-16-8.clearfix>DIV.article>DIV>TABLE TR.item>TD>DIV.pl2>A').items():
            self.crawl(each.attr.href, priority=9, callback=self.detail_page)
        # 翻页
        for each in response.doc('HTML>BODY>DIV#wrapper>DIV#content>DIV.grid-16-8.clearfix>DIV.article>DIV.paginator>A').items():
            self.crawl(each.attr.href, callback=self.list_page)

    @config(priority=3)
    def detail_page(self, response):
        return {
            "url": response.url,
            "title": response.doc('HTML>BODY>DIV#wrapper>DIV#content>H1>SPAN').text(),
            "rating": response.doc('#interest_sectl > div.rating_wrap.clearbox > div.rating_self.clearfix > strong').text(),
            "导演": [x.text() for x in response.doc('a[rel="v:directedBy"]').items()],
        }
```
## 实战4——pyspider爬取所有豆瓣书籍（翻页操作+多级网页爬取+上级页面元素传递到下级页面）
```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2017-04-25 19:48:12
# Project: douban_books

from pyspider.libs.base_handler import *
# import re


class Handler(BaseHandler):
    headers = {
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        "Accept-Encoding": "gzip, deflate, sdch",
        "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4",
        "Cache-Control": "max-age=0",
        "Connection": "keep-alive",
        "host": 'book.douban.com',
        "Referer": "https://www.baidu.com/link?url=vr9Qbd26mPEXzNxKIoGn7ibVvn41ipOOk09fW6M5Q6_MaxI6lK0VRCU9OkS5Sb0d&wd=&eqid=be06030d000271120000000358ff3f2f",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36"
    }
    crawl_config = {
        "headers": headers,
        "itag": 'v1'  # 表示当前爬虫的版本，当itag发生变化时，爬虫强制重新爬取
    }

    @every(minutes=10 * 24 * 60)
    def on_start(self):
        self.crawl('https://book.douban.com/tag/',
                   callback=self.tag_page)  # 打开tag页URL，返回tag页

    @config(age=10 * 24 * 60 * 60)
    def tag_page(self, response):
        for each in response.doc('#content > div > div.article > div > div > table > tbody > tr > td > a').items():
            # 抓取tag页中每个tag对应的index页的URL，并返回index页
            self.crawl(each.attr.href, callback=self.index_page)

    def index_page(self, response):
        tag1 = response.doc('#content > h1').text()  # 获取每个标签tag页的tag
        tag2 = tag1[8:]  # tag1的值是'豆瓣图书标签: xxxx'因此只需要8位以后的标签
        for each in response.doc('#subject_list > ul > li > div.info > h2 > a').items():
            self.crawl(each.attr.href, callback=self.detail_page,
                       priority=9, save={'a': tag2})
        # 抓取每个tag页中每本书籍的详情页URL，并返回每本书的detail_page
        # 将tag值传递给下级页面
        for each in response.doc('.next > a').items():
            self.crawl(each.attr.href, callback=self.index_page, priority=5)
        # 抓取每个tag页中的"下一页"的url，并返回下一页的index_page

    @config(priority=2)
    def detail_page(self, response):
        bookname = response.doc('#wrapper > h1 > span').text()
        star = response.doc(
            '#interest_sectl > div > div.rating_self.clearfix > strong').text()
        num = response.doc(
            '#interest_sectl > div > div.rating_self.clearfix > div > div.rating_sum > span > a > span').text()
        catalog = response.doc(
            '#db-tags-section > div').text()  # .replace(' ', '，')
        if '\u3000' in catalog:
            catalog = catalog.replace('\u3000', '，')
        elif ' ' in catalog:
            catalog = catalog.replace(' ', '，')
        return {
            "书籍链接": response.url,
            "书名": bookname,
            "得分": star,
            "评分人数": num,
            "分类": catalog,
            '标签': response.save['a'],
        }
```
