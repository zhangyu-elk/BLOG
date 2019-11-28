# C++常对象-常函数-常成员变量

## C++常对象
**C++常对象, 就是使用const修饰的类实例!**
`const A a;`

### C++常对象有什么限制
1, 无法修改内部成员; 2, 无法调用普通函数
```
class TestA
{
public:
	TestA(){};
	int a;
	void fun1 (){}
};

int main()
{
	TestA const ta;
	ta.fun1();
	ta.a = 1;

	return 0;
}

//结果: 编译错误
test.cpp: In function 'int main()':
test.cpp:13:10: error: passing 'const TestA' as 'this' argument discards qualifiers [-fpermissive]
  ta.fun1();
          ^
test.cpp:7:7: note:   in call to 'void TestA::fun1()'
  void fun1 (){}
       ^~~~
test.cpp:14:9: error: assignment of member 'TestA::a' in read-only object
  ta.a = 1;
         ^
```

**结果**: 无法调用普通的函数和修改成员变量, 不过需要注意的是`passing 'const TestA' as 'this'`这一句; 说明我们需要定义首参为`const this`的成员函数. 

## C++常函数
**C++常函数, 就是类的成员函数, 在花括号之前添加一个`const`修饰`this`的函数**
`void fun1 () const{}`

ps: 如果是在`func1`前的`const`仅是修饰返回值而已. 这里的`cosnt`实际修饰的就是`this`

### 那么C++常对象和普通对象是否可以调用呢?
```
class TestA
{
public:
	TestA(){};
	int a;
	void fun1 () const{}
};

int main()
{
	TestA const ta;
	ta.fun1();

	TestA tb;
	tb.fun1();

	return 0;
}
```

**结果**:编译成功, 可以调用.

ps: 这里传入的是一个`const`修饰的常对象, 所以在常对象中我们无法调用非常函数和修改任何成员变量.

## C++常成员变量
`const int a;`

**C++常成员变量, 只能在初始化阵列中初始化, 此后都不再可以修改.**