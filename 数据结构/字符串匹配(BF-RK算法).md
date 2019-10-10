## 字符串匹配算法
> 字符串匹配想必任何人都不会陌生吧, 今天介绍两种字符串匹配中的常用算法

### BF(Brute Force)
> 强制匹配, 没有什么技术含量, 从头到尾逐个逐个调用`strncmp`这样的函数比较
	绝大多数时候, 使用这种方法就够了, 代码如下
```
/*Brute Force string match*/
bool BF(char *main, char *pattern)
{
    int mlen = strlen(main);
    int plen = strlen(pattern);
    for(int i = 0; i <= (mlen - plen); i++)
    {
        if(0 == strncmp(main + i, pattern, plen)) {
            //printf("BF match succ: index = %d\n", i, GetTickCount());
            return true;
        }
    }
    return false;
}
```
时间复杂度O(m\*n), (m-n+1)个点被访问, 每次需要比较n个字符(m为主串长度,n为子串长度)

### RK(Rabin Karp)算法
> 通过`hash`的方式减少比较, 不过`hash`的方式有点巧妙
	假设我们只有`a-z`这26个字母, 我们可以将这个字符串变为一个26进制的数
```
abc -> 012 -> 0*26^\^^2 + 1*26^\^^1 + 2
```
	这基本的换算, 程序员应该都是明白的, 以这样的方式进行如果`hash`的结果相同, 那么字符串也一定相同,
但是实际当中必然要考虑到冲突的问题以及最大数超出的问题(我在测试的时候, 生成的字符串间隔为65, 长度为20, 这样的数字根本无法保存在C语言中的类型里面), 于是采用了一种取巧的方式(每个数字相加)
```
abc -> 012 -> 0 + 1 + 2
```
	这样要求我们在`hash`值相同的情况下, 还是要比较一下字符串, 但是已经可以减少字符串比较了

```
/*Rabin Karp算法*/
bool RK(const char *main, const char *pattern)
{
    int mlen = strlen(main);
    int plen = strlen(pattern);

    int target = 0;
    for(int i = 0; i < plen; i++)
    {
        target += (pattern[i] - ' ');
    }
    int last = 0;
    for(int i = 0; i <= (mlen - plen); i++)
    {
        if(last > 0)
        {
            last = last - (main[i-1] - ' ') + (main[i+plen-1] - ' ');
        }
        else
        {
            for(int j = 0; j < plen; j++)
            {
                last += (main[i+j] - ' ');
            }
        }
        if(last == target && 0 == strncmp(main+i, pattern, plen))
        {
            //printf("RX match succ: index = %d\n", i);
            return  true;
        }
    }
    //printf("BF match fail!\n");
    return  false;
}
```
时间复杂度变为O(m)

### BM(boyer moore)算法
> BM算法也是一种字符串匹配算法, 分别包含坏字符规则和好后缀规则
> 假设主串`main`长度为`m`, 模式串`pattern`长度为`n`
#### 坏字符规则
```
a b c d a b c d
a b d
```
对于这样的一个匹配, 对于BF算法来说每次移动一位, 但是我们实质上可以一次移动3位, 而不必担心错过
```
a b c d a b c d
      a b c
```
因为`c`在匹配字符串中不存在, 移动`1,2`位都绝对不可能成功匹配

**坏字符规则指的是**:
```
a b c d e f g
c b a d 
```
对于这样的一个匹配过程来说, 当我们遍历`pattern`从后往前, `pattern[j]`是不匹配的(j=2), 然后从后往前第一个匹配的字符是`pattern[i]`(i=0), 那我们可以直接移动`j-i`位, 而不担心错误误判

##### 算法说明

&emsp;&emsp;我们可以事先记录以下每一个字符对应的位置(相同的字符以下标大的为准), ASCII字符也不过100多个. 这一步的时间复杂度和空间复杂度均为 O(n). 

