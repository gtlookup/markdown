# linux目录介绍

/：根目录，linux下只有一个根目录

/bin: /usr/bin：可执行二进制文件目录，如常用命令：ls、tar、mv、cat等

/boot：存放linux启动时用的一些文件，如linux内核文件：/boot/vmlinuz，系统引导管理器：/boot/grub

/dev：存放linux下的设备文件，访问该目录下某个文件相当于访问某个设备

/etc：存放系统配置文件的目录，不建议放可执行文件

/home：系统默认的用户目录，新增用户账号时的用户目录都存在该目录下

/lib: /usr/lib: /usr/local/lib：系统使用的函数库目录。该目录下的函数用于协助程序在执行过程中，需要获取的额外参数

/lost+fount：系统异常产生错误时，会将一些遗失的片断放在该目录下

/mnt:/media：光盘默认挂载点，通常光盘挂载于 /mnt/cdrom 下，也可以选择任意位置挂载

/opt：存放给主机额外安装软件的目录

/proc：此目录的数据都在内存中，如系统核心、外设、网络状态。由于数据在内存中，所以不占用磁盘空间

/root：系统管理员的用户根目录

/sbin:/usr/sbin:/usr/local/sbin：

- 存放系统管理员使用的可执行命令、如：fdisk、shutdown、mount等。
- 与/bin不同的是，这几个目录是给系统管理员 root 使用的命令，一般用户只能 “查看” ，而不能设置和使用

/tmp**：`**`一般用户或正在运行的程序临时存放文件的目录，任何人都可访问，重要数据不要放在此处

/srv：服务启动后需要访问的数据目录，如 www 服务需要访问的网页数据存放在 /srv/www 下

/usr：用户程序存放目录

- /usr/bin：放应用程序
- /usr/share：放共享数据
- /usr/lib：程序运行所需要的函数库文件
- /usr/local：存放软件升级包
- /usr/share/doc：存放系统说明文件
- /usr/share/man：存放应用说明文件

/var：存放系统执行过程中经常变化的文件

- /var/log：日志文件
- /var/message：所有的登陆文件
- /var/spool/mail：存放邮件
- /var/run：存放程序或服务启动后的PID

# 文件分类

白色：普通文件，绿色：可执行文件，红色：压缩文件，蓝色：目录，青色：链接文件，黄色：设备文件，灰色：其它文件



# 安装dev

```bash
# 类似于 centos 里 yum groupinstall "Development Tools"
sudo apt-get update
sudo apt-get install build-essential
```

# 无法装软件

```bash
# 一般在docker里运行的ubuntu会出现这个问题
Building dependency tree       
Reading state information... Done
E: Unable to locate package ****

# 解决办法
apt-get update
apt-get upgrade # 这句可以不执行
```

# ulimit

> 当通过 `tokio::spawn` 创建数万个文件时，发现文件数总是不够，经调查发现：

```bash
ulimit -a # 执行后结果如下

core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7185
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024       # 这里默认太小了
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) unlimited
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

ulimit -SHn 1048576 # 设置 open files 临时为 1048576，重启后恢复原值
```

# 查看cpu架构

```bash
uname -a # 方法一
# Linux vmachine 5.11.0-43-generic #47~20.04.2-Ubuntu SMP Mon Dec 13 11:06:56 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

uname -m # 方法二
arch     # 方法三
# x86_64
```

# man

是一个帮助命令，掌握后可以更好的学习linux。==还可以查看系统api说明==

```bash
man ls  # 查看ls命令的帮助
man man # 查看man命令的帮助
man cat
man printf # 查看系统api函数
man abs    # 查看系统api函数
```

# objdump

将可执行文件反编译成汇编

```bash
objdump -S -l 可执行文件 > x.txt
objdump -S hello > h.txt
```

# 文件中执行linux命令

```bash
vim f1 # 内容如下：
mkdir aaa
cd aaa
touch a.tat
cd ..
cd ..
# 设置f1为可执行文件
chmod 777 f1
./ f1 # 执行
```

# time

查看一个可执行程序跑完后花了多长时间，如可执行文件叫demo：

