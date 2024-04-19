### 认识docker

#### 没有docker产生的问题

- 配置繁琐

环境不一致 ——两个应用需要的版本可能不一致

- 资源隔离

内存、cpu、网络带宽、磁盘相互影响，内存泄漏问题，软件出现bug

- 交付流程复杂 

手动安装业务服务、基础服务

#### 概念

docker是一种容器引擎，管理容器的生命周期

容器：用于运行一个软件的容器，该容器中准备了软件运行的一系列依赖

镜像：容器的安装包，容器中需要哪些内容，以及对应的配置信息，全部打包在里面，要运行一个容器，必须要有一个镜像

镜像仓库：存放各个镜像，对镜像进行统一管理

#### 容器与虚拟机区别

资源损耗、启动时间

容器共享操作系统，共享内核

虚拟机又新创建了一套操作系统和内核

#### docker架构

- docker client 客户端

- docker daemon

docker server：服务端，接受请求

engine：容器引擎，真正负责执行对应的任务

- docker registry

镜像仓库，存储镜像，用户相关信息

官方dockerhub

阿里云docker镜像仓库

### docker安装

- 要求

内核版本>=3.10

- 版本

docker ce 官方社区版本

docker ee 企业版本

- 操作过程

开发者发送命令给docker client，client发送请求给docker server，镜像操作给镜像管理，去镜像仓库拉取镜像，容器操作去容器引擎，分成一个个小任务（job）

- 安装

配置阿里云镜像（选做）

```shell
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#yum源使用阿里云镜像网站

#配置阿里云镜像仓库
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
EOF

systemctl daemon-reload
# 启动docker引擎并设置开机启动
sudo systemctl start docker
sudo systemctl enable docker
```

docker ce安装

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2 #安装依赖包
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo #安装稳定测试版本

# 打开体验版与测试版
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-test

$ # 关闭体验版与测试版
$ sudo yum-config-manager --disable docker-ce-edge
$ sudo yum-config-manager --disable docker-ce-test

sudo yum install -y docker-ce #安装docker ce最新版

yum list docker-ce --showduplicates | sort -r #安装别的版本
sudo yum install -y docker-ce-18.03.0.ce
sudo systemctl start docker
```

### docker基本命令

- 镜像操作

```shell
docker search tomcat #从镜像仓库搜索镜像
docker version #版本详细信息
dockers -v 
docker pull openjdk:18#拉取镜像
docker images #查看本地镜像
docker rmi <IMAGE ID> #删除镜像
docker info  #配置信息
```

- 容器操作

```shell
docker ps #列出运行中的容器
docker ps -a #列出所有容器
docker rm <CONTAINER ID> #删除容器
docker inspect <CONTAINER ID> #查看容器内配置信息
docker exec -it <CONTAINER ID> <Command> #向容器内部发送命令执行

docker run <IMAGE NAME> #创建新容器并运行
-d  #后台进程方式运行
-p port1:port2 #指定端口映射，左边是宿主机，右边是容器
-P  #暴露容器中所有的端口，并且在主机中使用随机端口去映射到这些暴露的端口
--name  #命名容器
--rm #运行退出后删除容器
--restart on-failure:3 #失败后重启次数3 
always:总是启动
-e/--env #指定环境变量
-m 2g/2m/2k/2b #限制内存
--cpus 1 #限制cpu
docker stats <IMAGE NAME> #实时监听状态

