# STL容器之优先级队列pritrity_queue

## 优先级队列
&emsp;&emsp;优先级队列是一个队列, 但是和队列不同的是, 在队首的位置上往往存储的是一个根据二元谓词(一个判断大小的函数), 获得的最大或者最小值. 同样只能在队首操作该数据结构. 与之类似的是大顶堆和小顶堆.

## 头文件

```
#include <queue>
```

## 类的定义
```
template<class elementType,
		 class Container=vector<Type>,
		 class Compare=less<typename Container::value_type>>
class ppritrity_queue
```
从以上模板可以看出
* 1 这是一个模板类, 可以兼容各种数据类型
* 2 可以自定义内部数据的存储方式, 如`vector`或者`deque`, 默认是`vector`
* 3 可以自定义谓词, 即比较的函数

 $\color{red}{那么选取vector还是deque呢?}$
 答: `deque`和`vector`的区别在于允许在开头插入和删除, 但是我们实现堆的时候, 完全可以用数组实现; 我还没想到两者实现的比较大的区别!!!

## 是否支持迭代器?

答: 不支持, 事实上如果我们实现一个堆, 也是无法遍历的.

## 实例化方式
```
#include <queue>
using namespace std;

int main()
{
	priority_queue<int> a;
	priority_queue<int> b(a);
	priority_queue<int, vector<int>, greater<int>> c;
	return 1;
}
```

* 1 普通的不带参数初始化
* 2 使用另外一个优先级队列来初始化
* 3 通常的二元谓词包括`less<T>`和`greater<T>`两种
* 4 即使是二元谓词的不同, 也不允许用来初始化.
$\color{red}{由于不支持迭代器, 其他很多方式就不支持了}$

## 成员函数
&emsp;&emsp;这种数据结构还是一个队列, 所以支持的成员函数基本与队列相同.
* 1 push: 往队列中插入一个元素, 与队列不同的是, 不是简单的在末尾插入而是同时进行了排序
* 2 pop: 删除队首元素
* 3 top: 返回队首元素的引用
* 4 empty: 判断是否为空
* 5 size: 队列大小

```
#include <queue>
using namespace std;
#include <cstdio>
#include <stdlib.h>

int main()
{
	priority_queue<int> a;
	a.push(88);
	printf("top: %d\n", a.top());
	a.push(99);
	printf("top: %d\n", a.top());

	//a.top() = 77;
	a.pop();
	printf("top: %d\n", a.top());


	return 1;
}

//结果
top: 88
top: 99
top: 88
```

**总结**: `top`、`empty`、`size`的使用和其他基本一致, 但是注意`top`虽然是取到了引用但是无法修改(如果允许修改那么结构就变了); 同时注意虽然是在后插入的, 但是插入后进行了排序, 队首默认是最大元素.

## 重点: 怎样自定义二元谓词?

&emsp;&emsp;对于基础的数据结构, 我们完全可以使用`less<T>`和`greater<T>`获取最大堆和最小堆; 但是对于类如何继续比较呢? 我们需要注意的是, `less`和`greater`实质就是对`<`和`>`的重载. 

&emsp;&emsp;由于`less`是重载的`<`号, 并且队首是最大元素; 所以我们可以得出: 如果A/B进行比较, 返回`true`排在后否则排在前.

### 重载其`<`和`>`
```
#include <queue>
using namespace std;
#include <cstdio>
#include <stdlib.h>

class A
{
public:
	bool operator<(const A& b) const
	{
		return index < b.index;
	}
	int index;
	A(int i){
		index = i;
	}
	
};

int main()
{
	priority_queue<A> pa;
	pa.push(A(1));
	pa.push(A(2));


	printf("top: %d\n", pa.top().index);

	return 1;
}

//结果
top: 2
```

**总结**: 重载`<`成功实现了类的排序
**注意**: 由于调用了`<`所以该重载必须是常函数并且是`public`的

### 如果我们保存的是其指针呢?

&emsp;&emsp;我们是无法对类指针的大于号和小于号进行重载的, 那我们应当怎么实现? 我们需要额外实现一个类
```
#include <queue>
using namespace std;
#include <cstdio>
#include <stdlib.h>

class A
{
public:
	int index;
	A(int i){
		index = i;
	}
	
};

class ComA
	{
	public:
		bool operator () (A* &a, A* &b) const
		{
			return a->index < b->index;
		}
	};

int main()
{
	priority_queue<A*, vector<A*>, ComA> pa;
	pa.push(new A(1));
	pa.push(new A(9));
	pa.push(new A(8));
	pa.push(new A(6));
	pa.push(new A(2));

	printf("top: %d\n", pa.top()->index);

	return 1;
}

//结果
top: 9
```

**分析**: 为什么是这么实现原理我也不甚了解, 我们我们可以进行反向的分析
* 1 模板中第三个模板就是一个类, 所以我们需要提供的也是一个类;
* 2 肯定不会是调用这个另外定义类的大于等于号, 那么可能情况的确就是使用`()`再在内部调用`<>`



