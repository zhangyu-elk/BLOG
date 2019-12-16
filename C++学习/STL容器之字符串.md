# STL容器之字符串

## STL容器字符串的简单介绍

&emsp;&emsp;`#include <string>`作为头文件引入, 需要使用`std`的命名空间.

&emsp;&emsp;`string`提供的是一个动态字符串数据结构, 一方面减少字符串创建销毁的操作隐藏内存的操作; 另外一方面提供了方便的追加、截断等函数.

## `string`的实例化

```
#include <queue>
#include <string>
#include <iostream>
using namespace std;



int main()
{
	string a;
	string b = "xxx";
	string c = b;
	cout<<a<<" | "<<b<<" | "<<c<<endl;

	string d(b, 2);
	string e(10, 'x');
	cout<<d<<" | "<<e<<endl;
}

//结果
 | xxx | xxx
x | xxxxxxxxxx
```
* 1 无参数初始化
* 2 使用字符串`char*`初始化
* 3 使用另外一个`string`初始化
* 4 使用另外一个的前几个字符
* 5 指定数量的相同字符.

$\color{red}{**注意**:}$
	* 这里使用另外一个字符串来初始化的方式采用的是深度复制; 初始化后就没有关系
	* 不能使用`printf`来打印`string`

## 访问`string`的字符内容
```
#include <queue>
#include <string>
#include <iostream>
using namespace std;



int main()
{
	string a = "abcdefg";
	cout<<a[4]<<endl;
	printf("%c\n", a[4]);
	printf("%c\n", a[9]);

	a[4] = 'x';
	a[9] = 'x';
	cout<<a<<endl;
}

//结果
e
e

abcdxfg
```
* 可以使用类似于数组访问下标的方式访问数据
$\color{red}{**注意**:}$
	* 通过下标获得的是该字符的索引, 允许被更改
	* 如果访问下标超出了并不会引起崩溃和错误.

### 获得`char*`类型及迭代器
```
#include <queue>
#include <string>
#include <iostream>
using namespace std;



int main()
{
	string a = "abcdefg";
	string::iterator c = a.begin();
	printf("%s\n", c+2);
	*c = 'x';

	cout<<*(c)<<endl;

	printf("%s\n", a.c_str());
}

//结果
cdefg
x
xbcdefg
```
$\color{red}{**注意**:}$
	* 迭代器实质就是`char*`类型
	* `c_str`获得的是`const`类型, 不允许修改


## 拼接字符串

&emsp;&emsp;如果仅仅只有以上这些特性, `string`的优越性就体验不出来了; `string`提供字符串拼接功能, 可以使用函数`append`或者一下脚本语言中`+`运算符实现.

```
#include <queue>
#include <string>
#include <iostream>
using namespace std;



int main()
{
	string a = "abcdefg";
	string b = a + "ABC";

	cout<<b<<endl;
	b.append("DEF");
	cout<<b<<endl;
}

//结果
abcdefgABC
abcdefgABCDEF
```

$\color{red}{**注意**:}$
	* 这必然是实现的`string`类`+`的重载, 所以进行`+`的参数必须要有一个是`string`类(必须区分与`char*`类型).

## 字符串的查找

### C语言中的字符串中子字符串和字符的查抄

* `<string.h>`标准库文件

* `char *strchr(const char *str, int c)`: 用于查找指定字符第一次出现的位置, 每次记录位置多次即可查找所有; (char类型直接传为int型传入).

* `char *strstr(const char *haystack, const char *needle)`: 查找指定字符串第一次出现的位置.

### `string`的查找(find函数)

```
#include <queue>
#include <string>
#include <iostream>
using namespace std;



int main()
{
	string a = "day 1 day 2 day 3 day 4";

	size_t pos = -1;
	while(true)
	{
		pos = a.find("day", pos+1);
		if(pos == string::npos)
		{
			break;
		}
		else
		{
			cout<<"find: "<<pos<<endl;
		}
	}

	pos= -1;
	while(true)
	{
		pos = a.find('a', pos+1);
		if(pos == string::npos)
		{
			break;
		}
		else
		{
			cout<<"find: "<<pos<<endl;
		}
	}
}

//结果
find: 0
find: 6
find: 12
find: 18
find: 1
find: 7
find: 13
find: 19
```

