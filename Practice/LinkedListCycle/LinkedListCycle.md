# LinkedListCycle

---

## 模型分析

首先，链表的带环问题转换为逻辑图应为：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Practice-LinkedListCycle-p1.png)

若细化表示，则应该是这样：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Practice-LinkedListCycle-p2.png)

---

## 问题引出

* 判断链表中是否有环。
* 确定入环点的位置。 

---

## 问题剖析

### 判断链表中是否有环

可知若链表带环则链表顺序遍历是无法走到尽头的，而无环链表必然会走到 NULL，但这并不能作为判断依据，因为无法得知链表究竟能不能走完。故采用快慢指针法，其原理即为追及问题：若快指针一次走两步，慢指针一次走一步，则两者每次只相对走了一步，若走到空则说明链表无环，若两指针相遇则必然有环。

如图：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Practice-LinkedListCycle-p3.png)

（L：入环点前的链表长度；C：环长；X：slow 与 fast 的右距离；C - X：slow 与 fast 的左距离）

前面的 L 并无意义，因为必然存在一个时刻使得 slow 到入环点，此时 fast >= slow 恒成立，因为 fast 的速度 比 slow 快，所以 slow 刚到入环点，fast 只有两种情况：1.在 slow 的前面；2.刚好走完此环（不一定是第一圈）与 slow 直接相遇。

故只对环进行分析即可：

1.在 slow 前面：此时 fast 的速度是 2，slow 的速度是 1，则两者每次相对移动 1，环长为 C，设 slow 与 fast 相距 X（右距离），可知 C - X <= C 恒成立，C 为常数，则必然在 C - X 秒后相遇，X 为常数，则必然相遇。

2.直接和 slow 相遇，则直接判断有环。

故有环情况已判断完毕，无环情况只需判断空指针即可，需要注意的是，因为 fast 一次走两步，空指针的判断应同时判断 fast 和 fast->next。

```cpp
bool HasCycle(list* head)
{
    assert(head);
    
    list* slow = head;
    list* fast = head;
    
    while(fast && fast->next)
    {
        if(slow == fast)
        {
            return true;
		}
        
        slow = slow->next;
        fast = fast->next->next;
	}
    
    return false;
}
```



### 确定入环点的位置

同样以上图为例：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Practice-LinkedListCycle-p3.png)

在 slow 走到入环点时，两者路程如下：

slow：`L`

fast：`L + NC + X`

（由于 slow 到达入环点时 fast 可能走了多圈，而这个圈数是未知的，故设为 N ）

可知此时两者需要再走 (C - X)s 后相遇，此时两者路程为：

slow：`L + C - X`

fast：`L + NC + X + 2C - 2X`

又因为 fast 的速度始终是 slow 的两倍，则存在关系：`2L + 2C - 2X = L + NC + X + 2C - 2X`，化简得：`L = NC + X`（第一步 2L = L + NC + X 也可得出）

又易得知，只要路径长为：`L + NC`这这个点必然在入环点处。

相遇后如图：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Practice-LinkedListCycle-p4.png)

相遇点要走到入环点，需要行进 `X + NC` 的路程，而这一段路程恰好就是 L。

故得出结论，只需从相遇点和起点同时出发，则两者再次相遇则是入环点。

```cpp
list* FindInCycle(list* head)
{
    assert(head);
    
    list* slow = head;
    list* fast = head;
    
    //找第一个相遇点（slow 和 fast 相遇）
    list* meet = NULL;
    while(fast && fast->next)
    {
        if(slow == fast)
        {
            meet = slow;
            break;
		}
        
        slow = slow->next;
        fast = fast->next->next;
	}
    
    //找第二个相遇点（meet 和 head 相遇）
    while(meet) //循环条件为 meet 可以一并处理 meet 为 NULL 的情况
    {
        if(head == meet)
        {
			break;
        }
        
        head = head->next;
        meet = meet->next;
	}
    
    return meet;
}
```

此时通过已知的入环点，可以求出任意带环链表的：L、C、X等所有未知量，若有类似问题也可一并解决。

---

## 补充说明

* 本内容所提供的仅为某类型问题的分析思路，不代表具体题目，应掌握后具体问题具体分析。
