# 系统学习Docker践行DevOps理念

## 容器和虚拟机的区别

Docker能干什么：

    简化配置
    整合服务器
    代码流水线管理
    调试能力
    提高开发效率
    多租户
    隔离应用
    快速部署

Docker + kubernetes(k8s)

K8S(容器编排工具)，Docker官方的容器编排工具是Docker Swarm。

## 第一章：容器技术和Docker简介

虚拟化的局限性：

    每一个虚拟机都是一个完整的操作系统，要给其分配资源，当虚拟机数量增多时，操作系统本身消耗的资源势必增多。

容器解决了什么问题？

    解决了开发和运维之间的矛盾
    在开发和运维之间搭建了一个桥梁，是实现devops的最佳解决方案。

什么是容器？

    对软件和其依赖的标准化打包；
    应用之间相互隔离
    共享同一个OS Kernel
    可以运行在很多主流操作系统上；

容器是APP层面的隔离，虚拟化是物理资源层面上的隔离

虚拟化+容器： 在虚拟化主机中使用容器；


Docker-容器技术的一种实现，另一种常用实现CoreOS。

## 第二章：Docker环境的各种搭建方法

docker-community edition、docker-enterprise(basic,standard,advanced) edition

Docker是一个Linux Application，要准备一个安装好了Docker的Linux系统。

* 1. Install Docker For Mac。(included Docker Engine，Docker Compose，Docker Machine and Kitematic(GUI))

* 2. Install Docker For Windows.

    要求： win10 或 winserver2016，requires 64bit windows 10 Pro with Hyper-V available。

* 3. windows + vagrant + virtualbox + docker 

```bash
    vagrant --help
    mkdir centos7 && cd centos7
    vagrant init centos/7
    vagrant up
    vagrant ssh(sudo yum update)
    exit
    vagrant status
    vagrant halt    挂起虚拟机
    vagrant destroy   删除虚拟机
```

vagrantfile

vagrant也可以驱动vmware，不建议使用； vagrant up --provider=vmware_fusion

* 在centos上安装docker：

```bash
# Uninstall old version
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

```bash
# Install requirements
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

```bash
# setup stable repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

```bash
# Install docker-ce
sudo yum install docker-ce docker-ce-cli containerd.io
```

```bash
sudo systemctl start docker   # start docker
sudo docker run hello-world   # test  docker
```

* 修改vagrantfile，使得vagrant启动虚拟机时自动安装docker：

```bash
xiong@sslinux-development MINGW64 /f/vagrant/centos
$ tail -10 Vagrantfile
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
        sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine -y
        sudo yum install -y yum-utils device-mapper-persistent-data lvm2
        sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        sudo yum install -y docker-ce docker-ce-cli containerd.io
        sudo systemctl start docker
        sudo systemctl enable docker
  SHELL
end
```

* Docker Machine

[Doc for Docker-Machine](https://docs.docker.com/machine/install-machine/)

Install docker-machine on winwons with GitBash:

```bash
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  mkdir -p "$HOME/bin" &&
  curl -L $base/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe" &&
  chmod +x "$HOME/bin/docker-machine.exe"
```

`使用docker-machine在本地创建包含docker的虚拟机：`

```bash
docker-machine create demo   # 创建虚拟机
docker-machine ls            # 查看通过docker-machine创建的虚拟机状态；
docker-machine ssh demo      # ssh连接虚拟机；
docker-machine stop demo     # 停止虚拟机；
docker-machine env demo      # 查看demo中的环境设置

eval $(docker-machine env daemon)    # 将本地的docker server设置为daemon中的；

# 取消上述设置：
docker-machine env --help
eval $(docker-machine env --unset)   # 将docker server设置为本地主机；
```

docker-machine的driver可以管理常见的云服务厂商，即可以使用docker-machine命令在云平台上创建已经包含docker环境的主机。

```bash
$ docker-machine create --driver digitalocean --digitalocean-access-token xxxxx docker-sandbox

