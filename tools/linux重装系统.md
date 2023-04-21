# linux重装系统
前提系统是centos8


### 重装步骤
1. 创建目录
2. 安装jdk (除了yum也可以去官网下载jdk，再配环境变量。yum安装的是开源精简版，官网下的是oracle jdk 全一点)
```
// 查看网络可用jdk
yum -y list java*  或者   yum search jdk
// 安装jdk，-devel是开发版， java-1.8.0-openjdk.x86_64 这个是jre运行环境，其他的demo、src是示例、源码、文档等等
yum install java-1.8.0-openjdk-devel.x86_64
// 验证版本
java -version
// 查看安装位置 
which java  
    // 显示/usr/bin/java，但其实这个java是一个超链接。查看超链接地址
    ls -lrt /usr/bin/java
    // 显示  /usr/bin/java -> /etc/alternatives/java   还是超链接，继续查看
    ls -lrt /etc/alternatives/java
    // 显示 /etc/alternatives/java -> /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-2.el8_5.x86_64/jre/bin/java
    // 则jdk位置在  /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-2.el8_5.x86_64
```
3. 安装mysql 5.7
```
// cd mysql目录
cd mysql
// 下载mysql
wget http://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
// 安装mysql的安装工具
rpm -ivh mysql80-community-release-el7-3.noarch.rpm
// 此时默认装的是MySQL8.0，我们需要的是MySQL5.7，需要对配置进行更改
cd /etc/yum.repos.d/
vim mysql-community.repo
// [mysql57 - community]下的enabled=0改为1
// [mysql80 - community]下的enabled=1改为0
// 保存退出
// 安装mysql
yum module disable mysql
yum -y install mysql-community-server
// 启动mysql
systemctl start mysqld.service
// 查看mysql状态
systemctl status mysqld.service
// 停止mysql
systemctl stop mysqld.service
// 重启
systemctl restart mysqld.service
// 配置mysql服务器自启动
systemctl enable mysqld.service
// 配置一个用户允许远程连接
GRANT ALL PRIVILEGES ON *.* TO 'david'@'%' IDENTIFIED BY 'David@123' WITH GRANT OPTION;
// 修改字符编码 utf-8
vim /etc/my.cnf
// 插入或修改
[mysqld]
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
// 重启mysql后可以看到字符集已改变
mysql> show variables like 'character%';
```
> 如果报错
// 如果报错  错误：没有任何匹配: mysql-community-server 则
yum module disable mysql
yum install mysql-community-server
// 报 "公钥没有安装" 错误  则
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
yum install mysql-community-server

```
// 查看自动生成的临时密码
grep 'temporary password' /var/log/mysqld.log
// 修改密码，密码有格式要求
set password for root@localhost = password('密码')

// 设置开机启动
systemctl enable mysqld
systemctl daemon-reload

```

再去服务器防火墙开放3306端口

4. 部署spring项目
```
1. idea package打包（默认jar，也可以打war包再在服务器启一个tomcat）
2. java -jar springboot-demo.jar   (或者 nohup java -jar springboot-deom.jar &)

// 关闭
ps -ef | grep java
kill -9 pid
```

### linux环境变量
/etc/profile
修改后要执行 source /etc/profile 才生效