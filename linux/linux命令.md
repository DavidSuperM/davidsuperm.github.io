- 查看所有端口
```
netstat -ntlp
```

- 查看特定端口
```
lsof -i:5005
```

- 开启一个端口用于输入测试数据
nc -l 9000

- 查看文件
```
ls
ll
which flink   // 查找flink位置
```

- 杀掉端口对应的进程 5005是进程号
```
ps -ef | grep banned

502 96445 94824   0  6:19下午 ttys004    0:00.04 python -u banned_book_incremental_spider_start.py

kill -9 96445
killall python   // 杀掉所有python的进程
pkill -f python
pkill -f banned
```
killall和pkill是相似的,不过如果给出的进程名不完整，killall会报错。pkill或者pgrep只要给出进程名的一部分就可以终止进程。
pkill = pgrep + kill
pgrep = ps + grep

- 查看服务器运行状况
```
top
top -b        // 显示所有进程
top -b -n 1      // 显示所有进程
```

- 终端连接远程服务器
```
ssh 用户名@服务器地址
[输入密码]
```

- 查看服务器进程
```
ps -ef    //-e 显示每个现在运行的进程 -f 生成一个完全的列表
ps -ef | grep python    //筛选出有python关键字的     
```

- 查看文件内容
```
cat test.log
cat test.log | grep 'Exception'
grep 'Exception' test.log
cat test.log | grep -10 'Exception' //打印匹配行的前后各10行
cat test.log | grep -c 'Exception' //统计匹配的行数

head -100 test.log    // 查看前100行
head -n 100 test.log

tail -100 test.log    //查看最后100行
tail -200f log.txt   //滚动查看最后200行 -f表示循环读取
tail -200f log.txt | grep --color -10 '123'   // --color表示加颜色,也可以在文件里永久配置

cat tmp.txt | grep 'qidian' | tail -2     // 查看匹配的最后2行
```

- 删除
```
rm -rf test     // 删除test文件夹以及里面的内容  -f：强制删除文件或目录；-r或-R：递归处理，将指定目录下的所有文件与子目录一并处理；
rm -r *        //删除当前目录下的所有东西
```

- 命令行curl请求
```
// get
curl http://9.155.22.33:10099/pirate/book?pageSize=10&currentPage=1
// post requestParam
curl http://127.0.0.1:8080/test -X POST -d "pageSize=10&currentPage=1&bookName='临渊行'&pirateSiteUrl='www.tianxiabachang.cn'" 
// post requestBody
curl http://127.0.0.1:8080/test -X POST -H "Content-type:application/json"  -d '{"id":1,"name":qdw}'
// curl设置代理 -x可以换成--proxy ，http:// 是代理的协议，也可能是 https:// socks4://  socks5:// 等   (有待验证)
curl -x http://43.243.166.221:8080 www.baidu.com
curl --proxy 43.243.166.221:8080 www.baidu.com
// curl代理设置用户名密码    (有待验证)
curl -x http://name:pwd@127.1.1.2:8080 http://127.0.0.1:8080/test
curl -x http://127.1.1.2:8080 -U name:pwd http://127.0.0.1:8080/test
```