$ docker-machine create --driver amazonec2 --amazonec2-access-key AKI******* --amazonec2-secret-key 8T93C*******  aws-sandbox
```

`使用docker-machine在aliyun上创建包含docker的虚拟机:`

AliyunECS:

需要自己安装相关驱动： https://github.com/AliyunContainerService/docker-machine-driver-aliyunecs

保证账户余额大于100元；
访问控制——用户详情——创建AccessKey

```bash
docker-machine create -d aliyunecs --aliyunecs-io-optimized=optimized --aliyunecs-instance-type=ecs.c5.large --aliyunecs-access-key-id=********* --aliyunecs-access-key-secret=****************** --aliyunecs-region=cn-qingdao sslinux
```

docker-machine rm sslinux   # 删除虚拟机，ECS中的实例是要收费的，不用的时候一定要删除；


`AWS，EC2:需要绑定信用卡`

Security --》 IAM --》 创建AccessKey

[详细文档](https://docs.docker.com/machine/drivers/aws/)

docker-machien的awsec2驱动比aliyunecs稳定得多，记得删除避免扣费；


### docker-playground： 无法创建docker环境时使用；

使用docker的账号登录；

临时的，一段时间不使用，就会被销毁；


### Docker安装的总结：
* 1.在Mac上玩Docker

        docker for Mac直接安装；
        通过Virtualbox或者Vmware虚拟化软件直接创建Linux虚拟机，在虚拟机里安装使用Docker。
        通过Vagrant + virtualbox快速构建Docker host。
        通过docker-machine快速搭建Docker host。

* 2.在Windows上玩docker

        Docker for Windows直接安装(队系统要求高至少要win10 pro)，开启Hyper-V；
        通过Virtualbox或者Vmware虚拟化软件直接创建Linux虚拟机，在虚拟机里安装使用Docker；
        通过Vagrant + VitualBox快速搭建Docker host
        通过docker-machine快速单间Docker host

* 3.在linux上玩docker

        Linux主机
        Linux虚拟机(支持虚拟化的任何操作系统或者平台)

* 4.在云上玩docker

        docker-machine + driver (AWS,Aliyun等)
        直接使用与服务上提供的容器服务：
            AWS的ECS(Amazon Elastic Container Service)
            Aliyun的Container Service

---


## 第三章：Docker的镜像和容器

Docker的架构和底层技术：

        Docker提供了一个开发，打包，运行app的平台
        把app和底层infrastructure隔离开来；

Docker Engine:

        后台进程(dockerd)
        REST API Server
        CLI接口(docker)


Docker底层技术支持：

        Namespaces： 做隔离pid，net，ipc，mnt，uts
        Control groups： 做资源限制
        Union file systems: Container和image的分层

### Docker Image：

什么是Image：  

        文件和meta data的集合(root filesystem)
        分层的，并且每一层都可以添加改变删除文件，成为一个新的image；
        不同的image可以共享相同的layer；
        image本身是read-only的

        [vagrant@localhost ~]$ sudo docker image ls


Image的获取：

* 1.Build from Dockerfile：

```bash
mkdir redis
cd redis
cat > Dockerfile << EOF
    FROM ubuntu:14.04
    LABEL maintainer="Guiyin Xiong <guiyin.xiong@gmail.com>"
    RUN apt-get update && apt-get install -y redis-server
    EXPOSE 6379
    ENTRYPOINT [ "/usr/bin/redis-server" ]

sudo docker build -t sslinux/redis:latest .
```


* 2.Pull from Registry

拉取官方image： sudo docker pull IMAGE:TAG

非官方image：   sudo docker pull USERNAME/IMAGE:TAG

```bash
$ docker pull ubuntu:14.04
```

将当前用户加入docker组，使得运行docker命令时不必再用sudo。

        sudo gpasswd -a vagrant docker


* 3.制作base image：

```bash
[vagrant@localhost ~]$ docker pull hello-world
[vagrant@localhost ~]$ docker run hello-world
```

```bash
[vagrant@localhost ~]$ mkdir hello-world
[vagrant@localhost ~]$ vim hello.c
[vagrant@localhost ~]$ cat hello.c
#include<stdio.h>

