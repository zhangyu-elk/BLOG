# KMP字符串匹配算法

## 先总结一下之前的几种字符串匹配算法

* 1 BF算法, 最简单的字符串匹配算法, 可以直接使用`strncmp`逐个匹配过去
* 2 RK算法, 利用了HASH的方式, 将字符串匹配变为数值比对
* 3 BM算法, 坏字符规则和好后缀规则
* 4 KMP算法, 和BM算法类似尽可能的一次多移动一些

## KMP算法思路

假设有如下的两个字符串
```
a b c d a b c d
a b c d e f
```
我们遇到了第一个坏字符`e`, 在BM算法中的好后缀规则考虑的是坏字符后的后缀, 而KMP算法考虑的则是前缀, 我们在往后滑动时必然也是需要考虑到前缀的匹配问题的.

比如如下的字符串匹配:
```
a b c a b c a b c a b c a b c
a b c a b c e
```
第一个坏字符`e`, 考虑前缀匹配的话, 如果向右移动的话可以直接移动三位
```
a b c a b c a b c a b c a b c
	  a b c a b c e
```

**总结的规则就是:**模式串已匹配的前缀`a b c a b c`好前缀, 在移动后好前缀的前缀(第二行)`a b c`需要匹配好前缀的后缀(第一行)
```
a b c a b c
	  a b c a b c
```

**注意**: 如果每一次字符串匹配的话都以最粗暴的方式进行好前缀匹配的话, 其效率显然也不会高

## 失效数组
&emsp;&emsp; 失效数组就是我们提前计算的一个数组用于计算好前缀的移动位置, 以`a b c a b c e`为例子

好前缀|前缀字符串结尾下标|最长可匹配前缀子串结尾下标
:--|:--|:--
a|0|-1(表示不存在)
a b|1|-1
a b c|2|-1
$\color{red}{a}$ b c $\color{red}{a}$|3|1
$\color{red}{a b}$ c $\color{red}{a b}$|4|2
$\color{red}{a b c}$ $\color{red}{a b c}$|5|3

这里做一下说明, 对于`a b c a b c e`如上5个均有可能作为匹配的前缀子串; 对于`a b c a b c`这样的好前缀(这6个与主串匹配,下一个不匹配), 其可移动的位数就是3位. 以如上表建立一个数据`next`,  对于遇到坏字符的好前缀, 如果该前缀字符串结尾下标为`n`, 那么我们可移动`index - next[n]`(index为坏字符的位置)

## 先写一下代码框架
```
bool KMP(const char *main, const char *pattern)
{
	int mlen = strlen(main);
    int plen = strlen(pattern);

    int *next = new int[plen];
    getNext(next, pattern);

    for(int i = 0; i < mlen; i++)
    {
    	while(j > 0 && a[i] != b[j])
    	{
    		j = next[i] + 1;
    	}

    	if(a[i] == b[j])
    	{
    		++j;
    	}
    	if(j == m)
    	{
    		return i - m  + 1;
    	}
    }
}
```

## 失效数组的建立

### 简单思路
* 1 假设`next[n] = k`, 即长度为`n`的前缀, 其前`k`个字符匹配后`k`个字符, 那么如果`next[n + 1] = next[k + 1]`, 可得`next[n + 1]=k+1`

* 2 如果上述不成立的话, 那么`next[n+1]`的最长可匹配前缀子串必然是`next[n]`的最长可匹配前缀子串的子串; 一种粗暴的方式就是在`next[n]`的最长可匹配前缀子串中找到`next[n+1]`字符在直接用`strncmp`比较

**代码**
```
bool KMP(const char *main, const char *pattern)
{
    int mlen = strlen(main);
    int plen = strlen(pattern);

    int *next = new int[plen];
    getNext(next, pattern);

    int j = 0;
    for(int i = 0; i < mlen; i++)
    {
        while(j > 0 && i > 0 && main[i] != pattern[j])
        {
            j = next[j - 1] + 1;
        }

        if(main[i] == pattern[j])
        {
            ++j;
        }
        if(j == plen)
        {
        	printf("KMP match succ: index = %d\n", i - plen  + 1);
            //return i - plen  + 1;
           	return true;
        }
    }
    return false;
}
```
很粗暴的写法, 但是实际是可以用的, 因为模式串通常不会太长

### 深一点的思路
&emsp;&emsp;1, 假设`next[n]=k`, 如果`next[n+1]=next[k+1]`, 那么`next[n+1]=k+1`; 如果不符合该规则, 那么`n`的最长可匹配后缀子串必然是`n-1`的次级后缀子串加上`next[n]`; 
&emsp;&emsp;假设该次级后缀子串`next[n]=y`, 那么存在关系
```
①pattern[0,k] = pattern[i-k,i]
②pattern[0,y] = pattern[i-y,i] 
```
对于①式子, 可以做变换
```
pattern[0+k-y,k] = pattern[i-k+k-y,i]
pattern[k-y,k] = pattern[i-y,i]
```
配合②式子可以得到
```
pattern[k-y,k] = pattern[0,y]
```
于是我们可以得出, `n`的次级后缀子串必然是其高一级的最长可匹配后缀子串

**代码:**第`n`次的最长可匹配后缀子串必然是和`n-1`的最长可匹配后缀子串相关联的
```
void getNext(int *next, const char *pattern)
{
	int plen = strlen(pattern);
	for(int i = 0; i < plen; i++)
	{
		next[i] = -1;
	}

	int j = -1;
	for(int i = 1; i < plen; i++)
	{

		while(j != -1 && pattern[j + 1] != pattern[i])
		{
			j = next[j];
		}
		if(pattern[j + 1] == pattern[i])
		{
			j++;
		}
		next[i] = j;
	}
}
```

### 一次简单的运行时间比对
```
BF time: 156
RK time: 125
BM time: 78
KMP time: 94
```

## 总结
&emsp;&emsp;**自此, 几种字符串匹配算法基本讲完了, 实际工作当中不必追求极限的效率, 绝大多数用普通的`BF`算法就够了; 对于KMP算法的失效数组其实以粗暴的解法问题也不是特别大. 重要的是算法思路学习!**













