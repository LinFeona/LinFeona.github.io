

# C语言 libuv tcp socket


> 头文件引入

```C
#include "uv.h"
```

> loop 循环

```C
  //循环句柄 最好定义成全局的 
 static uv_loop_t* loop = NULL; 
  //给循环句柄初始化
 loop = uv_default_loop(); 
 //启动事件循环
 uv_run(loop, UV_RUN_DEFAULT); //这里没有事件需要循环去等待 循环就会结束
 
 ```
 
 > TCP监听服务
 
 ```C
 
  //循环句柄 最好定义成全局的 
 static uv_loop_t* loop = NULL; 
 static uv_tcp_t l_server; // 监听句柄 最好也要定为全局的
 
 
 void tcp(){
 
  loop = uv_default_loop(); 
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
 
  //最后需要启动循环
  uv_run(loop, UV_RUN_DEFAULT);
 
 }
 
 
```

> 监听客户端接入回调

```C
static void uv_connection(uv_stream_t* server, int status) {
	printf("new client comming\n");
	
	// 接入客户端;
	uv_tcp_t* client = malloc(sizeof(uv_tcp_t));
	memset(client, 0, sizeof(uv_tcp_t));
	uv_tcp_init(loop, client);
	uv_accept(server, (uv_stream_t*)client);
	// end

	// 将这个客户端加入事件循环 监听这个客户端的事件
	//这里需要两个回调 一个是客户端发来数据分配内存的回调 一个是读取数据回调 调用顺序 uv_alloc_buf   after_read
	uv_read_start((uv_stream_t*)client, uv_alloc_buf, after_read);
}
```

> 分配内存回调函数

```C
参数( 发生事件的句柄  需要的字节长度 需要将内存分配的容器)
static void uv_alloc_buf(uv_handle_t* handle,size_t suggested_size,uv_buf_t* buf) {

	if (handle->data != NULL) {
		free(handle->data);
		handle->data = NULL;
	}

	buf->base = malloc(suggested_size + 1); //给容器分配内存 +1因为需要1个字节存结尾符
	buf->len = suggested_size; 
	handle->data = buf->base;
}

```

> 读取数据回调函数

```C

static void after_read(uv_stream_t* stream,
                       ssize_t nread,
					   const uv_buf_t* buf) {
	// 连接断开了;
	if (nread < 0) {
		uv_shutdown_t* reg = malloc(sizeof(uv_shutdown_t));
		memset(reg, 0, sizeof(uv_shutdown_t));
		uv_shutdown(reg, stream, on_shutdown);
		return;
	}
	// end

	buf->base[nread] = 0;
	printf("recv %d\n", nread);
	printf("%s\n", buf->base);

	// 测试发送给我们的 客户端;
	uv_write_t* w_req = malloc(sizeof(uv_write_t));
	uv_buf_t* w_buf = malloc(sizeof(uv_buf_t));
	w_buf->base = buf->base;
	w_buf->len = nread;
	w_req->data = w_buf;
	uv_write(w_req, stream, w_buf, 1, after_write);
}

```


