# LeetCode-23-合并K个有序链表

## 题目
合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

示例:

输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/merge-k-sorted-lists
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

* 1 采用暴力破解的方式, 类似于合并两个有序链表的方式, 每次从`n`个头节点中选取最大的一个; 那么时间复杂度是`O(k*N)`(k是链表个数,N是总节点数量)

* 2 遍历链表数组每次合并两个, 时间复杂度为`O(k*N)`, 和上面一样每个节点约需要比较`k`次.

* 3 分治算法: 采用类似于排序分治类似的思想, 两两分为一组, 每次所有的组都进行一次合并; 时间复杂度为`O(N*log(k))`, 每一次合并时所有链表的所有节点都会进行一次比较, 而我们只需要合并`log(k)`次;

$\color{red}{这一题的难度是困难, 但是其实没有什么特别值得深入研究的算法; 不过也很难想到使用分治算法来解决(不过可以看标签).}$

## C++事项
```
class Solution {
	ListNode* mergeT(ListNode *l1, ListNode *l2)
	{
		if(!l1)
			return l2;
		if(!l2)
			return l1;

		if(l1->val < l2->val)
		{
			l1->next = mergeT(l1->next, l2);
			return l1;
		}
		else
		{
			l2->next = mergeT(l1, l2->next);
			return l2;
		}
	}
public:
	ListNode* mergeK(vector<ListNode*>& lists, int l, int r)
	{
		if(l > r)	return NULL;
		if(l == r - 1)	return mergeT(lists[l], lists[r]);
		if(l == r)	return lists[l];

		int m = (l+r)/2;
		return mergeT(mergeK(lists, l, m), mergeK(lists, m+1, r));
	}
    ListNode* mergeKLists(vector<ListNode*>& lists) {
    	int l = 0;
    	int r = lists.size();
    	int m = (l+r)/2;
        return mergeK(lists, 0, lists.size()-1);
    }
};

```