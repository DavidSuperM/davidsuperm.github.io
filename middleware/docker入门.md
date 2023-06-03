# docker

### 基础命令

> docker安装的应用，记得要去防火墙打开端口，否则外部连不上

```
// 搜索镜像
docker search mysql
// 拉取远程镜像
docker pull mysql:5.7
// 创建镜像，镜像名为 springboot_demo_image，当前目录需要有dickerfile。
docker build -t springboot_demo_image .
// 启动mysql镜像,(mysql:后面是image的tag信息，前面是image的name)
docker run --name mysql5.7 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=*** mysql:5.7-debian   
// 启动mysql镜像,8080:8080,前一个8080是本机端口，后一个是docker服务端口，根目录下执行  -d 代表后台运行 (-d后台运行可能导致docker ps找不到container，要用 docker ps -a)
docker run --name springboot_demo -p 8080:8080 -d springboot_demo_image
// 终止容器
docker stop <container_id_or_name>
// 删除容器
docker rm -f <container_id_or_name>
// 再次启动容器
docker start <container_id_or_name>
// 删除镜像
docker rmi <image_id>
// 查看本地image
docker images
// 查看本地docker进程
docker ps
docker ps -a 
docker container ls -a
// 查看docker ip
docker inspect <container id>
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id
// 所有容器名称、ip
docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)
// 查看日志
docker logs <container_id>
docker logs -f <container_id>
// 进入容器
docker exec -it <container_id_or_name> /bin/bash
// 退出容器
exit
```



### mac下载安装
官网下载docker桌面版
https://www.docker.com/
参考<https://www.runoob.com/docker/docker-install-mysql.html>

### docker启动mysql
docker内搜索mysql镜像，选择5.7版本，pull。
启动参数设置name和在Environment variables中设置MYSQL_ROOT_PASSWORD密码

// 如果是命令行则如下。3307:3306将本机的3307端口映射到docker的3306，***是密码
docker run --name mysql5.7 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=*** mysql:5.7

// 本地连接docker mysql
mysql -h 127.0.0.1 -P 3307 -uroot -p

### docker启动springboot项目
1. idea安装docker插件，并配置preferences->build->docker-> + > 选择docker for mac，下方出现connection successful -> 确定。命令行输入 docker iamges可以看到镜像列表
1. 将springboot打包生成jar(idea右侧maven package按钮)，移动到项目根目录
2. 在根目录创建Dockerfile文件（无后缀）（只要将Dockerfile文件和jar放一起就行，或者修改Dockerfile内容，能找到jar即可）
   Dockerfile内容：
```
# Dockerfile
# 基础镜像，仓库是openjdk，标签用8-jdk-alpine
FROM openjdk:8-jdk-alpine
VOLUME /tmp
#将打包好的jar包拷贝到容器中的指定位置
ADD *.jar app.jar
# -Djava.security.egd=file:/dev/./urandom 可解决tomcat可能启动慢的问题
# 具体可查看：https://www.cnblogs.com/mightyvincent/p/7685310.html
# -Dspring.profiles.active=dev 是运行时参数，运行dev配置文件
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-Dspring.profiles.active=dev","-jar","/app.jar"]
# 对外端口，如果项目server.port配置了8081，则下面也改为8081
EXPOSE 8080
```
3. 构建镜像。 springboot_demo_image是镜像名
```
// 根目录下执行,创建镜像，镜像名为 springboot_demo_image
docker build -t springboot_demo_image .
```
4. 在docker桌面版启动镜像。命令行则是  
```
// 8080:8080,前一个8080是本机端口，后一个是docker服务端口，根目录下执行
docker run --name springboot_demo -p 8080:8080 -d springboot_demo_image
```
5. 浏览器访问项目
http://localhost:8080/springboot-demo/hello/index
   
> package打包和build镜像也可以放在一起做。需要 1. 在pom.xml添加插件<artifactId>docker-maven-plugin</artifactId>（详见https://blog.csdn.net/weixin_41012767/article/details/122509035）2. package和build用 mvn package -Dmaven.test.skip=true docker:build 代替。（应该是这样，没试过）

## k8s安装
参考<https://developer.aliyun.com/article/929808>
参考<https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/>
1.在docker桌面版->设置->kubernetes->勾选Enable Kubernetes即可（需要挂vpn，自动下载）
安装完成后使用kubectl检查下版本
```
// 查看版本
kubectl version
// 查看唯一的master节点
kubectl get node
// 查看默认的名称空间
kubectl get namespace
// 查看默认的pods:
kubectl get pods --all-namespaces
```

2. 安装dashboard 
参考<https://github.com/kubernetes/dashboard>
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```
其实v2.7.0应该是要选对应的版本，先在docker的设置页面看到kubernetes版本，然后找到对应的dashboard版本，然后下载。
3. 启动dashboard
```
kubectl proxy
```
4. 打开dashboard ui界面
   http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
   会提示需要token才能登录
   
5. 得到token。创建dashboard-adminuser.yaml文件(其实文件名无所谓)
内容：
```
apiVersion: v1
kind: ServiceAccount
metadata:
name: admin-user
namespace: kubernetes-dashboard
   ```
创建dashboard-adminuser2.yaml 文件
内容：
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```
命令行执行
```
kubectl apply -f dashboard-adminuser.yaml
kubectl apply -f dashboard-adminuser2.yaml
// 可以得到token
kubectl -n kubernetes-dashboard create token admin-user
```

