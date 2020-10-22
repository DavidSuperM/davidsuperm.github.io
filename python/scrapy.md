pip install scrapy -i http://pypi.douban.com/simple

pip install fake-useragent -i http://pypi.douban.com/simple
pip install Faker 
pip install cos-python-sdk-v5 -i http://pypi.douban.com/simple
pip install selenium -i http://pypi.douban.com/simple
pip install twisted -i http://pypi.douban.com/simple
pip install requests -i http://pypi.douban.com/simple
pip install datetime -i http://pypi.douban.com/simple
pip install scrapy==1.8.0    //linux上python2.7则必须装这个版本
pip install scrapy-fake-useragent
pip install scrapy-user-agents
安装phantomjs https://bitbucket.org/ariya/phantomjs/downloads/ 找官网下载 
放环境变量 sudo ln -sf /usr/local/app/.local/lib/python2.7/site-packages/phantomjs /usr/local/bin/phantomjs
或者(executable_path="/usr/local/app/.local/lib/python2.7/site-packages/phantomjs")
执行phantomjs -v
yum install bitmap-fonts bitmap-fonts-cjk


mac安装phantomjs 
brew update && brew install phantomjs
phantomjs -v