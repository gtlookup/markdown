# 环境

## 1. 概念

### 1.1 GNU

linux下编程永远绕不开`GNU`；是 `GNU's Not Unix` 的缩写，意思是`GNU`不是`Unix`。GNU下的软件都是开源免费的

GNU下的知名软件：

- C编译器：GNU C，就是传说中的GCC
- C++编译器：G++
- GNU C Library（GLIBC）
- Bash Shell

### 1.2 GCC与G++

**gcc**是GNU的C编译器，**g++**是GNU的C++编译器，两者都是 `GNU Compiler Collection` 的一部分

编译器从拿到一个c文件到生成一个可执行程序，中间经历了四步：

```mermaid
graph LR
0[.c]
1[.i]
2[.s]
3[.o]
4[.out]
0 --> |预处理 `gcc -E` |1
1 --> |编译器 `gcc -S`|2
2 --> |汇编器 `gcc -c`|3
3 --> |链接器|4
```

```bash
# 假如源文件为：pid.c
gcc -E pid.c -o pid.i # 预处理：头文件展开、宏替换、去掉注释
gcc -S pid.i -o pid.s # 生成汇编文件
gcc -c pid.s -o pid.o # 生成目标（二进制）文件
gcc pid.o -o pid      # 生成可执行文件
```

### 1.3 GDB

GDB（GNU Debugger）是一个调试程序，可以做四个主要事情：

1. 开始一个程序，指定任何可能影响其行为的事情
2. 让程序在特定条件下暂停
3. 程序停止时检查发生了什么
4. 改变程序中变量或数据

可调试的语言有：Ada、C / C++、Objective-C、Pascal等。**支持本地调试和远程调试**，可运行在windows和unix平台



### 1.4 makefile

是程序编译规则，其记录整个工程的编译规则，如：源文件的编译顺序、依赖关系等，通过**make**工具进行编译

make根据makefile定义的规则将源代码编译成二进制文件。

在跨平台（特别是类Unix系统）程序中，一般都会通过makefile来进行编译

## 2. 开发工具

安装搜`yum groupinstall`，centos/ubuntu都有

### 2.1 命令

```bash
-v / --version # 查看编译器版本，-v字多，另一个字少
-g             # 包含调试信息
-Wall          # 显示所有警告信息
-Wall -Werror  # 将警告信息当错误信息处理
-D             # 编译时定义宏，如：
gcc/g++ xx.c -o xxx -DHOME=7 # 此时，代码里就可以用HOME宏了(printf("%d", HOME);)

gcc/g++ xxx.c/cpp    # 编译链接并生成可执行文件（a.out）
gcc/g++ -c xxx.c/cpp # 编译成目标文件不链接，生成.o文件
gcc/g++ -O xxx.c/cpp -c # 编译成目标文件并优化执行速度，生成.o文件

# 将目标文件链接成二进制（可执行）文件
gcc 目标文件(.o) -o 可执行文件名 -lstdc++
g++ 目标文件(.o) -o 可执行文件名

gcc/g++ 源代码文件(.c/.cpp) -o 可执行文件名 # 直接编译成可执行文件
```

#### 2.1.1 hello world

```c++
// 1. vim hello.cpp
#include <iostream>
int main() {
    // gcc hello.cpp 会报错，因为c不识别c++的库
    std::cout << "hello world!!!" << std::endl;
    // 如果关文件换成 #include <stdio.h>
    // 则 gcc hello.cpp / g++ hello.cpp 都好用
    printf("hello world!!!\n");
}
// 2. 编译：g++ hello.cpp
// 3. 执行：./a.out
// 结果：hello world!!!
```

#### 2.1.2 多个源文件

```c++
// dyn.h
#include <iostream>
void test();
// dyn.cpp
#include "dyn.h"
void test() { std::cout << "test" << std::endl; }
// main.cpp
#include "dyn.h"
int main() { test(); }
```

```bash
# 编译成可执行文件命令：
g++ -o abc main.cpp dyn.cpp
# 把代码里的c++库换成c库（iostream => stdio.h）
gcc -o abc main.cpp dyn.cpp
./abc # 结果：test
```

### 2.3 g++与gcc区别

上面说g++是c++编译器，gcc是c编译器；但是g++也能编译c，gcc也能编译c++，区别为：

1. gcc会把.c文件当成c程序，g++会把.c文件当成c++程序。gcc / g++都会把.cpp文件当成c++程序

2. 编译阶段g++会调用gcc，但gcc不会自动连c++标准库，所以c和c++程序一般都用g++来编译。

3. gcc想要编译c++标准库需要加上选项：

   ```c++
   #include <iostream>
   int main() {
       std::cout << "hello world!!!" << std::endl;
   }
   // 这段代码通过 gcc hello.cpp -lstdc++ 就可以编译
   ```

# 动态库

动态链接：链接器在链接时仅仅建立与所需库函数之间的链接关系，在程序运行时才将所需的库调入可执行程序

```bash
# 生成动态库(.so)
#   -shared：指定生成动态库
#   -fpic：编译为独立的代码（pic小写大写都行）
#   -include：包含头文件，很少用，因为工程中一般都有相应的头文件
#   -I(大写i)：指定头文件路径，可用相对路径
gcc/g++ -shared x1.cpp x2.cpp xn.cpp -fpic -o libxx.so
# 生成.so后，可以通过下面命令查看是否包含源文件里的方法名：
nm libxx.so
nm libxx.so | grep "xxx" # 或者直接查找方法名
```