int main()
{
        printf("hello docker\n")
}
[vagrant@localhost ~]$ sudo yum install -y gcc glibc-static
[vagrant@localhost ~]$ gcc -static hello.c -o hello

[vagrant@localhost hello-world]$ vim Dockerfile
[vagrant@localhost hello-world]$ cat Dockerfile
FROM scratch
ADD hello /
CMD ["/hello"]


[vagrant@localhost hello-world]$ docker build -t sslinux/hello-world .
Sending build context to Docker daemon  869.9kB
Step 1/3 : FROM scratch
 --->
Step 2/3 : ADD hello /
 ---> 28d43280360b
Step 3/3 : CMD ["/hello"]
 ---> Running in 20e177bdd13e
Removing intermediate container 20e177bdd13e
 ---> 6cfb19e6e5f9
Successfully built 6cfb19e6e5f9
Successfully tagged sslinux/hello-world:latest


[vagrant@localhost hello-world]$ docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
# 只有800多k
sslinux/hello-world   latest              6cfb19e6e5f9        27 seconds ago      857kB
sslinux/redis         latest              085d7cef2f2c        21 minutes ago      205MB
ubuntu                14.04               5dbc3f318ea5        5 weeks ago         188MB
hello-world           latest              fce289e99eb9        2 months ago        1.84kB

# 查看某镜像的分层
[vagrant@localhost hello-world]$ docker history 6cfb19e6e5f9
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
6cfb19e6e5f9        2 minutes ago       /bin/sh -c #(nop)  CMD ["/hello"]               0B
28d43280360b        2 minutes ago       /bin/sh -c #(nop) ADD file:0bd91ef318c5fa6bf鈥?   857kB


[vagrant@localhost hello-world]$ docker run sslinux/hello-world
hello docker
[vagrant@localhost hello-world]$
```

### 什么是container？

        通过Image创建(copy)
        在Image layer之上建立一个container layer(可读写)
        类比面向对象：类和实例
        Image负责app的存储和分发，Container负责运行app；

```bash
# 查看当前正在运行的container
[vagrant@localhost hello-world]$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

# 查看所有的container，包括已经运行结束的；
[vagrant@localhost hello-world]$ docker container ls -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS                      PORTS               NAMES
f2b21d14b2d3        sslinux/hello-world   "/hello"            6 minutes ago       Exited (13) 6 minutes ago                       gallant_leavitt
fab54bbc5330        hello-world           "/hello"            23 minutes ago      Exited (0) 23 minutes ago                       peaceful_joliot
```

```bash
docker image ls == docker images
docker container ls == docker ps

docker rm    # 删除container
docker rmi   # 删除image

[vagrant@localhost ~]$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[vagrant@localhost ~]$ docker container ls -aq　　 # 这里列出的是所有container，包括up的；
f2b21d14b2d3
fab54bbc5330
[vagrant@localhost ~]$ docker rm $(docker container ls -aq) 
f2b21d14b2d3
fab54bbc5330

[vagrant@localhost ~]$ docker container ls -f "status=exited" -q
[vagrant@localhost ~]$ docker rm $(docker container ls -f "status=exited" -q)
```

* docker container commit  == docker commit
* docker image build == docker build

```bash

```



第四章：Docker的网络


第五章：Docker的持久化存储和数据共享
第六章：Docker Compose多容器部署
第七章：容器编排Docker Swarm
第八章：DevOps初体验——Docker Cloud和Docker企业版
第九章：容器编排Kubernetes
第十章：容器的运维和监控
第十一章：Docker+DevOps实战——过程和工具
第十二章：总结



