# Protobuf Cmake vs 2019 编译 

- 下载protobuf源码：https://github.com/google/protobuf
- 下载构建工具cmake: https://cmake.org/download/
- (可阅读 protobuf/cmake/README.md)文档获取详细内容
- 下载 https://github.com/google/googlemock.git 改名为gmock 放到protobuf目录下
- 下载 https://github.com/google/googletest.git 改名为 gtest 放到 protobuf/gmock目录下
- 打开Cmake工具 构建目录选择 protobuf/cmake目录 输出目录自定义 
- configcue 选择vs版本 即 等待 Configuring done 输出后 点击generate生成

 ---
 
 - 打开vs工程 需要构建三个项目
 -- libprotobuf 自己的项目需要用
 -- libprotoc 协议代码 在运行编码解码时候需要调度的库
 -- protoc protobuf的编译器 可生成不同语言版本协议文件
 - 头文件在protobuf/src/google/protobuf目录下 c/c++目录
 - 在自己项目的 项目运行时的目录中创建protobuf目录 将编译出来的libprotobuf.lib libprotocd.lib放进去
 - 将protobuf项目目录 放到 自己项目的 3rd中
 -- 目录结构 --- 根目录->(3rd win32 src proto)(第三方 运行时 代码 协议)
 - 将协议工具 protoc.exe 放到 自己项目proto目录中 
 -- 将头文件包含目录 指定到 protobuf/src中
 -- 链接库文件包含目录 指定到 运行时目录下 protobuf中
 -- 在库文件命令行中添加库文件libprotobuf.lib libprotocd.lib
