
# C语言 libuv 定时器


## 传统的libuv 定时器

```C
static uv_timer_t timer; //时间句柄
uv_timer_init(event_loop, &timer); //提交一个时间请求
uv_timer_start(&timer, on_uv_timer, 5000, 1000);//添加一个时间事件
uv_timer_stop(handle); // 停止这个timer 
```

- code

```C

static uv_loop_t* event_loop = NULL; //事件循环
static uv_timer_t timer; //时间句柄

// 时间到了 回调
static void on_uv_timer(uv_timer_t* handle) {
	printf("timer called\n");
	uv_timer_stop(handle); // 停止这个timer
}


void stratTime() {

	event_loop = uv_default_loop();
	uv_timer_init(event_loop, &timer);
  // timeout: 第一运行的时候是隔多少时间运行 repeat: 后面每隔多少时间(毫秒)调用一次;
	uv_timer_start(&timer, on_uv_timer, 5000, 1000);


	uv_run(event_loop, UV_RUN_DEFAULT);

}
  
```

---

## 封装定时器

- 获取系统时间搓

```C
 
// 获取当前系统从开机到现在运行了多少毫秒;
#ifdef WIN32
#include <windows.h>
static unsigned int
get_cur_ms() {
	return GetTickCount();
}
#else
#include <sys/time.h>  
#include <time.h> 
#include <limits.h>

static unsigned int
get_cur_ms() {
	struct timeval tv;
	// struct timezone tz;
	gettimeofday(&tv, NULL);

	return ((tv.tv_usec / 1000) + tv.tv_sec * 1000);
}
#endif 

```

---

- code dome 
- time_list.h

```C

#ifndef __MY_TIMER_LIST_H__
#define __MY_TIMER_LIST_H__

// on_timer是一个回掉函数,当timer触发的时候调用;
// udata: 是用户传的自定义的数据结构参数;
// after_sec: 多少秒开始执行;
// repeat_count: 执行多少次, repeat_count == -1一直执行;
// 返回timer的句柄;
struct timer;
struct timer* schedule(void(*on_timer)(void* udata), void* udata, int after_msec,int repeat_count);


// 取消掉这个timer;
void cancel_timer(struct timer* t);

//只调用一次 无repeat_count参数
struct timer*schedule_once(void(*on_timer)(void* udata), void* udata,int after_msec);
#endif

```

--- 

- time_list.c

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#include "uv.h"
#include "time_list.h"

#define my_malloc malloc
#define my_free free


//计时器参数结构
struct timer {
	uv_timer_t uv_timer; // libuv timer handle
	void(*on_timer)(void* udata);
	void* udata;
	int repeat_count; // -1一直循环;
};

//time结构内存分配 并初始化
static struct timer* alloc_timer(void(*on_timer)(void* udata),void* udata, int repeat_count) {

	struct timer* t = my_malloc(sizeof(struct timer));
	memset(t, 0, sizeof(struct timer));

	t->on_timer = on_timer;
	t->repeat_count = repeat_count;
	t->udata = udata;
  //将时间结构的 uv_timer句柄 事件提交到loop
	uv_timer_init(uv_default_loop(), &t->uv_timer);
	return t;
}

//释放掉一个计时器
static void free_timer(struct timer* t) {
	my_free(t);
}


//计时器 回调函数
static void on_uv_timer(uv_timer_t* handle) {
	struct timer* t = handle->data;
	if (t->repeat_count < 0) { // 不断的触发;
    //用户定于的函数
		t->on_timer(t->udata);
	}
	else {
		t->repeat_count --;
		t->on_timer(t->udata);
		if (t->repeat_count == 0) { // 函数time结束
			uv_timer_stop(&t->uv_timer); // 停止这个timer
			free_timer(t);
		}
	}
	
}


//启动一个计时器
struct timer*schedule(void(*on_timer)(void* udata),void* udata,int after_msec,int repeat_count) {

  //将计时器结构初始化
	struct timer* t = alloc_timer(on_timer, udata, repeat_count);

	// 启动一个timer;
	t->uv_timer.data = t;//将用户参数 赋给 句柄—>data
  
  //开始启用UV计时
	uv_timer_start(&t->uv_timer, on_uv_timer, after_msec, after_msec);
	// end 
	return t;
}

//释放掉一个计时器
void cancel_timer(struct timer* t) {
  //如果已经计时结束了
	if (t->repeat_count == 0) { // 全部触发完成，;
		return;
	}
  //还未结束就终止它
	uv_timer_stop(&t->uv_timer);
	free_timer(t);
}

//启动一个计时器
struct timer*schedule_once(void(*on_timer)(void* udata)void* udata,int after_msec) {
	return schedule(on_timer, udata, after_msec, 1);
}


```