```bash
# 使用动态库
#   -L.：表示要链接的目录为当前目录。-L/usr/lib 表在要链接库的目录是/usr/lib
#   -l(小写L)：动态库的名称(并非动态库的文件名)，比如libabc.so，则名称为abc
gcc/g++ xx.cpp -L. -lxx -o 可执行文件名
```

```bash
# 当执行程序时一般会报：
# error while loading shared libraries: libxx.so: cannot open shared object file: No such file or directory
# 解决如下：
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH # 设置当前目录为动态库的搜索路径之一
# 注：每次重新打开控制台运行可执行文件前都要export
```

## 示例

```c++
// dyn.h
#include <stdio.h>
void test();
// dyn.cpp
#include "dyn.h"
void test() { printf("test\n");}
// main.cpp
#include "dyn.h"
int main() {test();}
```

```bash
gcc/g++ -shared dyn.cpp -fPIC -o libdyn.so # 1. 生成动态库
gcc/g++ main.cpp -L. -ldyn -o aaa          # 2. 链接生成可执行文件aaa
./aaa # 执行报错：./aaa: error while loading shared libraries: libdyn.so: connot open shared object file: No such file or directory
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH # 设置当前目录为动态库的搜索路径之一
./aaa # 再次执行结果：test
```

# 静态库

静态链接：由链接器在链接时将库的内容加入到可执行程序中

写个静态库步骤：

1. 将要生成静态库的源文件.c生成.o文件
2. 使用打包工具ar，将.o文件打包成.a文件（如：libxxx.a）
   - ar 工具参数：==r==更新，==c==创建，==s==建立索引

## 示例

```c
// add.h
int add(int,int);
// add.c
int add(int x, int y) { return x + y; }
// sub.h
int sub(int, int);
// sub.c
int sub(int x, int y) { return x - y; }
// main.c
#include "add.h"
#include "sub.h"
#include <stdio.h>
int main() {
    printf("7 + 77 = %d\n", add(7, 77));
    printf("77 - 7 = %d\n", sub(77, 7));
}
```

```bash
gcc -c add.c -o add.o
gcc -c sub.c -o sub.o        # 1. 生成目标文件
ar -rcs libxxx.a add.o sub.o # 2. 打包生成静态库
# 3. 将 add.h sub.h libxxx.a main.c 拷到另一个目录下（为验证.a是否生效）
gcc main.c -o main -L. -lxxx # 4. 生成可执行文件
# 5. 生成后，无论单独将 main 拷到哪里，都能跑
```



# mkfifo

**命名管道**，作用是进程间通信。本质是`linux`中的一个命令，用于创建一个0大小的文件（管道名）。

一个进程往该文件里写，另一个进程从该文件读取，就实现了进程间通信，和写普通文件一样

```bash
mkfifo [-m 权限(8进制数)] 管道名
mkfifo -m 666 test # 例：创建名叫test的管道，权限是666
```

```c
// 函数原型
#include <sys/stat.h>
int mkfifo(const char *pathname, mode_t mode); // 0：成功，1：失败
```

## 1. 写入端

```c++
#include <sys/stat.h>   // mkfifo
#include <stdio.h>
#include <fcntl.h>      // open、O_WRONLY
#include <unistd.h>     // write、close、sleep
#include <stdlib.h>     // exit(1)
#include <limits.h>     // PIPE_BUF
#define BUFES PIPE_BUF

int main() {
    char f_fifo[] = "pipe1";

    // 没有就创建，有就返回-1
    if (mkfifo(f_fifo, 0666) < 0) printf("mkfifo failed, maybe there exist a same fifo file\n");
    else printf("mkfifo ok\n");

    int fh, sz_buf;
    char buf[BUFES];
    // 打开
    if ((fh = open(f_fifo, O_WRONLY)) < 0) {
        printf("Open failed!\n");
        exit(1);
    } else printf("Open fifo ok!\n");

    for (int i = 0; i < 20000; i++) {
        sz_buf = sprintf(buf, "write data %d", i);
        printf("Send msg: %s\n", buf);
        if (write(fh, buf, sz_buf + 1) < 0) {
            printf("Write failed!\n");
            close(fh);
            exit(1);
        }

        usleep(1000); // 不能写的太快，否则监听端丢数据
    }

    close(fh);
    unlink(f_fifo);
    exit(0);
}
```

## 2. 监听端

```c++
#include <sys/stat.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include <fcntl.h>
#include <unistd.h>
#define BUFES PIPE_BUF

int main() {
    int fh, len;
    char f_fifo[] = "pipe1";
    char buf[BUFES];

    if ((fh = open(f_fifo, O_RDONLY)) < 0) {
        printf("Open failed!\n");
        exit(1);
    }

    while ((len = read(fh, buf, BUFES)) > 0) printf("Read_fifo read:%s\n", buf);
    
    close(fh);

    exit(0);
}
```

## 3. java监听

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class RecvFifo {
    public static void main(String args[]) {
        while (true) {
            try {
                BufferedReader in = new BufferedReader(new FileReader("pipe1"));
                char buf[] = new char[4096];
                while (in.read(buf) > 0) {
                    System.out.println(new String(buf));
                }
            } catch (IOException e) {}
        }
    }
}
```

说明：先运行写入端，会在`open`处阻塞；当监听端以读权限打开`pipe1`文件时，写入端才会写数据



https://www.bilibili.com/video/BV1Yo4y1D7Ap?p=48&vd_source=e611dc7ed99505a0ef548dd66f0a11dd  13:48
