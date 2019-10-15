# C++11多线程互斥锁`mutex`,`unique_lock`,`lock_guard`

## 互斥锁
&emsp;&emsp;互斥锁是线程中常用的线程同步手段, 在C++11后使用互斥互斥锁的方式包括两种`pthread_mutex_t`和`std::mutex`

## pthread_mutex_t
> 这是Linux下pthread的锁, [介绍](https://www.jianshu.com/p/88027a0e5807)

## std::mutex
> 我们这里介绍一下C++11中`std::mutex`的基本使用

**头文件**: `#include<mutex>`

**成员函数:**
* 构造函数, `std::mutex _mutex;`不必传入参数, 不允许拷贝构造和`move`构造
* `lock()`: 上锁, 如果其他线程已经持有锁的话会一直阻塞
* `unlock()`: 解锁
* `try_lock()`: 与`lock`相同用于加锁, 如果其他线程持有锁的话立刻返回`false`


## 扩展
&emsp;&emsp;事实上还存在有其他类型的锁, 比如:`recursive_mutex`和`time_mutex`这些锁个人基本没用到过, 可以实现递归加锁解锁/加锁超时的限制, 如果有需要自行了解


## 进阶版使用, `unique_lock`,`lock_guard`
> 对于以上的简单使用其实与C语言相差不大, 但是我们可以使用`RAII`(通过类的构造析构)来实现更好的编码方式.

ps: C++相较于C引入了很多新的特性, 比如可以在代码中抛出异常, 如果还是按照以前的加锁解锁的话代码会极为复杂繁琐

**代码:**
```
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <mutex>          // std::mutex, std::lock_guard
#include <stdexcept>      // std::logic_error

std::mutex mtx;

void print_even (int x) {
    if (x%2==0) std::cout << x << " is even\n";
    else throw (std::logic_error("not even"));
}

void print_thread_id (int id) {
    try {
        // using a local lock_guard to lock mtx guarantees unlocking on destruction / exception:
        std::lock_guard<std::mutex> lck (mtx);
        print_even(id);
    }
    catch (std::logic_error&) {
        std::cout << "[exception caught]\n";
    }
}

int main ()
{
    std::thread threads[10];
    // spawn 10 threads:
    for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_thread_id,i+1);

    for (auto& th : threads) th.join();

    return 0;
}
```

这里的`lock_guard`换成`unique_lock`也是一样的, 我并未深入研究内部实现; 但是可以很简单的猜到, 在构造函数中加锁,析构函数中解锁

## `unique_lock`,`lock_guard`的区别

&emsp;&emsp; `unique_lock`与`lock_guard`都能实现自动加锁和解锁，但是前者更加灵活，能实现更多的功能。`unique_lock`可以进行临时解锁和再上锁，如在构造对象之后使用`lck.unlock()`就可以进行解锁，`lck.lock()`进行上锁，而不必等到析构时自动解锁。



## `unique_lock`扩展条件变量
> C++11提供`std::condition_variable`可以和`std::mutex`配合使用, 不过往往是配合`unique_lock`进行使用, 所以在这里介绍一下

```
#include <iostream>
#include <deque>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <unistd.h>

std::deque<int> q;
std::mutex mu;
std::condition_variable cond;

void fun1() {
    while (true) {
        std::unique_lock<std::mutex> locker(mu);
        q.push_front(count);
        locker.unlock();
        cond.notify_one();  
        sleep(10);
    }
}

void fun2() {
    while (true) {
        std::unique_lock<std::mutex> locker(mu);
        cond.wait(locker, [](){return !q.empty();});
        data = q.back();
        q.pop_back();
        locker.unlock();
        std::cout << "thread2 get value form thread1: " << data << std::endl;
    }
}
int main() {
    std::thread t1(fun1);
    std::thread t2(fun2);
    t1.join();
    t2.join();
    return 0;
}
```

&emsp;&emsp;条件变量的目的就是为了, 在没有获得某种提醒时一致休眠; 如果正常情况下, 我们需要一直循环(+sleep), 这样的问题就是`CPU消耗`+`时延问题`, 条件变量的意思是在`cond.wait`这里一直休眠直到`cond.notify_one`唤醒才开始执行下一句; 还有`cond.notify_all()`接口用于唤醒所有等待的线程.

**cond.wait(locker, [](){return !q.empty();});**: 条件变量可能会被意外唤醒, 可以额外传入一个函数只有被唤醒时同时函数返回值为`true`才会被真正唤醒

### 那么为什么必须使用`unique_lock`呢?

> 原因: 条件变量在`wait`时会进行`unlock`再进入休眠, `lock_guard`并无该操作接口

**wait**: 如果线程被唤醒或者超时那么会先进行`lock`获取锁, 再判断条件(传入的参数)是否成立, 如果成立则`waut`函数返回否则释放锁继续休眠
**notify**: 进行`notify`动作并不需要获取锁