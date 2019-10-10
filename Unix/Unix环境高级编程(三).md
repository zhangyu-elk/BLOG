## Unix环境高级编程--Linux线程
> 线程自然是会用的, 但是既然看到了这部分, 就做一个记录吧, 说明一下相关函数

### 线程数据类型
`pthread_t thread;`
对于这一种数据类型来说, 有可能实现的是向pid_t(进程id)一样实际实现是一个非负整数, 但是再编码中不应该直接对它做一些'整数类型'的操作, 尤其是当设计到跨平台时

`int pthread_equal(pthread_t tid1, pthread_t tid2);`
线程比较的函数, 就算可以直接用`==`获取结果, 也最后不要

### 线程基础操作函数

#### 创建线程
```
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);
```
**pthread**: 线程的数据类型, 需要先创建变量再传入指针
**attr**: 这里我不做介绍, 因为绝大部分传NULL即可, 如果真的需要再深入研究
**routine**: 传入一个void指针, 最后可以返回一个void指针(可以再主线程中获取到)
**arg**: 给函数传递的参数

***代码示例***
```
void* fun(void* arg)
{
	while(1)
	{
		//work
	}
	return NULL;
}

int main()
{
	pthread_t pt;
	int a;
	pthread_create(&pt, NULL, fun, &a);
}

```



ps: 对于主线程来说当然可以获得到子线程的`pthread_t`, 那么对于子线程呢? `pthread_t pthread_self();`

#### 子线程中线程终止
* 1 直接再函数中使用return返回即可
* 2 调用`pthread_exit`函数
* 3 被同一个进程其他线程取消

`void pthread_exit(void *retval);`
该函数与return相同, 返回一个void指针

#### 主线程监听子线程
> 对于第一段代码示例是有问题的, 因为父进程也就是main函数会直接退出去, 进程可能直接就终止了。
> 父进程可能希望有类似waitpid的方法等待子线程结束, 并且可以获得到子线程的返回值

`int pthread_join(pthread_t thread, void **retval);`
从这个函数就可以简单看出来其作用和参数, 第二个值就是获得子线程返回的void指针(也可以传入NULL)
该函数时阻塞的, 直到指定线程结束, 如果子线程已经结束, 则理解返回

ps: 注意返回值的问题, 注意如果是再栈中的会被销毁

#### 主线程分离子线程
> 主线程是必须对子线程进行管理的, 如果我们主线程并不需要获取子线程返回值, 也不存在竞争关系, 那么我们可以仅`pthread_create`之后就不做操作了吗？

	是不行的, 因为如果主线程不对子线程进行管理的化, 线程结束后资源并没有被回收(线程所占用堆栈和线程描述符(总计8K多),过多后会无法创建线程)

`int pthread_detach(pthread_t thread);`
	将线程与主线程分离, 在结束后自动回收线程, 可以在子线程中以`pthread_detach(pthread_self())`的方式分离自己, 总之要么join等待或者detach分离

注意: **分离后无法再pthread_join了**, **该属性可以再创建线程是通过attr参数设置**(线程栈的大小也可以通过这个设置), **不能多个线程pthread_join一个线程**

#### 主线程取消子线程
> 需要说明的是, 强制取消线程是不可取的行为
`int pthread_cancel(pthread_t thread);`
该函数也仅仅是发起请求, 并不等待终止, 具体行为和子线程属性有关

#### 介绍一下清理函数
```
void pthread_cleanup_push(void (*routine)(void *),
                                 void *arg);
void pthread_cleanup_pop(int execute);
```
一个push一个pop, 以堆栈的方式存储. pop弹出一个函数, 如果传入参数未0则不执行。
ps: 并不是`pop`才会执行, 再调用`prhead_exit`是会执行(return不会). 实际用的很少, 据说在一些平台还会有问题.


### 一个线程例子
```
void* fun(void* arg)
{
	int *a = (int*)arg;
	while(1)
	{
		printf("%d\n", *a);
		//work
	}
	return NULL;
}

int main()
{
	pthread_t pt;
	int a = 1;
	pthread_create(&pt, NULL, fun, &a);
	pthread_join(pt, NULL);
}
```

