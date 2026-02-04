
### mac安装python3
1. brew search python
2. brew install python@3.11
   主要看python安装位置，我是/usr/local/bin/python3
   
3. vim ~/.zshrc
   source ~/.bash_profile
   vim ~/.bash_profile
   export PATH=/usr/local/bin
   alias python="/usr/local/bin/python3"

4. 关闭iterm2，重启，输入python测试


### nacos
地址:
/Users/david/software/nacos/nacos/logs/startup.log
启动命令
sh ~/software/nacos/nacos/bin/startup.sh -m standalone
关闭命令
sh ~/software/nacos/nacos/bin/shutdown.sh

### rabbitmq
安装
brew install erlang
# 超时下不下来，就换下面的bottle源
# 清华镜像
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
验证
erl -version
brew install rabbitmq
# 启动 RabbitMQ 服务
brew services start rabbitmq

本地启动mq
brew services start rabbitmq
关闭
brew services stop rabbitmq

# 启用管理控制台插
rabbitmq-plugins enable rabbitmq_management
# 如果没有这个插件需要
brew --prefix rabbitmq
输出rabbit目录 比如 /opt/homebrew/opt/rabbitmq
echo 'export PATH="/opt/homebrew/opt/rabbitmq/sbin:$PATH"' >> ~/.zshrc
source ~/.zshrc
验证
which rabbitmq-plugins
rabbitmq-plugins list


http://localhost:15672
用户名: guest
密码: guest

### rabbit没有 x-delayed-message 启动报错
代码里声明了  x-delayed-message  延时队列，但是mq不支持，需要安装插件 否则启动项目报错
rabbitmqctl version  
记下版本号
去 https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases 下载对应版本的 .ez 文件
把下载的 .ez 文件放到 RabbitMQ 的插件目录里。
/opt/homebrew/Cellar/rabbitmq/<版本号>/plugins
放好后执行：
rabbitmq-plugins list | grep delayed
应该能看到 rabbitmq_delayed_message_exchange 了。
启用并重启服务：
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
brew services restart rabbitmq
验证
重启后进入管理后台，新增交换机时应该能看到 x-delayed-message 类型。


### mysql
brew services start mysql@8.0
brew services stop mysql@8.0

### redis
brew services start redis
brew services stop redis

### 合伙人表设计
需求：用户a可以邀请用户b注册，b则是a的下线，需要
查询任意一个用户的所有下线、查询用户的所有子孙的数量、子孙的消费金额、b注册将给a发放邀请奖励、a的下线下单，a能获得佣金奖励

合伙人表:
```
create table partner
(
    id                         bigint auto_increment comment '主键'
        primary key,
    user_id                    bigint                                not null comment '用户id',
    parent_id                  bigint                                not null comment '父id(本表的主键id)',
    level                      int                                   not null comment '层级 1/2/3...',
    agency_id                  bigint                                null comment '机构id',
    status                     tinyint(1)                            not null comment '用户状态 0:正常,1:注销,2:成为合伙人'
)
    comment '合伙人分销表';
```
问题：
1. 如果合伙人树有20层，则查询用户子孙数量和子孙消费总金额，需要循环去mysql partner 表查询，费时

解决 
1. 所以在 partner 加上这几个字段，用户每次注册，和下单，给所有祖先的 down_line_count， down_line_deal_total_money 加值
```
invited_number             int                                   null comment '已邀请人数',
acquired_bonus             bigint                                null comment '已获得佣金',
down_line_count            bigint                                null comment '下线人数，一直统计到叶节点',
down_line_deal_total_money bigint                                null comment '下线消费总金额，一直统计到叶节点',
```

问题：
1. 每次注册下单，仍然要查询多次上线，多次循环查表，费时

解决
1. Materialized Path 方案， partner表增加 path 字段， 保存祖先路径， 比如：  a1/b1/c1/d1
d1下单，找到 a1,b1,c1,更新 down_line_count， down_line_deal_total_money
问题：
1. path字段得设很长，因为不知道路径有多长. 实际存的是id，id本身也很长。得Text或者json 能存几千上万个id，但是对于复杂查询则比较麻烦
2. 如果不加 down_line_count 这两个字段，就要 path like 'a1/b1%' 这样查，索引的长度是有限的，并且查出所有下线id，还要去金额表查询金额，也费时，所以down_line_count 这两个字段还是要加

解决：
1. 合伙人表，新增一个字段， root_id 。查询时把root_id所有子孙查出来，代码里整理上线，然后更新人数和金额
问题：
1. 假如子孙有10万个，那一次下单，查10万个数据，处理也很麻烦

