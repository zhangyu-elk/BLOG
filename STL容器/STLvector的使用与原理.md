# STLvector的使用与原理

##  <a name="使用">vector介绍</a>
&emsp;&emsp;`vecotr`是一个模板类, 提供各种类型的动态数组的功能; 具有如下特性: 1, 具有和数组一样的使用下标访问指定内容的功能; 2, 可以在末尾添加和删除元素; 3, 数组长度不做限制, 可以不断的添加元素, 不需要手动的管理内存. 以下均基于`C++11`

## 使用

### 头文件
```
#include <vector>
using namespace std;
```
需要引入`vector`头文件, 并且如果不使用`std`域的话, 需要以`std::vector`的方式使用

### 声明
```
std::vectro<int> std;
```
`vector`是一个模板类, 在初始化一个变量时需要显示的在`<>`中写明存储的类型。

### 初始化
* 1 用一个`{}`包括的数组初始化
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec1 = {1, 2, 3};
	vector<int> vec2{1,2, 3};

	printf("vec1 size: %d\n", vec1.size());
	printf("vec2 size: %d\n", vec2.size());
	return 0;
}

//结果:
vec1 size: 3
vec2 size: 3
```
* 2 指定长度
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec1;
	vector<int> vec2(10);
	vector<int> vec3(10, 100);

	printf("vec1 size: %d\n", vec1.size());
	printf("vec2 size: %d\n", vec2.size());
	printf("vec2[9]: %d\n", vec2[9]);
	printf("vec3 size: %d\n", vec3.size());
	printf("vec3[9]: %d\n", vec3[9]);
	return 0;
}

vec1 size: 0
vec2 size: 10
vec2[9]: 0
vec3 size: 10
vec3[9]: 100
```
可以`选择`指定长度/初始化值, 来初始化一个`vector`

* 3 以另外一个`vector`来初始化
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec1;
	vector<int> vec2(10, 99);
	vector<int> vec3(vec2);
	vector<int> vec4(vec2.begin(), vec2.begin() + 4);

	printf("vec1 size: %d\n", vec1.size());
	printf("vec2 size: %d\n", vec2.size());
	printf("vec2[9]: %d\n", vec2[9]);
	printf("vec3 size: %d\n", vec3.size());
	printf("vec3[9]: %d\n", vec3[9]);
	printf("vec4 size: %d\n", vec4.size());
	printf("vec4[9]: %d\n", vec4[3]);
	return 0;
}

//结果:
vec1 size: 0
vec2 size: 10
vec2[9]: 99
vec3 size: 10
vec3[9]: 99
vec4 size: 4
vec4[9]: 99
```
可以直接用另外一个`vector`来初始化新的`vector`, 也可以用另外`vector`的迭代器来初始化;
**注意:**使用迭代器初始化, 初始化情况是`[a, b)`. 所以在`vec4`中, 我们进行了`+4`但是长度是4而不是5.

### 插入元素
* 1 在末尾插入元素`push_back`
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec2(10, 99);
	printf("vec2 size: %d\n", vec2.size());

	vec2.push_back(777);
	printf("vec2 size: %d\n", vec2.push_back(777););
	printf("vec2[9]: %d\n", vec2[10]);
	return 0;
}

//结果:
vec2 size: 10
vec2 size: 11
vec2[9]: 777
```
使用`push_back`在末尾插入元素呢?

* 2 在指定位置插入元素`insert`
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec2(10, 99);
	printf("vec2 size: %d\n", vec2.size());

	printf("vec2[0]: %d\n", vec2[0]);
	vec2.insert(vec2.begin(), 777);
	printf("vec2 size: %d\n", vec2.size());
	printf("vec2[0]: %d\n", vec2[0]);

	vec2.insert(vec2.end(), 3, 999);
	printf("vec2 size: %d\n", vec2.size());
	printf("vec2[13]: %d\n", vec2[13]);

	vector<int> vec3{888,888,888};
	vec2.insert(vec2.end(), vec3.begin(), vec3.end());
	printf("vec2 size: %d\n", vec2.size());
	printf("vec2[16]: %d\n", vec2[16]);

	return 0;
}

//结果:
vec2 size: 10
vec2[0]: 99
vec2 size: 11
vec2[0]: 777
vec2 size: 14
vec2[13]: 999
vec2 size: 17
vec2[16]: 888
```
使用迭代器作为索引; 可以单个插入, 可以插入指定个数的相同值(也可以仅指定个数插入0), 甚至可以使用另外一个`vector`的迭代器.

### 访问元素
&emsp;&emsp;访问元素的方法有两种, 第一种如上使用下标访问, 第二种就是使用迭代器访问

```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec2{1, 2, 3};
	printf("vec2[2]: %d\n", vec2[2]);

	vector<int>::iterator it = vec2.end() - 1;
	printf("vec2.end: %d\n", *it);


	return 0;
}

//结果:
vec2[2]: 3
vec2.end: 3
```
**注意**: `end`执行结尾的下一个; 通过上面的写法可以了解到`iterator`必然是`vector`的一个子类。 $\color{red}{使用索引获得的值是一个引用, 对该值的使用会直接更改到容器内部}$

### 删除元素(使用`pop_back`在末尾删除元素)
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec2(10, 99);
	printf("vec2 size: %d\n", vec2.size());

	vec2.pop_back();
	printf("vec2 size: %d\n", vec2.size());


	return 0;
}

//结果
vec2 size: 10
vec2 size: 9
```
可以看到, 长度减少了一个

### 其他
* 1 `size()`获取存储元素的个数
* 2 `capacity()`获取容量大小, 动态属于通常预先分配一个较大的内存, 这就是这个实际纷飞内存的大小

```

#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec2;
	printf("vec2 size: %d, capactity: %d\n", vec2.size(), vec2.capacity());

	vec2.push_back(1);
	printf("vec2 size: %d, capactity: %d\n", vec2.size(), vec2.capacity());

	vec2.push_back(1);
	printf("vec2 size: %d, capactity: %d\n", vec2.size(), vec2.capacity());

	vec2.push_back(1);
	printf("vec2 size: %d, capactity: %d\n", vec2.size(), vec2.capacity());

	return 0;
}

//结果:
vec2 size: 0, capactity: 0
vec2 size: 1, capactity: 1
vec2 size: 2, capactity: 2
vec2 size: 3, capactity: 4
```
可以看到`vector`初始化时分配长度为0, 通常会预分配内存.


### 问题

* 1 如果访问越界会怎么样?
```
#include <stdio.h>
#include <stdlib.h>
#include <vector>
using namespace std;

int main()
{
	vector<int> vec2(3, 99);
	printf("vec2[3]: %d\n", vec2[3]);

	return 0;
}

//结果:
vec2[3]: 119945
```
不会崩溃但是, 获取的值时错误的;



