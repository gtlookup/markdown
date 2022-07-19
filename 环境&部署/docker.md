# 1 简介

https://hub.docker.com/  官方镜像，可以查到想用的镜像配置

## 1.1 虚拟化

一种资源管理技术。如：软硬件/内存/网络/桌面/服务虚拟化，以及虚拟机等待

## 1.2 什么是 docker

go 语言实现，2013年诞生的开源项目。实现轻量级操作系统虚拟化解决方案。**里面装着打包好的环境**

## 1.3 组件

### 1.3.1 服务器与客户端

docker 是一个 C/S 架构程序。客户端只需向服务器或者守护进程发出请求，服务器或守护进程将完成所有工作并返回结果。提供了一个命令行工具及一整套RESTful API。docker客户端和服务器可以在一台机器上也可以在不同机器上。

### 1.3.2 镜像与容器

- 镜像：用来运行容器的一组文件的集合。代表一个容器的==模板==
- 容器：基于镜像启动，一旦启动，就可以登到容器里安装软件或服务

### 1.3.3 注册中心

- 保存用户构建的镜像

- 分为公有/私有两种注册中心
- 在docker 公司运营的 docker hub 上注册账号并可以保存自己的镜像（巨慢，可构建私有的）



# 2 安装

```bash
# 1. yum 包更新到最新
sudo yum update
# 2. 安装需要的软件包
#    yum-util 提供 yum-config-manager 功能
#    device-mapper-persistent-data和lvm2 是驱动依赖
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 3. 设置阿里镜像
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 4. 安装 docker
#    docker-ce 为社区版（免费）
#    docker-ee 为企业版（收费）
sudo yum install docker-ce

# 如果 centos 8 执行上面报错，则执行下
dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
# 再执行安装 sudo yum install docker-ce
# 安装完后如果报 
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
# 这种错误，则把下面 2.1 的 daemon.json 改成 daemon.conf 再启动即可
```

## 2.1 设置 ustc 镜像

```bash
# 手动创建文件 /etc/docker/daemon.json
# 内容：
{"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]}
# 或者 https://vdw43ozc.mirror.aliyuncs.com
# 阿里云主页上搜 “容器镜像服务”，找镜像加速器
```

## 2.2 启动/停止

```bash
# 启动
systemctl start docker
# 查看运行状态
systemctl status docker
# 停止
systemctl stop docker
# 重启
systemctl restart docker
# 开机启动
systemctl enable docker
```



# 3 常用命令

```bash
docker -v     # 完成后查看版本
docker info   # 查看信息
docker stats  # 查看占CPU/内存情况
```

## 3.1 镜像相关

### 3.3.1 查看镜像

```bash
docker images
```

> **镜像存储在宿主机 /var/lib/docker 目录下**

- REPOSITORY：镜像名称
- TAG：镜像标签，当有同名镜像可用于存版本标识
- IMAGE_ID：镜像ID
- CREATED：镜像创建日期
- SIZE：镜像大小

### 3.1.2 搜索镜像

- 搜的是互联网上的镜像

```bash
docker search 镜像名 # 如 centos
```

- NAME：镜像名
- DESCRIPTION：镜像名
- STARTS：用户评价，反映受欢迎程度
- OFFICIAL：是否官方
- AUTOMATED：自动构建，表示由 docker hub 自动构建流程创建的

### 3.1.3 拉取镜像

```bash
# 默认拉到最新版本
docker pull 镜像名
```

### 3.1.4 删除镜像

```bash
# 属于这个镜像的容器全部停止运行才能删除该镜像
docker rmi 镜像名或id（最好给id）
```

### 3.1.5 删除所有镜像

```bash
docker rmi `docker images -q`
```

## 3.2 窗口相关

### 3.2.1 查看容器

```bash
# 查看当前正在运行的容器
docker ps
# 查看所有容器(含没运行的)
docker ps -a
# 查看最后一次运行的容器
docker ps -l
# 查看停止的容器
docker ps -f status-exited
```

