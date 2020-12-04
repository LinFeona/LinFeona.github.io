

# C TCP socket 简单 client/server 连接收发

- 配置windows socket环境
- 头文件 lib库引入 

```C
#ifdef WIN32 // WIN32 才有的宏, Linux宏不存在
#include <WinSock2.h>
#include <Windows.h>
#pragma comment (lib, "WSOCK32.LIB")
#endif
```

- 配置socket版本 

```C
#ifdef WIN32
	WORD wVersionRequested;
	WSADATA wsaData;
	wVersionRequested = MAKEWORD(2, 2);
	ret = WSAStartup(wVersionRequested, &wsaData);
	if (ret != 0) {
		printf("WSAStart up failed\n");
		system("pause");
		return -1;
	}
#endif

```

- 监听
```C
	// step1 创建一个监听的socket;
	int s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP); // TCP
	if (s == INVALID_SOCKET) { // 创建
		goto failed;
	}
	// end 

	// ip地址 + 端口,监听到哪个IP地址和端口上;
	struct sockaddr_in sockaddr;
	sockaddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	sockaddr.sin_family = AF_INET;
	sockaddr.sin_port = htons(6080); // 127.0.0.1: 6080端口上;
	ret = bind(s, (const struct sockaddr*)&sockaddr, sizeof(sockaddr));
	if (ret != 0) {
		goto failed;
	}
	// 开启监听
	ret = listen(s, 1); 

```

- 客户端接入

```C
    struct sockaddr_in c_address; // 客户端的IP地址;
		int address_len = sizeof(c_address);
		// cs 是我们服务端为客户端创建的配对的socket;
		// c_address 就是我们客户端的IP地址和端口;
		printf("waiting....!!!!\n");
		int cs = accept(s, (struct sockaddr*)&c_address, &address_len); // 在这里像卡住了一样的；OS
		printf("new client %s：%d\n", inet_ntoa(c_address.sin_addr), ntohs(c_address.sin_port));
    
```

## 客户端配置 与服务器一致
  
- 客户端连接

```C

//创建与服务器连接的socket
int s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (s == INVALID_SOCKET) {
		goto failed;
	}
	// 配置一下要连接服务器的socket
	// 127.0.0.1 本机IP地址;
	struct sockaddr_in sockaddr;
	sockaddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	sockaddr.sin_family = AF_INET;
	sockaddr.sin_port = htons(6080); // 连接信息要发送给监听socket;
	// 发送连接请求到服务端的监听socket;
	ret = connect(s, &sockaddr, sizeof(sockaddr));
	if (ret != 0) {
		goto failed;
	}

	// 连接成功, s与服务器对应的socket就会建立连接;
	// 客户端在连接的时候他也需要一个IP地址+端口;
	// 端口是服务器端口。不是，客户端一个没有使用的端口就可以了;
	// 客户端自己也会分配一个IP + 端口(只要是没有使用的就可以了);
	char buf[11];
	memset(buf, 0, 11);
	send(s, "Hello", 5, 0);
	recv(s, buf, 5, 0);
	printf("%s\n", buf);

```






- server 代码

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// 配置windows socket环境
#ifdef WIN32 // WIN32 宏, Linux宏不存在
#include <WinSock2.h>
#include <Windows.h>
#pragma comment (lib, "WSOCK32.LIB")
#endif
// end 


int main(int argc, char** argv) {
	int ret;
	// 配置一下windows socket 版本
	// 一定要加上这个，否者低版本的socket会出很多莫名的问题;
#ifdef WIN32
	WORD wVersionRequested;
	WSADATA wsaData;
	wVersionRequested = MAKEWORD(2, 2);
	ret = WSAStartup(wVersionRequested, &wsaData);
	if (ret != 0) {
		printf("WSAStart up failed\n");
		system("pause");
		return -1;
	}
#endif

	// step1 创建一个监听的socket;
	int s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP); // TCP
	if (s == INVALID_SOCKET) { // 创建
		goto failed;
	}
	// end 

	// ip地址 + 端口,监听到哪个IP地址和端口上;
	struct sockaddr_in sockaddr;
	sockaddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	sockaddr.sin_family = AF_INET;
	sockaddr.sin_port = htons(6080); // 127.0.0.1: 6080端口上;
	ret = bind(s, (const struct sockaddr*)&sockaddr, sizeof(sockaddr));
	if (ret != 0) {
		goto failed;
	}
	// 开启监听
	ret = listen(s, 1); 

	while (1) {
		// 等待客户介入进来;
		struct sockaddr_in c_address; // 客户端的IP地址;
		int address_len = sizeof(c_address);
		// cs 是服务端为客户端创建的配对的socket;
		// c_address 就是客户端的IP地址和端口;
		printf("waiting....!!!!\n");
		int cs = accept(s, (struct sockaddr*)&c_address, &address_len); // 在这里任务挂起
		printf("new client %s：%d\n", inet_ntoa(c_address.sin_addr), ntohs(c_address.sin_port));

		// 收数据;
		char buf[11];
		memset(buf, 0, 11);
		recv(cs, buf, 5, 0);
		printf("recv: %s\n", buf);
		// end 

		// 发数据给客户端
		send(cs, buf, 5, 0);
		// end 
		closesocket(cs);
	}
	
failed:
	if (s != INVALID_SOCKET) {
		closesocket(s);
	}
// 结束的时候也要清理
#ifdef WIN32
	WSACleanup();
#endif
// end 
	system("pause");
	return 0;
}
    
```



- 客户端代码

```C

#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#ifdef WIN32 // WIN32 宏, Linux宏不存在
#include <WinSock2.h>
#include <Windows.h>
#pragma comment (lib, "WSOCK32.LIB")
#endif


int main(int argc, char** argv) {
	int ret;
	// 配置一下windows socket 版本
	// 一定要加上这个，否者低版本的socket会出很多莫名的问题;
#ifdef WIN32
	WORD wVersionRequested;
	WSADATA wsaData;
	wVersionRequested = MAKEWORD(2, 2);
	ret = WSAStartup(wVersionRequested, &wsaData);
	if (ret != 0) {
		printf("WSAStart up failed\n");
		system("pause");
		return -1;
	}
#endif

	int s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (s == INVALID_SOCKET) {
		goto failed;
	}
	// 配置一下要连接服务器的socket
	// 127.0.0.1 本机IP地址;
	struct sockaddr_in sockaddr;
	sockaddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	sockaddr.sin_family = AF_INET;
	sockaddr.sin_port = htons(6080); // 连接信息要发送给监听socket;
	// 发送连接请求到我们服务端的监听socket;
	ret = connect(s, &sockaddr, sizeof(sockaddr));
	if (ret != 0) {
		goto failed;
	}

	// 连接成功, s与服务器对应的socket就会建立连接;
	// 客户端在连接的时候他也需要一个IP地址+端口;
	// 端口是服务器端口。不是，客户端一个没有使用的端口就可以了;
	// 客户端自己也会分配一个IP + 端口(只要是没有使用的就可以了);
	// 
	char buf[11];
	memset(buf, 0, 11);
	send(s, "Hello", 5, 0);
	recv(s, buf, 5, 0);
	printf("%s\n", buf);

failed:
	if (s != INVALID_SOCKET) {
		closesocket(s);
		s = INVALID_SOCKET;
	}
	
#ifdef WIN32
	WSACleanup();
#endif

	system("pause");
	return 0;
}



```