```bash
time ./demo # 结果：
# real    0m0.128s 实际用时
# user    0m0.002s 用户态用时
# sys     0m0.001s 内核态用时（多线程下用时较多，因为涉及到创建线程及上下文切换所耗费时间）
```

# 快捷键

```bash
ctrl + L     # 清屏
tab          # 补齐或选择文件
ctrl + a | e # home end
ctrl + c     # 结束进程
ctrl + u     # 删除整行命令
```

# file

查看文件类型

# 查看文件内容

```bash
cat：加 `-n` 给内容（含空行，`-b` 不含空行）加行号
less：可以上下滚动，按 `q` 退出
head：从文件关向下查看内容。`-c 10` 查看前10个字符，`-n 2` 查看前两行，`-v *.c` 显示所有.c文件（带文件名，-q不带文件名）
tail：从文件尾向上查看内容。`-c 10` 查看后10个字符，`-n 2` 查看后两行，`-v *.c` 显示所有.c文件（带文件名，-q不带文件名）
     `-f a` 始终（实时）查看文件末尾内容 
```

# find

功能非常强大，通常用于在特定目录下搜索符合条件的文件，也可以搜索特定用户属主的文件

```bash
find 路径 -name 文件名  # 按文件名查找
find 路径 -size 范围    # 按文件大小查找
# +100k：表示大于100k
# -100k：表示小于100k
#  100k：表示等于100k
# 注意：单位k必须小写，M必须大写
```

# grep

强大的文本搜索工具（数据查找与定位）

```bash
# 选项： -e 或 -f 等等，还可以连用（如：-ion）
#		-i：忽略大小写
#		-o：精准匹配
#		-n：显示行号
#       -v：不包含匹配的行
# pattern：要匹配的字符串
# 文件名：可以是多个文件（如：file1 file2 file3）
grep [选项] '查找内容' 文件名
grep -r '查找内容' 路径  # 搜索指定目录
```

# 管道(|)

一个命令的输出通过管道(|)作为另一个命令的输入

```bash
cat thr.c | grep -on main # 根据cat到的文件内容，查找`main`关键字
```

# 重定向

将原本输出到屏幕的内容，重定向（>、>>）输出到别地方

```bash
ls dir > a.txt  # 将dir下的文件名输出到a.txt
ls dir >> a.txt # 将dir下的文件名追加到a.txt
skdfjdsk 2> err # 将错误信息输出到err文件中
ls dir > /dev/null # 不管什么内容，输出到黑洞文件后就没有了，再也查不到了
ls ./ abc &> a.txt # 将标信息和错误（abc目录不存在，所以报错）信息都输出到a.txt，用 &>
```

# tree

以树状形式查看目录内容，安装：`apt install tree`

```bash
tree [目录名] [-L n] # 默认查看全部层，可以指定要查看的目录，-L n 表示只查看n层
```

# ln

用于创建某文件的链接（快捷方式），源文件路径最好用绝对路径，链接文件分为：

- 软链接(-s)：不占用磁盘空间，源文件删除则链接文件失效
- 硬链接：只能链接==普通文件==，不能链接目录。两个文件占用相同磁盘大小，源文件删除，链接文件依然有效

```bash
ln -s 源文件 链接文件 # 软链接
ln 源文件 链接文件    # 硬链接
```

# vim

是 `vi` 的升级版，兼容 `vi` 所有功能；新特性如：无限撤销、自动补全、代码高亮等。`vimtutor` 打开帮助文档

四种模式：