docker logs (-f -n) <IMAGE NAME> #查看容器日志
docker exec -it <IMAGE NAME>  /bin/bash(/bin/sh) #进入容器内部
```

注意：docker容器中的文件系统与宿主机文件系统共用，通过不同文件夹隔离

附图：

<img src="https://raw.githubusercontent.com/NilPointerError/MdImage/main/img/image-20240315134952262.png" alt="image-20240315134952262" width="700" />

### 数据卷

作用：关联容器内与容器外的文件数据，容器删除后，数据还存在。实现在容器外更改生效。

- 绑定

```shell
docker volume ls
docker volume inspect
```

- 匿名绑定

只起到文件共享作用，匿名绑定的volume在删除容器后，数据卷也会删除

- 具名绑定

容器删除后，数据卷不会删除

```shell
docker run --rm -d -p80:80 --name nginx_vol -v nginx-html:/usr/share/nginx/html nginx
```

数据卷目录：/var/lib/docker/volumes/nginx-html/_data

- Bind Mount

绑定并加载主机的某个文件目录到容器中

```shell
docker run --rm -d -p80:80 --name nginx_vol -v /usr/local/nginx-1.6/html:/usr/share/nginx/html nginx
```

### 网络 network

- 是什么？


是docker对容器网络隔离的一项技术，提供了多种不同的模式供用户使用，选择不同的网络模式来实现容器网络的互通以及彻底的隔离

- 为什么需要？

  - 容器间的网络隔离


  - 实现部分容器之间的网络共享

  - 管理多个子网下容器的ip 


- 能干什么？

  - 提供了多种模式，可以定制化的为每个容器置顶不同的网络


  - 自定义网络模式，划分不同的子网以及网关、dns等配置

  - 网络互通  
    - 实现不同子网之间的网络互通，
    - 基于容器名（主机名）网络访问


- 网络模式


```
docker network ls
```

bridge（默认）：在主机中创建一个docker0的虚拟网桥，在docker0创建一对虚拟网卡，有一半在主机上vethxxx，还有一半在容器内eth0

容器间通信需要知道对应的ip地址

host：容器不再拥有自己的网络空间，而是直接与主机共享网络空间，那么基于该模式创建的容器对应的ip实际就是与主机同一个子网，同一个网段。

none：dockers会拥有自己的网络空间，不与主机共享，在这个网络模式下的容器，不会被分配网卡、IP、路由等相关信息。

特点：完全隔离，与外部任何机器都无网络访问，只有自己的IO 本地网络127.0.0.1

container：不会创建自己的网络空间，与其他容器共享网络空间，直接使用指定容器的ip/端口等。

自定义（推荐）：不使用docker自带的网络模式，自己去定制化自己特有的网络模式

```shell
docker network ls
docker network create --driver bridge --subnet 192.168.133.0/24 --gateway 192.168.133.1 wolfcode
docker network  inspect wolfcode
docker run -d --rm -P --name nginx_network1 --net wolfcode centos ping 127.0.0.1
docker exec -it nginx_network1 ping nginx_network2
docker network connect wolfcode net1 #把net1加入到wolfcode网络实现跨网络通信
```

### dockerfile

- 是什么？

docker为我们提供的一个用于自定义构建镜像的一个配置文件：描述如何构建一个对象

利用docker提供的build命令，指定dockerfile文件，就可以按照配置的内容将镜像构建出来

- 为什么需要？

作为开发者需要将自己开发好的项目打包成docker镜像，便于后面直接作为docker容器运行

作为运维人员需要构建更精确的基础设施服务镜像，满足公司的需要以及尽可能减少冗余的功能占用过多的资源

- 能干什么？
  - 可以自定义镜像内容
  - 构建公共镜像减少其他镜像配置
  - 开源程序的快速部署
  - 实现企业内部项目的快速交付

commit命令

基于一个现有的容器，构建一个新的镜像

```shell
docker commit -a="wolfcode" -m="first image" centos7 mycentos:7

OPTIONS：
-a：镜像的作者
-c：使用 Dockerfile 指令来构建镜像
-m：提交时的描述
-p：在 commit 时暂停容器
```

build命令

```
docker build -t <image name> <dockerfile dir>  #构建镜像
```

编写dockerfile

```shell
# 先指定当前镜像的基础镜像是什么
FROM openjdk:8
# 描述这个镜像的作者，以及联系方式
MAINTAINER
# 镜像的标签信息
LABEL version="1.0"
LABEL description=“这是我的第一个dockerfile”
#环境变量配置
ENV JAVA_ENV dev
# 构建镜像是，需要执行的shell命令
RUN ls -al
RUN mkdir /www/dockerfile/test
# 将主机中的指定文件复制到容器的目标位置
ADD /www/wolfcode.cn/index.html /www/server
ADD ["/www/wolfcode.cn/index.html", "/www/server"]
# 设置容器中的工作目录，如果该目录不存在，那么会自己创建
WORKDIR /app
# 在设置完工作目录后，执行pwd命令，打印的目录就是/app
RUN pwd
# 镜像数据卷绑定，将主机中的指定目录挂载到容器中
VOLUME ["/www/wolfcode.cn"]
# 设置容器启动后要暴露的端口
EXPOSE 8080
# 容器启动时执行的命令
# CMD
# ENTRYPOINT ping 127.0.0.1
ENTRYPOINT ["sh", "-c", "ping 127.0.0.1"]
```

springboot项目镜像配置

```shell
# 关联基础镜像 =>jdk
FROM openjdk:8
# 将项目 jar包加入到容器中
ADD *.jar /app.jar