> ***代码在之后一并给出, 需要说明如果单纯的只有坏字符规则,其效率未必会比BF算法来的好, 第一其最小也可能之移动一位, 第二BF算法可以使用`strcmp`或者`memcmp`等算法(strcmp采用四字节一比较), 实际效率难以考量***

#### 好后缀

```
a b c d e f g h i j k 
f g e d h f g
```
对于这样的一个字符串比较, 如果使用坏字符规则, 其可以移动`2`个字节(`e`字符)

但是我们要注意, `fg`两个字符是已经匹配相等的, 我们可以非常清楚的看到移动两个字节是绝对不可能成功的, 因为`fg`不匹配

**好后缀的规则是:**
&emsp;&emsp;第一, 对于以上的字符串匹配, 假设已经存在一段字符串`fg`成功匹配(在第一个不匹配的字符`h`之后), 那么我们类似于坏字符也必须要在`pattern`前部分`f g e d`之中找到和`fg`相匹配的子串才有可能成功

**移动如下:** 
```
a b c d e f g h i j k 
          f g e d h f g
```

&emsp;&emsp;或者, 在`pattern`中找到和匹配字符串`efg`的后缀子串相匹配的前缀子串, 如下

```
a b c d e f g h i j k 
f g c c e f g
```
**移动如下**
```
a b c d e f g h i j k 
          f g c c e f g
```

##### 算法说明
&emsp;&emsp;我们需要也必须提前生成一些数据, 如果每次计算那么其优势就不大了; 我们必须记录每一个后缀子串 与之 相同的其他子串 的起始下标
&emsp;&emsp;思路是: 可以用一个`int`数组, `key`为后缀子串的长度, `value`为相匹配子串的起始下标(如果有多个,记录下标大的); 如果存在匹配的起始下标为`0` 的子串, 那么该后缀子串可以考虑好后缀规则的第二条, 可以用一个`bool`类型数组, `key`为后缀子串的长度

这里空间复杂度是O(p), 时间复杂度O(n^\^2^)


