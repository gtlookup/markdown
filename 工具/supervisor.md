# Supervisor

py 开发的一套进程管理工具，能将挂掉的进程重新启动，又叫 `看门狗`

https://www.jianshu.com/p/0b9054b33db3

## 1. 安装

```bash
yum install supervisor
```

## 2. 配置

```bash
# /etc/supervisord.conf
[inet_http_server]            # HTTP服务器，提供web管理界面
port=0.0.0.0:8081             # web管理后台运行的IP和端口，如果开放到公网，需要注意安全性
username=root                 # 登录管理后台的用户名
password=root                 # 登录管理后台的密码

[program:agilor-hub]          # 是被管理的进程配置参数，:后是进程的名称
command=/root/agilor-hub      # 可执行文件路径
autorestart=true              # 程序退出后自动重启,
                              # 可选值：[unexpected,true,false]，
                              # 默认为unexpected，表示进程意外杀死后才重启
user=root                     # 用哪个用户启动进程，默认是root
stdout_logfile=/root/exe.out  # 程序输出的log文件地址
```

## 3. 命令

```bash
systemctl start supervisord.service     # 启动supervisor并加载默认配置文件
systemctl stop supervisord.service      # 停止
systemctl enable supervisord.service    # 将supervisor加入开机启动项

supervisorctl status        # 查看所有进程的状态
supervisorctl stop es       # 停止es
supervisorctl start es      # 启动es
supervisorctl restart       # 重启es
supervisorctl update        # 配置文件修改后使用该命令加载新的配置
supervisorctl reload        # 重新启动配置中的所有程序
```

## 4. 离线安装

https://segmentfault.com/a/1190000039359656

```bash
# 1. 安装可通过 yum 下载的安装包
yum install yum-plugin-downloadonly
# 2. 下载安装 python3 所需依赖包
#    --downloaddir 是指定要导出rmp包的位置
#    reinstall 表示已经安装过了，如果没安装过要用 install（这里要install）
yum reinstall --downloadonly --downloaddir=/root/c7 zlib-devel bzip2-devel openssl-devel ncurses-devel epel-release gcc gcc-c++ xz-devel readline-devel gdbm-devel sqlite-devel tk-devel db4-devel libpcap-devel libffi-devells make
# 3. 安装 rpm 包
rpm -Uvh ./*.rpm --nodeps --force
# 4. 安装py3
tar -zxvf Python-3.6.8.tgz
cd Python-3.6.8
./configure --with-ssl
make
make install
python3 -V # 查看版本：Python 3.6.8
# 5. 安装 meld3
tar -zxvf meld3-2.0.1.tar.gz
cd meld3-2.0.1
python3 setup.py install
# 6. 安装 supervisor
tar -zxvf supervisor-4.2.2.tar.gz
cd supervisor-4.2.2
python3 setup.py install
supervisorctl --help # 查看是否安装成功
```