解决：改进后的先序树遍历，查询快
问题：但是新增修改的数据太多，新增1条数据，基本要把树结构大部分节点都更新一次，树有1万个
   https://cloud.tencent.com/developer/article/2239759

解决：
闭包表（最终采用）
```
create table partner_relation
(
    id            bigint auto_increment comment '主键'
        primary key,
    ancestor_id   bigint                                not null comment '祖先节点ID(t_distribute_partner.id)',
    descendant_id bigint                                not null comment '子孙节点ID(t_distribute_partner.id)',
    depth         int                                   not null comment '层级深度：0=自己,1=直属下线,2=下线的下线...',
)
    comment '合伙人节点关系表';
```
保存每个节点和所有子孙、所有祖先的关系
查询时，一条sql即可查出所有祖先或者子孙，并且知道深度。查询快
问题：
1. 当合伙人树有20层时，每次注册新增时，需要插20条数据。删除同理。并且数据量预估是partner表的8倍。如果是更新，比如第三层某个节点，成为合伙人，需要抽出来作为新树，那删除的数据量很大。
不过这个业务，成为新树，是重新新增一个节点，原树关系保留。所以没关系。所以闭包表适合查询多，更新少的，以空间换时间。

仍然存在的其他问题
1. 目前查出所有上级后，是 update partner set down_line_count = down_line_count +1 where id in (); 这个操作是行锁，并发没问题，
但是高并发下，可能有等待行锁的情况，目前每次下单，所有上线执行这个sql，都会行锁，如果下单频率很高，树很大，就是会高并发。

解决：
如果 down_line_count， down_line_deal_total_money 不加在表里，还是维护在redis里，这样不会行锁，更新快，但是redis需要做持久化处理，并且服务器突然断电的话，丢失数据，需要有任务能重新计算。


闭包表有更新情况的并发问题：
如果有更新的场景，成为合伙人，即中间节点b1，带领子树，成为一颗新树，操作过程是先查询新树和旧树祖先节点的关系，全部删除。
// 查询b1的所有子孙节点  结果是 deList
select descendant_id from partner_relation where ancestor_id=b1;
// 查询b1的所有祖先节点 结果是 anList
select ancestor_id from partner_relation where descendant_id=b1;
// 删除新树和旧树的关系
delete from partner_relation where descendant_id in (deList) and ancestor_id in (anList);

注册流程：注册d1,首先拿到d1的上线c1
select ancestor_id from partner_relation where descendant_id=c1;
构建d1的关系数据，
insert 

并发问题： d1注册先查出 ancestor_id  构建d1关系
然后c1成为合伙人,将c1和a1的关系删除了
然后注册流程插入了 d1和a1，

这样与实际不符了，d1是在c1的新树了，和a1没有关系了。但是实际有d1-a1的数据。

解决：（目前最优方案）
c1成为合伙人 ，对c1加锁  lockKey = partner_c1  ，检查c1下所有节点是否有加注册锁的，没有执行成为合伙人流程
d1注册，对的d1上级加锁 lockKey = register_c1, 然后检查c1即所有祖先节点是否有加合伙人锁的，没有执行成为合伙人流程。
问题：
高并发下 一棵树一直有注册的子节点，导致无法成为合伙人，一直不成功。
解决：
注册前先检查再加锁，成为合伙人则是加锁后，检查失败，等待3s再检查，这样因为加了合伙人锁，其他新注册会被挡住，旧注册流程也在3s内走完了，大概率就能执行合伙人流程了。


并发问题解决2：
注册和成为合伙人，都加rootId的锁。
问题：
同一棵树同时只能有一个人注册，等待长，即使这样并且仍然有问题，d1注册，查询根节点 a1,准备加锁，这时，b1成为合伙人，并执行成功，然后d1加了a1的锁，然后，c1成为合伙人，加b1的锁，但是这时c1是b1下的，但是加锁不一致，还是有并发问题


并发问题解决3：
注册时，从根节点往下，依次对每个节点加锁。
成为合伙人，对当前节点加锁。
不冲突，能解决问题，但是加锁太多，每次注册可能都要加20个锁。



## 激活idea  datagrip

## clash设置不走代理
打开clash的配置文件增,这样就不走代理
```
rules:
   - 'DOMAIN,my.xgcloud.pro,DIRECT'
   - 'IP-CIDR,127.0.0.0/32,DIRECT'
```
如果是hosts配置里域名
比如 
```
8.133.253.65    nacos.my.com
```
则在配置文件中增加
```
hosts:
  nacos.my.com: 8.133.253.65
```
但是上面的方法会被clash的自动订阅覆盖
所以直接在clashxpro的配置-更多设置-忽略域名，加上 *.my.com即可



