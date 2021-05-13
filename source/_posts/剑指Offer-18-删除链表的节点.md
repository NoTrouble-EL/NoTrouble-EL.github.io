---
title: 剑指Offer 18. 删除链表的节点
date: 2020-10-25 09:09:16
tags:
- 剑指Offer
categories: DataStructure
mathjax: true
---

# 剑指Offer 18. 删除链表的节点

## 给定单链表的头指针和一个要删除的节点值，定义一个函数删除该节点。返回删除后的链表的头节点。

```java
输入: head = [4,5,1,9], val = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

```java
输入: head = [4,5,1,9], val = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```

```java
class Solution
{
    public ListNode deleteNode(ListNode head, int val)
    {
        if(head == null)
        {
            return head;
        }
        if(head.val = val)
        {
            return head.next;
        }
        ListNode pre = null;
        ListNode cur = head;
        
        while(cur.val != val)
        {
            pre = cru;
            cru = cru.next;
        }
        pre.next = cru.next;
        return head;
    }
}
```

