# docker安装hadoop集群

为了我们使用方便，我们采用docker来安装我们的hadoop，顺带复习一下docker

## 构建centos基础镜像

我们的基础镜像使用centos

```shell
docker pull centos
```

拉取镜像后，我们新建一个dockerfile文件

```dockerfile
FROM centos

//下载ssh
RUN yum install -y openssh-server sudo 
RUN sed -i 's/UsePAM yes/UsePAM no/g' /etc/ssh/sshd_config
RUN yum  install -y openssh-clients

//设置账号密码，并生成密钥
RUN echo "root:******" | chpasswd
RUN echo "root   ALL=(ALL)       ALL" >> /etc/sudoers
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key

RUN mkdir /var/run/sshd
//开放端口
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

构建镜像

```shell
docker build -t centos-ssh .
```

## 构建Hadoop镜像

```dockerfile
FROM centos-ssh

ADD jdk-8u291-linux-x64.tar.gz /usr/local/
RUN mv /usr/local/jdk1.8.0_291 /usr/local/jdk1.8
ENV JAVA_HOME /usr/local/jdk1.8
ENV PATH $JAVA_HOME/bin:$PATH

ADD hadoop-3.1.3.tar.gz /usr/local
RUN mv /usr/local/hadoop-3.1.3 /usr/local/hadoop
ENV HADOOP_HOME /usr/local/hadoop
ENV PATH $HADOOP_HOME/bin:$PATH

RUN yum install -y which sudo
```

