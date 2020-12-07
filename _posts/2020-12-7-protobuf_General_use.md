

# C++ protobuf 封装




 
 - 只要传递协议描述 的消息名 就能构建出对应的实例 使用 基类保存子类
 
 - 协议
  
```proto
  syntax = "proto2"; 
  message Person {  
  required string name = 1;  
  required int32 age = 2;  
  optional string email = 3;  
  repeated int32 int_set = 4;
}

```
 
 
 - 构建工厂 
 
```C++
google::protobuf::Message* create_message(const char* type_name) {
	google::protobuf::Message* message = NULL;

	// 根据名字，找到message的秒速对象;
	const google::protobuf::Descriptor* descriptor =
		google::protobuf::DescriptorPool::generated_pool()->FindMessageTypeByName(type_name);
	if (descriptor) {
		// 根据描述对象到对象工厂里面，生成对应的模板对象;
		// 根据模板复制出来一个;
		const google::protobuf::Message* prototype =
			google::protobuf::MessageFactory::generated_factory()->GetPrototype(descriptor);
		if (prototype) {
			message = prototype->New();
		}
	}
	return message;
}


```

- 使用

```C++
  void test(){
    // 基类的message--> Person类的实例;
	google::protobuf::Message* msg = create_message("Person"); 

  //强转
	/*Person* blake = (Person*)msg;
	blake->set_name("blake");
	printf("%s\n", blake->name().c_str());*/


	/* 每一个Message对象有包含了两个对象; 
	   google::protobuf::Descriptor   --> 获取Message的描述信息-->包含每一个filed的描述; 
	   google::protobuf::Reflection   --> 使用它 + filed描述，能get/set 每个 filed的值;
	*/

	const google::protobuf::Descriptor* des = msg->GetDescriptor();
	const google::protobuf::Reflection* ref = msg->GetReflection();

	// 设置;
	// name:
	const google::protobuf::FieldDescriptor* fd_des = des->FindFieldByName("name");
	ref->SetString(msg, fd_des, "blake");

	// age
	// fd_des = des->FindFieldByName("age");
	fd_des = des->FindFieldByNumber(2);
	ref->SetInt32(msg, fd_des, 34);
	// end 

	// email
	fd_des = des->FindFieldByName("email");
	ref->SetString(msg, fd_des, "blake@bycwedu.com");
	// end

	// array
	fd_des = des->FindFieldByName("int_set");
	ref->AddInt32(msg, fd_des, 1);
	ref->AddInt32(msg, fd_des, 2);
	ref->AddInt32(msg, fd_des, 3);
	ref->AddInt32(msg, fd_des, 4);
	// end

	// 遍历我们的Descriptor里面的每一个filed;
	for (int i = 0; i < des->field_count(); i++) {
		// 获取每一个field的描述对象;
		const google::protobuf::FieldDescriptor* fd = des->field(i);
		// (1)获取我们的名字; fd->name()
		printf("%s\n", fd->name().c_str());

		if (fd->is_repeated()) { // 当前我们的字段是一个数组;	FieldSize数组的长度;
			for (int walk = 0; walk < ref->FieldSize(*msg, fd); walk++) {
				switch (fd->cpp_type()) {
				case google::protobuf::FieldDescriptor::CppType::CPPTYPE_INT32:
					printf("%d\n", ref->GetRepeatedInt32(*msg, fd, walk));
					break;
				case google::protobuf::FieldDescriptor::CppType::CPPTYPE_STRING:
					printf("%s\n", ref->GetRepeatedString(*msg, fd, walk).c_str());
					break;
				// ...
				}
			}
			continue;
		}

		switch (fd->cpp_type()) {
		case google::protobuf::FieldDescriptor::CppType::CPPTYPE_INT32:
			printf("%d\n", ref->GetInt32(*msg, fd));
			break;
		case google::protobuf::FieldDescriptor::CppType::CPPTYPE_STRING:
			printf("%s\n", ref->GetString(*msg, fd).c_str());
			break;
		// ...
		}
		// 表示filed是一个数组,  fd->is_optional, fd->is_repeated
	}

	// 删除掉这个msg;
	delete msg;
	
  
  }
  
  
```