### 3.2.2 创建/启动容器

```bash
# -i：表示运行容器
# -t：启动后进入命令行。-it：创建后就登进去，即分配个伪终端
# -d：创建一个守护式容器在后台运行。和 -t 区别在于创建后不进入命令行
# -v：目录映射关系
# -p：端口映射（方便通过宿主机端口操作容器内的软件）
# -e：环境配置
# --name：创建的容器起个名
docker run
```

- 创建交互式容器

```bash
# latest：是使用 docker images 后显示的 TAG 下的版本
docker run -it --name=mycentos centos:latest /bin/bash
# 退出容器终端，退出后容器也停止运行
exit
```

- 创建守护式容器

```bash
docker run -id --name=mycentos2 centos:latest
docker exec -it mycentos2 /bin/bash # 进入创建的 mycentos2 容器
docker exec -it -u root 容器名 bash  # 以 root 权限进入容器
exit                                # 退出后，容器仍然在运行
```

### 3.2.3 停止/启动容器

```bash
docker start 容器名或id  # 启动
docker stop 容器名或id   # 停止
```

### 3.2.4 文件拷贝

- **不管容器停止还是运行着都能拷贝**

```bash
# 将文件拷贝到容器
# docker cp 文件 容器名:容器目录
docker cp a.txt mycentos2:/usr
# 将文件拷出来
# docker cp 容器名:容器目录 拷贝到哪个路径
docker cp mycentos2:/usr/a.txt b.txt
```

### 3.2.5 目录挂载

- 将宿主机目录与容器目录映射，使容器和宿主可以共享同一个目录

```bash
# -v 宿主目录:容器目录
docker run -id --name=share -v /root/gt:/usr/gt centos:latest
```

### 3.2.6 查看容器信息

```bash
# 查看全部信息
docker inspect 容器名
# 查看某项信息
docker inspect --format='{{.NetworkSettings.SandboxKey}}' mycentos
```

### 3.2.7 删除容器

```bash
# 删除前先停掉，进行着的删不掉
docker rm 容器名
```

# 4 应用部署

## 4.1 mysql 部署

```bash
# 1. 拉取镜像
docker pull centos/mysql-57-centos7
# 2. 创建容器
#    -p 宿主端口:容器端口
#    -e MYSQL_ROOT_PASSWORD=123 设置root账号密码为123 
#    -- lower_case_table_names 让表名大小写不敏感
docker run -id --name=xxx -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123 centos/mysql-57-centos7 --lower_case_table_names=1
#	 mysql5.7
docker run -id --name=mysql5.7 -p 3306:3306 
		-v /data/mysql_dump_data:/dump_data 
		-e MYSQL_ROOT_PASSWORD=123456 
		-e MYSQL_LOWER_CASE_TABLE_NAMES=1 # 5.7是-e，而8是--
		centos/mysql-57-centos7 # 这个镜像
```

## 4.2 redis

```bash
# docker镜像的redis默认是无配置文件，因此
# 先复制一份redis.conf的内容到宿主机/root/docker/redis/redis.conf中
docker run -id --name=me-redis -p 6379:6379
		-v /root/docker/redis/conf:/etc/redis/conf   # 映射宿主:容器的配置文件目录
		-v /root/docker/redis/data:/data             # 映射宿主:容器的持久化目录
		redis                                        # 镜像名
		redis-server /etc/redis/conf/redis.conf      # 用配置文件启动（启动路径要和上面第4行的容器路径一致）
```

## 4.3 用完即删

```bash
docker run -it --rm tomcat:9.0 # 在没有 pull 情况下直接运行
# --rm 表示用完即删，当运行完后再停止，则该容器就不存在了；但镜像还在
```

## 4.4 可视化界面

docker 的可视化应用

```bash
docker run -d -p 8099:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
# 安装好后浏览器查看 http://xxx.xxx.xxx.xxx:8099
```

# 5 迁移与备份

## 5.1 将容器备份为镜像

