# 收藏

```bash
https://kubernetes.io/zh/  # 官网
https://kubernetes.io/zh/docs/home/ # 官方文档
https://kuboard.cn/learning/ # 教程（概念全）
https://kubernetes.org.cn/installkubectl # 中文文档
```

# 概念

是谷歌2014年开源的容器化==集群管理系统==。用于容器化应用部署，每个容器间相互隔离、有各自的文件系统、进程不相互影响。比docker功能更多

# 特性

1）自动装箱：对应用程序的运行环境和资源配置，进行自动部署

2）自我修复：

- 当容器失败时，会对容器进行重启
- 当部署的节点有问题时，会对容器进行重新部署和重新调度
- 当容器未通过监控检查时，会关闭此容器，直到容器能正常运行，才会对外提供服务

3）水平扩展：通过简单的命令、用户UI界面或基于CPU等资源使用情况，对容器进行扩大或缩小

4）服务发现：用户不需使用额外的服务发现机制，就能基于k8s自身能力实现服务发现和负载均衡

5）滚动更新：根据应用变化，对容器进行一次性或批量更新

6）版本回退：根据应用部署情况，进行历史版本即时回退

7）密钥和配置管理：应用配置、部署和更新密钥，不用重新构建镜像，类似热部署

8）存储编排：自动实现数据（存储系统etcd）挂载及应用，对有应用数据持久化非常重要。数据（etcd）可来自本地目录、网络（NFS、Cluster、Ceph等）、公共云存储服务

9）批处理：提供一次性任务，定时任务；满足批量数据处理和分析场景

# 集群架构组件

master：主控节点，用于管理工作节点

- API Server：集群对外的统一入口，用于接收请求（RESTFul），然后交给etcd存储
- scheduler：节点调度（选择工作节点部署，从多个 work node 节点的组件中选举一个来启动服务）
- controller-manager：向 work 节点的 kubelet 发送指令
- etcd：存储系统。存注册节点、服务、记录账号

work node：工作节点，用来做具体工作的节点

- kubelet：理解为 master 派来的领导，用于管理本机容器的操作
- kube-proxy：提供网络代理，如负载均衡等操作
- docker 或 rocket：容器引擎，运行容器

# 核心

1）Pod：

- 最小的部署单元，一组容器的集合。
- 一个 pod 中可以有一个或多个容器，又叫容器组、一级容器的集合
- 一个Pod中的容器是共享网络的。生命周期是短暂的（重新部署后就是一个新的Pod）

2）Controller：

- ReplicaSet：确保预期的Pod副本数量
- Deployment：无状态应用部署，拿来直接用，没有任何限制
- StatefulSet：有状态应用部署，需要特定条件才可使用，如：依赖存储或网络ip
- DaemonSet：确保所有node运行同一个Pod（用的不多）
- Job：一次性任务
- Cronjob：定时任务

3）Service：将一组Pod关联起来，提供一个统一的访问入口。即使pod地址发生改变也不会影响访问

4）Label：标签。一组pod有统一的标签，Service通过标签对一组pod进行关联的

5）Namespace：名称空间，用来隔离pod的运行环境，默认情况下pod是可以互相访问的

- 比如dev、test、prod三个环境是不应该互相访问的

# 运行流程

比如用户开启一个nginx服务，从发送指令到服务启动，流程如下：

1）用户通过 master.kubectl 发送指令给 master.apiserver

2）master.apiserver 通过 etcd 做 auth 认证，看用户是否有权限

3）如果有权限，master.apiserver 发送指令给 master.scheduler，然后去 etcd 里找可用的 work node 的 ip

4）ip 发给 controller-manager，然后通过 node.kubelet 调用 node.docker 启动一个具体pod

5）node.kube-proxy 给刚启动 pod 分配一个 ip，这样外部就可以访问了

# 环境搭建

## 1. 规划

1）单 master 集群：一个 master 节点管理多个 node 节点。缺点是当 master 挂了，就不能再管理 node 节点了。

2）多 master 集群：多个 master 管理多个 node，但在 master 和 node 之间多了一层负载均衡

## 2. 硬件要求

master：2核/4G/20G +，生产环境要求更高

node：   4核/8G/40G +，生产环境要求更高

## 3. 两种部署方式

kubeadm 部署门槛低，但屏蔽了很多细节，遇到问题难排查。二进制部署虽然麻烦点，但能学到工作原理，利于后期维护。

> 要禁止 swap 分区。==free -h== 可以查看

### 3.1 kubeadm

是一个k8s部署工具，提供 kubeadm init 和 kubeadm join 两个命令，用于==快速部署==k8s集群。

https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/  官网kubeadm

#### 3.1.1 事先准备

1）准备三台 centos7 虚拟机，分别叫 master、node1、node2

2）在每台上执行 linux 命令（临时的也要执行，不执行本次开机就没关）

