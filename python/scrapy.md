# scrapy入门
1. 安装python3 (建议安装anaconda3)
2. 安装scrapy
```
pip install scrapy -i https://pypi.tuna.tsinghua.edu.cn/simple
// 如果安装的是python2，则可能需要指定scrapy版本，如 pip install scrapy==1.8.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
```
3. 创建项目
```
scrapy startproject SpiderDemo
```

4. 在spiders目录下新建文件demospider.py
```
import scrapy
import logging


class DemoSpider(scrapy.Spider):
    name = "spider_demo"

    def start_requests(self):
        urls = [
            'https://www.qidian.com/rank/yuepiao',
            'https://www.qidian.com/rank/yuepiao?page=2',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        rule_book_name = '//*[@id="rank-view-list"]/div/ul/li[1]/div[2]/h4/a/text()'
        rule_book_url = '//*[@id="rank-view-list"]/div/ul/li[1]/div[2]/h4/a/@href'
        book_name = response.xpath(rule_book_name).get()
        book_url = response.xpath(rule_book_url).get()
        logging.info('book_name = ' + book_name)
        logging.info('book_url = ' + book_url)

    def close(self, reason):
        pass
        
```

5. 运行爬虫
```
cd SpiderDemo
scrapy crawl spider_demo
```

5.1 用idea启动爬虫
```
根目录下新建main.py

import os
import sys
from scrapy.cmdline import execute

if __name__ == '__main__':
    sys.path.append(os.path.dirname(os.path.abspath(__file__)))
    execute(['scrapy', 'crawl', 'spider_demo'])
```


### linux安装其他爬虫需要的其他包 用的清华源
```
// 操作mysql的包
pip install pymysql -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install fake-useragent -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install Faker -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install selenium -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install twisted -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install datetime -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install scrapy-fake-useragent -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install scrapy-user-agents -i https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 如果需要phantomjs来实现截图  
mac安装phantomjs 
```
brew update && brew install phantomjs
phantomjs -v
```
linux安装phantomJs
```
// 找官网下载 
https://bitbucket.org/ariya/phantomjs/downloads/ 
// 设置环境变量
sudo ln -sf /usr/local/app/.local/lib/python2.7/site-packages/phantomjs /usr/local/bin/phantomjs
// linux下phantomjs截图会有字体乱码情况，需要安装下面的字体
yum install bitmap-fonts bitmap-fonts-cjk
```

