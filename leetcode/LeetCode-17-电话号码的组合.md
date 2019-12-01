# LeetCode-17-电话号码的组合

## 题目

给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

![图片](https://assets.leetcode-cn.com/aliyun-lc-upload/original_images/17_telephone_keypad.png)

示例:

输入："23"
输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
说明:
尽管上面的答案是按字典序排列的，但是你可以任意选择答案输出的顺序。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 回溯算法

## C++实现 
```
class Solution {
private:
	const char *mapping[10] = {NULL, NULL, "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};
	void core(const char *digits, string str, vector<string> &res)
	{
		if(!*digits)
		{
            if(str.size())
			    res.push_back(str);
			return;
		}

		int n = *digits - '0';
		for(int i = 0; i < strlen(mapping[n]); i++)
		{
			core(digits + 1, str + mapping[n][i], res);
		}
	}
public:
    vector<string> letterCombinations(string digits) {
        vector<string> res;
        string str;

        core(digits.c_str(), str, res);
        return res;
    }
};
```