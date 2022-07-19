配置文档：https://docs.microsoft.com/zh-cn/windows/terminal/



# WSL安装centos

1.先用docker导出一份centos7的容器

```bash
docker pull centos:centos7               # 下载镜像
docker run centos:centos7                # 运行后自动停止
# 导出容器到centos7.tar
docker export 78112e26a952 > centos7.tar # 通过docker ps -a 取得78112e26a952 用于导出容器
```

2.用FinalShell将centos7下载到win10

3.用管理员身份打开Windows Terminal，然后cd到centos7.tar所在目录

4.用wsl安装

```bash
# wsl --import 名称 安装位置 容器位置
# 注：安装位置不能是D:\Program Files\centos7，因为有空格了
wsl --import centos7 D:\centos7 .\centos7.tar  # 导入容器
wsl -l                                         # 查看win10的子系统
# 结果：
#适用于 Linux 的 Windows 子系统分发版:
#centos7 (默认)
```

5.进入centos

```bash
wsl                 # 进入linux
cat /etc/*release*  # 查看系统版本
```

## wsl卸载子系统

```bash
wslconfig /l          # 同 wsl -l
wslconfig /u contos7  # 注销
```

==不好用，不如虚拟机。wsl里的centos根本跑不了后台进程，也就是systemctl start 任何进程都无法启动，闹着玩儿还可以。==

# 设置wsl默认打开目录

```bash
# 设置 -> 默认值 -> 常规
//wsl$/Ubuntu-20.04/root
```

# 清理C盘

```bash
# 清除 winsys 目录
Dism /Online /Cleanup-Image /AnalyzeComponentStore # 分析
Dism /online /Cleanup-Image /StartComponentCleanup # 清理
```

https://zhuanlan.zhihu.com/p/379071844
