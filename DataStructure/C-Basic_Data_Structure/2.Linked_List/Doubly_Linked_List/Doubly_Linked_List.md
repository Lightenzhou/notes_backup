# Doubly_Linked_List

---

## 结构类

此文章将讲解**双向带头循环链表**，或许此类结构有些骇人听闻，但其却比单向不带头不循环链表要简单得多。拆解如：既然是带头链表，则链表将具有一个哨兵位节点，其不存储有效数据，只负责 “站岗”； 既然是双向链表，则节点的结构需要具有 next 的同时还要具备 prev 能够找到前一个节点，这样就构成了双向的结构；循环则意味着头尾相连，头节点的前一个链接尾节点，尾节点的后一个链接头节点，这样就实现了循环。故双向带头循环链表节点如下：

```cpp
typedef int list_datatype; //定义链表数据类型

struct doubly_linked_list_node
{
    list_datatype val;
    struct doubly_linked_list_node* prev;
    struct doubly_linked_list_node* next;
};

typedef struct doubly_linked_list_node list_node;
```

其逻辑结构为：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-DoublyLinkedList-p1.png)

物理结构为：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-DoublyLinkedList-p2.png)

---

## 成员函数

### list_initialize

首先考虑一点，为何单向不带头不循环链表没有初始化函数而双向带头循环链表有初始化函数？**其主要原因是 “是否带头” 所导致的**。因为带头链表必须时刻保证哨兵位节点的存在，站岗的人必须一直在岗位上。

```cpp
list_node* list_initialize()
{
    list_node* stand_node = (list_node*)malloc(sizeof(list_node));
    assert(stand_node); //防止 malloc 失败
    
    stand_node->val = -1; //不存储有效值
    stand_node->prev = stand_node; //头节点的前一个节点链接尾节点，即其本身
    stand_node->next = stand_node; //尾节点的后一个节点链接头节点，即其本身
    
    return stand_node;
}
```

首先，本代码采用了返回值返回所开辟的哨兵位节点，其本质原因是因为后文的插入和删除都只需要传入一级指针，但此处如果要通过参数来改变该指针则必须采用二级指针，故为了代码格式统一，这里采用返回值返回的方式保证代码格式统一。



### list_node_create

节点的创建依然是必要的，为添加数据做复用准备。本代码仍不采用额外封装，仅通过操作节点的方式进行。

```cpp
list_node* list_node_create(list_datatype data)
{
    list_node* newnode = (list_node*)malloc(sizeof(list_node));
    assert(newnode);
    
    newnode->val = data;
    newnode->prev = NULL; //先置空，等接收节点处在链接
    newnode->next = NULL;
    
    return newnode;
}
```



### list_find

find 仍然需要为 insert 和 erase 做工作，配合他们使用。

```cpp
list_node* list_find(list_node* stand_node, list_datatype data)
{
    assert(stand_node); //哨兵位节点不能为空
    
    list_node* cur = stand_node->next; //链表的头是哨兵位的下一个节点
    while(cur != stand_node) //循环结束的标志是走到哨兵位节点
    {
        if(cur->val == data)
        {
            return cur;
		}
        
        cur = cur->next;
	}
    
    return NULL;
}
```

循环结束的条件不再是 cur 为空，而是 cur == stand_node。原本尾节点的下一个尾空，但在循环链表中，尾节点的下一个为哨兵位节点，则走到哨兵位则说明该链表已遍历完毕。



### list_push_back

由于带哨兵位节点的缘故，插入和删除操作将不会存在需要修改原指针的情况，所以一律采用一级指针即可完成该工作。且尾插找尾不再是 O(N) 的时间复杂度，而是 O(1)，因为哨兵位节点的前一个节点即是尾节点。

```cpp
void list_push_back(list_node* stand_node, list_datatype data)
{
    assert(stand_node);
    
    list_node* newnode = list_node_create(data);
    list_node* tail = stand_node->prev;
    
    //插入
    tail->next = newnode;
    newnode->prev = tail;
    newnode->next = stand_node;
    stand_node->prev = newnode;
}
```

对于链表为空仅有哨兵位节点的情况，tail 即 stand_node 本身，即使是这种情况依然可以完成正常链接，不需要特殊处理。