> 简单的使用就是这样


***2019年8月3日时间添加关于线程同步***
### 线程同步
> [本人写的C语言原子操作的一篇blog](https://www.jianshu.com/p/509d3f594aff)

#### 为什么有线程同步以及原子操作
> 所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何context switch
> [linux库中的一些原子操作](https://www.jianshu.com/p/4c6c56386c15), 对于非原子操作使用dup时, 可能在close对应文件描述符后, 其他线程使用了该线程(这种情况非常可能出现);对于原子操作不会出现问题。

在C语言中即使是`i++;`这样的操作也不是原子操作, 涉及三步`从内存中取到寄存器, +1, 从寄存器放入内存中`, 如果两个线程同时对一个变量进行这三步, 很可能出现问题
ps: 当然可能涉及到gcc编译优化使得这三步可能不同, 但是从编码角度来说, 寄希望于编译优化是不可取的

**示例代码(go开1个线程进行计数)**
```
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int count = 0;

static void* func(void* arg)
{
	int i = 1;
	for(i = 1; i <= 100000000; i++)
	{
		count ++;
		//sync_pre_add(count);
	}
	printf("count: %d\n", count);
	
}

int main()
{
	go(func);
	go(func);
	go(func);
	go(func);
	go(func);
	
	sleep(15);
	printf("count: %d\n", count);
	
	return 0;
}
```
**结果**:
```
count: 408232265
count: 425684700
count: 441569929
count: 443180749
count: 461046378
count: 461046378
```
可以看出来, 即使是++这样的操作也存在问题


#### 相关函数
* 1 gcc提供的相关函数(上面提到了)
* 2 互斥量`pthread_mutex_t`
```
int pthread_mutex_destroy(pthread_mutex_t *mutex);
int pthread_mutex_init(pthread_mutex_t *restrict mutex,
              const pthread_mutexattr_t *restrict attr);
```
互斥量的创建和删除, 对于attr属性默认传入NULL即可
```
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
加锁和解锁, 调用`pthread_mutex_lock`时如果其他线程已经加锁了, 则阻塞让出CPU直到其他线程Unlock, `pthread_mutex_trylock`如果不能加锁立即返回
**示例代码**
```
int count = 0;
pthread_mutex_t mutex;
static void* func(void* arg)
{
	int i = 1;
	for(i = 1; i <= 100000000; i++)
	{
		pthread_mutex_lock(&mutex);
		count ++;
		pthread_mutex_unlock(&mutex);
		//sync_pre_add(count);
	}
	printf("count: %d\n", count);
	
}

int main()
{
	pthread_mutex_init(&mutex, NULL);
	go(func);
	go(func);
	go(func);
	go(func);
	go(func);
	pthread_mutex_destroy(&mutex);

	sleep(15);
	printf("count: %d\n", count);
	
	return 0;
}
```

不会有两个线程同时执行`pthread_mutex_lock(&mutex);`和`pthread_mutex_unlock(&mutex);`之间的代码, 可以保证不会出现问题, 但是会极大的影响效率

#### 死锁

**直接上代码:**
```
//A thread

pthread_mutex_lock(&A);
//...
pthread_mutex_lock(&B);
//... unlock


//B thread
pthread_mutex_lock(&B);
//...
pthread_mutex_lock(&A);
//...unlock 
```
对于A线程获取了A锁, 然后尝试获取B锁
对于B线程获取了B锁, 然后尝试获取A锁
两个线程可能在之后会进行解锁操作, 但是现在两个线程都会永远阻塞在这里, A无法获取B锁, B也无法获取A锁
正常情况应该是保持一个顺序加锁, 这样B线程会阻塞但是A线程时可以自行解锁的, 解锁后B线程就可以获取锁了


#### 其他函数
```
int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex,
              const struct timespec *restrict abs_timeout);
```
与`mutex_lock`动作是一样的, 不过有一个超时时间

此外还有读写锁, 自旋锁之类的暂不做解释



