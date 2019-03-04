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


第三章：Docker的镜像和容器
第四章：Docker的网络
第五章：Docker的持久化存储和数据共享
第六章：Docker Compose多容器部署
第七章：容器编排Docker Swarm
第八章：DevOps初体验——Docker Cloud和Docker企业版
第九章：容器编排Kubernetes
第十章：容器的运维和监控
第十一章：Docker+DevOps实战——过程和工具
第十二章：总结



