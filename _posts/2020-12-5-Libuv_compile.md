

# Libuv c语言中的环境配置
  
## 简介
- libuv是跨平台的异步事件库

## libuv 下载
- [libuv官网](http://libuv.org/)
- [下载直通车](https://dist.libuv.org/dist/)
- 选择一个版本 下载 `gz` 后缀的源码

## 环境配置
- 项目目录 `src` `win32` `3rd`  目录 代码 运行时目录 第三方代码
- 将 libuv/src 与 include 复制到 项目的3rd/libuv目录中 无libuv目录则创建
- 将项目附件包含头文件目录 添加 3rd/libuv/include目录

- 引入头文件

```C
  #include<uv.h>
```

- 编译不通过则 右击项目解决方案 -> 属性 -> C/C++ -> 常规 -> SDL检查 设为否
- 链接lib库

```C
#pragma comment(lib, "ws2_32.lib")
#pragma comment(lib,"Psapi.lib")
#pragma comment(lib,"Iphlpapi.lib")
#pragma comment(lib, "Userenv.lib")
```

- 若还是编译不通过 则是头文件再包含一个 3rd/libuv/src 目录 因为有几个头文件在src目录 而C文件工作目录在src/win 则需要返回上一级

```C
#include "uv.h"
#include "internal.h"
#include "../queue.h" //这个
#include "handle-inl.h"
#include "../heap-inl.h" //像这个
#include "req-inl.h"
```


- 因为考虑到跨平台 链接库文件还需要放在 链接器的命令行中 因为#pragma comment 是win才有的 
- so 右击 项目解决方案 -> 属性 -> 链接器 -> 命令行 添加 ws2_32.lib Psapi.lib Iphlpapi.lib Userenv.lib  -> 应用

-则将原来的 pragma 删掉

```C
#pragma comment(lib, "ws2_32.lib")
#pragma comment(lib,"Psapi.lib")
#pragma comment(lib,"Iphlpapi.lib")
#pragma comment(lib, "Userenv.lib")
```


## 另外 libuv/test 目录下是 官方写的测试代码 可以去阅读查看


