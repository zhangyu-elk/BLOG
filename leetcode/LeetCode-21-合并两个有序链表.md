# LeetCode-21-合并两个有序链表

## 题目

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

示例：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/merge-two-sorted-lists
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



## C++实现
> 链表的基础题目, 类似与归并排序中的归并步骤
```
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode res(0);
        ListNode *l = &res;

        while(l1 && l2)
        {
        	if(l1->val < l2->val)
        	{
        		l->next = l1;
        		l = l1;
        		l1 = l1->next;
        	}
        	else
        	{
        		l->next = l2;
        		l = l2;
        		l2 = l2->next;
        	}
        }
        if(l1)
            l->next = l1;
        if(l2)
            l->next = l2;
        return res.next;
    }
};
```

## 递归解法
```
class Solution {
public:
	ListNode* merge(ListNode *l1, ListNode *l2)
	{
		if(!l1)
			return l2;
		if(!l2)
			return l1;

		if(l1->val < l2->val)
		{
			l1->next = merge(l1->next, l2);
			return l1;
		}
		else
		{
			l2->next = merge(l1, l2->next);
			return l2;
		}
	}
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {

        return merge(l1, l2);
    }
};
```

ps: LeetCode上测试的时候递归的效率更好