# 配置项目环境变量
ENV APP_OPTS=""
# JVM 环境变量
ENV JVM_OPTS="-Duser.timezone=Asia/Shanghai -Xms128m -Xmx128m"

#暴露端口
EXPOSE 8888

# 设置启动时命令
ENTRYPOINT ["sh", "-c", "java $JVM_OPTS -jar /app.jar $APP_OPTS"]
```

springweb项目镜像配置

```shell
FROM tomcat:9.0

WORKDIR /usr/local/tomcat/webapps

ADD *.war ROOT.war

ENTRYPOINT ["sh", "-c", "../bin/catalina.sh run"]
```

nginx镜像配置

```shell
# 设置基础镜像
FROM centos:7
# 镜像的维护人
MAINTAINER npe

RUN yum -y install wget  gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel

WORKDIR /usr/local/src/
# 添加源文件（自动解压）
ADD nginx-1.6.3.tar.gz /usr/local/src

WORKDIR /usr/local/src/nginx-1.6.3

RUN ./configure --prefix=/usr/local/nginx-1.6 --with-pcre  --with-http_stub_status_module --with-http_ssl_module  --with-http_gzip_static_module --with-http_realip_module

RUN make && make install
# 以前台方式运行，挂起容器
RUN echo 'daemon off;' >> /usr/local/nginx-1.6/conf/nginx.conf

# 环境变量
# 将 /usr/local/nginx/sbin 添加到 PATH 环境变量的开头
ENV PATH /usr/local/nginx-1.6/sbin:$PATH

EXPOSE 80
# 添加自定义页面
ADD index.html /usr/local/nginx-1.6/html

CMD ["nginx"]
```

### 镜像仓库 Registry

- 作用

  - 实现快速交付，可以更方便在其他机器上下载镜像运行容器


  - 可以存储公司内部私有镜像，避免暴露到外网

  - 提升镜像下载速度

#### Docker Hub

#### Aliyun

阿里云私有仓库的拉取与上传

#### Nexus

docker方式搭建私有镜像仓库

```shell
docker run -d --restart=always -p 8868:8081 -p 5001:5001 -p5000:5000   --name nexus -v /opt/docker/nexus:/nexus-data sonatype/nexus3

cat /opt/docker/nexus/admin.password

docker login -u admin 192.168.200.158:5000
docker tag efa77f318784 192.168.200.158:5000/npenginx
#推送
docker push 192.168.200.158:5000/npenginx
#拉取
docker pull 192.168.200.158:5000/npenginx
```

#### Harbor

除了DockerHub以外最早的一个比较受欢迎的Docker企业级Registry服务器

### 容器编排

- 是什么？

对容器生命周期的管理，对容器的生命周期进行更快速方便的方式管理。

- 为什么需要？（能干什么？）
  - 依赖管理，当一个容器必须再另一个容器运行完成后，才能运行时，就需要进行依赖管理
  - 副本数控制，容器有时候也需要集群，快速地对容器集群进行弹性伸缩
  - 配置共享，通过配置文件统一描述需要运行的服务相关信息，自动化的解析配置内容，并构建对应的服务

#### docker-compose

单机环境的容器编排

- 安装


```shell
sudo curl -L "http://mirrors.aliyun.com/docker-toolbox/linux/compose/1.21.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

将可执行权限应用于二进制文件：

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

创建软链：

```shell
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

测试是否安装成功：

```shell
docker-compose --version
```

**注意**： 对于 alpine，需要以下依赖包： py-pip，python-dev，libffi-dev，openssl-dev，gcc，libc-dev，和 make。

- docker-compose.yml 配置文件

https://docs.docker.com/compose/compose-file/compose-file-v2/

示例：

```yaml
version: "2.1"
services:
  nginx-demo:
    image: "nginx"
    restart: "always"
    networks:
      - npe_net
    volumes:
      - nginx_vol-html:/usr/share/nginx/html
    environment:
      APP_ENV: dev
    dns:
      - 114.114.115.115
    ports:
      - 80:80

networks:
  npe_net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 188.18.0.0/16
          gateway: 188.18.0.1

volumes:
  nginx_vol-html:
```

- 常用命令


```shell
docker-compose up -d #更新容器配置并重启
docker-compose ps #列出
docker-compose create <service name> 
docker-compose scale <service name>=3 #弹性扩容缩容
```

#### Swarm

docker官方提供的容器编排项目，swarm可以在多个服务器或主机上创建多个容器集群服务



