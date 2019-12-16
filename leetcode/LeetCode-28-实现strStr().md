# LeetCode-28-实现strStr()

## 题目
实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:

输入: haystack = "hello", needle = "ll"
输出: 2
示例 2:

输入: haystack = "aaaaa", needle = "bba"
输出: -1
说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/implement-strstr
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 最简单的逐个遍历比较
* 字符串匹配算法有: BF/RK/BM/KMP算法; 笔者这里实现要给KMP算法, 之前介绍过该算法, 但是并没有手写实现过.

### KMP算法介绍

* KMP算法利用的是好前缀规则, 对于如下匹配:

![图1](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191214101405.png)

前缀`aba`匹配成功, `e`和`f`匹配失败; KMP算法的意义就是尽可能的实现往后移动, 如上图可见我们实际上可以一次移动两位而不需要考虑错误情况:

![图片2](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191214101646.png)

* 分析原理: 对于成功匹配的前缀`aba`来说, 我们应当一次性移动到该前缀的前缀(前面一个`a`)和后缀(后面一个`a`)匹配的的位置; 即: 我们需要考虑好前缀的: 前缀和后缀匹配情况.

### 辅助数组实现

&emsp;&emsp;我们需要提前计算所有前缀字符串的前缀和后缀匹配情况, 提前保存到一个数组中. 该数组只和模式串相关与主串无关.

&emsp;&emsp;数组`D`其`index`为前缀结尾下标, 其`value`为前缀最长好前缀的下标. 比如对于`aba`这样的前缀来说: `D[2]=0`。 数组的求取可以使用递归的方式实现.

* 1 对于如下这个数组来说: 
![图3](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191214102544.png)

有`D[2]=0`, 那么如果`data[3]=data[1]`则有`D[3]=2`; 推导就是: 
```
if data[n] = data[D[n-1]+1]
	D[n] = D[n-1]+1
```

* 2 如果不等于的话呢?
![图4](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191214103345.png)

对于如上图: `data[a,b]=data[c,d]`但是`data[b+1]!=data[d+1]`。 

对于这种情况而言: 我们需要找出最小的一个`e/f`使得`data[a,f]=data[e,d]`。

![图5](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191214103956.png)

但是我们同时需要注意: `data[a,b]=data[c,d]`。 
如上图看到的, 我们可以得到`data[a,f]`不但是`[a,d]`的好前缀还是`[a,b]`的好前缀.

* 3 之前已经计算过KMP的[算法实现了](https://www.jianshu.com/p/b8fa5754d407), 下面直接上代码

ps: 该题的难度只是简单, 感觉如果要用到KMP算法就不应当是简单了.

## C++实现
```
class Solution {
	void createNext(int next[], const char *p, int len)
	{
		for(int i = 0; i < len; i++)
		{
			next[i] = -1;
		}

		int j = -1;
        cout<<len<<endl;
		for(int i = 1; i < len; i++)
		{
			while(j != -1 && p[j+1] != p[i])
			{
				j = next[j];
			}
			if(p[j+1] == p[i])
			{
				j++;
			}

			next[i] = j;
		}
	}

public:
    int strStr(string haystack, string needle) {
        int len1 = haystack.size();
        int len2 = needle.size();

        if(len2 == 0)	return 0;

        int *next = new int[len2];
        createNext(next, needle.c_str(), len2);

        int j = 0;
      	for(int i = 0; i < len1; i++)
        {
        	while(j > 0 && i > 0 && needle[j] != haystack[i])
        	{
        		j = next[j-1] + 1;
        	}

        	if(haystack[i] == needle[j])
        	{
        		j++;
        	}
        	if(j == len2)
        	{
                delete []next;
        		return i - len2 + 1;
        	}
        }

        delete []next;
        return -1;
    }
};
```

