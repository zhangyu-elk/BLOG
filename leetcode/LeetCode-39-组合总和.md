# LeetCode-39-组合总和

## 题目
给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

说明：

所有数字（包括 target）都是正整数。
解集不能包含重复的组合。 
示例 1:

输入: candidates = [2,3,6,7], target = 7,
所求解集为:
[
  [7],
  [2,2,3]
]
示例 2:

输入: candidates = [2,3,5], target = 8,
所求解集为:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/combination-sum
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解决思路

* 1 本题是基于回溯算法
* 2 添加限制条件剪枝, 那么条件是什么呢?

### 回溯算法与深度遍历
![树](https://i.loli.net/2019/12/03/ij4TfClwLSy5RAm.png)

回溯算法其实类似与深度遍历, 沿着一条边一直走下去, 如果失败就回退一级

### 剪枝
```
7-2-...
7-3-2...
```
以上两条路径必然会产生重合, 因为`7-2`之后是可以再`-3`的, 所以我们应当限制后`-`的长度小于前的长度; 可以提前进行排序

## C++算法实现
```
class Solution {
	void core(vector<int>& candidates, int target, int index, vector<int> t, vector<vector<int>> &res)
	{
		if(target < 0)	return;
		if(target == 0)	
        {
            res.push_back(t);
            return;
        }

		for(int i = index; i < candidates.size(); i++)
		{
            int tmp = candidates[i];
            if(target >= tmp)
            {
                t.push_back(tmp);
                core(candidates, target - tmp, i, t, res);
            }
            else
            {
                break;
            }
            t.pop_back();
		}
		
	}
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> res;
        sort(candidates.begin(), candidates.end());
        core(candidates, target, 0, vector<int>(), res);
        return res;
    }
};
```