```bash
docker commit 容器名 镜像名 -m="备注信息" -a="作者" 
```

## 5.2 镜像备份

- 将镜像保存为.tar文件

```bash
docker save -o 文件名.tar 镜像名
```

## 5.3 镜像恢复与迁移

- 删掉镜像后可以恢复

```bash
docker load -i 文件名.tar
```

# 6. Dockerfile

基础镜像：操作系统级别的镜像（如：centos，ubuntu）

由一系列命令和参数构成的脚本，这些命令应用于**基础镜像**并最终构建一个新的镜像。就是用来构建docker镜像的文件，一组命令参数的脚本

构建步骤：

1. 编写dockerfile文件
2. docker build 构建镜像
3. docker run 运行镜像
4. docker push 发布镜像（发布到 dockerhub、阿里云镜像仓库）

## 6.1 常用命令

| 命令                      | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| FROM image_name:tag       | 基于哪个基础镜像启动构建，一切从这里开始                     |
| MAINTAINER user_name      | 声明镜像的创建者**（可不写，用于版权声明）**                 |
| ENV key value             | 设置环境变量（可写多条）                                     |
| RUN command               | dockerfile 核心部分（可写多条）                              |
| ADD 源文件 目标文件       | 将宿主机文件复制到容器内。若是压缩文件，复制后可以自动解压   |
| COPY 源文件 目标文件      | 和 ADD 相似，但压缩文件并不解压                              |
| WORKDIR path_dir          | 设置工作目录（进到容器后默认在哪个目录下）                   |
| VOLUME ["目录1", "目录n"] | 挂载的目录                                                   |
| EXPOSE                    | 指定对外暴露的端口（跟 -P 一样）                             |
| CMD                       | 指定容器启动时要运行的命令，如：CMD echo "xxx"               |
| ENTRYPOINT                | 跟 CMD 一样，区别为：docker run [命令] 时，CMD 是替换，而 ENTRYPOINT 是在后面追加 |
| ONBUILD                   | 当构建一个被继承的 dockerfile 时，就会运行 ONBUILD 指令（只是一个触发指令） |
| ARG                       | `docker build` 时使用命令 `--build-arg <varname>=<value>` 将其传给ARG指定一个变量 |

## 6.2 构建一个 jdk8 环境

```bash
# 1. 创建一个 docker jdk8 目录
# 2. 将 jdk8 拷贝进来
# 3. 创建名称为 Dockerfile 的文件（名字可随意，建议Dockerfile），添加以下内容
FROM centos:7       # 基础镜像
MAINTAINER gtlookup # 作者
WORKDIR /usr        # 相当cd
CMD echo "--haha--" # 打印log
VOLUME ["dir1","dir2"] # 挂载两个目录，当该镜像启动后，ls会看到 dir1 和 dir2，但是是匿名挂载。通过 inspect 可查到对应宿主机的目录
RUN mkdir /usr/local/java            # 创建 java 文件夹
ADD OpenJDK8.tar.gz /usr/local/java/ # 添加附件
ENV JAVA_HOME /usr/local/java/jdk8u242-b08 # 往下都是设置环境变量
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$JRE_HOME/bin:$PATH
# 4. 打包
#    -f 根据哪个文件创建镜像
#    -t 镜像名
#    . 当前目录为打包目录
docker build -f ./Dockerfile -t='jdk1.8' .
```

## 6.3 构建redis

