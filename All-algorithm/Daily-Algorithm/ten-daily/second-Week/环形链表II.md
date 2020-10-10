###  环形链表II

####  题目（142-中等）

给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意，pos 仅仅是用于标识环的情况，并不会作为参数传递到函数中。

####  思路

在环形链表I的基础上，进行发散思维

####  代码

```java
 public ListNode detectCycle(ListNode head) {
         ListNode slow = head;
        ListNode fast = head;
        while (fast!=null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
            if(slow == fast){
                fast = head;
                while(fast!=slow){
                    slow = slow.next;
                    fast = fast.next;
                }
                return  fast;
            }
        }
        return  null;
    }
```

