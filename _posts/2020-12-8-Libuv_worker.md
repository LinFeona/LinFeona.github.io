
# C语言Libuv 工作队列


- 相关API

1: 线程相关函数:
 
 ```C
    uv_thread_create: 创建一个线程;
    uv_thread_self: 获取当前线程id号;
    uv_thread_join: 等待线程结束;
```

2: 锁:

```
    uv_mutex_init: 初始化锁;
    uv_mutex_destroy: 销毁锁;
    uv_mutex_lock: 获取锁，如果被占用，就等待;
    uv_mutex_trylock: 尝试获取锁，如果获取不到，直接返回,不等待;
    uv_mutex_unlock: 释放锁;
```
    
3: 等待/触发事件;

```
    uv_cond_init: 创建条件事件;
    uv_cond_destroy: 销毁条件事件;
    uv_cond_signal: 触发条件事件;
    uv_cond_broadcast: 广播条件事件;
    uv_cond_wait/uv_cond_timedwait: 等待事件/等待事件超时;
```


- dome

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#include "uv.h"

// 工作队列的用处:
// 1：复杂的算法放到工作队列 ；
// 2：IO放到工作队列,获取数据库结果;
// ....

// 是在线程池里面另外一个线程里面调用，不在主线程;
static void 
thread_work(uv_work_t* req) {
	// 
	printf("user data = %d \n", (int)req->data);
	printf("thread_work id 0x%d:\n", uv_thread_self());
}

// 当工作队列里面的线程执行完上面的任务后，通知主线程;
// 主线程调用这个函数;
static void 
on_work_complete(uv_work_t* req, int status) {
	printf("on_work_complete thread id 0x%d:\n", uv_thread_self());
}

int main(int argc, char** argv) {
	uv_work_t uv_work;
	printf("main id 0x%d:\n", uv_thread_self());
	uv_work.data = (void*)6;
	uv_queue_work(uv_default_loop(), &uv_work, thread_work, on_work_complete);

	uv_run(uv_default_loop(), UV_RUN_DEFAULT);
	system("pause");
	return 0;
}
```


