# docker
### 下载安装
官网下载docker桌面版
https://www.docker.com/
参考<https://www.runoob.com/docker/docker-install-mysql.html>

### docker启动mysql
docker内搜索mysql镜像，选择5.7版本，pull。
启动参数设置name和在Environment variables中设置MYSQL_ROOT_PASSWORD密码

// 如果是命令行则如下。3307:3306将本机的3307端口映射到docker的3306，***是密码
docker run --name mysql5.7 -p 3307:3306 -e MYSQL_ROOT_PASSWORD=*** mysql:5.7

### docker启动springboot项目
1. idea安装docker插件，并配置preferences->build->docker-> + > 选择docker for mac，下方出现connection successful -> 确定。命令行输入 docker iamges可以看到镜像列表
1. 将springboot打包生成jar，移动到根目录（idea的maven package即可）
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
// 根目录下执行
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
