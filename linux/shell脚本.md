# 定时清理日志文件（日志文件以天为单位，每天自动创建一个）
1. 新建个 clean_log.sh 文件
2.
```
#!/bin/sh
for file in `find ./log/ -type f -name "*"`
    do
        local expired_time=$[30*24*60*60]        #此处定义文件的过期时间6天
        local currentDate=`date +%s`            #获取系统时间，所以时间格式为秒
        local modifyDate=$(stat -c %Y $file)    #获取文件修改时间
        local existTime=$[$currentDate-$modifyDate]     #对比时间，算出日志存在时间
        if [ $existTime -gt $expired_time ];
        then
            rm -rf $file    #删除文件
        fi
    done
```

# 守护进程，如果spider_start_script python脚本挂了，自动启动，配合crontab使用
新建monitor.sh
```
#!/bin/sh
source /etc/profile
ps -fe|grep spider_start_script |grep -v grep
if [ $? -ne 0 ]
then
  echo "start process....."
  killall phantomjs
  echo "killall phantomjs....."
  nohup python -u spider_start_script.py > run.out 2>&1 &
else
  echo "runing....."
fi
```