# 包含stdafx.h报错

>无法打开 源 文件 "stdafx.h"
>无法打开包括文件: “stdafx.h”: No such file or directory

`stdafx.h` 不是 `c++` 标准库里的，是 `visual studio` 使用的预编译头，一般不需要加

# 未定义标识符

如果工程引入第三方库(.h)时，出现`E0010 未定义标识符"xxx"` 错误，此时需要在第三方库头文件上面添加：

- DWORD：`#include <windows.h>`
- _TCHAR：`#include <tchar.h>`

# 文件无效或损坏

> LNK1107 文件无效或损坏:无法在 0x2D0 处读取

原因：链接时使用了dll，而不是lib。lib是编译时需要的，dll是运行时需要的。

## windows 下的动/静态库

动态库：

- 生成动态库时除了dll还有与之对应的lib；
- 此时的lib不是**静态库**而是编译时**动态库的导入库**，因此编译时需要dll和lib；
- 运行时需要dll

静态库：只有一个lib库，编译时需要将lib编译到程序中，因此运行时不再需要lib