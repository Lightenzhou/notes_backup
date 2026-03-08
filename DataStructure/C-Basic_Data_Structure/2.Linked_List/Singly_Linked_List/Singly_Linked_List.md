# Singly_Linked_List

---

## 结构类

链表的形式很多，总共分为：是否有哨兵位节点、双向或单向、循环或不循环几种类型。组合一共有八种链表形式，但并无需一一阐述，一般而言，如果理解了**单向不带头不循环链表**和**双向带头循环链表**后，其他六种应是如鱼得水，信手拈来。**此文章讲述单向不带头不循环链表。**

于链表而言，其结构**并非如顺序表一样是地址连续的线性结构，但其通过链式结构相连接，依然为链式线性结构**，链表是通过一个个独立的空间（节点）通过此节点内的指针指向下一个节点的地址的数据结构。由此可见，节点在保存数据的同时还需要保存下一个节点的地址，这样就诞生了节点的数据结构。

```cpp
typedef int list_datatype; //定义链表数据类型

struct singly_linked_list_node
{
    list_datatype val;
    struct singly_linked_list_node* next;
};

typedef struct singly_linked_list_node list_node;
```

这里并非是结构体嵌套结构体，不会导致无限嵌套，因为指针可以为 NULL，相当于定义了结束条件和位置。

对于 typedef，这样定义也是被允许的：
```cpp
typedef int list_datatype;

typedef struct singly_linked_list_node
{
    list_datatype val;
    struct singly_linked_list_node* next;
}list_node;

//这是被绝对禁止的：
typedef struct singly_linked_list_node
{
    list_datatype val;
    list_node* next; //错误
}list_node;
```

**typedef 只有在结构体完整结束后才生效**，所以不允许在结构体内这样使用，故我更提倡前一种写法，这样更加清晰，也不易混淆。

链表的逻辑结构：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/SinglyLinkedList-p1.png)

但如果你想真正意义上的理解链表，你需要理解其物理结构：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/SinglyLinkedList-p2.png)

从物理上而言，箭头是不存在的。在遍历时，也不会有一个指针跟随移动，实时的指向遍历到的当前位置，实际上只是这个指针只是在重复被不断赋值和解引用的操作，指针变量本身也有地址，它从未改变。

---

## 成员函数

### list_node_create

根据链表的数据结构得知，其添加一个数据就需要创建一个节点，故所有的涉及到插入的操作都需要这一步，先实现链表节点的创建无疑可以被之后的插入函数复用。（*未实现初始化函数是因为我并未对头节点和其它例如 size 等成员再次封装形成链表，而是直接操作其节点并在使用处创建头节点对其进行操作，这对于链表来说是允许的，或许不太符合规范，但可以降低理解难度*）

```cpp
list_node* list_node_create(list_datatype data)
{
    list_node* ret = (list_node*)malloc(sizeof(list_node));
    if(ret == NULL) //不允许开辟失败
    {
        assert(0);
	}
    
    ret->val = data;
    ret->next = NULL;
        
    return ret;
}
```



### list_find

find 的工作是为了返回所找到对应值的节点地址，其主要是为了配合 insert 和 erase 使用，找不到会返回空指针。

```cpp
list_node* list_find(list_node* head, list_datatype data)
{
    while(head != NULL && head->val != data)
    {
		head = head->next;
	}
    
    return head;
}
```



### list_push_back

这将是决定链表有无哨兵位节点的关键函数：
```cpp
void list_push_back(list_node* head, list_datatype data); //1
void list_push_back(list_Node** head, list_datatype data); //2
```

对于第一种写法：头节点必须不为空，如果为空则插入操作会因为对空节点的访问而异常中止，即无论如何必须保证哨兵位节点的存在。

对于第二种写法：头节点如果为空，则会给头节点赋值且正常插入，新插入的节点变为头节点。我们遵循刚开始的约定，从而使用第二种写法。

```cpp
void list_push_back(list_node** head, list_datatype data)
{
    assert(head); //对于 *head 是允许为空的，如果是带哨兵位节点的插入不需要对 head 进行 assert，但 head 是不允许为空的，因为头节点可以存储空指针，但                   //头节点的地址不可能为空。
    
    list_node* newnode = list_node_create(data);
    
    if(*head == NULL) //处理头节点为空的情况
    {
        *head = newnode;
        return;
	}
    
    list_node* tail = *head; //头节点不为空则需要找到尾节点来进行链接
    while(tail->next != NULL) //这里是 tail->next == NULL 而非 tail == NULL，因为是需要走到 NULL 的前一个位置，而非走到                               //NULL
    {
        tail = tail->next;
	}
    
    tail->next = newnode;
}
```



### list_push_front

头插需要改变头节点，故也需要传递二级指针。