本人认为复述链表尾插是如何完成的没有太大意义，一切秘密尽在代码之中。



### list_push_front

头插同理，代码如下。

```cpp
void list_push_front(list_node* stand_node, list_datatype data)
{
    assert(stand_node);
    
    list_node* newnode = list_node_create(data);
    list_node* head = stand_node->next;
    
    stand_node->next = newnode;
    newnode->prev = stand_node;
    newnode->next = head;
    head->prev = newnode;
}
```



### list_insert

insert 实际上可以被 push_back 和 push_front 复用，当然，下面同时也会展示头插和尾插的复用版本。先阐述头插和尾插保证了可以梳理清楚链接关系，并发现一些特性：

* 循环链表找头找尾非常快，时间复杂度为 O(1)。

* 双向链表可以直接找到当前节点的 prev 和 next，这为任意位置的插入节省了大量的消耗，正是这一特性让双向链表的插入时间复杂度为 O(1)。

* 带头不需要考虑无节点时的特殊情况。

我想这正是详述头插尾插再复用的意义，在这里，并不会为了一味追求极简而省略他们且忽略理解的重要性。

```cpp
void list_insert(list_node* pos, list_datatype data) //依然是前插
{
    assert(pos);
    list_node* newnode = list_node_create(data);
    list_node* pos_prev = pos->prev; 
    
    pos_prev->next = newnode;
    newnode->prev = pos_prev;
    newnode->next = pos;
    pos->prev = newnode;
}
```

于此，复用的意义显现。请观察头插、尾插以及插入函数，其四步改变链接关系的步骤都相似度极高。这和顺序表有些类似，你会发现顺序表无论是头插、头删还是插入，都是通过 for 循环的统一方式实现，即此才为复用的意义。但例如单链表如果要复用，尾插需要找尾，则需要先判断该节点是否是尾节点，依然需要进行一遍遍历找尾的过程，这个过程通过复用并无法省略这些代码，故复用的价值实际不大。（*除非尾节点被单独记录*）所以在不带头单向不循环链表中，并没有复用的身影出现。

头插复用如下：

```cpp
void list_push_front(list_node* stand_node, list_datatype data)
{
    assert(stand_node);
   	list_insert(stand_node->next, data); // 由于 insert 是前插，则头插即在 stand_node 的下一个节点处插入
}
```

尾插复用如下：

```cpp
void list_push_back(list_node* stand_node, list_datatype data)
{
    assert(stand_node);
    list_insert(stand_node, data); //由于 insert 是前插，则尾插即在 stand_node 的前一个节点插入
}
```



### list_empty

在开始删除操作前，判空是必不可少的一步。空链表不允许删除，对于每个链表情况不一：对于本链表判空，是要判断哨兵位节点的下一个是不是它本身，如果是则为空链表；对于单向不带头不循环链表只需判断 *head 是否为空即可。而此函数出现的意义是理解至上，无疑，直接在函数体内写上 `assert(stand_node != stand_node->next)`同样是可行的，但这相比于 `assert(list_empty(stand_node))`而言，后者或许更容易理解。

```cpp
bool list_empty(list_node* node)
{
    return node == node->next;
}
```



### list_erase

同理，则删除不再具体叙述，erase 可被头删和尾删复用。

```cpp
void list_erase(list_node* pos)
{
    assert(pos);
    assert(!list_empty(pos)); //防止链表为空
    
    list_node* pos_prev = pos->prev;
    list_node* pos_next = pos->next;
    
    pos_prev->next = pos_next;
    pos_next->prev = pos_prev;
    
    free(pos);
}
```



### list_pop_front

```cpp
void list_pop_front(list_node* stand_node)
{
    assert(stand_node);
    assert(!list_empty(stand_node));
    
    list_erase(stand_node->next);
}
```



### list_pop_back

```cpp
void list_pop_back(list_node* stand_node)
{
    assert(stand_node);
    assert(!list_empty(stand_node));
    
    list_erase(stand_node->prev);
}
```



### list_destroy

对于动态开辟的空间依然遵循用完释放的原则。

