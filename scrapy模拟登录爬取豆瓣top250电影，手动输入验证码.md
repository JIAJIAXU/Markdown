---
title: scrapy模拟登录爬取豆瓣top250电影，手动输入验证码
tags: python, scrapy, 豆瓣, 验证码, 模拟登录
grammar_cjkRuby: true
---


```python
# -*- coding: utf-8 -*-
# import scrapy
from scrapy.spiders import CrawlSpider
from scrapy.http import Request, FormRequest
from douban_login1.items import DoubanLogin1Item
# import urllib


class DoubanMovieTop250Spider(CrawlSpider):
    name = 'douban_login1'  # 此处name必须与项目名字一致
    allowed_domians = ['douban.com']
    start_urls = 'https://movie.douban.com/top250'

    def start_requests(self):
        # , headers={'User_Agent': 'Mozilla/5.0 (Windows NT 10.0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'})
        yield Request("https://accounts.douban.com/login", meta={'cookiejar': 1}, callback=self.post_login)

    def post_login(self, response):
        print('准备登录...')
        captcha_id = response.xpath('//*[@name="captcha-id"]/@value').extract()
        captcha_url = response.xpath(
            '//img[@id="captcha_image"]/@src').extract()[0]
        print('验证码URL:' + captcha_url)
        captcha_solution = input('请输入验证码：')
        form_data = {
            'soruce': 'movie',
            'redir': 'https://movie.douban.com/top250',
            'form_email': '1923229320@qq.com',
            'form_password': '2011322xjb',
            'captcha-solution': captcha_solution,
            'captcha-id': captcha_id,
            'login': '登录',
        }
        yield FormRequest.from_response(response, meta={'cookiejar': response.meta['cookiejar']}, callback=self.after, formdata=form_data, dont_filter=True)

    def after_login(self, response):
        url = self.start_urls
        # yield self.make_requests_from_url(url)
        yield Request(url, callback=self.parse)

    def parse(self, response):
        next_page = response.xpath(
            '//*[contains(@class,"next")]//@href').extract()
        for url in next_page:
            yield Request(response.urljoin(url), callback=self.parse)

        selector_items = response.xpath(
            '//div[@class="item"]')
        for item in selector_items:
            yield self.parse_item(item, response)

    def parse_item(self, selector, response):
        movie = DoubanLogin1Item()
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