```bash
# 再来个简单的例子
# 1. 创建dockerfile文件dkf1
FROM redis
VOLUME ["dir01","dir02"]
CMD echo "--haha--"
# 2. build
docker build -f ./dkf1 -t 'dkf1' .
#    build结果：
Step 1/3 : FROM redis
 ---> bd571e6529f3
Step 2/3 : VOLUME ["dir01","dir02"]
 ---> Using cache
 ---> bb2ce5b1e84c
Step 3/3 : CMD echo "--haha--"
 ---> Using cache
 ---> 92659b8c3915
Successfully built 92659b8c3915
Successfully tagged dkf1:latest
# 3. docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
dkf1                      latest              92659b8c3915        2 minutes ago       104MB
# 4. 运行并进入到容器后 ls，发现 dir01 和 dir02
bin  boot  data  dev  dir01  dir02  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

## 6.4 多容器共用目录

```bash
# 接上一个dkf1镜像
docker run -it --name r1 dkf1 /bin/bash                    # 先运行一个容器（父容器）
docker run -it --name r2 --volumes-from r1 dkf1 /bin/bash  # 再根据r1运行一个子容器（将r2挂载到r1上） --volumes-from r1
# 此时这两个容器就可以共享目录了。此时删除r1（父）容器，dir01和dir02依旧存在宿主中，直到没有容器再使用该共享目录为止
```

## 6.5 构建springboot

```bash
# 1. 先参照 `Spring Boot.md 的 8.3` 创建一个能直接跑的jar包
# 2. 将jar包上传centos，然后创建Dockerfile
FROM java:8             # 基于java8镜像
ADD app.jar /app.jar    # 将centos上的jar包拷到容器里
EXPOSE 8080             # 暴露8080端口（jar包的代码里没添加application.yml，所以默认是8080端口）
CMD java -jar /app.jar  # docker run 时要执行的命令
# 3. build 镜像
docker build -f ./Dockerfile -t='springboot' . # 最后那个 . 别忘写
# 4. 运行
docker run -id --name app -p 80:8080 springboot
# 5. 访问
http://47.105.141.18/ # 页面显示 hello world
```

> ==坑一：==
>
> 当 **docker build** 后发现：`Sending build context to Docker daemon  2.571GB`，怎么这么大？
>
> 原来 jar 包和 Dockerfile 所在的目录还有其它文件，这时 build 的话会全部发送到 daemon 里。
>
> 解决：新建个空目录，将 jar 和 Dockerfile 拷进去，然后再 build，OK 正常了
>
> 结果显示：`Sending build context to Docker daemon  17.31MB`

> ==坑二：==
>
> 当 **docker run** 时报 `-p tcp -d 0/0 --dport 8080 -j DNAT --to-destination 172.17.0.4:8080 ! -i docker0: iptables: No chain/target/match by that name.` 错
>
> 解决：**systemctl restart docker** 重启下再 run 就好了

## 6.6 构建rust

https://dev.to/rogertorres/first-steps-with-docker-rust-30oi

https://kerkour.com/rust-small-docker-image/

### hello world

一个最简单的构建，没有任何外部依赖库那种。因为 `FROM rust:latest` 的容器上不去网，无法下载依赖库

```bash
cargo new hello # 创建新项目
cd hello
touch Dockerfile # 创建docker文件，内容如下：
```

```dockerfile
FROM rust:latest
COPY ./ ./

RUN cargo build --release
CMD ["./target/release/poem-demo"]
```

```bash
docker build -t hello . # 创建镜像
```



# 7 私有仓库

相当于 svn git 服务器

## 7.1 创建私仓

```bash
# 1. 拉到私有仓库镜像
docker pull registry
# 2. 启动
docker run -id --name=registry -p 5000:5000 registry
# 3. 浏览器访问 http://192.168.2.13:5000/v2/_catalog
#    结果：{"repositories":[]}
# 4. vi /etc/docker/daemon.json
#    添加内容
{"insecure-registries":["192.168.2.13:5000"]}
# 5. 重启docker
systemctl restart docker
```

## 7.2 上传到私仓

```bash
# docker tag 镜像名 私服ip:端口/镜像名
docker tag jdk1.8 192.168.2.13:5000/jdk1.8
# docker push 私服ip:端口/镜像名
docker push 192.168.2.13:5000/jdk1.8