```cpp
void list_push_front(list_node** head, list_datatype data)
{
    assert(head);
    
    list_node* newnode = list_node_create(data);
    
    //记录新旧节点是为了降低理解难度
    list_node* old_head = *head;
    list_node* new_head = newnode;
    
    new_head->next = old_head;
    *head = new_head;
}
```

注意，头插不需要判断头节点是否为空，即使为空也会正常执行，会把 NULL 赋值给 new_head 的 next，一石二鸟。



### list_insert

拘于二级指针的约束，这里头插尾插被优先讲解，否则二级指针的理解将会晦涩难懂，且本身 insert 和头插尾插并无太大联系，因 insert 主要需要配合 find 使用，并无复用必要。后续的 erase 和头删尾删一致，头删尾删将会被优先讲解。

```cpp
void list_insert(list_node** head, list_node* pos, list_datatype data)
{
    assert(head && pos);
    
    list_node* prev = NULL; //找 pos 的 prev
    list_node* cur = *head; //找 pos
    while(cur != NULL && cur != pos)
    {
        prev = cur;
    	cur = cur->next;
	}
    
    if(cur == NULL) //如果为空则未找到 pos
    {
        assert(0);
    }
    
    //找到后前插（严格意义上来讲，你可以任意实现前插或后插，只是个人认为前插可以更充足的帮助理解二级指针故才采用<前插会需要面临更新头节点的情况>）
    list_node* newnode = list_node_create(data);
    if(prev == NULL) //prev 为空则表示只有一个数据的情况（因为未进入 while 循环）
    {
        *head = newnode;
        newnode->next = cur;
	}
    else
    {
        prev->next = newnode;
    	newnode->next = cur;
	}
}
```



### list_pop_back

尾删同样面对只有最后一个元素的删除情况，仍然需要更新头节点，故依然传入二级指针。

```cpp
void list_pop_back(list_node** head)
{
    assert(head && *head); //链表为空不允许执行删除
    
    list_node* tail_prev = NULL; //你需要让删除的前一个节点的 next 链接为 NULL，记录前一个节点是必要的
    list_node* tail = *head;
    while(tail->next != NULL)
    {
        tail_prev = tail;
        tail = tail->next;
	}
    
    //删除
    free(tail);
    tail = NULL;
    if(tail_prev == NULL) //tail_prev 为空表示链表中只有一个节点，则删除后需要更新头节点
    {
        *head = NULL;
    }
    else
    {
        tail_prev->next = NULL;
	}
}
```



### list_pop_front

头删每一次都会更新头节点，则必须传递二级指针。但相对头删比尾删要容易一些，毕竟它没有繁琐的 “找尾” 步骤。

```cpp
void list_pop_front(list_node** head)
{
    assert(head && *head);

    list_node* newhead = (*head)->next;

    free(*head);
    *head = NULL;

    *head = newhead;
}
```



### list_erase

erase 也是搭配 find 使用。

```cpp
void list_erase(list_node** head, list_node* pos)
{
    assert(head && *head && pos);

    list_node* prev = NULL; //找 pos 的 prev
    list_node* cur = *head; //找 pos

    while (cur != NULL && cur != pos)
    {
        prev = cur;
        cur = cur->next;
    }

    if (cur == NULL) //如果为空则未找到 pos
    {
        assert(0);
    }

    if (prev == NULL) //prev 为空则表示第一个节点就是 pos（未进入 while）
    {
        list_node* newhead = cur->next;
        free(*head);
        *head = newhead;
        return;
    }

    list_node* del = cur;
    cur = cur->next; //记录要删除的节点后 cur 走到后一个位置准备和 prev 链接
    prev->next = cur;

    free(del); //free 要在最后，因为前面还使用到了 cur 节点
    del = NULL;
}
```



### list_destroy

动态开辟的空间一律需要回收，这里需要注意的是销毁了当前节点无法找到下一节点，故要额外记录。

```cpp
void list_destroy(list_node** head)
{    
    assert(head);
    
    list_node* del = NULL;
    list_node* cur = *head;
    while(cur)
    {
        del = cur;
        cur = cur->next;
        free(del);
	}
    
    *head = NULL; //这个置空可在外完成也可在内完成
}
```



### 整体实现

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

typedef int list_datatype; //定义链表数据类型

struct singly_linked_list_node
{
    list_datatype val;
    struct singly_linked_list_node* next;
};

typedef struct singly_linked_list_node list_node;

list_node* list_node_create(list_datatype data)
{
    list_node* ret = (list_node*)malloc(sizeof(list_node));
    if (ret == NULL) //不允许开辟失败
    {
        assert(0);
    }

    ret->val = data;
    ret->next = NULL;

    return ret;
}

