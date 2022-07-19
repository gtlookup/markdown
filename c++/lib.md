# ctime

https://www.twle.cn/l/yufei/cplusplus/cplusplus-basic-ctime.html

```c++
#include <ctime>
// 当前时间
time_t t = time(NULL); // 当前时间，结果：1652091062

// 根据 tm 结构获取时间
tm dt = {0,0,0,1,4,122}; // 秒分时日月年，122 = 2022 - 1900
time_t t = mktime(&dt);  // 根据 tm 结构获取时间戳

char buf[32];
strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", &dt); // 结果：2022-05-01 00:00:00
```

# iostream

```c++
#include <iostream>

// 输入：std::cin
int a = 0;
std::cin >> a;
// 输出：std::cout
std::cout << a;
```

# 格式化

```c++
#include <sstream>
#include <iostream>

std::ostringstream os;
os << "haha," << 3 << endl;
cout << os.str(); // 结果：haha,3
```