- pip更改下载源
```
pip install scrapy -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 连接mysql
```
mysql -h host -P 8008 --default-character-set=utf8 -uuser -pqdw -Dtest_table;
```

## 导出mysql表数据
```
select * from pirate_book_chapter_update into outfile /tmp/project/chapter.xls //需要有权限
mysqldump -u[Mysql用户名] -h[mysqlIP] -P[mysql端口] -p[mysql密码] --default-character-set=utf8 --skip-opt --create-options -q -B --databases [库名] --tables [表名] --skip-tz-utc >>/tmp/project/table.sql
//--skip-tz-utc 是因为不加的话，导出来的timestamp时间会少8个小时
mysql -h[mysqlIp] -u[用户名] -p[密码] -P[端口] --default-character-set=utf8 -e"use database1;select * from user  where status=1" > /data/user/data1.xlsx
```

## 后台执行python脚本
```
nohup python -u run.py > run.out 2>&1 &
[1] 25584
```

## linux设置环境变量（未验证）
1.对当前用户生效
vim ~/.bash.profile
HOSTNAME = '/usr/bin/hostname 2>/dev/null'   // 得到 9-151-18-99
export CLASSPATH=./JAVA_HOME/lib;$JAVA_HOME/jre/lib
source ~/.bash_profile    //让环境 变量立即生效，不然就是下次重启生效

2.对所有用户生效
vim /etc/profile    
export CLASSPATH=./JAVA_HOME/lib;$JAVA_HOME/jre/lib
source /etc/profile

## crontab定时任务
```
crontab -e
//编辑文件，添加下面一行语句，作用：每分钟执行一次脚本
* * * * cd /tmp/project/ && sh ./monitor.sh >./shell_log.txt 2>&1   
```
语句复制进去的可能不能保存成功，需要手打，因为复制进去的可能会有乱码不合适的字符看不见

## top命令解析
![top命令](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/20201231_top%E5%91%BD%E4%BB%A4.png)

top命令扩展（在进入top后使用）
P：以占据CPU百分比排序
M：以占据内存百分比排序
T：以累积占用CPU时间排序
1: 查看每个cpu运行情况（逻辑cpu）

#### 第1行
| 字符 | 含义 |
| ---- | ---- |
| 23:49:09   |   当前系统时间  |
|  up 94 days,7:21  |  系统已运行时间   |
|  11users  |  当前11个用户登录系统   |
|  load average 2.59,2.80,2.72  |  cpu 1分钟，5分钟，10分钟的负载情况   |

> 负载：假如是单核cpu，load=0.5表示cpu还有一半的资源可用处理其他的进程请求，load=1表示CPU所有的资源都在处理请求，没有剩余的资源可以利用了，而load=2则表示CPU已经超负荷运作，另外还有一倍的线程正在等待处理。
> 所以，对于单核机器来说，理想状态下，Load Average要小于1。同理，对于4核处理器来说，cpu满载时 Load Average是4。

#### 第2行
| 字符 | 含义 |
| ---- | ---- |
|  Tasks 438 total  |  系统共有438多个进程   |
| 5 running  |  正在运行   |
|   380 sleeping |   休眠  |
|  34 stopped  |   停止  |
| 19 zombie  |   僵尸进程数  |

>僵尸进程:一个子进程在其父进程没有调用wait()或waitpid()的情况下退出。这个子进程就是僵尸进程。如果其父进程还存在而一直不调用wait，则该僵尸进程将无法回收，等到其父进程退出后该进程将被init回收。

#### 第3行
| 字符 | 含义 |
| ---- | ---- |
|  %Cpu(s) 12.1 us  |  用户空间占用CPU的百分比   |
|  0.7 sy  |  内核空间占用CPU的百分比   |
|  0.0 ni  |  改变过优先级的进程占用CPU的百分比   |
|  87.0 id   |  空闲CPU百分比   |
|  0.0 wa  |  IO等待占用CPU的百分比   |
|  0.0 hi  |  硬中断（Hardware IRQ）占用CPU的百分比   |
|   0.2 si |   软中断（Software Interrupts）占用CPU的百分比   |
|   0.0 st |   这个虚拟机被hypervisor偷去的CPU时间（译注：如果当前处于一个hypervisor下的vm，实际上hypervisor也是要消耗一部分CPU处理时间的）   |

>Cpu(s)表示的是 所有用户进程占用整个cpu(逻辑cpu)的平均值，由于每个核心占用的百分比不同，所以按平均值来算比较有参考意义

> cpu利用率和负载的意义区别
CPU利用率反映的是CPU被使用的情况。而Load Average却从另一个角度来展现对于CPU使用状态的描述，Load Average越高说明对于CPU资源的竞争越激烈，CPU资源比较短缺。

#### 第4行
| 字符 | 含义 |
| ---- | ---- |
|  KiB Mem 32606428 total  |  内存总量   |
|  6649240 free   |  空闲内存   |
|  5945196 used  |  已使用内存   |
|   20011992 buff/cache  | 缓存的内存量    |

#### 第5行
| 字符 | 含义 |
| ---- | ---- |
| KiB Swap 0 total   |  交换区总量   |
|  0 free  |  交换区空闲量  |
|  0 used   |   交换区使用量   |
| 26117616 availMem    | 可用内存量   |

#### 第6行
| 字符 | 含义 |
| ---- | ---- |
|  PID    |   进程ID   | 
|  USER    |   进程创建者   | 
|  PR    |   进程优先级   | 
|  NI    |   nice值。越小优先级越高，最小-20，最大20（用户设置最大19）   | 
|  VIRT    |   进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES   | 
|  RES    |   进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA  | 
|  SHR    |   共享内存大小，单位kb  | 
|  S    |   进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程  | 
|  %CPU    |   进程占用cpu百分比  | 
|  %MEM    |   进程占用内存百分比  | 
|  TIME+    |   进程运行时间 | 
|  COMMAND    |  进程名称  | 

>PR 越低优先级 越高，PRI(new)=PRI(old)+nice
 PR中的rt为实时进程优先级即rt_priority，prio=MAX_RT_PRIO - 1- p->rt_priority
 MAX_RT_PRIO = 99，prio大小决定最终优先级。这样意味着rt_priority值越大，优先级越高而内核提供的修改优先级的函数，是修改rt_priority的值，所以越大，优先级越高。
 例：改变优先级：进入top后按“r”–>输入进程PID–>输入nice值

> %CPU显示的是进程占用一个核（逻辑核）的百分比，而不是整个cpu（16核）的百分比，有时候可能大于100，那是因为该进程启用了多线程占用了多个核心，所以有时候我们看该值得时候会超过100%，但不会超过总核数*100。
 
## 查看cpu信息
- 物理cpu的个数
```
cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l
```

- 每个cpu物理核的个数
```
cat /proc/cpuinfo |grep "cores"|uniq
```

- 逻辑CPU的个数
```
cat /proc/cpuinfo |grep "processor"|wc -l
```

- 查看cpu信息
```
cat /proc/cpuinfo
```
![cpu信息](https://github.com/DavidSuperM/davidsuperm.github.io/blob/master/images/20210101_cpu_info.png)
processor : 逻辑cpu的编号
physical id ： 物理cpu的id
cpu cores ： 物理cpu的每个cpu的核心数

上面top命令解析中指的cpu都是逻辑cpu

逻辑CPU数量=物理cpu数量 x cpu cores x 2(如果支持并开启ht,ht是inter的超线程技术，它可以在逻辑上分一倍数量的cpu出来)

- 重命名文件 
```
mv file1 file2
```


## SecureCRT操作
```
Ctrl + a ：移到命令行首
Ctrl + e ：移到命令行尾
Ctrl + xx：在命令行首和光标之间移动

