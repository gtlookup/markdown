```bash
# centos7 先升级到 gcc9.3
yum -y install centos-release-scl 
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils 
scl enable devtoolset-9 bash

# 需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。如果要长期使用gcc 9.3的话：

echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```



https://www.csdn.net/tags/NtzaYg5sNTYzMDEtYmxvZwO0O0OO0O0O.html 怎么用