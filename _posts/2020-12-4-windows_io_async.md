
# C语言 window平台 异步IO 读写操作


- 头文件

```C
#include <Windows.h> 
```

## 同步

- 创建

```C
//io 句柄
HANDLE hfile = INVALID_HANDLE_VALUE;
//创建文件 
  // L 表示 unicode字符串: 每个字符是2个字节 GENERIC_READ 表示读文件 OPEN_EXISTING打开已经存在的文件 否报错
	hfile = CreateFile(L"in.txt", GENERIC_READ, 0, NULL,OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
	if (hfile == INVALID_HANDLE_VALUE) {
		printf("open error\n");
    CloseHandle(hfile); // 关闭一个文件
		system("pause");
	  return 0;
	}
  
```

- 读

```C
	// 准备好内存;
	char buf[1024];
	int readed;
	// 这个任务将会被挂起,直到磁盘读好了数据，才会唤醒;
	ReadFile(hfile, buf, 1024, &readed, NULL); 
	buf[readed] = 0;
	printf("%s\n", buf);
	CloseHandle(hfile); // 关闭一个文件;
```


## 异步

- 创建

```C
//FILE_FLAG_OVERLAPPED模式;
HANDLE hfile = CreateFile(L"in.txt", GENERIC_READ, 0, NULL,
		OPEN_EXISTING, FILE_FLAG_OVERLAPPED | 
		FILE_ATTRIBUTE_NORMAL, NULL);
	if (hfile == INVALID_HANDLE_VALUE) {
		printf("open error\n");
    CloseHandle(hfile); // 关闭一个文件
		system("pause");
	  return 0;
	}

```

- 读

```C
//创建异步OVERLAPPED句柄 
  OVERLAPPED ov;
  //创建事件
	HANDLE hevent = CreateEvent(NULL, FALSE, FALSE, NULL);
	memset(&ov, 0, sizeof(ov));
	ov.hEvent = hevent; // 事件对象，完成请求请求触发;
	ov.Offset = 2; // 从哪里开始读起;
	// 马上返回,IO挂起,没有读到数据
	ReadFile(hfile, buf, 1024, &readed, &ov); // 不是马上会有的;
	if (GetLastError() == ERROR_IO_PENDING) { // 表示正在等待;
		// 等待数据来读完成;
		WaitForSingleObject(hevent, INFINITE);
		readed = ov.InternalHigh; // 读到了几个字节的数据;
		buf[readed] = 0;
		printf("%s\n", buf);
		CloseHandle(hfile); // 关闭一个文件
	}
	else {
		CloseHandle(hfile); // 关闭一个文件
	}



```



## dome

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#include <Windows.h> // 你将获得大部分的Windows API接口
// 句柄: 这个句柄是这个对象的唯一的标识;
// unicode字符串: 每个字符是2个字节, L
// OPEN_EXISTING 打开存在文件，如果不存在就会失败;
int main(int argc, char** argv) {
	// sync模式
	HANDLE hfile = INVALID_HANDLE_VALUE;
	hfile = CreateFile(L"in.txt", GENERIC_READ, 0, NULL,
		OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
	if (hfile == INVALID_HANDLE_VALUE) {
		printf("open error\n");
		goto failed;
	}
	// 准备好内存;
	char buf[1024];
	int readed;
	// 当我们正在读取这个磁盘的数据的时候,
	// 这个任务将会被挂起,直到磁盘读好了数据，才会唤醒;
	// 同步;
	ReadFile(hfile, buf, 1024, &readed, NULL); 
	buf[readed] = 0;
	printf("%s\n", buf);
	CloseHandle(hfile); // 关闭一个文件;

	// async 异步, FILE_FLAG_OVERLAPPED模式;
	// step1: 以异步的模式打开;
	hfile = CreateFile(L"in.txt", GENERIC_READ, 0, NULL,
		OPEN_EXISTING, FILE_FLAG_OVERLAPPED | 
		FILE_ATTRIBUTE_NORMAL, NULL);
	if (hfile == INVALID_HANDLE_VALUE) {
		printf("open error\n");
		goto failed;
	}

	// step2: 读时候，发送请求, 准备一个OVERLAPPED这个对象;
	// 传给我们的OS，等我们的OS读完以后，
	// 会通过OVERLAPPED来给我们发送一个事件;
	OVERLAPPED ov;
	HANDLE hevent = CreateEvent(NULL, FALSE, FALSE, NULL);
	memset(&ov, 0, sizeof(ov));
	ov.hEvent = hevent; // 事件对象，完成请求请求触发;
	ov.Offset = 2; // 从哪里开始读起;
	// 马上返回,IO挂起,没有读到数据
	ReadFile(hfile, buf, 1024, &readed, &ov); // 不是马上会有的;
	if (GetLastError() == ERROR_IO_PENDING) { // 表示正在等待;
		// 等待数据来读完成;
		WaitForSingleObject(hevent, INFINITE);
		readed = ov.InternalHigh; // 读到了几个字节的数据;
		buf[readed] = 0;
		printf("%s\n", buf);
		CloseHandle(hfile); // 关闭一个文件
	}
	else {
		CloseHandle(hfile); // 关闭一个文件
	}
	// 数据还没有准备好,不能马上使用，但是你可以做别的;

	// ...
	// ...
	// ..
	
	// end 

	// 都是等，有什么区别呢？
	// 1> 同步等，API内部等，直接挂起;
	// 2> 异步等，是用户自己来写代码来等待;
	// 3> 我们用户，可以同时等待多个请求, 并发请求数;
	// WaitForMultipleObjects
failed:
	system("pause");
	return 0;
}
```


