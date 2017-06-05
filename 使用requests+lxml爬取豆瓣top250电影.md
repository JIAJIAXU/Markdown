---
title: 使用requests+lxml爬取豆瓣top250电影
tags: python,requests,lxml,xpath,豆瓣电影
grammar_cjkRuby: true
---

## requests爬取豆瓣电影
```python?linenums
import requests
from lxml import etree
from urllib.parse import urljoin
import time


def douban(url):
    #     start_url="http://movie.douban.com/top250"
    movie = {}  # 保存爬取信息
    headers = {  # 定制请求头参数
        "Accept": "*/*",
        "Accept-Encoding": "gzip,deflate",
        "Accept-Language": "en-US,en;q=0.8,zh-TW;q=0.6,zh;q=0.4",
        "Connection": "keep-alive",
        "Content-Type": "application/x-www-form-urlencoded; charset=UTF-8",
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.111 Safari/537.36",
    }
    s = requests.Session()  # 建立请求会话，保持cookie
    req = s.get(url, headers=headers)  # 发起get请求，指定请求头
    html = etree.HTML(req.text)  # 将请求返回的response转为xpath对象
    items = html.xpath('//div[@class="item"]')  # 从当前请求返回的页面提取存放所需信息的div部分
    for each in items:  # 遍历所有的item
        # 从items节点出发，匹配每个item中的电影名，xpath返回的是list,切片取所需部分
        movie['电影名'] = each.xpath('.//img/@alt')[0]
        movie['评分'] = each.xpath('.//span[@class="rating_num"]/text()')[0]
        movie['评分人数'] = each.xpath('.//div[@class="star"]/span[4]/text()')[0]
        movie['排名'] = each.xpath('.//em/text()')[0]
        print(movie)
        # with open("E://python_programming/crawler/requests_crawler/douban.txt1", 'w') as db:
        #     db.write(str(db))
        # 获取当前页面的'下一页'的url列表list
    next_page = html.xpath('//span[@class="next"]/a/@href')
    if next_page:  # 如果next_page不为空，则进行下一页操作，否则到达下一页时next_page为空，爬取完毕
        next_url = urljoin(req.url, html.xpath(  # 将相对链接转为绝对链接
            '//span[@class="next"]/a/@href')[0])
        print(next_url)
        time.sleep(2)
        return douban(next_url)
    else:
        print('爬虫抵达最后一页：爬取完毕！！！')


start_url = "http://movie.douban.com/top250"
douban(start_url)
```
## requests爬取图片
```python?linenums
import requests
import re

def Spider(url):
    head = 'http://www.xxx.com'
    r = requests.get(url).content
    pic_url = re.findall('class="mb10" src="(.*?)"', r, re.S)
    i=0
    for each  in pic_url:
        if '@' in each:
            each = each[0:each.find('@')]
        print each
        pic = requests.get(each)
        fp = open('pic\\'+str(i)+ '.jpg','wb')
        fp.write(pic.content)
        fp.close()
        i += 1

    nextPage = re.findall("<a href='(.*?)' btnmode='true' hideFocus class='pageNext'>", r)
    if len(nextPage)<=0:
        return
    nextPage = nextPage[0]
    print nextPage
    if nextPage.strip('')!='':
        nextPage = head+nextPage
    else:
        return
    Spider(nextPage)
```