```bash
# 1. 关防火墙
systemctl disable firewalld # 永久关，stop 是临时关

# 2. 关闭 selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config # 永久关
setenforce 0 # 临时关

# 3. 关闭 swap
swapoff -a # 临时关
sed -ri 's/.*swap.*/#&/' /etc/fstab # 永久关

# 4. 根据规划设置主机名
hostnamectl set-hostname 名称 # 设置完后通过 hostname 查看新名

# 5. 在 hosts 里添加 名称/ip（只在master里执行）
cat >> /etc/hosts << EOF
192.168.1.103 master
192.168.1.105 node1
192.168.1.106 node2
EOF

# 6. 将桥接的 IPv4 流量传递到 iptables（可理解为网络防火墙）
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system # 使 6 生效

# 7. 时间同步（三台系统时间都相同）
yum install ntpdate -y
ntpdate time.windows.com
```

#### 3.1.2 环境安装

三台机器都要安

1）添加阿里云 yum 源

```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

2）docker 安装，参照 docker.md

3）安装 kubeadm、kubelet、kubectl

```bash
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
systemctl enable kubelet # 设置开机启动
```

 4）部署 master

```bash
kubeadm init --apiserver-advertise-address=192.168.1.103                # 当前master的ip
			 --image-repository registry.aliyuncs.com/google_containers # 阿里镜像
			 --kubernetes-version v1.18.0                               # 当前版本
			 --service-cidr=10.96.0.0/12
			 --pod-network-cidr=10.244.0.0/16
# 此时 docker images 会看到正在拉取镜像
```

```bash
# init 成功后提示：
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.103:6443 --token 1ut0o0.fb9mjxf7hrvrcqfs \
    --discovery-token-ca-cert-hash sha256:80628d1c6f5ebc798026b97456ae04a1acf9a10288cbfdf3529330e870b448e3 
```

```bash
# 将上面成功信息复制出来一段执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config    # 此上三行在 master 里运行
kubectl get nodes # 查看节点，此时会有一个 master 节点

kubeadm join 192.168.1.103:6443 --token 1ut0o0.fb9mjxf7hrvrcqfs --discovery-token-ca-cert-hash sha256:80628d1c6f5ebc798026b97456ae04a1acf9a10288cbfdf3529330e870b448e3 # 在node节点上执行，表示把node节点加到master里
kubectl get nodes # 此时再查看节点，发现另两个node节点也显示了
```

```bash
# 默认token有效期为24小时，过期后需要重新创建
kubeadm token create --print-join-command
```

5）部署CNI网络插件

```bash
[root@192 ~] kubectl get nodes
NAME     STATUS     ROLES    AGE    VERSION
master   NotReady   master   8m1s   v1.18.0
node1    NotReady   <none>   10s    v1.18.0
node2    NotReady   <none>   4s     v1.18.0 # NotRead 说明缺少网络插件
```

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 此时会报：The connection to the server raw.githubusercontent.com was refused - did you specify the right host or port?
# 1. 到 http://ip.tool.chinaz.com/raw.githubusercontent.com 上查 raw.githubusercontent.com 的 ip，结果：185.199.110.133
# 2. 将 185.199.110.133 raw.githubusercontent.com 加到 /etc/hosts 文件里，再次执行报错那句，结果OK

kubectl get pods -n kube-system # 此时发现状态都是 Running
kubectl get nodes # 此时再查看节点状态就由 NotReady 变成了 Ready
```

6）测试k8s集群网络

```bash
# 在 k8s 集群中创建一个 pod，验证是否正常运行：
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc # 查询状态
NAME                        READY   STATUS              RESTARTS   AGE
pod/nginx-f89759699-7fmrp   0/1     ContainerCreating   0          25s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        45m
service/nginx        NodePort    10.109.58.38   <none>        80:30270/TCP   10s # 商品30270

# 浏览器访问3个节点都能正常打开nginx默认页面
http://192.168.1.103:30270/  # master节点
http://192.168.1.105:30270/  # node1节点
http://192.168.1.106:30270/  # node2节点
```



### 3.2 二进制包

从 github 下载发行版二进制包，手动部署每个组件，组成 k8s 集群



# 命令行工具kubectl

是 k8s 集群命令行工具，通过 kubectl 对集群本身进行管理，并在集群上进行容器化应用的安装部署

```bash
# kubectl 语法
kubectl [command] [type] [name] [flags]
# command：指对资源执行的操作，如 create、get、describe、delete
# type：指定资源类型，资源类型是大小写敏感的，开发者能以单数、复数和缩略的形式，如：
kubectl get pod pod1
kubectl get pods pod1
kubectl get po pod1
# name：资源名称，也大小写敏感。若省略名称则显示所有资源，如：
kubectl get pods
# flags：可选参数。如，-s 或 -server 指定 k8s api server 的地址和端口
```

