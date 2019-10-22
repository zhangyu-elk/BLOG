
# C++事件模型
> 实现一个事件模型, 能够进行事件的绑定和触发


## 功能
* 1 预先绑定指定字符串事件类型的回调函数
* 2 触发时, 之前所有绑定的函数均被触发
* 3 触发时, 所有参数以`key-value`的形式传递, 仅支持字符串传递 
* 4 暂时不支持取消事件绑定
* 5 保证线程安全

## 代码
> 提供`hpp`文件, [github地址](https://github.com/zhangyu012/Xevent_handler.git) 

### **Xevent_handler.hpp**
* 1 使用单例模式, 全局运行一个实例一个线程, 顺序执行触发的事件
* 2 使用信号量, 仅当有值时触发线程顺序执行, 并不是瞬间执行的
* 3 触发事件是非阻塞的
* 4 `pfifo`是类似与[唤醒缓冲队列](https://www.jianshu.com/p/d378a904429f), 在一个线程读取一个线程写入的情况下是安全的

**问题**: 在`wait`第一次挂起之前会不会先判断一下条件呢? 笔者并不清楚, 所以需要额外一个`atomic`辅助判断

```

#ifndef XEVENT_HANDLER_H
#define XEVENT_HANDLER_H

#include <stdio.h>
#include <stdlib.h>
#include <string>
#include <map>
#include <vector>
#include <mutex>
#include <atomic>
#include <thread>
#include <functional>
#include <condition_variable>

#include "pfifo.hpp"


/*一个event不要在多个线程中操作*/
class Xevent
{
private:
	std::string _key;
	std::map<std::string, std::string> _header;
public:
	Xevent(const char *key);
	~Xevent();

	void add_header_string(const char *key, const char *value);
	const char* get_header(const char *key);
};

Xevent::Xevent(const char *key):_key(key)
{

}

void Xevent::add_header_string(const char *key, const char *value)
{
	_header.insert(std::make_pair(key, value));
}

const char* Xevent::get_header(const char *key)
{
	auto iter = _header.find(key);
	if(iter != _header.end())
	{
		return (iter->second).c_str();
	}
	else
	{
		return nullptr;
	}
}


typedef void(*EventCallback)(Xevent *ev);

class Xtask
{
public:
	Xtask():_cb(NULL),_ev(NULL){}
	Xtask(EventCallback cb, Xevent *ev):_cb(cb),_ev(ev){}
	Xtask(Xtask const & t)
	{
		_cb = t._cb;
		_ev = t._ev;
	}
	void run()
	{
		if(_cb)  _cb(_ev);
	}
private:
	EventCallback 	_cb;
	Xevent 			*_ev;
};

class XeHandler
{
public:
	~XeHandler()
	{
		running = false;
		
		std::unique_lock<std::mutex> lk(_cdmutex);
		cond.notify_one();
		_active = true;
		lk.unlock();

		_thr.join();
	}
	static XeHandler* getInstance()
	{
		
		return &handler;
	}
	Xevent* event_create(const char *key);
	void bind(const char *key, EventCallback cb);
	void fire(const char *key, Xevent *ev);
private:
	std::map<std::string, std::vector<EventCallback>> _cbmap;

	std::thread _thr;
	std::mutex _mutex;

	std::condition_variable cond;
	std::mutex _cdmutex;
	std::atomic<bool> _active;

	pfifo<Xtask> _task;

	bool running;
	void Loop();

	static XeHandler handler;
	XeHandler():_task(8),_active(false){
		running = true;
		_thr = std::thread(std::mem_fn(&XeHandler::Loop), this);
	}
	XeHandler(XeHandler const & handler);
	XeHandler& operator = (const XeHandler& handler);
};

XeHandler XeHandler::handler;

XeHandler* GetXeHandler()
{
	return XeHandler::getInstance();
}

void XeHandler::Loop()
{
	while(running)
	{
		std::unique_lock<std::mutex> lk(_cdmutex);	//加锁
		if(!_active) cond.wait(lk, [this](){return !_task.empty() || !running;});
		lk.unlock();

		Xtask t;
		while(_task.pop(&t))
		{
			t.run();
		}
	}
	printf("%s\n", "byebye");
}

void XeHandler::fire(const char *key, Xevent *ev)
{
	std::lock_guard<std::mutex> lk(_mutex);

	auto iter = _cbmap.find(key);
	if(iter == _cbmap.end())
	{
		return;
	}
	else
	{
		for(auto cb : iter->second)
		{
			_task.push(Xtask(cb, ev));
		}

		std::unique_lock<std::mutex> lk(_cdmutex);
		cond.notify_one();
		_active = true;
	}
}

void XeHandler::bind(const char *key, EventCallback cb)
{
	std::lock_guard<std::mutex> lk(_mutex);

	auto iter = _cbmap.find(key);
	if(iter != _cbmap.end())
	{
		iter->second.push_back(cb);
	}
	else
	{
		std::vector<EventCallback> vec(1, cb);
		_cbmap.insert(std::make_pair(key, vec));
	}
}

Xevent* XeHandler::event_create(const char *key)
{
	Xevent *ev = new Xevent(key);
	return ev;
}


#endif

```

## **pfifo.hpp**
```
/*
* Linux内核中kfifo的Cpp实现
*/

#ifndef PFIFO_H
#define PFIFO_H

#include <assert.h>

//判断数是不是2的幂次方
inline int is_power_of_2(unsigned int num)
{
	return (num != 0 && ((num & (num - 1)) == 0)) ? 1 : 0;
}
//将数扩展为2的幂次方
inline long roundup_pow_of_two(unsigned long num)
{
	unsigned long ret = 1;
	unsigned long num_bk = num;
	while(num >>= 1)
	{
		ret <<= 1;
	}
	return (ret < num_bk) ? (ret << 1) : ret;
}

template<class T>
class pfifo
{
public:
	pfifo(uint32_t size);
	~pfifo();

	bool pop(T *);
	bool push(T t);
	bool empty();
private:
	T 			*_queue;
	uint32_t 	_capacity;
	uint32_t	_in;
	uint32_t	_out;
};

template<class T>
bool pfifo<T>::empty()
{
	return _in > _out;
}


template<class T>
pfifo<T>::pfifo(uint32_t size)
{
	if(!is_power_of_2(size))
	{
		size = roundup_pow_of_two(size);
	}

	_queue = new T[size];
	assert(_queue);

	_capacity = size;
	_in = 0;
	_out = 0;
}


template<class T>
pfifo<T>::~pfifo()
{
	delete []_queue;
}

template<class T>
bool pfifo<T>::pop(T *t)
{

	uint32_t len = _in - _out;
	if(len == 0)
	{
		return false;
	}
	else
	{
		*t = _queue[_out & (_capacity - 1)];
		_out++;
		return true;
	}
}

template<class T>
bool pfifo<T>::push(T t)
{
	uint32_t len = _in - _out;
	if(len >= _capacity)
	{
		return false;
	}
	else
	{
		_queue[_in & (_capacity - 1)] = t;
		_in++;
		return true;
	}
}

#endif
```

## **test.cpp**
> 简单工作运行
```
#include "Xevent_handler.hpp"
#include <iostream>
#include <map>
#include <string>
#include <unistd.h>
#include <mutex>

using namespace std;

void func1(Xevent *ev)
{
    cout<<"func1"<<endl;
	cout<<ev->get_header("test1")<<endl;
	cout<<ev->get_header("test3")<<endl;
}

void func2(Xevent *ev)
{
    cout<<"func2"<<endl;
    cout<<ev->get_header("test1")<<endl;
    cout<<ev->get_header("test2")<<endl;
}


int main(int argc, char const *argv[])
{
	cout<<"Test begin..."<<endl;
	XeHandler *handler = GetXeHandler();

	handler->bind("test", func1);
	handler->bind("test", func2);

	Xevent *ev = handler->event_create("good");
	ev->add_header_string("test1", "bb");
	ev->add_header_string("test2", "cc");
	ev->add_header_string("test3", "dd");

	handler->fire("test", ev);
    handler->fire("test1", ev);
    handler->fire("test", ev);
	return 0;
}
```
