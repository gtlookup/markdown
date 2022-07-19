# 编译运行

## 1. 单个文件

```java
// Hello.java
public class Hello {
    public static void main(String args[]) {
        System.out.println("haha");
    }
}
```

```bash
javac Hello.java # 编译
java Hello       # 运行结果：haha
```

## 2. 带jar包

```bash
# 貌似-cp == -classpath
# 如果windows则 `.;`
# 如果linux则 `.:`
javac -cp .:xx1.jar:xx2.jar:xxn.jar xx.java # 编译
java -cp .:xx1.jar:xx2.jar:xxn.jar xx # 运行
```

`Main.jax`引用了`fastjson-1.2.76.jar`文件

```java
import com.alibaba.fastjson.JSON;
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String args[]) {
        List<Integer> list = new ArrayList(){{add(1);add(2);add(3);}};
        String s = JSON.toJSONString(list);
        System.out.println(list);
    }
}
```

```bash
javac -cp .:fastjson-1.2.76.jar Main.java # 编译
java -cp .:fastjson-1.2.76.jar Main       # 运行结果：[1,2,3]
```

# 调动态库(.so)

java.lang.UnsatisfiedLinkError: Error looking up function 'test': ./libdyn.so: undefined symbol: test

```c++
// 1. dyn.h
#include <stdio.h>
void test();
// 2. dyn.cpp
#include "dyn.h"
void test() { printf("haha test \n"); }
```

```bash
g++ -shared dyn.cpp -fPIC -o libdyn.so # 3. 编译动态库
```

```java
// 4. Main.java
import com.sun.jna.Library;
import com.sun.jna.Native;

public class Main {
    public interface Clib extends Library {
        Clib INS = (Clib) Native.loadLibrary("dyn", Clib.class);
        void test();
    }

    public static void main(String args[]) {
        Clib.INS.test();
    }
}
```

```bash
# 5. 把libdyn.so拷到Main.java同级目录下
# 6. 把jna-3.0.9.jar拷到Main.java同级目录下
java -cp .:jna-3.0.9.jar Main # 7. 编译
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH # 8. 设置so搜索目录
java -cp .:jna-3.0.9.jar Main # 9. 运行后报错：
# java.lang.UnsatisfiedLinkError: Error looking up function 'test': ./libdyn.so: undefined symbol: test
# 意思说test在libdyn.so里没找到，那查看下有没有：
um libdyn.so # 10. 查看动态库里的方法
# 结果没真没有 test 方法
```

```c++
// 11. 修改 dyn.h
#include <stdio.h>

#ifdef __cplusplus
extern "C" {
    void test();
}
#endif
// 12. 再次编译后放到Main.java一起
nm libdyn.so // 13. 再次查看发现有test了
// 14. java侧再编译再运行，ok 结果对了：
OpenJDK 64-Bit Server VM warning: You have loaded library /tmp/jna2443187850364428701.tmp which might have disabled stack guard. The VM will try to fix the stack guard now.
It's highly recommended that you fix the library with 'execstack -c <libfile>', or link it with '-z noexecstack'.
haha test // 这里
```

