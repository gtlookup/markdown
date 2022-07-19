# wsl2

https://zhuanlan.zhihu.com/p/347461016

## 1. 安装wsl

```bash
# 控制面板 -> 卸载程序 -> 启动或关闭 Windows 功能（在左侧）
# 勾选倒数第3项目：适用于 Linux 的 Windows 子系统
# 根据提示重启
# 以管理员身份打开powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## 2. 安装其它

以==管理员身==份进入 powershell：

```bash
# 安装 Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
# 安装 LxRunOffline
choco install lxrunoffline
```

## 3. 尝试启动

```bash
LxRunOffline run -n centos # 启动方式一
wsl -d centos              # 启动方式二
exit # 退出
```

## 4. 升级为wsl2

```bash
wsl -l -v # 列出已经安装的wsl信息
wsl --set-version Ubuntu-20.04 2 # 如果报下面第3句
#正在进行转换，这可能需要几分钟时间...
#有关与 WSL 2 的主要区别的信息，请访问 https://aka.ms/wsl2
#WSL 2 需要更新其内核组件。有关信息，请访问 https://aka.ms/wsl2kernel

# 则到提示的网站上下载 wsl_update_x64.msi 安装后重新设置版本

wsl --unregister Ubuntu-20.04 # 卸载

# 显示wsl2的ip
# 一般wsl2里的web服务只能通过localhost来访问，127.0.0.1都访问不到
# 所以，通过该命令查看ip，将localhost换成指定的ip就可以访问web服务了
wsl -- ifconfig eth0
```

## 5. 重启wsl2

```bash
# 以管理员身份进cmd
net stop LxssManager  # 停止
net start LxssManager # 启动。直接打开也行
```

# ubuntu 安装 docker

```bash
apt update && apt upgrade -y # 更新
curl -fsSL https://get.docker.com -o get-docker.sh # 下载docker
sudo sh get-docker.sh # 安装
sudo service docker start # 启动
service docker status # 查看状态
```

# wsl 迁移其它盘

```bash
# 1. 关闭子系统
wsl --shutdown
# 2. 将原 C 盘子系统导出镜像到D盘
wsl --export Ubuntu-20.04 D:\wsl2\Ubuntu-20.04.tar
# 3. 卸载原 C 盘子系统
wsl --unregister Ubuntu-20.04
# 4. 查看是否卸载掉
wsl --list
# 5. 重新导入
wsl --import Ubuntu-20.04 D:\wsl2\Ubuntu-20.04 D:\wsl2\Ubuntu-20.04.tar --version 2
```

# 解决命令行不高亮

```bash
vim ~/.bashrc          # 1.
force_color_prompt=yes # 2. 放开注释
bash                   # 3. 
```

