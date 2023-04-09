# redis安装-mac
```
brew install redis
```
1.启动redis服务
brew services start redis

2.关闭redis服务
brew services stop redis

3.重启redis服务
brew services restart redis

4.打开图形化界面
redis-cli

7.停止redis服务
redis-cli shutdown

8. redis配置文件位置
/usr/local/etc/redis.conf

9.卸载redis
brew uninstall redis rm ~/Library/LaunchAgents/homebrew.mxcl.redis.plist