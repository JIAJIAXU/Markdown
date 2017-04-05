---
title: pyspider爬取豆瓣TOP250电影
tags: python,爬虫,pyspider
grammar_cjkRuby: true
---

``` python
from pyspider.libs.base_handler import *
# import re


class Handler(BaseHandler):
    crawl_config = {
    }

    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('http://movie.douban.com/top250', callback=self.index_page)
        for i in range(1, 10):
            url2 = 'movie.douban.com/top250?start=' + \
                str(25 * i)
            self.crawl(url2, callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('html > body > #wrapper > #content > .clearfix > .article > .grid_view > li > div a[href^="http"]').items():
            self.crawl(each.attr.href, callback=self.detail_page)

    @config(priority=2)
    def detail_page(self, response):
        return {
            "title": response.doc('html > body > #wrapper > #content > h1 > span').text(),
            "rating": response.doc('html > body > #wrapper > #content > .clearfix > .article > .clearfix > .subjectwrap > #interest_sectl > .clearbox > .rating_self > .rating_num').text()
        }
```
