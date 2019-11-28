# LeetCode-923-三数之和的多种可能性

## 题目
给定一个整数数组 A，以及一个整数 target 作为目标值，返回满足 i < j < k 且 A[i] + A[j] + A[k] == target 的元组 i, j, k 的数量。

由于结果会非常大，请返回 结果除以 10^9 + 7 的余数。

 

示例 1：

输入：A = [1,1,2,2,3,3,4,4,5,5], target = 8
输出：20
解释：
按值枚举（A[i]，A[j]，A[k]）：
(1, 2, 5) 出现 8 次；
(1, 3, 4) 出现 8 次；
(2, 2, 4) 出现 2 次；
(2, 3, 3) 出现 2 次。
示例 2：

输入：A = [1,1,2,2,2,2], target = 5
输出：12
解释：
A[i] = 1，A[j] = A[k] = 2 出现 12 次：
我们从 [1,1] 中选择一个 1，有 2 种情况，
从 [2,2,2,2] 中选出两个 2，有 6 种情况。
 

提示：

3 <= A.length <= 3000
0 <= A[i] <= 100
0 <= target <= 300

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/3sum-with-multiplicity
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
(这个我也不删, 可以跳过去自己看)

## 解题思路

&emsp;&emsp;就像我们之前解决三数之和`[LeetCode-15]()`的解法是类似的, 可以采用三索引的方式进行, 这样事件复杂度能够降低为`O(N2)`。 

&emsp;&emsp;数学知识: 再N个人中选取`n`个人, 其种类共计`C(N,n) = N!/(2*(N-n)!)`, 对于`C(N,2)=(N*(N-1))/2`

## C++实现 
```
class Solution {
public:
    int threeSumMulti(vector<int>& A, int target) {
        sort(A.begin(), A.end());
        long res = 0;
        long M = 1e9 + 7;

        int len = A.size();
        for(int i = 0; i < len - 2; i++)
        {
        	int l = i + 1;
        	int r = len - 1;
            while(l < r)
            {
                int total = A[i]  + A[l] + A[r];
                if(total == target)
                {
                    //cout<<A[i]<<","<<A[l]<<","<<A[r]<<","<<res<<endl;
                    if(A[l] == A[r])
                    {
                        res += (r - l + 1) * (r - l) / 2;
                        break;
                    }
                    else
                    {
                        int tmp = 1;
                        while(A[l] == A[++l]) tmp++;
                        res += tmp;
                        while(A[r] == A[--r]) res+=tmp;
                    }
                }
                else if(total > target)
                {
                    r--;
                }
                else
                {
                    l++;
                }
            }

        }
        return res % M;

    }
};
``` 

## 解题思路2

&emsp;&emsp;上面我们运用到一个数学知识, 就是从`N`个中选取`n`个的计算公式; 假设建立一个`count`数组包含各种数出现的数量, 选取的三个值为`X/Y/Z` 那么就有:

* 1 如果三个值`X != Y != Z`互不相等. 可能性为: `cout[X]*count[Y]*count[Z]`
* 2 如果三个值`X == Y != Z`, 那么可能性为: `C(count[X],2)*count[Z]`
* 3 如果三个值`X==Y==X`, 那么可能性为: `C(count[X],3)`

## C++代码实现
```
class Solution {
public:
    int threeSumMulti(vector<int>& A, int target) {
    	map<int ,int> map;
        sort(A.begin(), A.end());
        long res = 0;
        long M = 1e9 + 7;

        int len = A.size();
        for(int i = 0; i < len; i++)
        {
        	map[A[i]]++;
        }

        for(int i = 0; i < len - 2; i++)
        {
        	int l = i + 1;
        	int r = len - 1;
            while(l < r)
            {
                int total = A[i]  + A[l] + A[r];
                if(total == target)
                {
                    int same = 0;
                    int diff = 0;
                    int same_num = 0;
                    if(A[i] == A[l])
                    {
                    	same_num++;
                    	same = A[i];
                    	diff = A[r];
                    }
                    if(A[i] == A[r])
                    {
                    	same_num++;
                    	same = A[i];
                    	diff = A[l];
                    }
                    if(A[r] == A[l])
                    {
                    	same_num++;
                    	same = A[r];
                    	diff = A[i];
                    }

                    if(same_num == 0)
                    {
                    	res += map[A[i]] * map[A[l]] * map[A[r]];
                    }
                    else if(same_num == 1)
                    {
                    	long tmp = map[same];
                    	res += (tmp * (tmp - 1)) / 2 * map[diff];
                    }
                    else if(same_num == 3)
                    {
                        long tmp = map[same];
                    	res += ((tmp * (tmp - 1)) / 2 * (tmp - 2) / 3) % M; 
                    }
                    while(A[l] == A[++l] && l < r);
                    while(A[r] == A[--r] && l < r);
                }
                else if(total > target)
                {
                    r--;
                }
                else
                {
                    l++;
                }
            }
            while(i < len - 2 && A[i] == A[i + 1]) i++;

        }
        return res % M;

    }
};
```

ps: 再leetcode上测试的时候, 后者的执行用时较短.