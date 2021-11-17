# Docker简介

## docker概述

在云时代，开发者创建的应用必须要能很方便地在网络上传播，也就是说应用必须脱离底层物理硬件的限制；同时必须满足“任何时间任何地点”可获取可使用的特点。因此，开发者们需要一种新型的创建分布式应用程序的方式，快速分发部署，而这正是Docker所能够提供的最大优势。Docker提供了一种更为聪明的方式，通过容器来打包应用、解耦应用和运行平台。这意味着迁移的时候，只需要在新的服务器上启动需要的容器就可以了，无论新旧服务器是否是同一类别的平台。这无疑帮助我们节约了大量的宝贵时间，并降低部署过程出现问题的风险。 

### 什么是容器

容器就是在隔离的环境运行的一个进程，如果进程停止，容器就会退出。隔离的环境拥有自己的系统文件，ip地址，主机名等 

### 为什么使用Docker？

 **1、更快的交付和部署：**
使用Docker，开发人员可以使用镜像来快速构建一套标准的开发环境；开发完之后，测试和运维人员可以直接使用完全相同的环境来部署代码。只要是开发测试过的代码，就可以确保在生产环境无缝运行。Docker可以快速创建和删除容器，实现快速迭代，节约开发、测试及部署的时间。
**2、更高效的利用资源：**
运行Docker容器不需要额外的虚拟化管理程序的支持，Docker是内核级的虚拟化，可以实现更高的性能，同时对资源的额外需求很低，与传统的虚拟机方式相比，Docker的性能要提高1~2个数量级。
**3、更轻松的迁移和扩展：**
Docker容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑等等，同时支持主流的操作系统发行版本。这种兼容性能让用户可以在不同的平台之间轻松的迁移应用。
**4、更轻松的管理和更新：**
使用Dockerfile，只需要小小的配置修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的容器管理。

###  Docker容器与传统虚拟化的区别 

- 虚拟机: 硬件cpu支持(vt虚拟化),模拟计算硬件,走正常的开机启动

- 容器: 不需要走开机启动流程，不需要硬件cpu的支持,共用宿主机内核去启动容器的第一个进程。
  **容器优势:** 启动快,性能高,损耗少,轻量级
  	100个虚拟机运行100个服务需要10台物理机
  	100个容器运行100个服务需要大约6台物理机
  
- 为什么Docker比虚拟机快？

  1.Docker有着比虚拟机更少的抽象层，由于Docker不需要Hypervisor实现硬件资源虚拟化，运行在Docker容器上的程序直接使用的都是实际物理机的硬件资源，因此在Cpu、内存利用率上Docker将会在效率上有明显优势。

  2.Docker利用的是宿主机的内核，而不需要Guest OS，因此，当新建一个容器时，Docker不需要和虚拟机一样重新加载一个操作系统，避免了引导、加载操作系统内核这个比较费时费资源的过程，当新建一个虚拟机时，虚拟机软件需要加载Guest OS，这个新建过程是分钟级别的，而Docker由于直接利用宿主机的操作系统则省略了这个过程，因此新建一个Docker容器只需要几秒钟。

  |            | Docker容器              | 虚拟机（VM）                |
  | ---------- | ----------------------- | --------------------------- |
  | 操作系统   | 与宿主机共享OS          | 宿主机OS上运行宿主机OS      |
  | 存储大小   | 镜像小，便于存储与传输  | 镜像庞大（vmdk等）          |
  | 运行性能   | 几乎无额外性能损失      | 操作系统额外的cpu、内存消耗 |
  | 移植性     | 轻便、灵活、适用于Linux | 笨重、与虚拟化技术耦合度高  |
  | 硬件亲和性 | 面向软件开发者          | 面向硬件运维者              |

### Docker的核心概念

#### 镜像（Image）

Docker镜像（Image），就相当于是一个root文件系统。比如官方镜像ubuntu:16.04就包含了完整的一套Ubuntu16.04最小系统的root文件系统。

#### 容器（Container）

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等

#### 仓库（Repository）

用来保存镜像的仓库。当我们构建好自己的镜像之后，需要存放在仓库中，当我们需要启动一个镜像时，可以在仓库中下载下来。（类似于git）

## docker安装

1. 卸载旧版本

   ```bush
   yum remove docker \
                     docker-client \
                     docker-client-latest \
                     docker-common \
                     docker-latest \
                     docker-latest-logrotate \
                     docker-logrotate \
                     docker-engine
   ```

   

2. 使用存储库安装

   在新主机上首次安装Docker Engine之前，需要设置Docker存储库。之后，您可以从存储库安装和更新Docker。

   设置存储库

   安装`yum-utils`软件包（提供`yum-config-manager` 实用程序）并设置**稳定的**存储库。

   ```bush
   yum install -y yum-utils
   yum-config-manager \
       --add-repo \
       http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ```

3. 安装docker引擎

   ```bush
   yum install docker-ce docker-ce-cli containerd.io
   ```

4. 更新yum索引

   ```bush
   yum makecache fast
   ```

5. 安装docker相关

   ```bush
   yum install docker-ce docker-ce-cli containerd.io
   ```

6. 启动docker

   ```bush
   systemctl start docker
   ```

7. 查看是否安装成功

   ```bush
   docker version
   ```

8. 跑第一个镜像

   ```bush
    docker run hello-world
   ```

9. 查看镜像列表

   ```bush
   docker images
   ```

## docker卸载

1.  卸载Docker Engine，CLI和Containerd软件包： 

   ```bush
   yum remove docker-ce docker-ce-cli containerd.io
   ```

2.  主机上的映像，容器，卷或自定义配置文件不会自动删除。要删除所有图像，容器和卷： 

   ```bush
   rm -rf /var/lib/docker
   rm -rf /var/lib/containerd
   ```

##  docker原理

我们来看看上面 `docker run hello-world`

[![](https://pic.imgdb.cn/item/60967984d1a9ae528f2a9e02.jpg)](https://pic.imgdb.cn/item/60967984d1a9ae528f2a9e02.jpg)