# 再次浏览器访问 http://192.168.2.13:5000/v2/_catalog
# 结果：{"repositories":[jdk1.8]}
```

# 8. 容器安所卷

当数据在容器中，如果容器删除，那么数据也丢失了。==需求：数据持久化==。比如mysql容器删了，还有本地存储的数据；即容器之间可以数据共享。

再直白说是将容器中的目录挂载到本地（多个容器可以共用一个本地目录）。==参考：3.2.5==

## 8.1 匿名挂载

```bash
docker volume ls # 查看所有卷的情况
# 显示结果：下面这些都是匿名卷，在挂载（-v）的时候没有指定容器外的路径
# DRIVER              VOLUME NAME
# local               1b97356e098174884bb2ff4d784e5f4042245b989cc4416caefe3a176d86d798
# local               2f1082849d4a3236bbbd6ee45524787b21d5b1e2d9a1c49c2ca6fc01e6a8d4bd
# local               6ef5a21765293437172b991978a84e363d08488b77267f1149ad5baaf1eb442d
```

## 8.2 具名挂载

```bash
docker run -d --name redis1 -v dir-redis:/etc/redis redis # 创建一个具名挂载的容器，-v 具名:容器内路径
# 显示结果：
# DRIVER              VOLUME NAME
# local               dec3a7baf4d17d2841e3e646e878fa7e64617e6fd386969ec6ccc65110b95e4d
# local               dir-redis      # 具名了
docker volume inspect dir-redis      # 查看具名的具体位置
#[
#    {
#        "CreatedAt": "2021-05-30T09:26:50+08:00",
#        "Driver": "local",
#        "Labels": null,
#        "Mountpoint": "/var/lib/docker/volumes/dir-redis/_data",    # 挂载到本地的路径
#        "Name": "dir-redis",
#        "Options": null,
#        "Scope": "local"
#    }
#]
```

==建议使用具名挂载==。-v 三种形态：

```bash
-v 容器内路径                  # 匿名挂载
-v 卷名:容器内路径              # 具名挂载
-v /宿主机路径/:容器内路径       # 指定路径挂载
```

## 8.3 读写权限

```bash
# ro：代表只读 -> readonly。一旦设置只读，只能在宿主机上修改，不能在容器中修改
# rw：可读可写 -> readwrite（默认）
docker run -d --name redis1 -v dir-redis:/etc/redis:ro redis # ro
docker run -d --name redis1 -v dir-redis:/etc/redis:rw redis # rw
```



# 9. 填坑

## 1. Error OCI runtime

> 当docker start me-mysql时报错：Error response from daemon: OCI runtime create failed: container with id exists: 01283448be87e7e18289d2f003aba51e21362872f34e67e1a2b634908b922c12: unknown
> Error: failed to start containers: me-mysql

```bash
find / -name 01283448be87e7e18289d2f003aba51e21362872f34e67e1a2b634908b922c12
# 结果：/run/docker/runtime-runc/moby/01283448be87e7e18289d2f003aba51e21362872f34e67e1a2b634908b922c12
rm -rf /run/docker/runtime-runc/moby/01283448be87e7e18289d2f003aba51e21362872f34e67e1a2b634908b922c12 # 删除
docker ps -a # 查看容器的id
docker start 01283448be87 # 用id启动
```

## 2. 停不了

> 当执行 systemctl stop docker 时报：`Warning: Stopping docker.service, but it can still be activated by: docker.socket`

```bash
systemctl stop docker.socket
```

## 3. docker磁盘满了

```bash
[root@1_231 data]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 5.8G     0  5.8G    0% /dev
tmpfs                    5.8G     0  5.8G    0% /dev/shm
tmpfs                    5.8G   26M  5.8G    1% /run
tmpfs                    5.8G     0  5.8G    0% /sys/fs/cgroup
/dev/mapper/centos-root   50G   50G   13M  100% /           # df -h 发现满了
/dev/sda2               1014M  221M  794M   22% /boot
/dev/sda1                200M   12M  189M    6% /boot/efi
/dev/mapper/centos-home   55G   33M   55G    1% /home
tmpfs                    1.2G     0  1.2G    0% /run/user/0
/dev/sdb                 459G   24G  412G    6% /data       # 而 /data 够大，那么就把 docker 盘符改到这里

