---
title: LeetCode_206
date: 2020-10-24 20:45:10
tags:
- LeetCode
categories: DataStructure
mathjax: true
---

# Leetcode 206

## 反转一个单链表。

```java
public class ListNode
{
    int val;
    ListNode next;
    public ListNode(int val)
    {
        this.val = val;
    }
}
```



```java
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

### 遍历法

```java
class Solution
{
    public ListNode reverseList(ListNode head)
    {
        if(head == null || head.next == null)
        {
            return head;
        }
        ListNode pre = null;
        ListNode cur = head;
        
        while(cur != null)
        {
            ListNode temp = cur.next;
            
            cur.next = pre;
            pre = cur;
            cur = temp;
        }
        return pre;
    }
}
```

### 递归法

```java
class Solutin
{
    public ListNode reverseList(ListNode head)
    {
        if(head == null || head.next == null)
        {
            return head;
        }
        
        ListNode newNode = reverseList(head.next);
        head.next.next = head;
        head.next = null;
        return newNode;
    }
}
```

