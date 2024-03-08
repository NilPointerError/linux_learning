#### nginx简介

##### 1、nginx是什么？

1.nginx是一个高性能的http和反向代理服务器，特点是占有内存少，并发能力强
2.nginx专门为性能优化开发，能承担最大并发连接数50000

##### 2、反向代理

1.正向代理
在客户端（浏览器）配置代理服务器（比如clash监听7890端口），通过代理服务器来访问
2.反向代理
客户端将请求发送到反向代理服务器，由反向代理服务器转发到目标服务器，再从目标服务器返回数据给客户端。反向代理服务器和目标服务器对外是一个服务器，从而隐藏了真实服务器ip地址

![image-20240305104359903](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240305104359903.png)

##### 3、负载均衡

增加服务器数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发（平均分发？）到不同的服务器，就是负载均衡

![image-20240305104454776](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240305104454776.png)

##### 4、动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析。nginx处理静态页面，tomcat处理动态页面

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240305104208336.png" alt="image-20240305104208336"  />

#### nginx安装

版本：nginx-1.6.3

```shell
yum -y install gcc gcc-c++ make libtool zlib zlib-devel openssl openssl-devel pcre pcre-devel  #安装依赖包
cd /usr/local/src/
wget https://nginx.org/download/nginx-1.6.3.tar.gz #下载压缩包 
tar -xzvf nginx-1.6.3.tar.gz #解压
./configure --prefix=/usr/local/nginx-1.6 --with-pcre \
> --with-http_stub_status_module --with-http_ssl_module \
> --with-http_gzip_static_module --with-http_realip_module \
#编译
make && make install #安装
```

安装成功后，多出来文件夹/usr/local/nginx，nginx/sbin有启动脚本

#### nginx常用命令

```shell
cd /usr/local/nginx/sbin

./nginx #启动nginx
./nginx  -v #查看版本号
./nginx -s stop #关闭nginx
./nginx -s reload #重新加载nginx
```

#### nginx配置文件

路径：/usr/local/nginx-1.6/conf/nginx.conf

1、全局块

设置一些影响nginx服务器整体运行的配置指令

work_processes：支持的并发处理量

2、events块

影响nginx服务器与用户的网络连接

work_connections：支持的最大连接数

3、http块

nginx配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里

http全局块 、server块



#### 反向代理

apache tomcat下载解压完后 

进入bin目录./startup.sh启动tomcat服务器

1、需求：访问www.123.com实际上访问到tomcat网站

![image-20240305160228837](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240305160228837.png)

注意：只需要开放80端口，不需要开放8080端口

配置信息：

![image-20240307153536983](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240307153536983.png)

2、需求：不同的路径转发到不同的端口

访问192.168.17.129:9001/edu/ 跳转到 127.0.0.1:8080/edu/

访问192.168.17.129:9001/vod/跳转到 127.0.0.1:8081/vod/

配置信息：

![image-20240307153440273](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240307153440273.png)





**扩展：location后不同标识的作用？**

#### 负载均衡

刷新时实现不同端口的跳转

配置信息：

![image-20240307160824714](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240307160824714.png)



**扩展：**

nginx分配服务器策略：

1、轮询（默认）

每个请求按照时间顺序依次分配到不同的服务器，如果服务器down掉，则自动剔除

2、weight

weight代表权重默认为1，权重越高被分配的客户端越多

3、ip_hash

每个请求按访问的ip 的hash结果分配，这样每个访客固定访问一个服务器，可以解决session问题

4、fair（第三方）

根据服务器响应时间分配请求，响应时间短的优先分配

#### 动静分离

1、把静态文件独立成单独的域名，放在独立的服务器上

2、动态和静态文件混合在一起发布，通过nginx分开发布

配置信息：

<img src="C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308135314669.png" alt="image-20240308135314669" style="zoom: 67%;" />





tomcat：

tomcat是一种web应用服务器，可以响应html页面的请求，它还是一个servlet和jsp容器



#### 高可用的集群

##### 主备模式

![image-20240308170632787](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308170632787.png)

准备两台服务器，需要安装nginx、keepalived

配置信息：

/etc/keepalived.conf

主：

![image-20240308171216317](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308171216317.png)

从：

![image-20240308171115822](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308171115822.png)

脚本文件：

/usr/local/src

不能启动切换到另一个服务器

![image-20240308171252104](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308171252104.png)

#### nginx的原理解析

![image-20240308173826367](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308173826367.png)

![image-20240308174202710](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20240308174202710.png)



##### 一个master和多个worker的好处

1、可以使用nginx -s reload 热部署，利用nginx进行热部署操作（无需停掉nginx服务即可更改配置）

2、每个worker是独立的进程，如果有其中的一个woker出现问题，其他worker独立的，继续进行争抢，实现请求过程，不会造成服务中断

##### 设置多少个worker合适

worker数和服务器的cpu数相等是最为适宜的

##### 连接数work_connection

1、发送请求，占用了worker的几个连接数？

2个或4个

2、nginx有一个master，有四个worker，每个worker支持最大连接数据(work_connection)1024，支持的最大并发数是多少？

普通的静态访问最大并发数：work_connection*work_processes/2;

http作反向代理，最大并发数是work_connection*work_processes/4;

