# C++懒汉式单例模式遇到多线程
> 单例模式是一个创建型设计模式, 就是保证在整个程序运行中仅存在该类的一个实例, 比如数据库句柄/日志等.

## 单例模式要求
* 1 构造函数必须私有, 防止使用new的方式初始化
* 2 拷贝构造函数和赋值操作必须私有, 避免赋值创建
* 3 提供一个`static`类型的函数作为对外接口, 负责获取对象

## 懒汉式
> 懒汉式的意思是知道我第一次使用该实例的时候才会去初始化它, 如果从未使用就不初始化。
> 但是在多线程中使用懒汉式, 必然存在问题

## 懒汉式单例模式的实现
* 1 使用静态变量
```
// 局部静态变量
class Singleton{
public:
	// 返回指针避免拷贝复制
	static Singleton* getInstance(){
		static Singleton instance;
		return &instance;
	}
private:
	/*避免new方法和赋值操作*/
	Singleton(){}
	Singleton(Singleton const & single);
	Singleton& operator = (const Singleton& single);
};
```
注意:**多线程中局部静态变量初始化并不保证线程安全**
如果按如上写法, 那么只有一个办法就是加锁, 而且必然是很粗暴的上下加锁的一种

* 2 普通实现
```
class Singleton{
public:
	static Singleton* getInstatce(){
	      if( NULL == lazy ){
	             lazy = new Singleton;
	      }
	      return lazy;
	}
private:
	static Singleton* lazy;
	Singleton(){}
	Singleton(Singleton const & single);
	Singleton& operator = (const Singleton& single);
};
//类静态变量必须外部初始化
Singleton* Singleton::lazy = NULL;
```
同样存在多线程的问题, 但是加锁的方式可以优化一下, 因为可以先进行一下判断在加锁

ps:**如果采用指针的方式, 那么必须注意释放的问题**




## 完整实现(包括加锁和最后的资源释放)
```
class Singleton{
public:
    typedef std::shared_ptr<Singleton> Ptr;
    static Ptr get_instance(){
        if(m_instance_ptr==nullptr){
            std::lock_guard<std::mutex> lk(mutex);
            if(m_instance_ptr == nullptr){
              m_instance_ptr = std::shared_ptr<Singleton>(new Singleton);
            }
        }
        return m_instance_ptr;
    }

private:
	Singleton(){}
	Singleton(Singleton const & single);
	Singleton& operator = (const Singleton& single);
	static Ptr m_instance_ptr;
	static std::mutex mutex;
};

// 初始话静态变量
Singleton::Ptr Singleton::m_instance_ptr = nullptr;
std::mutex Singleton::mutex;
```
* 只有在最开始调用时, 才会出现`m_instance_ptr`是`null`, 之后永远有值则不用加锁, 能够极大的优化效率
* 通过智能指针的方式, 实现资源释放(起始只有进程退出时才需要释放资源, 如果要求不高可以直接系统回收)

**扩展:**
* 如果不使用智能指针的方式, 可以创建初始化一个静态成员类, 在该成员析构时,	`delete`对象
```
class Singleton{
public:
	static Singleton* getInstatce(){
	      if( NULL == m_instance ){
	             std::lock_guard<std::mutex> lock(mutex);
	             if( NULL == m_instance ){
	                   m_instance = new Singleton;
	             }
	      }
	      return m_instance;
	}
private:
	Singleton(){}
	Singleton(Singleton const & single);
	Singleton& operator = (const Singleton& single);

	static Singleton* m_instance;
	static mutex mutex;

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
Singleton* Singleton::m_instance = NULL;
std::mutex Singleton::mutex;
Singleton::GC Singleton::gc;
```

**可以通过初始化一个私有的静态变量的的方式, 在最后析构时调用函数释放单例的资源**





