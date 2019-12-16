# LeetCode-19-删除链表的倒数第N个节点

## 题目
给定一个链表，删除链表的倒数第 n 个节点，并且返回链表的头结点。

示例：

给定一个链表: 1->2->3->4->5, 和 n = 2.

当删除了倒数第二个节点后，链表变为 1->2->3->5.
说明：

给定的 n 保证是有效的。

进阶：

你能尝试使用一趟扫描实现吗？

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 解题思路

&emsp;&emsp;这其实是一个比较常见的链表问题了, 如果接触过链表的一些算法题的话; 大概都知道解法: 使用双指针`a,b`并且`b`是`a`之后的第`n-1`个节点, 那么当`a/b`同时往后移动并且`b`移动至末尾的时候, `a`指向的就是倒数第`n`个节点; 当然还需要一个指针指向`a`前驱节点.

## C++实现
```
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if(!head)	return head;

        ListNode *prev = NULL;
        ListNode *a = head;
        ListNode *b = a;
        for(int i = 0; i < n - 1; i++)
        	b = b->next;

        while(b->next)
        {
        	prev = a;
        	a = a->next;
        	b = b->next;
        }

        if(!prev)
        	return a->next;

        prev->next = a->next;

        return head;
    }
};
```