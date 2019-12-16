# C++函数对象、二元谓词、lambda表达式

## C风格的函数指针

&emsp;&emsp;在C语言中当我们希望使用`回调函数`这种动作时, 我们就需要使用函数指针了; 在C语言中函数指针和普通指针的保存或者传递并没有特别大的区别. 该指针指向的地方就是函数的地址.

&emsp;&emsp;典型的应用就是在排序当中, 我们可能对于不同的情况可能使用的比较函数是不一样的, 希望在比较的时候传递一个自定义的比较函数.

### 请看例子
```
#include <stdio.h>
#include <stdlib.h>

typedef void (*pfun)(int);

void print(void (*fun)(int), int a)
{
	fun(a);
}

void print_v2(pfun fun, int a)
{
	fun(a);
}

void myfun(int a)
{
	printf("input %d\n", a);
}

int main()
{
	print(myfun, 10);
	print_v2(myfun, 10);
	print_v2(&myfun, 30);
	return 1;
}

//结果: 
input 10
input 10
input 30
```

* 1 函数指针和普通数据的指针是一样的,
* 2 函数指针类型说明
```
void (*fun)(int)
```
前面的`void`表示返回值类型
括号中的`(int)`表示该函数传入的参数类型
`(*fun)`声明一个名字叫做`fun`的函数指针.

* 3 `typedef void (*pfun)(int)`: 表示定义函数指针的别名
* 4 如下两种方式是一致的, 加上`&`只不过是显示说明罢了; 编译器内部一定继续了转换实现.s
```
pfun fun = myfun;
pfun fun = &myfun;
```

## C++函数对象

&emsp;&emsp;C++的函数对象实际上就是`实现了函数功能的对象`, 说穿了就是重载了`()`运算符.

### 谓词

&emsp;&emsp;C++在使用STL算法时经常会说到二元谓词或者一元谓词之类的, 其实就是:

* 一元谓词: `bool fun(x)`, 接受一个参数返回一个bool值的函数
* 二元谓词: `bool fun(x,y)`, 接受两个参数返回一个bool值的函数

## lambda表达式

&emsp;&emsp;在比如JS语言中天然支持类似于lambda表达式一样的写法, 而lambda就是类似与这样的语法:

JS
```
setTimeout(20, function(){})
```
C++
```
#include <stdio.h>
#include <stdlib.h>

typedef void (*pfun)(int);

void print_v2(pfun fun, int a)
{
	fun(a);
}

void myfun(int a)
{
	printf("input %d\n", a);
}


int main()
{
	print_v2(myfun, 10);
	print_v2([](int a){printf("input %d\n", a);}, 30);
	return 1;
}

//结果
input 10
input 30
```

从这里看出, 最简单的`lambda`表达式的作用就是: 实现一个`匿名的`、`临时定义的`函数指针(与C风格的函数指针兼容)


### 语法
`[](Type elem)->void{//expression}`

以上共分为四部分:
* 第二部分`[Type elem]`: 类似于普通函数的参数, 类型+名称; 可以传递多个.
* 第三部分`->void`: 指定返回值类型, 如果返回`void`可以忽略该语句
* 第四部分`//expression`: 表达时语句


### 闭包
&emsp;&emsp;`[](Type elem)->void{//expression}`表达式第一项表示获取的外部变量, 类似于JS中的闭包.

#### 注意
```
#include <stdio.h>
#include <stdlib.h>

typedef void (*pfun)(int);

void print_v2(pfun fun, int a)
{
	fun(a);
}

int main()
{
	int extra = 100;
	print_v2([extra](int a)->void{printf("input %d, empure: %d\n", a, 0);}, 30);
	return 1;
}
```
$\color{red}{在debian gcc 8.3 中该语句是无法通过的; 如果带有额外变量的函数是无法转为C风格的函数指针的, 需要额外注意(C风格的指针本身也无法完成全部lambda的功能的, 这是必然的)}$

#### lambda在C++中的类型是什么呢?

有上面可以了解到, lambda不完全是函数指针, 那么当我们将其作为参数传递时, 其数据类型是什么呢? 我们可以查一下`_algo`的源码.
```
template <class _InputIter, class _Function>
_STLP_INLINE_LOOP _Function
for_each(_InputIter __first, _InputIter __last, _Function __f) {
  for ( ; __first != __last; ++__first)
    __f(*__first);
  return __f;
}
```

$\color{red}{结果}$: 是一个实现了`()`重载的class, 即函数对象; 实际可以等同于`std::function`类(头文件`#include <functional>`)

```
#include <stdio.h>
#include <stdlib.h>
#include <functional>

typedef void (*pfun)(int);

void print_v2(std::function<void(int)> fun, int a)
{
	fun(a);
}

class Pr
{
public:
	void operator()(int a)const
	{
		printf("input %d\n", a);
	}
};

int main()
{
	int extra = 100;
	print_v2([extra](int a)->void{printf("input %d, empure: %d\n", a, extra);}, 30);
	print_v2(Pr(),40);
	return 1;
}
```

`Pr()`这样的方式可以直接获得重载的`()`类型.

#### 闭包
```
#include <stdio.h>
#include <stdlib.h>
#include <functional>
#include <cstring>

void print_v2(std::function<void(int)> fun, int a)
{
	fun(a);
}

std::function<void(int)> getFun()
{
	int extra = 99;
	char *str = new char[10];
	strcpy(str, "abc");
	auto b = [=](int a)->void{printf("str %s, empure: %d\n", str, extra);};
	delete str;
	return b;
}

int main()
{
	int extra  = 199;
	print_v2(getFun(), 30);
	return 1;
}

//结果
str , empure: 99
```


* 在`[]`中指定闭包
* 如果是`[]`则说明获取所有的外部变量
* 需要注意和JS的不同, 需要考虑外部变量的释放问题

## 总结

* 1 C风格函数指针和C++并不能完全通用
* 2 对于很多STL算法而言, 自定义二元谓词可能是实现了类的模板(实际传递一个类), 可以传递任意一个实现了`()`重载的类.




