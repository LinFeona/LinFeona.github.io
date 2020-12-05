

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

	//检查句柄寄存器 数据内存有没有被释放
	if (handle->data != NULL) {
		free(handle->data);
		handle->data = NULL;
	}

	buf->base = malloc(suggested_size + 1); //给容器分配内存 +1因为需要1个字节存结尾符
	buf->len = suggested_size; 
	handle->data = buf->base; //给句柄的存储器指向这块地址 方便下一次使用释放 避免内存泄漏的问题
}

```

> 读取数据回调函数
 
```C
//参数 发生事件的句柄 读取到的长度 数据内存
static void after_read(uv_stream_t* stream,ssize_t nread,const uv_buf_t* buf) {

	// 连接断开了;
	if (nread < 0) {
		uv_shutdown_t* reg = malloc(sizeof(uv_shutdown_t));
		memset(reg, 0, sizeof(uv_shutdown_t));
		//链接断开回调设定
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

> 客户端断开 回调函数

```C
static void on_shutdown(uv_shutdown_t* req, int status) {
	//绑定完全断开事件
	uv_close((uv_handle_t*)req->handle, on_close);
	free(req);
}

```

> 客户端完全断开

```C
static void on_close(uv_handle_t* handle) {
	printf("close client\n");
	//如果句柄储存器里还有数据释放
	if (handle->data) {
		free(handle->data);
		handle->data = NULL;
	}
}


```


> 服务器发送数据给客户端

```C
//回调
static void after_write(uv_write_t* req, int status) {
	if (status == 0) {
		printf("write success\n");
	}
	uv_buf_t* w_buf = req->data;
	if (w_buf) {
		free(w_buf);
	}

	free(req);
}

void send_data(){
	// 测试发送给我们的 客户端;
	uv_write_t* w_req = malloc(sizeof(uv_write_t));
	uv_buf_t* w_buf = malloc(sizeof(uv_buf_t));
	w_buf->base = data;
	w_buf->len = len;
	w_req->data = w_buf;
	uv_write(w_req, stream, w_buf, 1, after_write);

}

```

> Dome

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#include "uv.h"

/*
uv_handle_s 数据结构:
UV_HANDLE_FIELDS

uv_stream_t  数据结构;
UV_HANDLE_FIELDS
UV_STREAM_FIELDS

uv_tcp_t 数据结构;
UV_HANDLE_FIELDS
UV_STREAM_FIELDS
UV_TCP_PRIVATE_FIELDS

uv_tcp_t is uv_stream_t is uv_handle_t;
*/

static uv_loop_t* loop = NULL;
static uv_tcp_t l_server; // 监听句柄;

// 当我们的event loop检车到handle上有数据可以读的时候,
// 就会调用这个函数, 让这个函数给event loop准备好读入数据的内存;
// event loop知道有多少数据,suggested_size,
// handle: 发生读时间的handle;
// suggested_size: 建议我们分配多大的内存来保存这个数据;
// uv_buf_t: 我们准备好的内存，通过uv_buf_t,告诉even loop;
static void 
uv_alloc_buf(uv_handle_t* handle,
                  size_t suggested_size,
				  uv_buf_t* buf) {

	if (handle->data != NULL) {
		free(handle->data);
		handle->data = NULL;
	}

	buf->base = malloc(suggested_size + 1);
	buf->len = suggested_size;
	handle->data = buf->base;
}

static void
on_close(uv_handle_t* handle) {
	printf("close client\n");
	if (handle->data) {
		free(handle->data);
		handle->data = NULL;
	}
}

static void 
on_shutdown(uv_shutdown_t* req, int status) {
	uv_close((uv_handle_t*)req->handle, on_close);
	free(req);
}

static void 
after_write(uv_write_t* req, int status) {
	if (status == 0) {
		printf("write success\n");
	}
	uv_buf_t* w_buf = req->data;
	if (w_buf) {
		free(w_buf);
	}

	free(req);
}

// 参数: 
// uv_stream_t* handle --> uv_tcp_t;
// nread: 读到了多少字节的数据;
// uv_buf_t: 我们的数据都读到到了哪个buf里面, base;
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

static void
uv_connection(uv_stream_t* server, int status) {
	printf("new client comming\n");
	// 接入客户端;
	uv_tcp_t* client = malloc(sizeof(uv_tcp_t));
	memset(client, 0, sizeof(uv_tcp_t));
	uv_tcp_init(loop, client);
	uv_accept(server, (uv_stream_t*)client);
	// end

	// 告诉event loop,让他帮你管理哪个事件;
	uv_read_start((uv_stream_t*)client, uv_alloc_buf, after_read);
}

int main(int argc, char** argv) {
	int ret;
	loop = uv_default_loop();
	// Tcp 监听服务;
	uv_tcp_init(loop, &l_server); // 将l_server监听句柄加入到event loop里面;
	// 你需要event loop来给你做那种管理呢？配置你要的管理类型;
	struct sockaddr_in addr;
	uv_ip4_addr("0.0.0.0", 6080, &addr); // ip地址, 端口
	ret = uv_tcp_bind(&l_server, (const struct sockaddr*) &addr, 0);
	if (ret != 0) {
		goto failed;
	}
	// 让event loop来做监听管理，当我们的l_server句柄上有人连接的时候;
	// event loop就会调用用户指定的这个处理函数uv_connection;
	uv_listen((uv_stream_t*)&l_server, SOMAXCONN, uv_connection);

	uv_run(loop, UV_RUN_DEFAULT);

failed:
	printf("end\n");
	system("pause");
	return 0;
}



```