```cpp
void list_destory(list_node* stand_node)
{
	list_node* cur = stand_node->next;
	list_node* next = NULL; //记录下一个否则释放当前节点无法找到下一个节点

	while(cur != stand_node)
	{
		next = cur->next;
		free(cur);

		cur = next;
	}

	free(stand_node); //stand_node 释放后需要在函数体外手动置空，否则需要传递二级指针
}
```



### 整体实现

```cpp
#include <stdio.h>
#include <stdbool.h>
#include <assert.h>

typedef int list_datatype; //定义链表数据类型

struct doubly_linked_list_node
{
    list_datatype val;
    struct doubly_linked_list_node* prev;
    struct doubly_linked_list_node* next;
};

typedef struct doubly_linked_list_node list_node;

list_node* list_initialize()
{
    list_node* stand_node = (list_node*)malloc(sizeof(list_node));
    assert(stand_node); //防止 malloc 失败

    stand_node->val = -1; //不存储有效值
    stand_node->prev = stand_node; //头节点的前一个节点链接尾节点，即其本身
    stand_node->next = stand_node; //尾节点的后一个节点链接头节点，即其本身

    return stand_node;
}

list_node* list_node_create(list_datatype data)
{
    list_node* newnode = (list_node*)malloc(sizeof(list_node));
    assert(newnode);

    newnode->val = data;
    newnode->prev = NULL; //先置空，等接收节点处在链接
    newnode->next = NULL;

    return newnode;
}

list_node* list_find(list_node* stand_node, list_datatype data)
{
    assert(stand_node); //哨兵位节点不能为空

    list_node* cur = stand_node->next; //链表的头是哨兵位的下一个节点
    while (cur != stand_node) //循环结束的标志是走到哨兵位节点
    {
        if (cur->val == data)
        {
            return cur;
        }

        cur = cur->next;
    }

    return NULL;
}

void list_insert(list_node* pos, list_datatype data) //依然是前插
{
    assert(pos);
    list_node* newnode = list_node_create(data);
    list_node* pos_prev = pos->prev;

    pos_prev->next = newnode;
    newnode->prev = pos_prev;
    newnode->next = pos;
    pos->prev = newnode;
}

void list_push_front(list_node* stand_node, list_datatype data)
{
    assert(stand_node);
    list_insert(stand_node->next, data); // 由于 insert 是前插，则头插即在 stand_node 的下一个节点处插入
}

void list_push_back(list_node* stand_node, list_datatype data)
{
    assert(stand_node);
    list_insert(stand_node, data); //由于 insert 是前插，则尾插即在 stand_node 的前一个节点插入
}

bool list_empty(list_node* node)
{
    return node == node->next;
}

void list_erase(list_node* pos)
{
    assert(pos);
    assert(!list_empty(pos)); //防止链表为空

    list_node* pos_prev = pos->prev;
    list_node* pos_next = pos->next;

    pos_prev->next = pos_next;
    pos_next->prev = pos_prev;

    free(pos);
}

void list_pop_front(list_node* stand_node)
{
    assert(stand_node);
    assert(!list_empty(stand_node));

    list_erase(stand_node->next);
}

void list_pop_back(list_node* stand_node)
{
    assert(stand_node);
    assert(!list_empty(stand_node));

    list_erase(stand_node->prev);
}

void list_destory(list_node* stand_node)
{
    list_node* cur = stand_node->next;
    list_node* next = NULL; //记录下一个否则释放当前节点无法找到下一个节点

    while (cur != stand_node)
    {
        next = cur->next;
        free(cur);

        cur = next;
    }

    free(stand_node); //stand_node 释放后需要在函数体外手动置空，否则需要传递二级指针
}   
```



---

## 效率分析

* 头插：O(1) - 无需查找，直接通过哨兵位节点找到头并插入
* 尾插：O(1) - 无需查找，直接通过哨兵位节点找到尾并插入
* 插入：O(1) - 无需查找，直接通过所给 pos 找到 pos_prev 并插入
* 头删：O(1) - 无需查找，直接通过哨兵位节点找到头并删除
* 尾删：O(1) - 无需查找，直接通过哨兵位节点找到尾并删除
* 删除：O(1) - 无需查找，直接通过所给 pos 找到 pos_prev 和 pos_next 并删除

---

## 补充说明

* 对于其它六种类型的链表，都可依据现所讲述的两种链表扩展而出，大同小异，故不再对其进行详细讲解。

