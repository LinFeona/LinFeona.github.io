

# C语言 libuv tcp socket


- 头文件引入

```C
#include "uv.h"
```

- API

```C
  //循环句柄 最好定义成全局的 
 static uv_loop_t* loop = NULL; 

  //给循环句柄初始化
 loop = uv_default_loop();
 
 //启动事件循环
 uv_run(loop, UV_RUN_DEFAULT); //这里没有事件需要循环去等待 循环就会结束
 
 //TCP监听服务
 static uv_tcp_t l_server; // 监听句柄;
 // 将l_server监听句柄加入到event loop里面;
 uv_tcp_init(loop, &l_server);  
 //配置管理类型
 struct sockaddr_in addr;
	uv_ip4_addr("0.0.0.0", 6080, &addr); // ip地址, 端口
	ret = uv_tcp_bind(&l_server, (const struct sockaddr*) &addr, 0);
	if (ret != 0) {
		printf("end\n");
	  system("pause");
	  return 0;
	}
  // 给这个事件加上回调函数
  // 监听的事件 监听的数目 回调函数 这个回调的参数(uv_stream_t* server, int status)
  //如果有链接进来了就会调用 回调函数  
 uv_listen((uv_stream_t*)&l_server, SOMAXCONN, 回调);
 
 
 
 //最后还需要重新启动循环
 uv_run(loop, UV_RUN_DEFAULT);
 
```




