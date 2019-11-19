# C传递可变参数

## `va_list`
&emsp;&emsp;`va_list`配合`va_start`/`va_arg`/`va_end`宏定义, 可以完成C语言中可变参数的传递获取. 需要注意的是, 必须有一个固定参数; 其次三个宏的使用必须严格一致。

### 示例
```
#include <stdarg.h>
#include <stdio.h>

void test1(int n, ...)
{
	va_list ap;
	va_start(ap, n);
	const char *tmp = NULL;

	for(int i = 0; i < n; i++)
	{
		tmp = va_arg(ap, const char*);
		if(tmp)
		{
			printf("%s  ", tmp);
		}
	}
}

int main()
{
	test1(3, "a", "b", "c", "d");

	return 0;
}
```
**结果**
```
a  b  c  
```

**注意事项**
* 1 我们通常使用第一个值作为可变参数的个数; 但是实际中我们再函数内无法得到传递的参数个数, 我们可以无限制的调用`va_arg`,  假设我们将循环次数改为4次, 最后一次获取的值是不确定的, 如下是乱码.
```
a  b  c ?(
```
* 2 这个函数的使用是不严格且不安全的

## 自己实现可变参数传递
&emsp;&emsp;首先我们需要明白函数入栈的规则, 函数参数入栈从右到左; 如果我们知道第一个参数的地址并且知道参数的类型, 我们就可以得到其余参数的地址了. 这就是`va_list`需要一个固定参数的原因, 如果没有该固定参数我们是无法进行任何操作的.

### 参数类型是否必须一致?
```
#include <stdio.h>
#include <stdlib.h>

void func(int a, ...)
{

}

void main()
{
	int a = 10;
	short b = 10;
	printf("sizeof int: %d\n", sizeof(int));
	printf("sizeof ptr: %d\n", sizeof(char*));
	printf("sizeof short: %d\n", sizeof(short));
	func(a, "hello", b, "world");
}
```

**结果是**:
```
sizeof int: 4
sizeof ptr: 4
sizeof short: 2
```
可以成功编译通过并执行, 可以得知编译器内部并没有做特别详细的判断和区分; 那么就会引入一个新的问题, 由于不同类型的字节数不同, 会不会产生字节对齐的问题?

### 会不会产生字节对齐?
```
#include <stdio.h>
#include <stdlib.h>

void func(int a, ...)
{
	const char *argv = (const char *)&a;					//指向栈的指针

	int Arg1 = a;
	printf("Arg[1]: %d\n", Arg1);

	const char *Arg2 = *(const char **)(argv + 4);			
	//将指向栈中第二个参数的指针转换为为指向字符串指针的指针, 再进行解引用就可以获得栈中的字符串指针

	printf("arg[2]: %s\n", Arg2);

	short Arg3 = *(short*)(argv + 8);
	printf("arg[3]: %d\n", Arg3);

	const char *Arg4 = *(const char**)(argv + 12);
	printf("arg[4]: %s\n", Arg4);
}

void main()
{
	int a = 10;
	short b = 100;
	printf("sizeof int: %d\n", sizeof(int));
	printf("sizeof ptr: %d\n", sizeof(char*));
	printf("sizeof short: %d\n", sizeof(short));
	func(a, "hello", b, "world");
}
```
**结果**:
```
sizeof int: 4
sizeof ptr: 4
sizeof short: 2
Arg[1]: 10
arg[2]: hello
arg[3]: 100
arg[4]: world
```
结论: 产生了字节对齐