### docker安装mysql
```
docker pull mysql:5.7

docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7

docker exec -it mysql /bin/bash

mysql -uroot -proot --default-character-set=utf8
```
防火墙打开端口
阿里云服务器的话直接页面上配置

### docker安装redis
```
docker pull redis:7

docker run -p 6379:6379 --name redis \
-v /mydata/redis/data:/data \
-d redis:7 redis-server --appendonly yes

docker exec -it redis redis-cli
```
如有必要打开防火墙

### docker安装nginx
```
docker pull nginx:1.22

// 先运行一次容器（为了拷贝配置文件）：
docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-d nginx:1.22

docker container cp nginx:/etc/nginx /mydata/nginx/

cd /mydata/nginx/

// 修改文件名称
mv nginx conf

docker stop nginx
docker rm nginx

docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.22

```

如有必要打开防火墙

### docker安装RabbitMQ
```
docker search rabbitmq
// 安装的是最新版镜像，如果要指定版本则：docker pull rabbitmq:3.8-management
docker pull rabbitmq
// 启动
docker run -d --hostname my-rabbit --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq
// 安装插件
docker ps -a
docker exec -it 镜像ID /bin/bash
rabbitmq-plugins enable rabbitmq_management
```
// 访问
http://ip地址:15672  
用户名：guest
密码：guest
权限配置参考：<https://www.macrozheng.com/mall/deploy/mall_deploy_docker.html#rabbitmq%E5%AE%89%E8%A3%85>

(防火墙打开端口15672)
firewall-cmd --zone=public --add-port=15672/tcp --permanent
firewall-cmd --reload

### docker安装es、es-head
```
docker pull elasticsearch:7.17.3

docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-e "ES_JAVA_OPTS=-Xms512m -Xmx1024m" \
-v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-d elasticsearch:7.17.3

// 启动时会发现/usr/share/elasticsearch/data目录没有访问权限，只需要修改/mydata/elasticsearch/data目录的权限，再重新启动即可；
chmod 777 /mydata/elasticsearch/data/

// 安装中文分词器IKAnalyzer
下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases
解压到 Elasticsearch的/mydata/elasticsearch/plugins/文件夹下
例如解压后的文件命名为 analysis-ik（名字可以变换） 解压后的若干文件路径为 /mydata/elasticsearch/plugins/analysis-ik  

// 开启防火墙（阿里云服务器的话直接控制台页面设置9200）
firewall-cmd --zone=public --add-port=9200/tcp --permanent
firewall-cmd --reload

访问 http://106.15.106.216:9200/ 会看到版本信息

分词器测试：
postman:
post请求： http://106.15.106.216:9200/_analyze
参数raw json:
{
    "analyzer":"ik_max_word",
    "text":"中华人民共和国"
}

// 安装es head
docker pull mobz/elasticsearch-head:5
docker run --name es-head -p 9100:9100 -d mobz/elasticsearch-head:5
// 访问：http://106.15.106.216:9100/
// 页面连接 http://106.15.106.216:9200/ 的es时会报跨域问题
// 解决：
docker exec -it elasticsearch /bin/bash
vi config/elasticsearch.yml
// 光标移动到最后一行，按"o"编辑文件,添加如下两行
http.cors.enabled: true 
http.cors.allow-origin: "*"
// 保存重启es即可

```

如有必要打开防火墙

### Logstash安装
```
docker pull logstash:7.17.3
mkdir /mydata/logstash
```
创建/mydata/logstash目录，并将Logstash的配置文件logstash.conf拷贝到该目录
（配置文件位置：https://github.com/macrozheng/mall/blob/master/document/elk/logstash.conf）
```
docker run --name logstash -p 4560:4560 -p 4561:4561 -p 4562:4562 -p 4563:4563 \
--link elasticsearch:es \
-v /mydata/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-d logstash:7.17.3

// 进入容器内部 安装json_lines插件。
logstash-plugin install logstash-codec-json_lines
```


### Kibana安装
```
docker pull kibana:7.17.3

docker run --name kibana -p 5601:5601 \
--link elasticsearch:es \
-e "elasticsearch.hosts=http://es:9200" \
-d kibana:7.17.3

firewall-cmd --zone=public --add-port=5601/tcp --permanent
firewall-cmd --reload
```
访问地址进行测试：http://106.15.106.216:5601

### MongoDB安装
```
docker pull mongo:4

// 因为端口冲突，我把本地端口改成了27018，即27018:27017
docker run -p 27017:27017 --name mongo \
-v /mydata/mongo/db:/data/db \
-d mongo:4
```

### MinIO安装
```
docker pull minio/minio

docker run -p 9090:9000 -p 9001:9001 --name minio \
-v /mydata/minio/data:/data \
-e MINIO_ROOT_USER=minioadmin \
-e MINIO_ROOT_PASSWORD=minioadmin \
-d minio/minio server /data --console-address ":9001"

// 防护墙开放端口
// 访问地址：<http://106.15.106.216:9090>
// 账号密码：
// minioadmin
// minioadmin
```