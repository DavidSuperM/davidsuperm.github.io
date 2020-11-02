
### linux安装scrapy准备工作 用的清华源
```
pip install scrapy -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install pymysql -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install fake-useragent -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install Faker -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install cos-python-sdk-v5 -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install selenium -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install twisted -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install datetime -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install scrapy==1.8.0 -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install scrapy-fake-useragent -i https://pypi.tuna.tsinghua.edu.cn/simple
pip install scrapy-user-agents -i https://pypi.tuna.tsinghua.edu.cn/simple
安装phantomjs https://bitbucket.org/ariya/phantomjs/downloads/ 找官网下载 
放环境变量 sudo ln -sf /usr/local/app/.local/lib/python2.7/site-packages/phantomjs /usr/local/bin/phantomjs
或者(executable_path="/usr/local/app/.local/lib/python2.7/site-packages/phantomjs")
执行phantomjs -v
yum install bitmap-fonts bitmap-fonts-cjk
```

mac安装phantomjs 
```
brew update && brew install phantomjs
phantomjs -v
```