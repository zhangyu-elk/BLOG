# LeetCode-53-最大子序和

## 题目
给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:

输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
进阶:

如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/maximum-subarray
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 在LeetCode上看到一中非常简明的解决方法, 一并说明一下
* 2 动态规划, 用数组记录以指定索引结尾的最大和: `dp[n+1] = max(dp[n]+data[n], dp[n])`

### JS实现方案1
```
class Solution {
    public int maxSubArray(int[] nums) {
        int ans = nums[0];
        int sum = 0;
        for(int num: nums) {
            if(sum > 0) {
                sum += num;
            } else {
                sum = num;
            }
            ans = Math.max(ans, sum);
        }
        return ans;
    }
}
```

#### 说明
![图片](https://raw.githubusercontent.com/zhangyu012/picture_picgo/master/blogImg/20191208192442.png)
* 1 `sum`记录的是前缀子串之和, 如果小于0那么说明这一段子串没有任何意义(和后面相加只会减少)
* 2 这其实也是和动态规划类似的, 假设`N[n]=sum`, 那么对于`N[n+1] = max(N[n]+data[n+1], data[n+1])`; 如果`sum`小于0则必然是直接等于`data[n+1]`

### C++实现标准动态规划
```
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
    	int len = nums.size();
    	if(len == 0)	return 0;

        int *dp = new int[len];
        int res = nums[0];

        dp[0] = nums[0];
        for(int i = 1; i < len; i++)
        {
        	dp[i] = (dp[i-1] > 0) ? (dp[i-1]+nums[i]) : nums[i];
        	res = res < dp[i] ? dp[i] : res;
        }

        delete []dp;
        return res;
    }
};
```

#### 说明
&emsp;&emsp;这里的实现其实很没营养, `dp[i-1]`仅被`dp[i]`需要, `res`也可当场比较; 所以实际不需要`n`的空间.