* 1 函数原型: 从这些原型就可以很简单的看出使用了, 就不做特别详细的介绍了
```
size_t find ( const string& str, size_t pos = 0 ) const;
size_t find ( const char* s, size_t pos, size_t n ) const;
size_t find ( const char* s, size_t pos = 0 ) const;
size_t find ( char c, size_t pos = 0 ) const;
```

* 2 `find`返回的是找到的索引下表,`npos`实质就是`-1`
* 3 这里实现的就是寻找找出所有的字符串和字符.

#### `find`的扩展(函数原型均和`find`一致)
* 1 `find_first_of`: 查找子字符串任意一个字符(也可以直接传入字符)在主串中第一次出现的位置.
* 2 `find_first_not_of()`: 查找主串中第一个和传入字符串(或直接传入字符)都不匹配的字符的位置。
* 3 `find_last_of`: 从后往前查找第一个在指定参数中的字符位置.
```
size_t find_last_of (const string& str, size_t pos = npos) const noexcept;
size_t find_last_of (const char* s, size_t pos = npos) const;
size_t find_last_of (const char* s, size_t pos, size_t n) const;
size_t find_last_of (char c, size_t pos = npos) const noexcept;
```
* 4 `find_last_not_of`: 顾名思义

$\color{red}{[C++文档](http://www.cplusplus.com/)}$

## 截短字符串(`erase`函数)

### 函数原型
```
string& erase (size_t pos = 0, size_t len = npos);
iterator erase (const_iterator p);
iterator erase (const_iterator first, const_iterator last);
```

### 代码测试
```
#include <queue>
#include <string>
#include <iostream>
using namespace std;



int main()
{
	string a = "day1day2day3day4";

	cout<<a<<endl;
	a.erase(0, 3);
	cout<<a<<endl;

	auto c = a.erase(a.begin()+1);
	cout<<a<<endl;
	printf("%s\n", c);
	c = a.erase(a.begin()+1, a.end()-1);
	cout<<a<<endl;
	printf("%s\n", c);
}

//结果
day1day2day3day4
1day2day3day4
1ay2day3day4
ay2day3day4
14
4
```

$\color{red}{结果分析}$

	* 1 从函数原型就大致知道使用方式了
	* 2 对于使用迭代器返回值也是迭代器, 返回删除字符串(或字符)的下一个位置的迭代器

ps: 很多函数其是看一下原型就知道使用了, 不过往往还是有一些细节不清楚.

### 扩展

* `clear` 清空字符串

* `being`/`length`/`size`等基本函数

## 截取字符串 `substr`和`substring`

> 函数原型: `string substr (size_t pos = 0, size_t len = npos) const;`

```
#include <queue>
#include <string>
#include <iostream>
#include <algorithm>
using namespace std;



int main()
{
	string a = "abcba";
	string b = a.substr(1);
	string c = a.substr(2, 2);
	string d = a.substr(2, -1);
	cout<<a<<" | "<<b<<" | "<<c<<" | "<<" | "<<d<<endl;

}

//结果
abcba | bcba | cb |  | cba
```
第一个参数是从第几个索引开始
第二个参数是截取的长度, 如果不带该参数或者传入负数默认截取索引后所有, 0则返回空字符串。


## 泛型函数`reverse`和`transform`
```
#include <queue>
#include <string>
#include <iostream>
#include <algorithm>
using namespace std;



int main()
{
	string a = "abcba";
	cout<<a<<endl;
	reverse(a.begin(), a.end());

	cout<<a<<endl;
	transform(a.begin(), a.end(), a.begin(), ::toupper);
	cout<<a<<endl;

}

//结果
abcba
abcba
ABCBA
```

$\color{red}{结果分析}$:
	* `transform`第三个参数是输出值
	* 笔者每看过源码, 但是可以从这里推测 1, 所有模板类必然继承同一个基类; 2, 模板算法都放在`algorithm`中


