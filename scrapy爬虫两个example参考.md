---
title: scrapy爬虫两个example参考
tags: python, scrapy
grammar_cjkRuby: true
---


## 例1
```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:  # 判断下一页的链接是否为空
            next_page = response.urljoin(next_page)  # 将下一页的链接转为绝对链接
			# response.urljoin(url)与urllib.parse.urljoin(response.url,url)作用一样
            yield scrapy.Request(next_page, callback=self.parse)

```
## 例2
```python
import scrapy


class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author + a::attr(href)').extract():
            yield scrapy.Request(response.urljoin(href),
                                 callback=self.parse_author)

        # follow pagination links
        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:  # 判断下一页链接是否为None
            next_page = response.urljoin(next_page)
            # urllib.parse.urljoin(response.url,
            # url)与response.urljoin(next_page)的作用一样
            yield scrapy.Request(next_page, callback=self.parse)  # self.parse为默认回调函数

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }
```
### 注：
* 在获取'下一页'的URL时，如果下一页有很多，那么就要使用for循环来Request每个URL
* 当循环到最后一页时，'下一页'的URL为None,因此就不需要Request最后一页的下一页，所以加个if语句判断一下
* 从页面获取的URL有可能是相对链接，在Request中要使用绝对URL，因此需要使用urljoin方法处理，推荐使用scrapy自带的response.urljoin()
* parse函数是scrapy中默认的Request之后的返回函数，在start_url与start_requests中都是默认返回parse函数，在翻页操作中，没有指定callback函数，意味着调用默认返回函数parse，正常情况下应该把每一个Request的操作中加入相应的回调函数如：parse_item