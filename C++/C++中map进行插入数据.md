# std::map的插入操作
> `map`是C++中的映射容器类, 支持`key-value`的存储方式, 那么在插入时是进行的复制还是引用呢

## 插入方式
* 1 `_map.insert(make_pair(key, value))`: 通过`make_pair`生成一个`pair`对象, 并且无需写明类型(那么可能出现一些类型问题)
* 2 `_map.insert(pair<int, string>(key, value))`: 进行类型转换
* 3 `_map.insert(map<int, string>::value_type(key,value))`: 也是进行类型转换

## 问题: map进行insert操作时是进行拷贝还是引用?
> 答案: 拷贝, 无需考虑资源的问题

## 代码实验

```
#include <iostream>
#include <map>
#include <string>

using namespace std;
class TestA
{
public:
	TestA(TestA const &ta){printf("%s\n", "copy create TestA!");}
	TestA(){printf("%s\n", "create TestA");}
	~TestA(){printf("%s\n", "delete TestA");};
	void test(){printf("%s\n", "TestA is testing!");}
};

int main(int argc, char const *argv[])
{
	map<int, TestA> _map;

	{
		TestA a;
		_map.insert(make_pair(1, a));
	}

	printf("%s\n", "insert over!");

	auto iter = _map.find(2);
	if(iter != _map.end())
	{
		iter->second.test();
	}

	return 0;
}
```
**结果是:**
```
create TestA
copy create TestA!
copy create TestA!
delete TestA
delete TestA
insert over!
delete TestA
``` 
**结论:**
* 1 在`insert`操作是必然是进行的复制操作, 而不是引用
* 3 具体时进行深度复制还是浅度复制, 就看构造函数和拷贝构造函数

## 扩展实验
> 如上情况我们会进行两次构造函数, 这是为什么呢?

```
#include <iostream>
#include <map>
#include <string>

using namespace std;
class TestA
{
public:
	TestA(TestA const &ta){printf("%s\n", "copy create TestA!");}
	TestA(){printf("%s\n", "create TestA");}
	~TestA(){printf("%s\n", "delete TestA");};
	void test(){printf("%s\n", "TestA is testing!");}
};

int main(int argc, char const *argv[])
{
	map<int, TestA> _map;

	{
		TestA a;
		_map.insert(pair<int TestA>(1, a));
		printf("%s\n", "inserting!");
	}

	printf("%s\n", "insert over!");

	auto iter = _map.find(2);
	if(iter != _map.end())
	{
		iter->second.test();
	}

	return 0;
}
```
**结果:**
```
create TestA
copy create TestA!
copy create TestA!
delete TestA
inserting!
delete TestA
insert over!
delete TestA
```
**结论:**
* 1 `pair<int TestA>(1, a)`: `a`作为值进行了一次拷贝复制(make_pair之类的结果也是一致的), 产生了一个临时值传给了`insert`, 所以这一行结束就进行了一次析构

* 2 在`insert`内部必然重新进行了一次拷贝复制, 所以外部的资源销毁了也没关系, 我们可以看到到最后`map`销毁时才一并销毁了该对象



***`map`进行插入时, 会调用拷贝构造函数, 如果拷贝复制函数进行的时深度复制, 那么外部资源插入后就无关了. 对于这样插入的对象在`map`最后销毁的时候会一并调用析构函数. 典型的对于`string`这种类型不需要使用`new`分配空间***