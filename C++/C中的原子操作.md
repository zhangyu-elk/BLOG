## C中的原子操作

### volitate关键字
> 以前也知道这个关键字, 但是直到现在也没有深入了解过, 看了网上的一些说明, 做一个记录
**直接上代码**
```
static int i=0;
int main(void)
{
...
	while (1)
	{
	if (i) do_something();
	}
}
```
这样的代码非常常见, 等待另外一个线程修改标志位, 然后进行一些操作
**问题**:实际这段代码可能永远不会被调用
**原因**:编译器在优化的时候将i值读入寄存器, 然后等待那个寄存器变为1(其他线程改变了值,该寄存器的值不会改变), 当然这个编译器优化等级有关
**volitate**: 每次获取值都从内存中读取
ps: 该关键词经常和嵌入式相关, 可以和const一起用

### gcc提供的原子操作
> 再C语言中即使是一个简单的自加操作也涉及到三步(从内存取到寄存器, + 1, 写入内存), 需要加锁
> [网上例子](https://blog.csdn.net/hzhsan/article/details/25837189)
#### 要求
gcc.version > 4.1.2
ps: 编译时可能需要加上 `-march=i686`

#### 函数族
```
//在用gcc编译的时候要加上选项 -march=i686
type __sync_fetch_and_add (type *ptr, type value, ...);	//+
type __sync_fetch_and_sub (type *ptr, type value, ...);	//-
type __sync_fetch_and_or (type *ptr, type value, ...);	// |
type __sync_fetch_and_and (type *ptr, type value, ...);	// &
type __sync_fetch_and_xor (type *ptr, type value, ...);	// ^
type __sync_fetch_and_nand (type *ptr, type value, ...); //与非(与门和非门叠加)
type __sync_add_and_fetch (type *ptr, type value, ...);
type __sync_sub_and_fetch (type *ptr, type value, ...);
type __sync_or_and_fetch (type *ptr, type value, ...);
type __sync_and_and_fetch (type *ptr, type value, ...);
type __sync_xor_and_fetch (type *ptr, type value, ...);
type __sync_nand_and_fetch (type *ptr, type value, ...);
```
说明
```
int  i = 1;
__sync_fetch_and_add(&i, 1)	//相当于i++
__snyc_add_and_fetch(&i, 1)	//相当于++i
```
其他的看名字就能够明白其操作, 今天仅做一下记录说明, 需要用的时候再深入研究(好像也不需要研究, 用就可以)



### 自旋锁
```
//第1和第2就是典型的CAS, 如果*ptr = oldValue, 就将newValue写入*ptr
//第一个函数在相等并写入的情况下返回true
//第二个函数返回操作之前的值

bool __sync_bool_compare_and_swap(type* ptr, type oldValue, type newValue, ....);

type __sync_val_compare_and_swap(type* ptr, type oldValue, type newValue, ....);

//将*ptr设为value并返回*ptr操作之前的值
type __sync_lock_test_and_set(type *ptr, type value, ....);

//置*ptr为0
void __sync_lock_release(type* ptr, ....);
```

> 自旋锁与互斥锁的区别就是, 不会让出CPU睡眠
**优点**:效率高
**缺点**:一直占用CPU, 如果一直没有获得锁, CPU效率降低
```
while (!(__sync_bool_compare_and_swap (&mutex,lock, 1) ))
```
这就是一个类似自旋锁, 不断的判断而不是阻塞睡眠, 但是如果用的不好效率可能更低



### __sync_synchronize函数

> [在网上找的一篇说明](https://blog.csdn.net/sz76211822/article/details/84965238)
> [第二篇](https://www.cnblogs.com/FrankTan/archive/2010/12/11/1903377.html)

**在代码中的一个简单顺序赋值指令, 实际执行顺序是不定的**
该函数时为了解决上述问题的, 实际发出一个full barrier

***以下转载自第二篇文章中的***
关于memory barrier,cpu会对我们的指令进行排序，一般说来会提高程序的效率，但有时候可能造成我们不希望得到的结果，举一个例子，比如我们有一个硬件设备，它有4个寄存器，当你发出一个操作指令的时候，一个寄存器存的是你的操作指令（比如READ），两个寄存器存的是参数（比如是地址和size），最后一个寄存器是控制寄存器，在所有的参数都设置好之后向其发出指令，设备开始读取参数，执行命令，程序可能如下：
```
    write1(dev.register_size,size);
    write1(dev.register_addr,addr);
    write1(dev.register_cmd,READ);
    write1(dev.register_control,GO);
```

如果最后一条write1被换到了前几条语句之前，那么肯定不是我们所期望的，这时候我们可以在最后一条语句之前加入一个memory barrier,强制cpu执行完前面的写入以后再执行最后一条：
```
    write1(dev.register_size,size);
    write1(dev.register_addr,addr);
    write1(dev.register_cmd,READ);
    __sync_synchronize();
    write1(dev.register_control,GO);
```
memory barrier有几种类型：

* acquire barrier : 不允许将barrier之后的内存读取指令移到barrier之前（linux kernel中的wmb()）。
* release barrier : 不允许将barrier之前的内存读取指令移到barrier之后 (linux kernel中的rmb())。
* full barrier    : 以上两种barrier的合集(linux kernel中的mb())。






