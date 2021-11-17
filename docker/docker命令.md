# docker命令

[官网](https://docs.docker.com/reference/)

## 镜像命令

#### 列出镜像

```shell
docker images

# 输出结果
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   2 months ago   13.3kB
# 解释
镜像的仓库源     标签		id				创建时间		大小

# 可选项

--all ， -a		显示所有图像（默认隐藏中间图像）
--digests		显示摘要
--filter ， -f		根据提供的条件过滤输出
--format		使用Go模板打印漂亮的图像
--no-trunc		不要截断输出
--quiet ， -q		仅显示镜像ID
```

#### 搜索镜像

```shell
docker search （要搜的内容）
```



#### 获取镜像

```shell
docker pull （name：version）
```

#### 删除镜像

```shell
docker rmi -f id
```

## 容器命令

**说明：必须先有镜像才能创建容器！**

#### 新建容器并运行

```shell
docker run image

# 参数说明
--name Name 	容器名字，用来区分容器
-d  			后台方式运行
-it				使用交互方式运行
-p				指定容器端口 -p 8080：8080
	-p	主机端口：容器端口
	-p  容器端口
	
```

#### 退出容器

```shell
exit		  直接退出并停止
ctrl + P + Q  容器不停止退出
```



#### 查看容器

```shell
docker ps 		查看正在运行容器
docker ps -a	查看所有容器
-n=？			最近？个容器
-q				仅显示容器id
```

#### 删除容器

```shell
docker rm 容器id 
```

#### 启动及停止容器

```
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id
```

## 其他常用命令

#### 后台启动命令问题

```shell
-d
#启动服务发现马上停止，是因为docker没有发现前台应用
```

#### 日志操作

```shell
docker logs -f -t --tail 数量 容器id
```

#### 查看进程信息

```shell
docker top 容器id  
```

#### 查看容器元数据

```shell
docker inspect 容器id
```

#### 进入当前正在运行的容器

```shell
#方式1（进入容器后开启一个新的终端（常用））
docker exec -it 容器id 以什么方式打开

#方式2（进入容器正在执行的终端）
docker attach 容器id
```

#### 从容器内拷贝文件到主机

```shell
docker cp 参数1 参数2
#参数1	要移动的文件
#参数2	移动到哪
#注意：	容器id：/文件目录
```

## 提交

我们修改容器后可以使用`commit`指令

```shell
#例
docker commit -a="zx" -m="add webapps" e9a720907d5a tomcat02:1.0
```

