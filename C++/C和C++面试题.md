## 1, 信号的声明周期

答: 信号的产生 -> 信号在进程中注册 -> 信号在进程中的注销 -> 执行信号处理函数

## 2, 信号的产生方式

答: 1, `bash`命令行下的`kill`命令; 2, `C`中的信号发送函数`kill(), sigqueue(), raise(), alarm(), setitimer(), pause()，abort()`等; 3, 一些软硬件异常产生的信号

## 3, 信号的处理方式

答: 1,  默认处理方式, 比如`PIPE`信号会导致进程终止; 2, 忽略处理或者自定义函数处理(`signal`或者`signaction`函数)

## 4, 隐式类型转换以及如何消除？ 

答: 如果可以通过一个实参创建一个类, 那么我们可以隐式的完成该实参类型到该类类型的转换, 通过该构造函数前加上`explicit`避免这种隐式的转换

## 5, 重载/重写/隐藏的区别

答: 重载指的是函数的重载(函数名相同和参数不同的函数), 重写指的是虚函数, 隐藏指的是子类的函数名与父类相同就会隐藏

## 6, volatile功能

答: 在实际中我们可能遇到一个线程等待一个参数在另外一个线程发生改变; 但是由于代码优化每次都从寄存器中取值, 而另外线程的修改没有影响到此线程的寄存器, 导致无法检测变化; 该关键字指的就是每次从内存去取值.

## 7, malloc/new/free/delete区别

答: new/delete是描述符, 仅属于C++; 而后两者是库函数, C中可以使用. `new`错误会抛出`bad_alloc`异常, 而`malloc`失败返回NULL; `new`无需指明大小, 其返回的是指定的类型, `malloc`必须指明大小并返回`void`指针; `new`会调用构造函数, 后者不会; 都是在堆中分配释放内存.

## 8, `free`是怎么知道释放多大内存的呢?

答: 一般在分配的内存前几个字节有一个结构体保存一些信息, 在`redis`中实现的动态字符串就是这种形式.

-------------------2019年11月9日16点35分----------------------

## 9 `__stdcall`和`__cdecl`的区别

答: 函数参数均从右到左入栈, 但是前者被调用的函数自行清理堆栈, 后者是主动调用该函数的进行清理堆栈。 也因此, 前者需要确定的参数数量, 不能用于变参数.

## 10, Linux有哪些调试宏

答: `__FILE__`/`__LINE__`/`__FUNCTION__`分别指明文件名/行数/函数名

## 11, 线程安全的单例模式

**饿汉式**
```
class Singleton
{
private:
	Singleton();
	Singleton(Singleton const &);
	Singleton & operator = (Singleton const &);
	static Singleton *instance;
public:
	static Singleton* getInstatce()
	{
		return instance;
	} 
	
};

Singleton Singleton::instance = new Singleton();
```


**懒汉式**
```
class Singleton
{
private:
	Singleton();
	Singleton(Singleton const &);
	Singleton & operator = (Singleton const &);
	static Singleton *instance;
	static std::mutex _mutex;
public:
	static Singleton* getInstatce()
	{
		if(nullptr == instance)
		{
			std::lock_guard<std::mutex> lk(_mutex);
			if(nullptr == instance)
			{
				instance = new Singleton();
			}
		}

		return instance;
	} 
	
};


std::mutex Singleton::_mutex;
```

**注意**:可以额外添加以静态成员用以销毁资源

## 12, 引用和指针的区别?

* 1 引用只是一个别名, 必须初始化, 不占用额外空间; 指针本身就是一种数据结构可以为`nullptr`, 占用4/8字节
* 2 引用只能为一个成员的引用, 指针如果不加`const`限制的话可以修改其指向

## 13, 出现异常时, `try`和`catch`分别干了什么?

答: `try{}`用于包含可能抛出异常的语句; `catch{}`用于包含处理异常的语句

## 14, C++如何处理多个异常
```
try
{
    //可能抛出异常的语句
}
catch (异常类型1)
{
    //异常类型1的处理程序
}
catch (异常类型2)
{
    //异常类型2的处理程序
}
// ……
catch (异常类型n)
{
    //异常类型n的处理程序
}
```


