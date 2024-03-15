#### 认识docker

##### 没有docker产生的问题

- 配置繁琐

环境不一致 ——两个应用需要的版本可能不一致

- 资源隔离

内存、cpu、网络带宽、磁盘相互影响，内存泄漏问题，软件出现bug

- 交付流程复杂 

手动安装业务服务、基础服务

##### 概念

docker是一种容器引擎，管理容器的生命周期

容器：用于运行一个软件的容器，该容器中准备了软件运行的一系列依赖

镜像：容器的安装包，容器中需要哪些内容，以及对应的配置信息，全部打包在里面，要运行一个容器，必须要有一个镜像

镜像仓库：存放各个镜像，对镜像进行统一管理

##### 容器与虚拟机区别

资源损耗、启动时间

容器共享操作系统，共享内核

虚拟机又新创建了一套操作系统和内核

##### docker架构

- docker client 客户端

- docker daemon

docker server：服务端，接受请求

engine：容器引擎，真正负责执行对应的任务

- docker registry

镜像仓库，存储镜像，用户相关信息

官方dockerhub

阿里云docker镜像仓库

#### docker安装

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

#### docker基本命令

- 镜像操作

```shell
docker search tomcat #从镜像仓库搜索镜像
docker version #版本详细信息
dockers -v 
docker pull openjdk:18#拉取镜像
docker images #查看本地镜像
docker rmi <IMAGE ID> #删除镜像
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

#### 数据卷

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



#### docker-compose

更新docker-compose

```sh
docker-compose up -d
```

列出所用docker下的容器

```shell
docker ps
docer-compose ps

docker images
```