- 命令模式：任何模式按 `Esc` 都能切回命令模式

  ```bash
  ZZ # 保存并退出
  
  # ------- 进入编辑模式 --------
  i  # 在光标当前位置进入编辑模式
  I  # 将光标移到当前行开始位置，再进入编辑模式
  o  # 在光标下一行插入空行并进入编辑模式
  O  # 在光标上一行插入空行并进入编辑模式
  a  # 光标右移一格并进入编辑模式
  A  # 光标移到行末并进入编辑模式
  s  # 删除光标右边一个字符并进入编辑模式
  S  # 删除当前行并进入编辑模式
  
  # ------- 光标移动 --------
  l # 右移
  h # 左移
  k # 上移
  j # 下移
  ctrl + f：# 向上滚一屏
  ctrl + b：# 向下滚一屏
  gg # 到文件第一行
  G # 到文件最后一行
  0(数字)：# 光标移到行首
  ^ # 光标移动该行第一个有效字符
  mG或mgg # 到指定行。光标移到m行首个有效字符上
  $ # 光标移到和尾
  
  # ------- 复制粘贴 --------
  [n]yy # 复制当前行往下n行
  p # 在当前行下面粘贴
  
  # ------- 删除 --------
  [n]x  # 删除光标后 n 个字符
  [n]X  # 删除光标前 n 个字符
  D     # 删除光标所在位置到行末所有内容
  [n]dd # 剪切当前行往下n行内容
  dG    # 删除光标所在位置到文件末尾所有内容
  dw    # 删除光标所在位置后的字符（按单词删）
  d0(数字) # 删除本行光标前的内容
  dgg   # 删除光标前到文件首所有内容
  
  # ------- 撤销恢复 --------
  . # 执行上一次操作
  u # 撤销上一次操作
  ctrl + r # 反撤销
  n. # 执行n次上一次操作
  
  # ------- 查找 --------
  /内容 enter # 当前光标 enter 后，n 向下找，N 向上找
  ?内容 enter # 当前光标 enter 后，n 向上找，N 向下找
  
  # ------- 替换 --------
  r 字符   # 按下r后同志按个字符，替换当前光标后的一个字符
  R 字符串 # 按下R后一直输入字符，替换当前光标后的字符串，按Esc退出
  ```

- 编辑模式：

- 可视模式：

  ```bash
  v # 以单个字符为单位，配合 hjkl 进行选取内容，按 d 删除，按 y 复制
  V # 以行为单位，配合 hjkl 进行选取内容，按 d 删除，按 y 复制
  ctrl + v # 以列为单位，配合 hjkl 进行选取内容，按 d 删除，按 y 复制（注意：win的cmd下不好用，得用wsl专属橘黄那个）
  ```

- 末行模式：shif + ; 该模式下可对文件进行查找、替换、定位行、保存、退出等

  ```bash
  :set nu     # 显示行号
  :set nonu   # 不显示行号
  :q # 退出文件
  :100        # 定位到100行
  :q! # 不保存退出
  :wq # 保存退出
  :x  # 保存退出
  :w 路径文件名 # 另存为
  :/要查找的字符 + enter # 查找内容
  
  # ------- 替换 --------
  :s/替换前/替换后/      # 替换光标所在行第一个匹配内容
  :s/替换前/替换后/g     # 替换光标所在行的所有匹配内容
  :1,10s/替换前/替换后/g # 替换1到10行的所有匹配内容
  :7,14s/^/\/\/ # 7到14行加注释，即替换可以写正则
  :%s/替换前/替换后/g    # 替换该文件所有匹配内容
  :1,$s/替换前/替换后/g  # 替换该文件所有匹配内容
  :%s/替换前/替换后/gc   # 替换该文件所有匹配内容，每按下y替换一个
  
  # ------- 分屏 --------
  :sp  # 水平分屏
  :vsp # 垂直分屏
  :sp 文件名  # 当前文件和另一个文件水平分屏
  :vsp 文件名 # 当前文件和另一个文件垂直分屏
  ctrl + w w # 光标在多个分屏间切换
  :wall | :wqall | :qall  # 保存所有分屏 | 保存并退出所有分屏 | 退出所有分屏
  vim -O 文件1 文件2 # 垂直分屏打开俩文件
  vim -o 文件1 文件2 # 水平分屏打开俩文件
  
  # ------- 其它用法 --------
  :!man printf # 直接在vim里查看printf，按q退出，再回车回到vim
  :r !ls # 将ls结果写到当前光标下方，!ls 可以是任何命令
  :r 文件路径  # 将指定文件的内容写到当前光标下方
  :w 文件路径  # 将当前文件内容写入到另一个文件
  :w! 文件路径 # 强制将当前文件内容写入到另一个文件（比如目标文件已存在，要加w!）
  ```
  
  



