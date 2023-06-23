# linux重装系统
前提系统是centos8


## 重装步骤
### 创建目录
### 安装jdk (除了yum也可以去官网下载jdk，再配环境变量。yum安装的是开源精简版，官网下的是oracle jdk 全一点)
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
### 安装mysql 5.7
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

### 部署spring项目
```
1. idea package打包（默认jar，也可以打war包再在服务器启一个tomcat）
2. java -jar springboot-demo.jar   (或者 nohup java -jar springboot-deom.jar &)

// 关闭
ps -ef | grep java
kill -9 pid
```

### 安装nodejs、npm
```
sudo yum install nodejs
node -v
```

### 安装mongodb
```
// 创建存储库文件。
$ sudo vi /etc/yum.repos.d/mongodb-org-4.repo

[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```
保存并关闭文件
安装mongodb及其依赖包
```
sudo yum install -y mongodb-org
```
安装 MongoDB 软件包后，将创建以下文件和目录。
/etc/mongod.conf：MongoDB 的配置文件。默认 localhost IP (127.0.0.1) 是绑定 IP，27017 是默认端口
/var/lib/mongo：MongoDB 的数据目录
/var/log/mongodb/mongod.log：MongoDB 的日志文件

selinux是linux的权限控制，如果是启用会导致mongodb无法正常启动
```
$ getenforce
Enforcing
$ sudo setenforce 0
$ sudo sed -i s/^SELINUX=.*$/SELINUX=permissive/ /etc/selinux/config
```

运行以下命令以在重新启动时启动并启用 mongodb 服务。
```
$ sudo systemctl start mongod
$ systemctl stop mongodb.service
$ sudo systemctl enable mongod
$ sudo systemctl status mongod
```

连接mongodb
```
$ mongo
```

参考<https://zhuanlan.zhihu.com/p/521761109>

### 安装yapi
前提安装
   nodejs（7.6+)
   mongodb（2.6+）
   
```
npm install -g yapi-cli --registry https://registry.npm.taobao.org
yapi server
```
然后按照提示打开网址，开始界面配置yapi
配置好后，按照提示，启动服务器
cd yapi安装目录
node vendors/server/app.js
或者 nohup node vendors/server/app.js &
打开页面使用
http://127.0.0.1:3000
默认登录名是上面填的邮箱
密码是 ymfe.org

关闭服务
lsof -i tcp:3000
kill -9 进程id
参考
<https://hellosean1025.github.io/yapi/devops/index.html#%E5%AE%89%E8%A3%85>
<https://zhuanlan.zhihu.com/p/94297858>
<https://cloud.tencent.com/developer/article/1552445>

### linux安装docker 
   
1. 安装yum-utils：
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```
2. 为yum源添加docker仓库位置：
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
// 或者  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
// 遇到问题 Invalid configuration value: failovermethod=priority in /etc/yum.repos.d/CentOS-epel.repo; 配置：ID 为 "failovermethod" 的 OptionBinding 不存在
// 解决方案：进入/etc/yum.repos.d/CentOS-epel.repo文件，注释 failovermethod=priority 这一行
```
3. 查看可以安装的docker版本
```
yum list docker-ce --showduplicates
```
4. 安装最新版t
```
yum -y install docker-ce
```
5. 查看docker安装是否成功 
```
docker -v
```
6. 启动docker
```
systemctl start docker
```

### docker安装mysql
```
// 拉镜像
docker pull mysql:5.7
// 启动
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```
参数说明：
-p 3306:3306：将容器的3306端口（后一个3306）映射到主机的3306（前一个3306）端口
-v /mydata/mysql/conf:/etc/mysql：将配置文件夹挂在到主机
-v /mydata/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
-v /mydata/mysql/data:/var/lib/mysql/：将数据文件夹挂载到主机
-e MYSQL_ROOT_PASSWORD=root：初始化root用户的密码



## linux环境变量
/etc/profile
修改后要执行 source /etc/profile 才生效



