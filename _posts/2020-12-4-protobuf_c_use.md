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

-- proto 文件
```proto
  message Person {  
  required string name = 1;   required 表示必须要设置值的
  required int32 age = 2;  
  optional string email = 3;  optional 可以不设值
}  
```


-- 生成编码解码器

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
 
 - 编译如果编译不通过 
 -- 右键解决方案-> 属性->c/c++ -> 常规 ->SDL检查 设置为否
 -- 右键解决方案-> 属性->c/c++ -> 代码生成 -> 运行库 设为多线程调试(/MTD) 或者MT 模式
 
 
 ### get/set数据  
 
  --- protobuf流程 将数据序列化 -> 传输 --> 反解 --> get数据
  
  ```C++
  
   //存
    Person p; // 编辑proto协议文件中的类名
    p.set_name("name");
    p.set_age(3);
    p.set_email("email");

    //取  .c_str() 是因为返回的是string 不是传统的C 
    printf("%s \n %d \n%s\n", p.name().c_str(), p.age(), p.email().c_str());

    //序列化
    string out;
    p.SerializeToString(&out);
   
      //反序列化
    Person p2;
    p2.ParseFromString(out);
    cout << p2.name() << p2.age() << p2.email()<<endl;
  
  ```
  
  
  
  

