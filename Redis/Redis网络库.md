
## Redis网络模块
> 在这里学习Redis网络库的代码实现, 这里仅仅学习Redis网络库, 而不是Redis服务器逻辑
> ae.c ，以及任意一个 ae_*.c 文件
> [网上对这些代码的整理](https://blog.csdn.net/yangbodong22011/article/details/65444273)
> [或者也可以直接git网络部分的代码](https://github.com/hurley25/ANet.git)

### Redis核心
> Redis是单线程模型, 核心使用I/O多路复用
#### I/O多路复用
在源码存在`ae_epoll.c`,`ae_select.c`等文件, 根据不同的系统选择不同的接口, 以下是在 `ae.c`中的代码
```
#ifdef HAVE_EVPORT
#include "ae_evport.c"
#else
    #ifdef HAVE_EPOLL
    #include "ae_epoll.c"
    #else
        #ifdef HAVE_KQUEUE
        #include "ae_kqueue.c"
        #else
        #include "ae_select.c"
        #endif
    #endif
#endif
```
默认情况下, Linux会使用`epoll`, windows下会使用`select`

#### 核心逻辑
> 对于这种I/O多路复用, 其核心逻辑必然是处于一个循环当中的

```
//ae.c:496
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        if (eventLoop->beforesleep != NULL)
            eventLoop->beforesleep(eventLoop);
        aeProcessEvents(eventLoop, AE_ALL_EVENTS|AE_CALL_AFTER_SLEEP);
    }
}
```
以上是Redis网络库的核心逻辑正式开始的部分, 暂时不考虑`beforesleep`的工作, 让我们进入`aeProcessEvents`真正处理的地方吧

#### 相关结构体
> 在进入之前先了解以下结构体
```
//ae.h:97
typedef struct aeEventLoop {
    int maxfd;   					/* 当前注册的最大文件描述符 */
    int setsize; 					/* 监控的最大文件描述符数 */
    long long timeEventNextId;		/* 定时事件ID */
    time_t lastTime;     			/* 最近一次处理定时事件的时间 */
    aeFileEvent *events; 			/* 注册事件链表 */
    aeFiredEvent *fired; 			/* 发生事件链表 */
    aeTimeEvent *timeEventHead;		/* 定时事件链表*/
    int stop;						/* 是否停止循环*/
    void *apidata; 					/* 特定接口的特定数据*/
    aeBeforeSleepProc *beforesleep;	/*在sleep之前执行的程序*/
    aeBeforeSleepProc *aftersleep;	/*在sleep之后执行的程序*/
} aeEventLoop;
```
以上是Redis的核心结构体, 我们可以看到循环的开始就是使用的该结构体, 在代码中对结构体做一些说明

```
#define AE_FILE_EVENTS 1								/*文件描述符事件*/
#define AE_TIME_EVENTS 2								/*定时器事件*/
#define AE_ALL_EVENTS (AE_FILE_EVENTS|AE_TIME_EVENTS)	/*两者的并集*/
#define AE_DONT_WAIT 4									
/*从代码上可以看出, 如果设置了该标志位, 不处理定时器相关事件, 多路复用IO也不等待*/
#define AE_CALL_AFTER_SLEEP 8
```

#### I/O多路复用的地方`aeProcessEvents`
```
//ae.h:71
typedef struct aeFileEvent {
    int mask; 					/*AE_(READABLE|WRITABLE|BARRIER), 事件的标志位 */
    aeFileProc *rfileProc;		/*可读事件的程序*/
    aeFileProc *wfileProc;		/*可写事件的程序*/
    void *clientData;			/*参数*/
} aeFileEvent;

int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
	int processed = 0, numevents;
	
	if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;
	/*在`aeMain`中传递的flag包含`AE_ALL_EVENTS(AE_FILE_EVENTS|AE_TIME_EVENTS), 这里直接跳过`*/

	if (eventLoop->maxfd != -1 ||
		((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
		/*只要当前有注册的文件描述符, 或者有监控定时器事件*/
        int j;
        aeTimeEvent *shortest = NULL;
        struct timeval tv, *tvp;

		if (flags & AE_TIME_EVENTS && !(flags & AE_DONT_WAIT))
			shortest = aeSearchNearestTimer(eventLoop);
		/*找到最近一个可能触发的定时器*/
        if (shortest) {
			long now_sec, now_ms;

            aeGetTime(&now_sec, &now_ms);
            tvp = &tv;
            /*获取当前时间, 很简单的函数, 不做介绍*/

			long long ms =
				(shortest->when_sec - now_sec)*1000 +
				shortest->when_ms - now_ms;

            if (ms > 0) {
                tvp->tv_sec = ms/1000;
                tvp->tv_usec = (ms % 1000)*1000;
            } else {
                tvp->tv_sec = 0;
                tvp->tv_usec = 0;
            }
            /*计算还需要等待的时长*/
        } else {
            if (flags & AE_DONT_WAIT) {
                tv.tv_sec = tv.tv_usec = 0;
                tvp = &tv;
            } else {
                tvp = NULL; /* wait forever */
            }
        }

        /*tvp将被设置为超时时间, 那么我们可以知道(假设对于select):
        * 	如果有定时器事件, 那么可以阻塞定时器对应的时长, 等待函数返回该定时器事件也刚好触发 
        *	如果设置了 `AE_DONT_WAIT`, 那么I/O函数立即返回, 也不处理定时器事件
        *	如果处理定时器事件, 但是没有可用定时器, 那么无限阻塞
        *	这样可以很好的把定时器和文件描述符事件结合
        */

		numevents = aeApiPoll(eventLoop, tvp);
		/*核心的多路复用 I/O, 调用后触发的事件就在`fired`中 */
		
		if (eventLoop->aftersleep != NULL && flags & AE_CALL_AFTER_SLEEP)
			eventLoop->aftersleep(eventLoop);


        for (j = 0; j < numevents; j++) {
            aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
            int mask = eventLoop->fired[j].mask;
            int fd = eventLoop->fired[j].fd;
            int fired = 0; 

            /*正常情况下, 先处理可读事件, 再处理可写事件
            * 如果设置了`AE_BARRIER`标志位, 则相反
            */
            int invert = fe->mask & AE_BARRIER;

            if (!invert && fe->mask & mask & AE_READABLE) {
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                fired++;
            }

            if (fe->mask & mask & AE_WRITABLE) {
                if (!fired || fe->wfileProc != fe->rfileProc) {
                    fe->wfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }

            if (invert && fe->mask & mask & AE_READABLE) {
                if (!fired || fe->wfileProc != fe->rfileProc) {
                    fe->rfileProc(eventLoop,fd,fe->clientData,mask);
                    fired++;
                }
            }
            /*执行可读可写相关程序, 如果可读可写程序一样的话, 不重复执行*/
            processed++;
        }
    }

    if (flags & AE_TIME_EVENTS)
        processed += processTimeEvents(eventLoop);

    return processed; /* 返回处理的事件个数 file/time events */
}

```
**注意**:
`aeFileEvent *events`该数组直接使用的就是文件描述符作为下标, Linux自然是可以的, 不过需要注意大小, 对于windows之后还需要走读一下事件初始化的代码


##### `aeSearchNearestTimer`
> 找到最近一个要触发的定时器

```
//ae.h:79
typedef struct aeTimeEvent {
    long long id; 	
    long when_sec; 							/*秒数*/
    long when_ms; 							/*毫秒*/
    aeTimeProc *timeProc;
    aeEventFinalizerProc *finalizerProc;
    void *clientData;
    struct aeTimeEvent *prev;				/*前一个*/
    struct aeTimeEvent *next;				/*下一个*/
} aeTimeEvent;

//ae.c:254
static aeTimeEvent *aeSearchNearestTimer(aeEventLoop *eventLoop)
{
    aeTimeEvent *te = eventLoop->timeEventHead;
    aeTimeEvent *nearest = NULL;

    while(te) {
        if (!nearest || te->when_sec < nearest->when_sec ||
                (te->when_sec == nearest->when_sec &&
                 te->when_ms < nearest->when_ms))
            nearest = te;
        te = te->next;
    }
    return nearest;
}
```
可以看出`timeEventHead`这个成员, 表示了一个定时器链表, 目前可以看出, 没有做什么特定的数据结构体排序
逐个比较, 找到时间最小的一个返回, 存储的应当是绝对时间

##### `aeApiPoll`
> 多路复用的IO, 根据系统情况的不同, 实际可能调用的也不一样, 我们这里仅仅介绍以下`epoll`
```
//ae_epoll.c:34
typedef struct aeApiState {
    int epfd;
    struct epoll_event *events;
} aeApiState;
/*epoll要用到的东西, epoll专用描述符/接受回传的结构体*/

//ae_epoll.c:108
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) {
    aeApiState *state = eventLoop->apidata;
    int retval, numevents = 0;

    retval = epoll_wait(state->epfd,state->events,eventLoop->setsize,
            tvp ? (tvp->tv_sec*1000 + tvp->tv_usec/1000) : -1);
    if (retval > 0) {
        int j;

        numevents = retval;
        for (j = 0; j < numevents; j++) {
            int mask = 0;
            struct epoll_event *e = state->events+j;

            if (e->events & EPOLLIN) mask |= AE_READABLE;
            if (e->events & EPOLLOUT) mask |= AE_WRITABLE;
            if (e->events & EPOLLERR) mask |= AE_WRITABLE;
            if (e->events & EPOLLHUP) mask |= AE_WRITABLE;
            eventLoop->fired[j].fd = e->data.fd;
            eventLoop->fired[j].mask = mask;
        }
    }
    return numevents;
}
```
和正常的处理相同, 调用`epoll_wait`等待事件触发, 将出发后的事件和类型存储到`fired`中

##### `processTimeEvents`

```
//ae.c:270
static int processTimeEvents(aeEventLoop *eventLoop) {
    int processed = 0;
    aeTimeEvent *te;
    long long maxId;
    time_t now = time(NULL);

    if (now < eventLoop->lastTime) {
        te = eventLoop->timeEventHead;
        while(te) {
            te->when_sec = 0;
            te = te->next;
        }
    }
    /*lastTime 表示上一次处理的时间
    *  	如果出现这种情况, 说明系统时间被迁移了
    *	提前处理所有的事件
    */

    eventLoop->lastTime = now;

    te = eventLoop->timeEventHead;
    maxId = eventLoop->timeEventNextId-1;
    while(te) {
		long now_sec, now_ms;
		long long id;
		
        if (te->id == AE_DELETED_EVENT_ID) {
            aeTimeEvent *next = te->next;
            if (te->prev)
                te->prev->next = te->next;
            else
                eventLoop->timeEventHead = te->next;
            if (te->next)
                te->next->prev = te->prev;
            if (te->finalizerProc)
                te->finalizerProc(eventLoop, te->clientData);
            zfree(te);
            te = next;
            continue;
        }
        /*如果id被设置为-1, 表示应该被删除*/

        if (te->id > maxId) {
            te = te->next;
            continue;
        }
        aeGetTime(&now_sec, &now_ms);
        if (now_sec > te->when_sec ||
            (now_sec == te->when_sec && now_ms >= te->when_ms))
        {
            int retval;

            id = te->id;
            retval = te->timeProc(eventLoop, id, te->clientData);
            processed++;
            if (retval != AE_NOMORE) {
                aeAddMillisecondsToNow(retval,&te->when_sec,&te->when_ms);
            } else {
                te->id = AE_DELETED_EVENT_ID;
            }
        }
        /*时间到了, 调用相关的处理函数, 如果有 AF_NOMORE, 则将ID设置为-1之后删除
        * 否则, 根据timeProc的返回值重新添加定时器
        */
        te = te->next;
    }
    return processed;
}
```

> ***以上就是核心的处理代码了, 定时器和文件描述符的结合很巧妙, 定时器还存在优化的可能性, 对于Linux来说其实可以直接使用timefd***

#### 对于结构体的操作函数
> 接下来介绍以下, 初始化/添加/删除等操作

##### `aeCreateEventLoop`
> 结构体的创建
```
//ae.c:63
aeEventLoop *aeCreateEventLoop(int setsize) {
    aeEventLoop *eventLoop;
    int i;

    if ((eventLoop = zmalloc(sizeof(*eventLoop))) == NULL) goto err;
    eventLoop->events = zmalloc(sizeof(aeFileEvent)*setsize);
    eventLoop->fired = zmalloc(sizeof(aeFiredEvent)*setsize);
    if (eventLoop->events == NULL || eventLoop->fired == NULL) goto err;
    eventLoop->setsize = setsize;
    eventLoop->lastTime = time(NULL);
    eventLoop->timeEventHead = NULL;
    eventLoop->timeEventNextId = 0;
    eventLoop->stop = 0;
    eventLoop->maxfd = -1;
    eventLoop->beforesleep = NULL;
    eventLoop->aftersleep = NULL;
    if (aeApiCreate(eventLoop) == -1) goto err;
    /* Events with mask == AE_NONE are not set. So let's initialize the
     * vector with it. */
    for (i = 0; i < setsize; i++)
        eventLoop->events[i].mask = AE_NONE;
    return eventLoop;

err:
    if (eventLoop) {
        zfree(eventLoop->events);
        zfree(eventLoop->fired);
        zfree(eventLoop);
    }
    return NULL;
}

```
* 1 根据传入的参数, 分配内存空间, `setsize`结构体就是最大的可接受文件描述符数量
* 2 调用`aeApiCreate`根据不同的系统创建异步I/O所需要的结构体
* 3 

##### `aeApiCreate`
> 根据系统不同, 选择的一步IO, 该函数的实际实现也不同, 这里介绍以下`select`和`epoll`的实现
```
//ae_epoll.c:39
static int aeApiCreate(aeEventLoop *eventLoop) {
    aeApiState *state = zmalloc(sizeof(aeApiState));

    if (!state) return -1;
    state->events = zmalloc(sizeof(struct epoll_event)*eventLoop->setsize);
    if (!state->events) {
        zfree(state);
        return -1;
    }
    state->epfd = epoll_create(1024); /* 1024 is just a hint for the kernel */
    if (state->epfd == -1) {
        zfree(state->events);
        zfree(state);
        return -1;
    }
    eventLoop->apidata = state;
    return 0;
}

```
主要是分配相关资源, 包括结构体空间/epoll_wait获取返回值的event/epool_fd

```
static int aeApiCreate(aeEventLoop *eventLoop) {
    aeApiState *state = zmalloc(sizeof(aeApiState));

    if (!state) {
        return -1;
    }

    FD_ZERO(&state->rfds);
    FD_ZERO(&state->wfds);
    eventLoop->apidata = state;

    return 0;
}
```
这里就分配需要使用的`fd_set`

###### 其他
其他操作函数也很简单, 只不过需要注意根据使用的I/O方式不同, 具体实现的差别
最后网络部分就是一些简单的函数库了