list_node* list_find(list_node* head, list_datatype data)
{
    while (head != NULL && head->val != data)
    {
        head = head->next;
    }

    return head;
}

void list_push_back(list_node** head, list_datatype data)
{
    assert(head);

    list_node* newnode = list_node_create(data);

    if (*head == NULL) //处理头节点为空的情况
    {
        *head = newnode;
        return;
    }

    list_node* tail = *head;
    while (tail->next != NULL)
    {
        tail = tail->next;
    }

    tail->next = newnode;
}

void list_push_front(list_node** head, list_datatype data)
{
    assert(head);

    list_node* newnode = list_node_create(data);

    //记录新旧节点是为了降低理解难度
    list_node* old_head = *head;
    list_node* new_head = newnode;

    new_head->next = old_head;
    *head = new_head;
}

void list_insert(list_node** head, list_node* pos, list_datatype data)
{
    assert(head && pos);

    list_node* prev = NULL; //找 pos 的 prev
    list_node* cur = *head; //找 pos
    while (cur != NULL && cur != pos)
    {
        prev = cur;
        cur = cur->next;
    }

    if (cur == NULL) //如果为空则未找到 pos
    {
        assert(0);
    }

    //找到后前插
    list_node* newnode = list_node_create(data);
    if (prev == NULL) //prev 为空则表示只有一个数据的情况（因为未进入 while 循环）
    {
        *head = newnode;
        newnode->next = cur;
    }
    else
    {
        prev->next = newnode;
        newnode->next = cur;
    }
}

void list_pop_back(list_node** head)
{
    assert(head && *head); //链表为空不允许执行删除

    list_node* tail_prev = NULL; //你需要让删除的前一个节点的 next 链接为 NULL，记录前一个节点是必要的
    list_node* tail = *head;
    while (tail->next != NULL)
    {
        tail_prev = tail;
        tail = tail->next;
    }

    //删除
    free(tail);
    tail = NULL;
    if (tail_prev == NULL) //tail_prev 为空表示链表中只有一个节点，则删除后需要更新头节点
    {
        *head = NULL;
    }
    else
    {
        tail_prev->next = NULL;
    }
}

void list_pop_front(list_node** head)
{
    assert(head && *head);

    list_node* newhead = (*head)->next;

    free(*head);
    *head = NULL;

    *head = newhead;
}

void list_erase(list_node** head, list_node* pos)
{
    assert(head && *head && pos);

    list_node* prev = NULL; //找 pos 的 prev
    list_node* cur = *head; //找 pos

    while (cur != NULL && cur != pos)
    {
        prev = cur;
        cur = cur->next;
    }

    if (cur == NULL) //如果为空则未找到 pos
    {
        assert(0);
    }

    if (prev == NULL) //prev 为空则表示第一个节点就是 pos（未进入 while）
    {
        list_node* newhead = cur->next;
        free(*head);
        *head = newhead;
        return;
    }

    list_node* del = cur;
    cur = cur->next; //记录要删除的节点后 cur 走到后一个位置准备和 prev 链接
    prev->next = cur;

    free(del); //free 要在最后，因为前面还使用到了 cur 节点
    del = NULL;
}

void list_destroy(list_node** head)
{
    assert(head);

    list_node* del = NULL;
    list_node* cur = *head;
    while (cur)
    {
        del = cur;
        cur = cur->next;
        free(del);
    }

    *head = NULL; //这个置空可在外完成也可在内完成
}
```



---

## 效率分析

- 头插：O(1) - 无需查找，直接执行插入
- 尾插：O(N) - 需要找尾再插入
- 插入：O(N) - 需要找到 pos 后插入
- 头删：O(1) - 直接删除并更新头节点
- 尾删：O(N) - 需要找尾再删除
- 删除：O(N) - 需要找到 pos 后删除

---

## 补充说明

- 本篇中大量函数并未复用实现，第一考虑到单链表复用效率并不高，第二更希望初学者可以更细致的理解细节。在后续的数据结构中，复用将显得极为常见。
- 成员函数中 data 和 val 都是表示链表数据类型，二者都使用是为了防止传入数据和结构体内数据名称一致而混淆。
- assert 很有趣，它只能在 debug 版本下使用，且本文所有成员函数一律采用 assert 的暴力检查，因为例如无数据时删除的情况是本不应该出现的，应该绝对禁止。
- 本文很多处循环所表述的都是 while(cur->next != NULL)，随着时间的推移，我会逐渐将其替换为 while(cur)。
- find 查找返回指针而不是 int，不使用 int 来表示 “第几个”，因为 C++ STL 中 find 返回迭代器，也就和指针类似，本文遵循 C++ STL 原则。
- 顺序表的线性体现在物理线性和逻辑线性，链表的线性体现在逻辑线性，故都是线性结构。