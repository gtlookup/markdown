linux 中==一切皆文件==；文件只有读、写、权限控制 3 种操作

https://www.bilibili.com/video/BV187411y7hF?p=4



# centos 命令

## 显示文件

```bash
ls      # 显示当前目录下所有文件
ls -l   # 以列表形式显示
ls -a   # 连隐藏文件及.开头文件都显示
ll net* # 显示所有叫net...的文件
ls -lR cache/*|grep "^-"|wc -l # 查看当前目录下cache下的文件数
```

## 显示敲过的命令

```bash
history
```

## 关机

```bash
poweroff
shutdown -h now #立刻关机
shutdown -h +10 #10分钟后关机
shutdown -h 12:00:00 #12点关机
halt #立刻关机
```

## 重启

```bash
shutdown -r now
reboot # 立刻重启
```

## 用户 / 密码

```bash
#查看当前用户
id -un
#添加用户
useradd 新用户
#修改密码
passwd 密码
#切换用户
su -l 用户
```

## 查看系统32还是64位

```bash
uname -m
```

## 查看应用的路径

```bash
which 应用名
#查看java安装路径
whereis java
```

## 文件/文件夹

```bash
# 创建文件夹
mkdir 文件夹名
mkdir -p a/b/c # 即使a/b文件夹不存在 -p 后都能给创建
# 创建文件
touch 文件名.扩展名
# 重命名或移动文件
mv old new
# 复制文件
cp form to
# 远程文件拷贝
# scp 本机文件 目标机:路径及文件
scp a.txt root@k8s-node1:/root/ # 将本地文件a.txt拷贝到k8s-node1里
scp /root/a.txt root@192.168.3.89:/root/gt/docker/ng02/a.txt # 本机的文件拷到3.89上
scp -r a root@192.168.3.89:/root/a # -r 表示拷贝文件夹

# 删除文件
rm -rf 目录/文件
# 查看当前目录
pwd
#往文件里输入内容
echo "内容" > a.txt  # 1.每次执行都覆盖 2.文件不存在则新建
echo "追加" >> a.txt # 不覆盖，末尾追加
echo a = \"haha\" >> a.txt # 结果：a = "haha"
#给文件加权限(如遇到：Cannot find ./startup.sh 可用)
chmod +x *.sh    # +加权限 x可执行(r可读w可写)
```

## 查看系统信息

```bash
# 系统版本
cat /etc/centos-release
# mac 地址
ifconfig -a
# 查看连本机的远程ip/mac
cat /proc/net/arp
```

## 根据url下载文件

```bash
wget url #wget也要先下载安装
```

## top

任务管理器 / 硬件信息

```bash
top #详细说明 https://www.cnblogs.com/mengchunchen/p/9669704.html
#查看硬件信息(CPU 内存 硬盘)
#https://blog.csdn.net/weixin_33734785/article/details/91926268
```

```bash
# 参数说明：
# top [-] [d] [p] [q] [c] [C] [S] [s]  [n]
d 指定每两次屏幕信息刷新之间的时间间隔。当然用户可以使用s交互命令来改变之。 
p 通过指定监控进程ID来仅仅监控某个进程的状态。 
q 该选项将使top没有任何延迟的进行刷新。如果调用程序有超级用户权限，那么top将以尽可能高的优先级运行。 
S 指定累计模式 
s 使top命令在安全模式中运行。这将去除交互命令所带来的潜在危险。 
i 使top不显示任何闲置或者僵死进程。 
c 显示整个命令行而不只是显示命令名
```



## 搜索命令

```bash
which java
whereis java
```

## jdk 环境变量

```bash
which java
ls -lrt /usr/bin/java
ls -lrt /etc/alternatives/java
#配置JAVA_HOME
vi /etc/profile            # 环境变量文件
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
source /etc/profile        # 最后让环境变量生效
```

## 防火墙

```bash
systemctl status firewalld      #查看防火墙状态
systemctl start/stop firewalld  #开启/关闭防火墙
systemctl disable firewalld     #禁止防火墙开机启动
```

## 端口操作

```bash
#开放端口
firewall-cmd --add-port=8080/tcp --permanent  #结果 success
#查看已经开放的端口
firewall-cmd --list-ports
#查看端口是否开放
firewall-cmd --query-port=8080/tcp   #结果 yes/no
#删除开放的端口(然后要reload才生效)
firewall-cmd --permanent --remove-port=8080/tcp
#重新载入端口，使以上操作生效
firewall-cmd --reload      #结果 success
```

## 获取加密密码

```bash
#                                      -sha1：是数字1，不是字母l
echo -n acc:123 | openssl dgst -binary -sha1 | openssl base64 # 账号：acc，密码：123
# 结果：kkSs0iUNGV03wPUYlKMU7tobjaA=
```

## 执行文件无权限

```bash
chmod 777 程序名
```

## curl

