# Protobuf Cmake vs2019的构建编译 C++ 平台

## 构建

- 下载protobuf源码：[protobuf传送门]https://github.com/google/protobuf
- 下载构建工具cmake: [cmake传送门]https://cmake.org/download/
- (可阅读 protobuf/cmake/README.md)文档获取详细内容
- 下载 [googlemock传送门]https://github.com/google/googlemock.git 改名为gmock 放到protobuf目录下
- 下载 [googletest传送门]https://github.com/google/googletest.git 改名为 gtest 放到 protobuf/gmock目录下
- 打开Cmake工具 构建目录选择 protobuf/cmake目录 输出目录自定义 
- configcue 选择vs版本 即 等待 Configuring done 输出后 点击generate生成

 ---
 
 ## 编译 
 
 - 打开vs工程 需要构建三个项目
 -- libprotobuf 自己的项目需要用
 -- libprotoc 协议代码 在运行编码解码时候需要调度的库
 -- protoc protobuf的编译器 可生成不同语言版本协议文件
 - 头文件在protobuf/src/google/protobuf目录下 c/c++目录
