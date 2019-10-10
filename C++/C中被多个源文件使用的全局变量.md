## C中被多个源文件使用的全局变量该怎么定义?
> 在工作中遇到了一个问题, 就是一个全局变量需要在多个文件中使用. 

### 直接在头文件中定义?
> 最开始我的做法就是直接在头文件中定义该变量, 但是这是有问题的!

#### `include`的功能
***#include 命令告诉预处理器将指定头文件的内容插入到预处理器命令的相应位置***
简单的说就是直接把代码放到`include`的地方, 如果我在一个头文件中包含一个全局变量, 实际上所有引入该头文件的源文件都包含了一个同名的全局变量

#### 重名符号规则
* 1 如果有多个重名的强符号，则报错。
* 2 如果有一个强符号，多个弱符号，则以强符号为准。
* 3 如果没有强符号，但有多个重名的弱符号，则任选一个弱符号

> ps: 我本人一直没有深入研究过强符号和弱符号, 只知道有这个东西的存在! 这里不做深入研究, 仅说明与本次实验相关的知识

#### 代码测试(一)
```
//main.c
#include <stdio.h>

#include "global.h"

int main() {
    A_func();
    B_func();
    printf("Hello, World: %d!\n", c);
    return 0;
}
```

```
//A.c
#include "global.h"

void A_func()
{
    printf("A file: %d\n", c++);
    return;
}

```

```
//B.c
#include "global.h"

void B_func()
{
    printf("B file: %d\n", c++);
    return;
}
```

```
//global.h
#ifndef CTEST_GLOBAL_H
#define CTEST_GLOBAL_H

#include <stdio.h>
#include <stdlib.h>

int c;

void A_func();
void B_func();


#endif //CTEST_GLOBAL_H
```

**结果:**
```
A file: 0
B file: 1
Hello, World: 2!
```
发现似乎与之前的说明时不相符的, 这是为什么呢?


**我们做一个相近的测试:**
```
int c = 1;
```
仅修改`global.h`中的这一行, 结果会有什么不一样呢?

```
CMakeFiles\CTest.dir/objects.a(A.c.obj):E:/CLionProjects/CTest/global.h:11: multiple definition of `c'
CMakeFiles\CTest.dir/objects.a(main.c.obj):E:/CLionProjects/CTest/global.h:11: first defined here
CMakeFiles\CTest.dir/objects.a(B.c.obj):E:/CLionProjects/CTest/global.h:11: multiple definition of `c'
CMakeFiles\CTest.dir/objects.a(main.c.obj):E:/CLionProjects/CTest/global.h:11: first defined here
```
直接报出了编译错误, 这与我们预料的结果是一致的, 这里就涉及到刚刚所说的强符号和弱符号了

> ***对于全局变量如果进行了初始化为强符号, 否则为弱符号. 初始化的符号在目标文件的bss段中，而初始化的符号在data段中***, 与符号规则一致, 对于弱符号选择的是最开始初始化的那一个弱符号, 对于强符号则冲突报错! 用更简单的话来说, gcc视这样的没有初始化的变量为extern而非define.


#### 正确做法
> 我个人认为将一些应该在代码上体现的东西由编译器实现是不合理的, 所以我认为最好像下面这样写
```
//main.c
#include <stdio.h>

#include "global.h"
int c = 100;
int main() {
    A_func();
    B_func();
    printf("Hello, World: %d!\n", c);
    return 0;
}
```

```
#ifndef CTEST_GLOBAL_H
#define CTEST_GLOBAL_H

#include <stdio.h>
#include <stdlib.h>

extern int c;

void A_func();
void B_func();


#endif //CTEST_GLOBAL_H
```
在头文件中声明, 在某个源文件中定义, 这样在所有的源文件中都是公用的

**结果**
```
A file: 100
B file: 101
Hello, World: 102!
```
结果正确

ps: **需要注意, GCC先编译后链接, 这样的手法仅能用于多个文件编译为同一个可执行文件或者链接库. 或者定义(强链接)在某个库中, 然后链接这个库也可以!**