```bash
#curl 命令行访问url
#-X http的请求方式  如：-XGET/POST/PUT/DELETE/HEAD
#-d 指定要传输的数据
#-H 指定http请求的头信息
curl -XGET 'http://localhost:8088'

curl -i # 显示请求结果
	 -XPOST "http://192.168.3.89:8713/agilorapi/v6/write?db=PI__" 
	 -H "Authorization: Token oxtviAO6OWBiOMmQhQXkSdu1Id8VO0KdRyhlgnHOsLsmVeNCj0z2-XKSoVZ6xFh2B5kmlB327nZlNHxR7NZBOQ==" 
	 -H "Content-Type: text/plain" 
	 --data-binary "@./rel.txt" # 指定本地文件
```

## yum

```bash
#查看可安装的java版本
yum -y list java*
#安装
yum install java-1.8.0-openjdk.x86_64
#卸载软件
yum remove xxx
#查看是否安装了某软件
yum list installed | grep 软件名
```

## cmake

```bash
yum groupinstall "Development Tools" # 打包安装全部 GNU 开发库
```

```bash
1.安装：yum install cmake
2.创建一个工程文件夹: mkdir dir_cmake
3.cd到这个文件夹
4.创建配置文件：touch CMakeLists.txt
5.编辑该配置文件：gedit CMakeLists.txt
   添加代码：
      # CMake 最低版本号要求
      cmake_minimum_required (VERSION 2.8)
      # 项目信息
      project (demo1)
      # 指定生成目标
      add_executable(demo1 main.c)
6.创建代码文件：touch main.c
7.添加代码：gedit main.c  
   #include <stdio.h>
   #include <stdlib.h>
   int main(int argc, char *argv[])
   {
       printf("haha 666");
   }
8.生成项目：cmake .
9.编译：make
10.执行：./demo1 x x    # 随便两个什么参数
   显示结果：haha 666
```

## tar

```bash
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
#下面的参数-f是必须的
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。

tar -zxvf ... # 解压
tar -zcvf x.tar.gz 目录(或文件或目录/文件)
```

## maven

```bash
#安装
#1.下载
#2.解压
#3.配置环境变量
vi /etc/profile
#最底下加上
export MAVEN_HOME=/opt/apache-maven-3.5.4
export PATH=$MAVEN_HOME/bin:$PATH
#配置文件生效
source /etc/profile
#查看版本
mvn -v
```

## lsof

```bash
# 查看8080是否被占用
lsof -i:8080
# 查看所有打开的文件(或理解为正在运行的进程)
lsof -i
```

## awk

https://www.runoob.com/linux/linux-comm-awk.html

## nohup

```bash
nohup ./exe > w_rtdb.log & # 后台运行可执行文件exe，并将log输出到 w_rtdb.log 中
```

## ssh

### 1. 基本连接

```bash
ssh 192.168.2.36 -l root    # 连接linux
ssh root@192.168.2.36       # 或者
ssh -p 22 root@192.168.2.36 # 指定端口
```

### 2. 端口转发

通过SSH登陆之后，在**SSH客户端**与**SSH服务端**之间建立了一个隧道，从而进行通信

#### 2.1. 本地端口转发

将本地端口转发给远程端口（访问本地相当于访问远程）

```bash
# 1. 先在远程 centos 上跑一个 web 服务，如：192.168.87.128:8080
simple-http-server -p 8080 --try-file ./index.html -i ./ # index.html 里就写 haha
# 2. 在本机执行端口转发
#    本地端口:目标ip:端口          远程centos用户@ip
ssh -L 8080:192.168.87.128:8080 root@192.168.87.128
# 3. 在本机浏览器访问
http://localhost:8080  # 结果：haha
#    将 localhost:8080 请求转发到 192.168.87.128:8080 上，请求 localhost 相当于请求 192.168.87.128
```

#### 2.2. 远程端口转发

将远程端口转发给本地端口（访问远程相当于访问本地）

```bash
# 1. 本机上跑一个 web 服务，如：10.10.20.44:8081（返回 "index"，本机服务要绑定0.0.0.0:8081）
# 2. 在本机上执行端口转发
#    远程端口:本机ip:本机端口    远程centos用户@ip
ssh -R 8080:10.10.20.44:8081 root@192.168.87.128
# 3. 在远程 centos 上访问
curl http://localhost:8080 # 结果：index
```

#### 2.3. 动态端口转发

可用于 ”科学上网“，浏览器访问的话需要配置 `socks5`

```bash
# 1. 在本机上设置动态转发
ssh -D 8080 root@192.168.87.128
# 2. 本机通过 8080 访问百度
curl --socks5 localhost:8080 http://www.baidu.com
# 结果：返回百度的html内容
```

#### 2.4. 停止转发

转发基于 `SSH` 链接，因此，当退出 `SSH` 链接后即可停止转发

## ps

只显示进程的静态快照，及瞬间的进程状态

```bash
ps [-aefFly] [-p pid] [-u userid]
# -a：与任何用户标识和终端相关的进程
# -e：所有进程（包括守护进程）
# -ef：显示所有用户进程，完整输出。等价于 aux
# -t：仅显示所有守护进程
# -x：显示没有控制终端(TTY)的进程
# -p：pid 与指定PID相关的进程
# -u：userid 与指定用户标识userid相关的进程
```

