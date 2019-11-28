# 源码赏析
&emsp;&emsp;STL源码有很多中, 由于SGI STL找不到了; 所以下载了STLport来学习。

[1, 构造函数](#构造函数)
* [指定长度构造函数](#指定长度构造函数)

[2, 插入数据](#插入数据)

## 宏定义
* `_STLP_DONT_SUP_DFLT_PARAM`: 如下的构造函数, 对于函数的影响不大; 如果未定义该宏, 对于`构造器`等值, 允许传入并给一个默认参数(其他函数也是类似); 否则直接在构造函数内部创建.


## 构造函数

* 1  `vector`构造函数(默认无参构造函数)
```
#if !defined (_STLP_DONT_SUP_DFLT_PARAM)
	explicit vector(const allocator_type& __a = allocator_type())
#else
	vector()
	: _STLP_PRIV _Vector_base<_Tp, _Alloc>(allocator_type()) {}
	vector(const allocator_type& __a)
#endif
	: _STLP_PRIV _Vector_base<_Tp, _Alloc>(__a) {}
```
最简单的无参构造函数, 仅初始化其`_Base`空间配置器


## 指定长度构造函数又是怎么样的呢?
```
explicit vector(size_type __n)
	: _STLP_PRIV _Vector_base<_Tp, _Alloc>(__n, allocator_type())			//空间配置器
	{ _M_initialize(__n); }													//真正的初始化操作
	
void _M_initialize(size_type __n, const _Tp& __val = _STLP_DEFAULT_CONSTRUCTED(_Tp))
	{ this->_M_finish = _STLP_PRIV __uninitialized_init(this->_M_start, __n, __val); }
```

`_STLP_DEFAULT_CONSTRUCTED(_Tp)`展开就是调用对应类型默认构造函数; `_M_finish`和`_M_start`是指定类型`Tp`的指针, 在`_Vector_base`父类中定义. 核心分配空间和初始化都在下面:


```
  _Vector_base(size_t __n, const _Alloc& __a)
    : _M_start(0), _M_finish(0), _M_end_of_storage(__a, 0) {
    _M_start = _M_end_of_storage.allocate(__n, __n);			//调用空间配置其分配空间
    _M_finish = _M_start;
    _M_end_of_storage._M_data = _M_start + __n;
    _STLP_MPWFIX_TRY _STLP_MPWFIX_CATCH
  }
```
`_M_start`执行分配空间的首地址, `_M_finish`执行结尾的下一个; \_M_end_of_storage.\_M_data指向结尾的下一个.

$\color{red}{`vector`继承了`_Vector_base`类, 该类负责空间的管理}$


## 插入数据
```
#if !defined (_STLP_DONT_SUP_DFLT_PARAM) && !defined (_STLP_NO_ANACHRONISMS)
	void push_back(const _Tp& __x = _STLP_DEFAULT_CONSTRUCTED(_Tp)) {
#else
	void push_back(const _Tp& __x) {
#endif
		if (this->_M_finish != this->_M_end_of_storage._M_data) {		//判断是否还有空间
			_Copy_Construct(this->_M_finish, __x);						//赋值操作, 会调用复制构造函数;
			++this->_M_finish;											//finish持续指向最后一个值的下一个空间
		}
		else {
			typedef typename __type_traits<_Tp>::has_trivial_assignment_operator _TrivialCopy;
			_M_insert_overflow(this->_M_finish, __x, _TrivialCopy(), 1, true);
		}
	}
```

**初步总结**: 抛开其他一些成员, `vector`与普通的数组类似(空间分布); 其次, 使用`_M_start`指向开头, `_M_finish`指向最后一个值的下一个, `_M_data`指向分配空间末尾的下一个.

**重新分配空间**:
&emsp;&emsp;这里面存在很多类以及宏定义, 但是不必关注
```
template <class _Tp, class _Alloc>
void vector<_Tp, _Alloc>::_M_insert_overflow_aux(pointer __pos, const _Tp& __x, const __false_type& /*DO NOT USE!!*/,
                                                 size_type __fill_len, bool __atend ) {
  typedef typename __type_traits<_Tp>::has_trivial_copy_constructor _TrivialUCopy;
#if !defined (_STLP_NO_MOVE_SEMANTIC)
  typedef typename __move_traits<_Tp>::implemented _Movable;
#endif
  size_type __len = _M_compute_next_size(__fill_len);
  pointer __new_start = this->_M_end_of_storage.allocate(__len, __len);
  pointer __new_finish = __new_start;
  _STLP_TRY {
    __new_finish = _STLP_PRIV __uninitialized_move(this->_M_start, __pos, __new_start, _TrivialUCopy(), _Movable());
    // handle insertion
    if (__fill_len == 1) {
      _Copy_Construct(__new_finish, __x);
      ++__new_finish;
    } else
      __new_finish = _STLP_PRIV __uninitialized_fill_n(__new_finish, __fill_len, __x);
    if (!__atend)
      __new_finish = _STLP_PRIV __uninitialized_move(__pos, this->_M_finish, __new_finish, _TrivialUCopy(), _Movable()); // copy remainder
  }
  _STLP_UNWIND((_STLP_STD::_Destroy_Range(__new_start,__new_finish),
               this->_M_end_of_storage.deallocate(__new_start,__len)))
  _M_clear_after_move();											//释放原有空间
  _M_set(__new_start, __new_finish, __new_start + __len);			//设置相应指针
}
```

`_M_compute_next_size`计算新增的长度, 如果新增长度`__n`小于原有的长度, 则原有长度\*2; 否则变为原有长度+新增长度.
调用空间配置器重新分配空间, 调整指针指向, 复制数据.


## 弹出数据
```
void pop_back() {
	--this->_M_finish;
	_STLP_STD::_Destroy(this->_M_finish);
}

//destory定义
template <class _Tp>
inline void __destroy_aux(_Tp* __pointer, const __false_type& /*_Trivial_destructor*/)
{ __pointer->~_Tp(); }

template <class _Tp>
inline void __destroy_aux(_Tp*, const __true_type& /*_Trivial_destructor*/) {}

template <class _Tp>
inline void _Destroy(_Tp* __pointer) {
	typedef typename __type_traits<_Tp>::has_trivial_destructor _Trivial_destructor;
	__destroy_aux(__pointer, _Trivial_destructor());
	#if defined (_STLP_DEBUG_UNINITIALIZED)
	memset(__REINTERPRET_CAST(char*, __pointer), _STLP_SHRED_BYTE, sizeof(_Tp));
	#endif
}
```
很简单的将指针迁移, 然后清空指向的空间; 如果又必要甚至需要调用析构函数. 这里空间之后有可能的话还可以接着利用.

$\color{red}{**说明**}$: `typedef typename __type_traits<_Tp>::has_trivial_destructor _Trivial_destructor;`到这里也大致明白其是用来判断是否是`class`还是仅仅是基础类型.

## 访问数据

&emsp;&emsp;到了这里我们明白内部数据的存储还是数组的形式, 那么我们仅需要访问对`[]`做一下重载就可以以索引访问数据了.
```
iterator begin()             { return this->_M_start; }
reference operator[](size_type __n) { return *(begin() + __n); }
```
简单的获取指针然后直接访问地址, 返回其引用

## 基本函数介绍
```
iterator begin()             { return this->_M_start; }                 //起始指针
const_iterator begin() const { return this->_M_start; }                 //起始const指针
iterator end()               { return this->_M_finish; }                //末尾指针
const_iterator end() const   { return this->_M_finish; }                //末尾const指针


size_type size() const        { return size_type(this->_M_finish - this->_M_start); }   //元素个数
size_type max_size() const {
  size_type __vector_max_size = size_type(-1) / sizeof(_Tp);
  typename allocator_type::size_type __alloc_max_size = this->_M_end_of_storage.max_size();
  return (__alloc_max_size < __vector_max_size)?__alloc_max_size:__vector_max_size;
}
//允许分配的最大元素个数

size_type capacity() const    { return size_type(this->_M_end_of_storage._M_data - this->_M_start); }
//已分配的空间大小
bool empty() const            { return this->_M_start == this->_M_finish; }
//数据是否又元素

reference operator[](size_type __n) { return *(begin() + __n); }
const_reference operator[](size_type __n) const { return *(begin() + __n); }      //返回引用

reference front()             { return *begin(); }
const_reference front() const { return *begin(); }
reference back()              { return *(end() - 1); }
const_reference back() const  { return *(end() - 1); }            //获得首位元素的指针
```

$\color{red}{从这里也可以看到, `iterator`属于每个容器独有的, 对于`vector`可以仅仅就是一个指针}$

## 迭代器
```
  typedef value_type* iterator;
  typedef const value_type* const_iterator;
```
迭代器就是最简单的指针, 遍历也就是最简单的指针移动.

## 反向迭代器
```
  reverse_iterator rbegin()              { return reverse_iterator(end()); }
  const_reverse_iterator rbegin() const  { return const_reverse_iterator(end()); }
  reverse_iterator rend()                { return reverse_iterator(begin()); }
  const_reverse_iterator rend() const    { return const_reverse_iterator(begin()); }
```
实际实现在`_iterator.h`中, 与迭代器大致类似; 但是`++`这种操作重载; `++`实际进行了`--`即反向


 