> ## k8s

#### 1、概念

- k8s是开源的容器化集群管理系统
- 使用k8s进行容器化应用部署
- 使用k8s利于应用扩展
- k8s目标是让部署容器化应用更加简洁和高效

#### 特性

1. 自动装箱
2. 自我修复
3. 水平扩展
4. 服务发现（负载均衡）
5. 滚动更新
6. 版本回退
7. 密钥和配置管理
8. 存储编排（管理虚拟磁盘 ）
9. 批处理

#### 集群架构组件



**说明：**

master（主控节点）和node（工作节点）

（1）master组件

apiserver

集群统一入口，以restful方式，交给etcd存储

scheduler

节点调度，选择node节点应用部署

controller-manager

处理集群中常规后台任务，一个资源对应一个控制器

etcd

存储系统，用于保存集群相关的数据

Controller

- 确保预期的pod副本数量
- 无状态应用部署（不会对本机环境产生任何依赖，例如不会存储数据到本地磁盘）
- 有状态应用部署（会对本地环境产生依赖，例如会存储数据到本地磁盘）
- 确保所有的node运行同一个pod
- 一次性任务和定时任务

3、Service

- 定义一组pod的访问规则





（2）node组件

kubelet

负责Pod的生命周期、存储、网络

master排到node节点代表，管理本机容器

kube-proxy

提供网络代理，负责Service的服务发现负载均衡（4层负载）

container-runtime

容器运行时环境：docker、containerd、CRI-O

Pod 

- 最小的部署资源单元
- 一组容器的集合
- 共享网络
- 生命周期是短暂的



Pod控制器

ReplicationController：帮助我们动态更新Pod的副本数

RelicaSet：帮助我们动态更新Pod的副本数，可以通过selector来选择对哪些Pod生效

Deployment：针对RS的更高级封装，提供了更丰富的部署相关的功能

SatefulSet：专门针对于有状态服务进行部署的一个控制器

DaemonSet：为每一个匹配的Node都部署一个守护进程



Service：解决k8s集群内部网络调用、负载均衡（四层负载）（节点 与节点之间）

Ingress：实现将k8s内部服务暴露给外网访问的服务

Ingress-nginx：反向代理、负载均衡（七层负载）

东西流量（横向流量）、南北流量（纵向流量）



Volume：共享数据，类似docker中的存储卷（puv，pvc）

CSI：制定的一个行业标准接口规范

ConfigMap：解决配置文件在容器固定的问题（明文存储）

Secret：加密

DownwardAPI：把Pod信息共享到容器内

Role-->ClusterRole

RoleBinding-->ClusterRoleBinding

#### 核心概念

资源和对象

资源的分类：元空间、集群、命名空间

命名空间级别的资源：作用在命名空间之上，通常只能在该命名空间范围内使用

集群级别的资源：作用与集群之上，集群下的所有资源都可以共享使用

元数据级别的资源：对于资源的元数据描述，每一个资源都可以使用元数据的数据







#### 搭建

环境要求：

（1）基于客户端工具kubeadm

提供kubeadm init和kubeadm join，用于快速部署kubernetes集群

（2）基于二进制包方式

从github下载，手动部署，虽然麻烦，但是可以排查问题



```shell
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6
 cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
kubelet --version
kubectl get nodes
kubeadm token list
kubeadm token create
journalctl -xeu kubelet


kubeadm join 192.168.150.134:6443 --token hm6jfh.tn62fuubrd6f7vrm --discovery-token-ca-cert-hash \
sha256:8e94801d5659fe4fea2b9686ac664d1f051946831765bb3a56104ae583a0b7f0


 cat /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]

}

```





#### 统一日志管理

#### 性能监控平台

#### 部署项目 