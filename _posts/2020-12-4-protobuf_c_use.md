# protobuf 在C++中的使用


## 前置配置

- 项目创建的目录结构 src win32 3rd proto 分别为 代码 运行时目录 第三方库 协议
- 将protobuf的三个库构建出来 – libprotobuf  libprotoc  protoc  
- lib库文件放置到win32/protobuf目录中 没有目录就创建
- protoc放到 proto目录中 用于编辑协议描述文件
- 将protobuf 未构建的目录 整个目录复制到3rd中
- 将项目头文件包含目录 指定到 protobuf/src
- 将项目链接库文件包含目录 指定到 运行时目录下 protobuf中
- 在库文件命令行中添加库文件libprotobuf.lib libprotocd.lib

## 协议描述文件的编辑使用

- proto 文件

```proto
  message Person {  
  required string name = 1;  
  required int32 age = 2;  
  optional string email = 3;  
}  

```

- 生成编码解码器
```cmd

cd proto
protoc.exe --cpp_out=./ filename.proto

//更多命令参数 请输入 protoc.exe -h

```

## C++项目中的使用
 - 将生成的.cc 与.h 添加到项目中
 - #include到模块中
 
 ```C++
 #include"../proto/Person.pb.h"
 ```
 