```bash
ps aux # 如下结果：

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0    896   532 ?        Sl   04:10   0:00 /init
root         8  0.0  0.0    896    88 ?        Ss   04:10   0:00 /init

# USER：哪个用户创建的
# PID：进程号
# %CPU：CPU占用率
# %MEM：内存占用率
# VSZ：虚拟地址
# RSS：空闲地址
# TTY：终端（如：vim、ps、pts/22）
# STAT：状态
# START：起始时间
# TIME：运行时间
# COMMAND：命令
```



## netstat

```bash
netstat -tunlp|grep port # 查看port是否被占用
# 结果：tcp6  0   0 :::8088   :::*   LISTEN   90181/java
ps -ef|grep 90181 # 然后查看 90181/java 这个进程，可以知道是哪个程序占用的端口
```

## df

全称 disk free，用于显示统计系统磁盘占用和空余情况

- -a：显示所有文件系统的磁盘使用情况
- -m：以1024字节为单位显示
- -h：以K、M、G为单位显示

```bash
# df [选项] [目录]
#	-h：以K，M，G为单位
df -h       # 查看硬盘容量
df -h /data # 查看 /data 容量信息
```

## du

全称 disk usage，用于显示目录或文件大小

- -a：递归显示指定目录中各个文件和子目录中文件占用的数据块
- -s：显示指定文件或目录占用的数据块
- -b：以字节为单位显示磁盘占用情况
- -h：以K、M、G为单位显示

```bash
# df [选项] [目录/文件]
#	--max-depth=0：0只显示该目录大小，1该目录下的第一层目录大小
du -h / --max-depth=1    # 显示当前目录下大小
du -h /var --max-depth=1 # 显示目录/var下大小
du -h db.dump            # 显示 db.dump 文件大小
```

## upx

https://www.jianshu.com/p/f51f4f64524a

UPX (the Ultimate Packer for eXecutables)是一款先进的可执行程序文件压缩器，压缩过的可执行文件体积缩小50%-70% ，这样减少了磁盘占用空间、网络上传下载的时间和其它分布以及存储费用。 通过 UPX 压缩过的程序和程序库完全没有功能损失和压缩之前一样可正常地运行，对于支持的大多数格式没有运行时间或内存的不利后果。 UPX 支持许多不同的可执行文件格式 包含 Windows 95/98/ME/NT/2000/XP/CE 程序和动态链接库、DOS 程序、 Linux 可执行文件和核心。

百科定义主要有两点：==加壳、压缩==

==小于40KB的文件不支持==

```bash
upx agilor-1 -o agilor-hub
upx agilor.exe -o agilor-hub.exe
```

## while...done

```bash
# 循环执行某一操作
while true;do curl http://localhost:81;sleep 1;done; # 循环访问81操作，每秒访问一次
```

# ==填坑==

> 报错：Failed to set locale, defaulting to C.UTF-8

```shell
echo "export LC_ALL=en_US.UTF8" >> /etc/profile
source /etc/profile
```

> jps command not found

```bash
# 1.卸载
yum remove java*
# 2.重新安装-devel版
yum install java-1.8.0-openjdk-devel
```

> 解决【阿里ECS】配置tomcat后无法访问

https://www.cnblogs.com/kingsonfu/p/9802537.html

> 杀掉挖矿病毒

```bash
# 查看cpu占用率
top -c
# 当文件无法删除时用 chattr -i 去掉无法删除的权限
# 要删除以下文件
chattr -i /etc/sysupdate
chattr -i /etc/networkservice
chattr -i /etc/update.sh
rm /etc/上面3个文件
# 最后通过 top -c 查看pid
kill pid # 删除sysupdate 和 networkservice进程
```

> final shell 上传失败

```bash
df -h # 发现 /dev/mapper/centos-root 磁盘满了
du -h --max-depth=1 # 然后 cd / 到根目录下执行，看看哪个文件夹大，一层一层找，最后发现是log满了----删！
```

> top -c 发现 ./kswapd0 进程 cpu 占用过高

当物理内存不足时，swap分区会将一部分硬盘当做虚拟内存来使用。

kswapd0 占用过高是因为物理内存不足，使用swap分区与内存换页操作交换数据，导致CPU占用过高。

```bash
ls -a /tmp/.X25-unix
# 结果：.  ..  dota3.tar.gz  .rsync
kill port # ./kswapd0 对应的进程
kill port # .X25-unix 对应的进程
```

>发现每天一来，输入三次密码后都会报 Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password). 

```bash
# vi /etc/ssh/sshd_config
PermitRootLogin yes        # 允许root账号用ssh登陆
PubkeyAuthentication no    # 取消密钥登录
PasswordAuthentication yes # 改用密码登陆

systemctl restart sshd.service # 重启sshd服务
```

> ssh root@阿里云ip时报：ssh_exchange_identification: read: Connection reset

换成手机热点访问就好用了，说明网络有问题

> 明明安装了wget，但还是不识别 wget命令

```bash
ll /usr/bin/wg* # 查看哪个命令像 wget，发现 wge 像，试试 wge http://xxx 确实能下载，说明wget被篡改成了wge
mv /usr/bin/wge /usr/bin/wget # 给改回来，问题解决！
```

