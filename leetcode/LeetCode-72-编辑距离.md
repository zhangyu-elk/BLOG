# LeetCode-72-编辑距离

## 题目
给定两个单词 word1 和 word2，计算出将 word1 转换成 word2 所使用的最少操作数 。

你可以对一个单词进行如下三种操作：

插入一个字符
删除一个字符
替换一个字符
示例 1:

输入: word1 = "horse", word2 = "ros"
输出: 3
解释: 
horse -> rorse (将 'h' 替换为 'r')
rorse -> rose (删除 'r')
rose -> ros (删除 'e')
示例 2:

输入: word1 = "intention", word2 = "execution"
输出: 5
解释: 
intention -> inention (删除 't')
inention -> enention (将 'i' 替换为 'e')
enention -> exention (将 'n' 替换为 'x')
exention -> exection (将 'n' 替换为 'c')
exection -> execution (插入 'u')

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/edit-distance
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路
[编辑距离算法]()

## C++实现
```
class Solution {
	//返回剩余操作次数
	int core(const char *word1, const char *word2, int i, int j, int **f)
	{
        if(word1[i] && word2[j] && f[i][j] >= 0)
        	return f[i][j];
        if(!word1[i] && !word2[j])
        	return 0;
		if(word1[i] == word2[j])
			return (f[i][j] = core(word1, word2, i+1, j+1, f));
		else
		{
			int t = INT_MAX;
			if(word1[i])
				t = min(t, core(word1, word2, i+1, j, f));
			if(word2[j])
				t = min(t, core(word1, word2, i, j+1, f));
			if(word1[i] && word2[j])
				t = min(t, core(word1, word2, i+1, j+1, f));
			return (f[i][j] = t + 1);
		}
	}

public:
    int minDistance(string word1, string word2) {
        int len1 = word1.size();
        int len2 = word2.size();
        int **f = new int*[len1+1];
        int *buffer = new int[(len1+1)*(len2+1)];
        for(int i = 0; i <= len1; i++)
        {
            f[i] = buffer + (len2+1)*i;
            for(int j = 0; j <= len2; j++)
                f[i][j] = -1;
        }
        int res = core(word1.c_str(), word2.c_str(), 0, 0, f);
        delete []f;
        delete []buffer;
        return res;
    }
};
```