# LeetCode-739-每日温度

## 题目
根据每日 气温 列表，请重新生成一个列表，对应位置的输入是你需要再等待多久温度才会升高超过该日的天数。如果之后都不会升高，请在该位置用 0 来代替。

例如，给定一个列表 temperatures = [73, 74, 75, 71, 69, 72, 76, 73]，你的输出应该是 [1, 1, 4, 2, 1, 1, 0, 0]。

提示：气温 列表长度的范围是 [1, 30000]。每个气温的值的均为华氏度，都是在 [30, 100] 范围内的整数。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/daily-temperatures
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

假设输出数组`R`

* 1 从后往前逐个遍历, 可以值得优化的一点是, 求取`R[i]`时可以通过`R[i+1]`的值快速求取: 如果`data[i+1]>data[i]`那么`R[i]=1`, 但是如果`data[i+1]<=data[i]`, 下一个可以直接从`i+1+R[i+1]`的位置开始
```
k = i + 1
while data[i] >= data[k]
	if R[k] > 0
		k = R[k] + k
	else
		break
if data[k] > data[i]
	R[i] = k - i
else
	R[i] = 0
```
此种问题的时间复杂度达到了`O(N2)`.

* 2 逐个遍历不可避免, 但是查找后续更大值的方式可以进行一些优化: 对于`data[index]=t`, 可以从后遍历使用`HASH`的方式记录每个温度`30-70`出现的最小下标, 遍历到`index`时仅需对`HASH`数组中的`t,100`进行遍历比较求取最小值. 
	* 时间复杂度降低到了`O(N*70)`, 实际如果`N`较小的情况下, 这种算法可能实际未必效率更高.

## C++算法实现
```
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& T) {
        int hash[100] = { 0 };			//可以使用map但是效率两说
    	int len = T.size();
    	vector<int> res(len, 0);
    	
        for(int i = len - 1; i >= 0; i--)
        {
        	hash[T[i]] = i;
        	int min = INT_MAX;
        	for(int j = T[i] + 1; j <= 100; j++)
        	{
        		min = (min > hash[j]) ? hash[j] : min;
        	}
        	if(min != INT_MAX)
        		res[i] = min - i;
        }
        return res;
    }
};
```