```bash
kubectl --help # help

Basic Commands (Beginner):
  create        通过文件或标准输入创建资源
  expose        使用 replication controller, service, deployment, pod 对外暴露一个Service
Service
  run           在集群中运行一个指定的镜像
  set           为 objects 设置一个指定的特征
Basic Commands (Intermediate):
  explain       查看资源的文档
  get           显示一个或更多 resources
  edit          在服务器上编辑一个资源
  delete        通过文件名、输入、资源名、标签选择器来删除资源
Deploy Commands:
  rollout       Manage the rollout of a resource
  scale         Set a new size for a Deployment, ReplicaSet or Replication Controller
  autoscale     自动调整一个 Deployment, ReplicaSet, 或者 ReplicationController 的副本数量
Cluster Management Commands:
  certificate   修改 certificate 资源.
  cluster-info  显示集群信息
  top           Display Resource (CPU/Memory/Storage) usage.
  cordon        标记 node 为 unschedulable
  uncordon      标记 node 为 schedulable
  drain         Drain node in preparation for maintenance
  taint         更新一个或者多个 node 上的 taints
Troubleshooting and Debugging Commands:
  describe      显示一个指定 resource 或者 group 的 resources 详情
  logs          输出容器在 pod 中的日志
  attach        Attach 到一个运行中的 container
  exec          在一个 container 中执行一个命令
  port-forward  Forward one or more local ports to a pod
  proxy         运行一个 proxy 到 Kubernetes API server
  cp            复制 files 和 directories 到 containers 和从容器中复制 files 和 directories.
  auth          Inspect authorization
Advanced Commands:
  diff          Diff live version against would-be applied version
  apply         通过文件名或标准输入流(stdin)对资源进行配置
  patch         使用 strategic merge patch 更新一个资源的 field(s)
  replace       通过 filename 或者 stdin替换一个资源
  wait          Experimental: Wait for a specific condition on one or many resources.
  convert       在不同的 API versions 转换配置文件
  kustomize     Build a kustomization target from a directory or a remote url.
Settings Commands:
  label         更新在这个资源上的 labels
  annotate      更新一个资源的注解
  completion    Output shell completion code for the specified shell (bash or zsh)
Other Commands:
  alpha         Commands for features in alpha
  api-resources Print the supported API resources on the server
  api-versions  Print the supported API versions on the server, in the form of "group/version"
  config        修改 kubeconfig 文件
  plugin        Provides utilities for interacting with plugins.
  version       输出 client 和 server 的版本信息
Usage:
  kubectl [flags] [options]
Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

## 1. 常用命令

```bash
kubectl config view # 查看kubeconfig配置内容
```

## 2. window10安装

下载：https://storage.googleapis.com/kubernetes-release/release/v1.16.2/bin/windows/amd64/kubectl.exe

后放到 C:\Windows\System32 下

# 集群yaml文件

k8s 对资源的管理、编排部署都可以通过 yaml 文件来解决

## 1. yaml 组成部分

```yaml
# 第1部分：控制器定义
apiVersion: apps/v1               # api版本，通过 kubectl api-version 查看
kind: Deployment                  # 资源类型
metadata:                         # 资源元数据
  name: nginx-deployment
  namespace: default
spec:                             # 资源规格
  replicas: 3                     # 副本数量
  selector:                       # 标签选择器
    matchLabels:
      app: nginx
# 第2部分：被控制的对象
template:                         # pod 模板
  metadata:                       # pod 元数据
    labels:
      app: nginx
    spec:                         # pod 规格
      containers:                 # 容器配置
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

## 2. 快速生成 yaml 文件

从头写不现实，可以通过 kubectl 命令快速生成 yaml 文件

> 第一种生成方式：

```bash
#	web --image=nginx：生成一个名叫 web 的 nginx
#	-o yaml：显示 yaml 文件内容
#	--dry-run：不去真正执行
kubectl create deployment web --image=nginx -o yaml --dry-run
# 结果显示：
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
#	加上 > a.yaml 表示把上面内容生成到当前目录下的 a.yaml 里
kubectl create deployment web --image=nginx -o yaml --dry-run > a.yaml
```

> 第二种生成方式：适用于已有的部署项目，就是给导出来

```bash
kubectl get deploy # 查看已部署的项目，假如有个 name=web 的
kubectl get deploy web -o=yaml --export > a.yaml # 导出到当前目录，名为 a.yaml
```



# 附录

## 1. master的ip变了

此时需要重置

```bash
# 1. 修改 /etc/hosts 将master的新ip改了
# 2. 重置
kubeadm reset
# 3. 重新初始化，即 3.1.2.4)。
#	 注意，--apiserver-advertise-address=master的新ip
# 4. 重置node节点
kubeadm reset
kubeadm join 主节点重置成功后这里显示的ip:port --token ...
# 5. 如果发现
kubectl get pods -n kube-system
NAME                             READY   STATUS              RESTARTS   AGE
coredns-7ff77c879f-g84vz         0/1     ContainerCreating   0          27m
coredns-7ff77c879f-jnjz9         0/1     ContainerCreating   0          27m # 这两个异常
#	那么再执行 3.1.2.5)
#	这个异常会导致创建个nginx后，无法访问
```

## 2. 查看pod错误信息

```bash
kubectl get pods # 查看pod名
NAME                   READY   STATUS              RESTARTS   AGE
web-5dcb957ccc-w4cdc   0/1     ContainerCreating   0          30m

kubectl describe pod web-5dcb957ccc-w4cdc # 显示该pod错误信息
```





https://www.bilibili.com/video/BV1GT4y1A756?p=22&spm_id_from=pageDriver