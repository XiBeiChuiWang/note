# Docker核心

## 容器数据卷

经过上面的学习，大家肯定能体会到非常方便，那么docker还有什么问题呢？

1. 我们的数据是放在容器中的，而镜像只保存应用和环境，数据是不会保留的，那么有啥办法对docker的数据进行持久化呢？
2. 我们想要修改docker里的配置，难道每次都得进容器吗？

总结一句：容器数据卷就是为了docker的持久化操作和同步操作。

### 使用数据卷

> 方式一：直接使用命令挂载   -v

```shell
docker run -it -v 主机目录：容器目录  #指定路径挂载
#例
 docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 87eca374c0ed
```

##### 具名挂载和匿名挂载

```shell
#具名挂载
-v 具名：容器目录
#匿名挂载
-v 容器目录
```

> 方式二：直接使用DockerFile挂载 

```shell
--volumes-from
```



## DockerFile

**DockerFile就是用来构建镜像的构建文件**

### DockerFile基本指令

```shell
FROM			#基础镜像，一切从这里开始
MAINTAINER		#镜像作者/邮箱
RUN				#镜像构建时需要的命令
ADD				#镜像添加内容
WORKDIR			#镜像工作目录
VOLUME			#镜像挂载目录
EXPOSE			#端口配置
CMD				#指定容器启动时要执行的命令，只有最后一个生效，可被替代
ENTRYPOINT		#指定容器启动时要执行的命令，可追加命令
ONBUILD			#当构建一个被继承的镜像会执行
COPY			#将我们的文件拷贝到镜像
ENV				#构建时设置环境变量
```

### 实战：创建我们自己的Centos

1. 首先我们创建一个dockerfile文件

   ```shell
   FROM centos
   
   MAINTAINER ZX<2860698744@qq.com>
   
   ENV MYPATH /usr/local
   
   WORKDIR $MYPATH
   
   RUN yum -y install vim
   RUN yum -y install net-tools
   
   EXPOSE 8001
   
   CMD echo $MYPATH
   CMD echo "----end-----"
   CMD /bin/bash
   ```

2. 通过这个文件构建镜像

   ```shell
   docker build -f 文件路径 -t 镜像名：[version]
   ```


### 实战：创建我们自己的tomcat

1. 首先我们创建一个dockerfile文件

   ```shell
   FROM centos
   
   MAINTAINER zxl<2860698744@qq.com>
   
   COPY readme.txt /usr/local/readme.txt
   
   ADD jdk-8u291-linux-x64.tar.gz /usr/local/
   ADD apache-tomcat-9.0.45.tar.gz /usr/local/
   
   RUN yum -y install vim
   
   ENV MYPATH /usr/local
   WORKDIR $MYPATH
   
   ENV JAVA_HOME /usr/local/jdk1.8.0_291
   ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.45
   ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.45
   ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
   
   EXPOSE 8080
   
   CMD /usr/local/apache-tomcat-9.0.45/bin/startup.sh && tail -F /url/local/apache-tomcat-9.0.45/bin/logs/catalina.out
   ```

2. 构建镜像

   ```shell
   docker build -f 文件路径 -t 镜像名：[version]
   ```

3. 运行镜像

   ```shell
   docker run -d --name zxtomcat -p 9090:8080 
   -v /home/zxl/tomcat/test:/usr/local/apache-tomcat-9.0.45/webapps/test 
   -v /home/zxl/tomcat/tomcatlogs:/usr/local/apache-tomcat-9.0.45/logs diytomcat:0.1
   ```

4. ok，通过测试

### 发布我们的镜像

> DockerHub

1. 注册DockerHub账号

2. 在我们的服务器上提交

   ```shell
   docker login -u username  #登录
   
   docker tag 镜像id 用户名/镜像名：版本
   
   docker push 用户名/镜像名：版本
   ```

