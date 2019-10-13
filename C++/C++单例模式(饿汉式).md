# C++单例模式(饿汉式)
> 上一次看到了懒汉式的问题, 这次也简单介绍一下饿汉式的单例模式吧!

> ***需要注意的时, 我认为自C语言以后的很多语言的代码都不信任本人以外的所有人(各种方法限制后续使用该类的人); 必须要强调, 如果代码足够规范的话, 我们甚至可以通过一个全局变量来简单的实现单例模式.***

## 代码
> 直接上代码, 网上的大部分实现都是通过`new`的方式创建, 但是我认为使用静态变量的方式应当也可以的, 并且可以避免资源释放的问题.
```
class Singleton{
public:
       static Singleton* getInstatce(){
              return &m_instance;
       }
private:
	Singleton(){}
	Singleton(Singleton const & single);
	Singleton& operator = (const Singleton& single);
	static Singleton* m_instance;

	class GC
	{
	public :
		~GC()
		{
			// 销毁所有资源
			if (m_Instance != NULL )
             {
                 delete m_instance;
                 m_instance = NULL ;
             }
         }
     };
     static GC gc;
};
//类外初始化
Singleton* Singleton::m_instance = new Singleton;
Singleton::GC Singleton::gc;
```
和懒汉式是一致的, 必须注意销毁资源, 不需要考虑多线程的问题, 因为在进程开始时初始化资源。

**注意:** 如果可以采用饿汉式的方式实现, 我认为更好的方式是使用`局部静态变量的懒汉式(不加锁版本)+早期主动调用`的方式实现, 因为资源的初始化可能有顺序问题(依赖于其他资源的初始化). 这样可以解决 多线程/初始化顺序/资源释放的问题. 

## 静态变量实现
```
class Singleton{
public:
       static Singleton* getInstatce(){
              return &m_instance;
       }
private:
	Singleton(){}
	Singleton(Singleton const & single);
	Singleton& operator = (const Singleton& single);
	static Singleton m_instance;
};
//类外初始化
Singleton Singleton::m_instance;
```
这样可以不用考虑资源释放的问题

ps: 虽然这样写很好, 可以从编译时就阻止错误创建, 但是也引出了很多的问题.