[root@1_231 data]# docker info
...
Server:
 ...
 Docker Root Dir: /var/lib/docker # 查看到 docker 存储位置
 ...
```

```bash
systemctl stop docker       # 1. 先停止
mkdir /data/docker          # 2. 创建个目录给docker用
vim /etc/docker/daemon.json # 3. 修改docker存储目录
{
  "registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],
  "graph": "/data/docker"   # 4. 加上这句
}
systemctl start docker      # 5. 启动
docker info                 # 6. 查看最新存储目录
Server:
 Docker Root Dir: /data/docker # 查看到 docker 存储位置
```

# 10 离线安装

https://www.cnblogs.com/helf/p/12889955.html

```bash
wget https://download.docker.com/linux/static/stable/x86_64/docker-19.03.8.tgz # 1. 下载
tar -zxvf docker-19.03.8.tgz # 2. 解压
cp docker/* /usr/bin/        # 3. 拷贝到 /usr/bin 目录下

# 4. 创建/etc/systemd/system/docker.service
cd /etc/systemd/system/
touch docker.service
# 内容为：
```

==注意：--insecure-registry=192.168.200.128 此处改为你自己服务器ip==

```bash
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd --selinux-enabled=false --insecure-registry=192.168.200.128
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

```bash
chmod 777 /etc/systemd/system/docker.service # 5. 加上可执行权限
systemctl daemon-reload # 6. 重新加载配置文件（每次修改 docker.service 时都要重新加载）
systemctl start docker  # 7. 启动
```

# 11. manifest

https://www.cnblogs.com/nhdlb/p/15233410.html

当创建容器时，该容器的镜像必须与宿主机架构一致（如：x86只能创建x86、arm只能创建arm）

manifest 特性支持用户在不同系统架构机器上分别运行不同架构的镜像

## 11.1 开启 manifest

```bash
# 1. 添加 config.json 文件
mkdir /root/.docker & cd .docker
vim config.json # 添加如下内容：
{
  "auth": {},
  "experimental": "enabled"   
}

# 2. 编辑 daemon.json 文件
vim /etc/docker/daemon.json # 添加如下内容：
{
	"registry-mirrors":["https://docker.mirrors.ustc.edu.cn"],
	"experimental": true # 关键添加这个
}

# 3. 重新加载服务配置文件
systemctl daemon-reload # ubuntu 下 systemctl 也能用
# 4. 重启 docker
systemctl restart docker
# 5. 查看是否开启 experimental 功能
Client: Docker Engine - Community
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.12
 Git commit:        e91ed57
 Built:             Mon Dec 13 11:45:33 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true # 表示已开启

Server: Docker Engine - Community
 Engine:
  Version:          20.10.12
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.12
  Git commit:       459d0df
  Built:            Mon Dec 13 11:43:42 2021
  OS/Arch:          linux/amd64
  Experimental:     true # 表示已开启
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## 11.2 运行 arm 容器

```bash
# 1. 安装 install qemu-user-static
#    x86 无法运行 arm 平台程序，但是 qemu 提供了机制，通过 qemu-arm-static 可以达到目的
apt install qemu-arm-static

# 2. 拉取一个arm架构的centos
docker pull --platform=arm64 centos:7

# 3. 运行容器
#    -v /usr/bin/qemu-arm-static：需要挂载 qemu 才能在 x86 上运行 arm
docker run -id --name=arm_c7 -v /usr/bin/qemu-arm-static:/usr/bin/qemu-arm-static centos:7

# 4. 进入容器后通过 arch 查看系统架构为：aarch64
```



# 附录

```bash
FROM scratch # centos7以它为基础镜像，99%的镜像都以它为基础镜像
```



https://www.bilibili.com/video/BV1og4y1q7M4?p=29&spm_id_from=pageDriver