Ctrl + u ：从光标处删除至命令行首
Ctrl + k ：从光标处删除至命令行尾
Ctrl + w ：从光标处删除至字首
```

1.  上传本地文件到服务器
```
cd /tmp
rz  //然后会弹出弹窗选择本地的文件
//文件传输也可以用sftp：右击此会话的标签栏，选择“连接SFTP会话”
//put [本地文件的地址] [服务器上文件存储的位置]
//get [服务器上文件存储的位置] [本地要存储的位置]
```
2. 下载服务器文件到本地
```
sz  filename   //下载到默认地址
```
默认地址设置：
Options->Session Options->Terminal->X/Y/ZModem->Download

3.双击复制并打开新session:
options -> global options -> Terminal -> Tabs 选择Double-click action的下拉框为Clone tab，这样就可以在已经打开的session标签中鼠标双击，打开一个完全一样的新session标签。 

4.保持连接：
options -> global options -> General -> Default Session,点击Edit default settings按钮，在Terminal中钩上Send protocol NO-OP, every 30 seconds，这样可以保证securecrt不会因为一段时间没有操作，而丢掉连接。

5. 按钮栏保存常用命令
点击查看（view）-> 按钮栏（Button Bar）
此时SecureCRT窗户底下会出现一条横杠。右击它，会出现“新建按钮”的选项。
