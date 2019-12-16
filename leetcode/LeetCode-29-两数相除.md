# LeetCode-29-两数相除

## 题目
给定两个整数，被除数 dividend 和除数 divisor。将两数相除，要求不使用乘法、除法和 mod 运算符。

返回被除数 dividend 除以除数 divisor 得到的商。

示例 1:

输入: dividend = 10, divisor = 3
输出: 3
示例 2:

输入: dividend = 7, divisor = -3
输出: -2
说明:

被除数和除数均为 32 位有符号整数。
除数不为 0。
假设我们的环境只能存储 32 位有符号整数，其数值范围是 [−2^31,  2^31 − 1]。本题中，如果除法结果溢出，则返回 231 − 1。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/divide-two-integers
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 暴力减法, 每次减去被除数
* 同样采取减法, 但是尽可能快速的逼近结果, 每次翻倍去减. 比如对于: `10/3`, 第一次减去3, 结果为7(>0); 那么第二次减去6, 结果为1(>0); 第三次减去12, 结果小于0, 再次从3开始减; 减去3小于0, 结束.

## C++实现
```
class Solution {
public:
    int divide(int dividend, int divisor) {
        if(divisor == 1) return dividend;
        if(divisor == -1){
            if(dividend>INT_MIN) return -dividend;
            return INT_MAX;
        }
        if(divisor == INT_MIN)
        {
            if(dividend == INT_MIN) return 1;
            return 0;
        }
        //排除特殊情况

        int minus = 1;
        if((dividend < 0 && divisor > 0) ||
        	(dividend > 0 && divisor < 0))
        {
        	minus = -1;
        }
        int res = 0;
        uint32_t a = dividend;
        uint32_t b = divisor;
        if(dividend < 0 && dividend > INT_MIN)
        {
            a = dividend * -1;
        }
        if(divisor < 0 && divisor > INT_MIN)
        {
            b = -1 * divisor;
        }
        //转为正数

        while(a >= b)
        {
            uint32_t t = b;
            uint32_t bit = 1;
            while(a >= t && t != 0)
            {
                a -= t;
                res += bit;
                bit = bit << 1;
                t = t << 1;
            }
        }

        if(res > INT_MAX)
            return INT_MAX;
		return res * minus;
    }
};
```

## 注意点
* `int`类型大小是`[−2^31,  2^31 − 1]`, 如果我们想要将负数转为正数; 对于`−2^31`直接进行强转就可以. 因为`−2^31`二进制为`1000...`, 强转为`uint32_r`该二进制就代表`2^32`。 如果直接和`-1`相乘会报错.

* 这里使用了倍数减法, 每次对被除数`*2`(移位也是同样的). 需要注意如果超出了最大值, `uint32_t`表示的值会从0开始, 所以需要排除`t=0`的情况.

