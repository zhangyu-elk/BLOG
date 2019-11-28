# STL空间配置器(allocator)
&emsp;&emsp;STL的一部分核心就是在于内存的管理, 这是隐藏在一切组件之下的工作. 以下均基于STLport的源码, 由于代码中涉及到很多的宏定义, 在不清楚定义之前不做特别详细的探究.

[1, 如何使用空间配置器](#如何使用空间配置器)
[2, 一个空间配置器应当是怎么样的呢?](#一个空间配置器应当是怎么样的呢?)


## 文件说明
文件名|功能
:--|:--
vector|作为引入的头文件, 没有实际的代码, 引入一些新的需要的头文件
std/_vector.h|包含`vector`的类定义
std/_alloc.h|空间配置器类`allocator`的定义


[特性1](#特性1)

## 如何使用空间配置器
```
template <class _Tp, _STLP_DFL_TMPL_PARAM(_Alloc, allocator<_Tp>) >
class vector : protected _STLP_PRIV _Vector_base<_Tp, _Alloc>
#if defined (_STLP_USE_PARTIAL_SPEC_WORKAROUND) && !defined (vector)
             , public __stlport_class<vector<_Tp, _Alloc> >
#endif
```
这是`STLport`中`vector`类的定义, 把宏展开后就是:
```
template <class _Tp, class classname = allocator<_Tp> >
```
可以看到我们实际生成一个`vector`的时候是`vetor<int, allocator<int>> vec`这样的, 默认会生成一个空间配置器。 这些都是隐藏在算法之下的, 我们实际完全不需要考虑这些.

## 一个空间配置器应当是怎么样的呢?
&emsp;&emsp;由上面可以看的出来, 首先必须是一个模板类; 事实上在STL规范中有说明其必要接口(我们也可以直接看一下头文件是怎么样的):
```
template <class T>
class allocator
{
public:
	typedef T		value_type;
	typedef T*		pointer;
	typedef const T*	const_pointer;
	typedef T&		reference;
	typedef const T&	const_reference;
	typedef size_t		size_type;
	typedef ptrdiff_t	difference_type;

	template <class U>
	struct rebind
	{
		typedef allocator<U> other;
	};

	pointer allocate(size_type n, const void* hint = 0)
	{
		T* tmp = (T*)(::operator new((size_t)(n * sizeof(T))));
		if (tmp == 0)
			cerr << "out of memory" << endl;

		return tmp;
	}

	void construct(pointer p, const T& value)
	{
		new(p) T1(value);
	}

	//析构函数
	void destroy(pointer p)
	{
		p->~T();
	}

	//释放内存  直接使用delete
	void deallocate(pointer p)
	{
		::operator delete(p);
	}

	//取地址
	pointer address(reference x)
	{
		return (pointer)&x;
	}

	//返回const对象的地址
	const_pointer const_address(const_reference x)
	{
		return (const_pointer)&x;
	}

	//可成功配置的最大量, 可寻址的最大数/类型的字节数
	size_type max_size() const
	{
		return size_type(UINT_MAX / sizeof(T));
	}
};
```

## 空间配置器的创建
&emsp;&emsp;以下均基于STLport中的`vector`中的迭代器使用进行说明(`_vector`文件)
```
	vector()
	: _STLP_PRIV _Vector_base<_Tp, _Alloc>(allocator_type()) {}
```
`alloctor_type`在`vector`中定义在父类(`_Vector_base`)中, 其实就是`alloctor<Tp>`, 我们也可以一窥空间配置器的构造函数(`_alloc.h`文件), 与上述的简易写法基本是一致的(代码很多就不贴了)。 无参的构造函数也没有进行什么操作


## 分配空间
```

```







