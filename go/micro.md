# 周边

## 1. Raft

consoul 和 etcd 的核心算法

https://www.topgoer.com/%E5%BE%AE%E6%9C%8D%E5%8A%A1/Raft.html

## 2. 云原生

一种构建和运行应用程序的方法，是一套技术体系和方法论。云原生（CloudNative）是一个组合词，Cloud+Native。Cloud表示应用程序位于云中，而不是传统的数据中心；Native表示应用程序从设计之初即考虑到云的环境，原生为云而设计，在云上以最佳姿势运行，充分利用和发挥云平台的弹性+分布式优势。

最新官网对云原生概括为4个要点：DevOps + 持续交付 + 微服务 + 容器

# 安装

## 1. go

```bash
go get github.com/micro/micro/v3
```

## 2. docker

```bash
docker pull micro/micro
docker run -id -p 8080-8081:8080-8081/tcp micro/micro server
```

## 3. helm

the package manager for k8s

```bash
helm repo add micro https://micro.github.io/helm
helm install micro micro/micro
```

## 4. OS

```bash
# MacOS
curl -fsSL https://raw.githubusercontent.com/micro/micro/master/scripts/install.sh | /bin/bash
# Linux
wget -q  https://raw.githubusercontent.com/micro/micro/master/scripts/install.sh -O - | /bin/bash
# Windows
powershell -Command "iwr -useb https://raw.githubusercontent.com/micro/micro/master/scripts/install.ps1 | iex"
```

# 命令

```bash
# 假设以 docker 安装
docker exec -it 名称 /bin/bash # 进入容器

# 登陆
/micro login # 一般认知是 ./micro，但这里只能 /micro 才能执行
Enter username：admin
Enter password：micro
Successfully logged in.

/micro status        # 查看服务状态（空的），不登陆没权限
/micro services      # 查看服务注册情况
/micro env set local # 设置本地环境
```





# 收藏

https://learnku.com/docs/go-micro/3.x/helloworld/9470  # 教程

https://zhuanlan.zhihu.com/p/368545133  # 一个系列
