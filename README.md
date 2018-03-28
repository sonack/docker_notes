# Docker
  
  
* [Docker](#docker )
	* [Overview](#overview )
		* [Docker是什么?](#docker是什么 )
		* [Docker架构](#docker架构 )
	* [Installation](#installation )
		* [Ubuntu](#ubuntu )
			* [使用脚本安装](#使用脚本安装 )
			* [卸载docker](#卸载docker )
	* [Docker的使用](#docker的使用 )
		* [An Example](#an-example )
		* [Interactive Container](#interactive-container )
		* [启动容器(后台模式)](#启动容器后台模式 )
		* [停止容器](#停止容器 )
	* [容器使用](#容器使用 )
		* [运行一个web应用](#运行一个web应用 )
		* [重启容器](#重启容器 )
			* [docker start和docker run的区别?](#docker-start和docker-run的区别 )
		* [删除容器](#删除容器 )
	* [镜像使用](#镜像使用 )
		* [列出镜像列表](#列出镜像列表 )
		* [获取新的镜像](#获取新的镜像 )
		* [查找镜像](#查找镜像 )
		* [创建镜像](#创建镜像 )
			* [更新镜像](#更新镜像 )
			* [构建镜像](#构建镜像 )
		* [设置镜像标签](#设置镜像标签 )
	* [容器连接](#容器连接 )
		* [网络端口映射](#网络端口映射 )
		* [Docker容器连接](#docker容器连接 )
	* [Questions](#questions )
  
## Overview
  
  
### Docker是什么?
  
  
[Docker](https://www.docker.com/ )是一个开源的应用容器引擎，可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的linux机器上，可以实现虚拟化。
由Go语言开发，遵循Apache2.0开源协议。
  
### Docker架构
  
  
(c/s)架构，可以使用远程API来创建和管理Docker容器。
  
Docker容器可以通过Docker镜像创建。
  
容器和镜像的关系，类似于对象与类。
  
镜像是类，可以创建多个对应的容器。
  
![Docker架构](http://www.runoob.com/wp-content/uploads/2016/04/576507-docker1.png )
  
**Docker Images**: 镜像是创建container的模板
**Docker Container**: 容器是独立运行的一个或一组应用
**client**: 客户端通过命令行或者其他工具使用Docker API与Docker的守护进程通信。
**host**: 一个物理或者虚拟的机器，用于执行Docker守护进程和容器。
**Registry**: 仓库，用于保存镜像，可以理解为代码控制中的代码仓库。DockerHub提供了庞大的镜像集合可以使用。
**Docker Machine**: DM是一个简化Docker安装的CLI工具，通过它可以在相应平台安装docker，如VirtualBox, Digit Ocean, Microsoft Azure.
  
  
## Installation
  
  
### Ubuntu
  
  
**前提条件:** Ubuntu的内核版本要高于3.10，可以通过以下命令查看:
  
```
➜  ~ uname -r
4.13.0-37-generic
```
  
#### 使用脚本安装
  
  
```
wget -qO- https://get.docker.com/ | sh
sudo usermod -aG docker runoob
  
# logout and login back
  
```
  
启动docker后台服务:
  
```
sudo service docker start
```
  
测试hello-world:
  
```
docker run hello-world
```
  
配置镜像:
  
/etc/docker/daemon.json
  
```
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
  
  
#### 卸载docker
  
  
```
sudo apt-get purge docker.io  # sudo apt-get purge runc containerd docker.io
sudo rm -rf /var/lib/docker
sudo rm -rf /etc/docker
  
  
# Remove docker from apparmor.d:
sudo rm /etc/apparmor.d/docker
# Remove docker group:
sudo groupdel docker
  
# 检查是否卸载干净
dpkg -l | grep -i docker
```
  
## Docker的使用
  
  
### An Example
  
docker run 命令在container中运行一个app
  
```
docker run ubuntu:15.10 /bin/echo "Hello world"
```
  
*ubuntu:15.10* 是一个指定的镜像，Docker实现从本地主机上查找镜像是否存在，如果不存在，则会从镜像仓库Docker Hub上拉取公共镜像。
*/bin/echo "Hello world"* 在启动的container中要执行的命令.
  
整体含义: docker以ubuntu 15.10镜像创建一个新容器，在该容器里执行'bin/echo "Hello world"'，然后输出结果。
  
### Interactive Container
  
  
通过参数 `-i -t` 让docker运行的容器实现"对话"能力。
其中`-t`表示在新容器内指定一个伪终端或终端, `-i`表示允许对容器内的stdin进行交互。
  
`cat /proc/version`
  
### 启动容器(后台模式)
  
  
```
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
创建了一个以进程方式运行的容器。
  
输出了 container ID
  
```
docker ps  # 确认运行中的容器
  
docker logs (container ID or Name)  # 查看ID对应容器的stdout
```
  
  
### 停止容器
  
  
```
docker stop (container ID or Name)
docker ps # 确认是否关闭
```
  
  
## 容器使用
  
  
help
  
```
docker
  
docker stats --help
```
  
### 运行一个web应用
  
  
```
docker pull training/webapp
docker run -d -P training/webapp python app.py
```
  
-d: 后台运行容器
-P: 将容器内部使用的网络端口映射到主机
  
利用ps命令查看，发现多了端口信息
  
```
PORTS
0.0.0.0:32769->5000/tcp
```
  
含义为docker内部开放了5000端口(Flask默认端口)，将其映射到主机的32769端口.
可以使用浏览器访问`your_IP_address:32769`访问.
  
也可以通过-p来指定映射端口
  
```
docker run -d -p 5000:5000 training/webapp python app.py
```
  
还可以使用docker port来查看指定ID或Name的容器的某个确定端口映射到host的端口号。
  
```
docker logs -f ID
```
  
-f 可以像tail -f一样实时输出展示log(stdout).
  
可以使用`docker top`来查看容器内部运行的进程。
`docker inspect`检查Docker容器的底层信息， 返回一个包含容器配置和状态信息的json文件。
  
### 重启容器
  
  
```
docker ps -l # 查询最后一次(last)创建的容器
docker stop ID
docker start ID
  
  
docker restart ID
```
  
#### docker start和docker run的区别?
  
  
1. Run: 创建一个镜像的新容器，并执行它，同一个Image可以创建N个clone的container, 命令是 `run IMAGE_ID` 而不是 CONTAINER_ID.
2. Start: 重新启动一个之前已经停止的Container，可以用`start CONTAINER_ID`来重新打开相同的container，而数据和设置都是一样的。
  
### 删除容器
  
  
docker rm ID_NAME
  
**注意： 删除时，容器必须停止!**
  
## 镜像使用
  
  
### 列出镜像列表
  
  
```
docker images # 列出本地主机镜像
```
  
REPOSITORY: 表示镜像的仓库源
TAG: 镜像的标签
IMAGE ID: 镜像ID
CREATED: 镜像创建时间
SIZE: 镜像大小
  
一个仓库源可以有多个TAG，代表不同版本，如ubuntu仓库源，有15.10、14.04等多个版本，使用`REPOSITORY:TAG`来标识不同的镜像.
如果不指定TAG,默认使用latest镜像。
  
### 获取新的镜像
  
当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 docker pull 命令来下载它。
  
### 查找镜像
  
  
```
docker search httpd
```
  
Fields 说明:
  
NAME: 镜像仓库源的名称
DESCRIPTION: 镜像的描述
OFFICIAL: 是否docker官方发布
  
  
### 创建镜像
  
  
可以通过以下两种方式对镜像进行更改:
  
1. 从已创建的容器中更新镜像，并且提交这个镜像
2. 使用Dockerfile指令来创建一个新的镜像
  
#### 更新镜像
  
  
首先创建一个容器
  
```
docker run -it ubuntu:15.10 /bin/bash
  
# 做一些修改
  
exit # 退出容器
  
docker commit -m='has update' -a='author' CONTAINER_ID runoob/ubuntu:v2
```
  
  
  
  
#### 构建镜像
  
  
使用docker build，从0开始构建一个新镜像，需要一个dockerfile文件，其中包含了一组指令，来告诉Docker如何构建镜像。
  
dockerfile的每一条指令都会在Image上创建一个新层，每个指令前缀都大写。
比如:
  
FROM 指定使用哪个镜像源
RUN 在镜像内执行命令，安装东东
  
```
docker build -t runoob/centos:6.7 .
```
`-t` 指定创建的目标镜像名
`.` 指定dockerfile文件所在目录，可以是dockerfile文件所在目录的绝对路径
  
eg:
  
CLIC:
  
```
docker build -t clic:v1 /home/snk/Desktop/CLIC/
```
  
### 设置镜像标签
  
  
```
docker tag IMAGE_ID runoob/centos:dev
```
  
  
runoob/centos:dev 用户名/repository_name:tag_name
  
  
## 容器连接
  
  
实现通过某个端口，连接到一个docker容器。
  
### 网络端口映射
  
  
```
docker run -d -P training/webapp python app.py
```
  
可以指定容器绑定的网络地址，比如127.0.0.1
  
使用-p标识来指定容器端口绑定到主机端口:
  
* -P: 容器内部端口随机映射到host的高端口
* -p: 容器内部端口绑定到指定的host的端口
  
```
docker run -d -p 127.0.0.1:5001:5000 training/webapp python app.py
```
可以访问`127.0.0.1:5001`
  
默认都是绑定tcp端口，如果要绑定udp端口，可以在端口后面加上/udp.
  
```
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```
  
  
```
docker port adoring_stonebraker 5000
```
  
  
### Docker容器连接
  
  
docker有一个连接系统，允许将多个容器连接在一起，共享连接信息。
docker连接会创建一个父子关系，其中父容器可以看到子容器的信息。
  
当我们创建一个容器的时候，docker会自动对它进行命名。另外，我们也可以使用--name标识来命名容器，例如：
  
```
docker run -d -P --name runoob training/webapp python app.py
```
  
---
  
## Questions
  
  
docker start -a  # continue Interactive shell
  
docker run default cmd [https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa]
  
repository name must be lowercase