### 全部代码
> 算法思路都已经说明了, 不过实际写代码可能涉及到很多的下标和距离计算(C以0开头的吗), 建议手写画图理解一下
```
/*Brute Force string match*/
bool BF(char *main, char *pattern)
{
    int mlen = strlen(main);
    int plen = strlen(pattern);
    for(int i = 0; i <= (mlen - plen); i++)
    {
        if(0 == strncmp(main + i, pattern, plen)) {
            //printf("BF match succ: index = %d\n", i, GetTickCount());
            return true;
        }
    }
    return false;
}

/*Rabin Karp算法*/
bool RK(const char *main, const char *pattern)
{
    int mlen = strlen(main);
    int plen = strlen(pattern);

    int target = 0;
    for(int i = 0; i < plen; i++)
    {
        target += (pattern[i] - ' ');
    }
    int last = 0;
    for(int i = 0; i <= (mlen - plen); i++)
    {
        if(last > 0)
        {
            last = last - (main[i-1] - ' ') + (main[i+plen-1] - ' ');
        }
        else
        {
            for(int j = 0; j < plen; j++)
            {
                last += (main[i+j] - ' ');
            }
        }
        if(last == target && 0 == strncmp(main+i, pattern, plen))
        {
            //printf("RX match succ: index = %d\n", i);
            return  true;
        }
    }
    //printf("BF match fail!\n");
    return  false;
}

void generateBC(const char* pattern, int bc[])
{
    for(int i = 0; i < strlen(pattern); i++)
    {
        bc[(int)pattern[i]] = i;
    }
}

void generateGS(const char* pattern, int suffix[], bool prefix[])
{
    int plen = strlen(pattern);
    for(int i = 0; i < plen; i++)
    {
        suffix[i] = -1;
        prefix[i] = false;
    }


    for(int i = 0; i < plen - 1; i++)
    {
        int j = i;
        int k = 0;
        while(j >= 0 && pattern[j] == pattern[plen - k - 1])
        {
            k++;
            j--;
            suffix[k] = j + 1;
        }

        if(j == -1)
        {
            prefix[k] = true;
        }
    }
}

int moveByGS(int j, int plen, int suffix[], bool prefix[])
{
    int k = plen - j - 1;   //后缀的长度

    if(suffix[k] != -1)
    {
        return j - suffix[k] + 1;
    }

    for(int i = j + 2; i < plen; i++)
    {
        if(prefix[plen - i])
        {
            return i;
        }
    }

    return plen;
}

/*boyer moore
 * bad char rule
 * good suffic shift
 * */
bool BM(const char *main, const char *pattern)
{
    int bc[128] = { -1 };
    generateBC(pattern, bc);

    int i = 0;
    int mlen = strlen(main);
    int plen = strlen(pattern);

    int *suffix = new int[plen];
    bool *prefix = new bool[plen];
    generateGS(pattern, suffix, prefix);
    while(i < (mlen - plen)) {
        int j;
        /*模式串从后往前匹配*/
        for (j = plen - 1; j >= 0; j--)
        {
            if (main[i + j] != pattern[j])
            {
                //坏字符对应的下边为j
                break;
            }
        }
        if (j < 0)
        {
            //printf("BM match succ: index = %d\n", i);
            return true; //匹配成功
        }

        int update = j - bc[(int) main[i+j]];
        if(j < plen - 1)
        {
            int GSupdate = moveByGS(j, plen, suffix, prefix);
            if(GSupdate > update)
            {
                update = GSupdate;
            }
        }

        i += update;
    }

    return false;
}

/*create a random string and it's a substring
 * between 64-126
 * */
static void random_string(char main[], int mlen, char sub[], int slen)
{
    srand(time(NULL));
    for(int i = 0; i < mlen; i++)
    {
        int r = rand() % (126 - 64) +64;
        main[i] = (char)r;
    }
    int r = rand() % (mlen - slen + 1);
    r = r % (mlen /2) + mlen / 2;
    strncpy(sub, main + r, slen);
    printf("main string: %s\nsub string: %s\n", main, sub);
}


int main()
{
    char mains[30000] = { 0 };
    char subs[101] = { 0 };
    random_string(mains, 29999, subs, 100);
    DWORD start = GetTickCount();
    for(int i = 0; i< 1000; i++)
    {
        BF(mains, subs);
    }
    printf("BF time: %ld\n", GetTickCount() - start);

    start = GetTickCount();
    for(int i = 0; i< 1000; i++)
    {
        RK(mains, subs);
    }
    printf("RK time: %ld\n", GetTickCount() - start);

    start = GetTickCount();
    for(int i = 0; i< 1000; i++)
    {
        BM(mains, subs);
    }
    printf("BM time: %ld\n", GetTickCount() - start);
    return 0;
}
```

### 时间计算
```
GetTickCount()
```
windows下可以相减计算毫秒数
```
gettimeofday()
```
Linux下可以获得微妙级别

### 测试结果
> windows下运行
#### 30000字节(1000次, 匹配100个)
```
BF time: 235
RK time: 188
BM time: 15

BF time: 203
RK time: 94
BM time: 16
```

#### 10000字节
```
BF time: 62
RK time: 31
BM time: 0

BF time: 47
RK time: 31
BM time: 0
```

#### 3000字节(10000次)
```
BF time: 156
RK time: 94
BM time: 31

BF time: 141
RK time: 78
BM time: 31
```

#### 199字节(10000次, 匹配20个)
```
BF time: 109
RK time: 94
BM time: 93

BF time: 140
RK time: 78
BM time: 141
```


### 结论
***对于超长字符串匹配, BM算法优势还是很大的, 比如文本的查找功能, 不过如果不是这么复杂的情况, 简单点就好, 如果有问题欢迎指正, 这里也没测试的特别详细***

