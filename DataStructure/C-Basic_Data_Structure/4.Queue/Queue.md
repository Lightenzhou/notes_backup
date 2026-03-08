# Queue

---

## 结构类

队列的性质是 “先进先出” 或 “后进后出”。相比于栈的单向开口而言，队列的结构是双向开口的。具体逻辑结构如下：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/1.png)

将 A B C 三个元素依次放入后：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/2.png)

由于先进先出的原则，只能从对头出元素，若出一个元素为：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/3.png)

队列的实现方法固然很多，但注意到队列出元素相当于头删，若采用顺序表会导致时间复杂度过高。那如果采用单链表，尾插则需要找尾，故可以考虑双向循环链表。但再次考虑到双向循环链表的优势只能在此体验一部分（仅需要找尾），则考虑单独采用记录尾的方式进行尾插，依然采取单链表的方式减少空间使用，故为最优解：

```cpp
typedef int queue_datatype;

struct queue_node
{
    queue_datatype val;
    strcut queue_node* next;
};

typedef struct queue_node queue_node;

struct queue
{
    queue_node* head;
    queue_node* tail;
};

typedef struct queue queue;
```



---

## 成员函数

### queue_initialize

初始化仍然是必要的，结构体中有两个指针变量，刚开始时其应该为空。

```cpp
void queue_initialize(queue* qu)
{
	qu->head = NULL;
	qu->tail = NULL;
}
```



### queue_node_create

在实现插入操作之前，节点的创建是必须的。

```cpp
queue_node* queue_node_create(queue_datatype data)
{
	queue_node* newnode = (queue_node*)malloc(sizeof(queue_node));
	assert(newnode);

	newnode->val = data;
	newnode->next = NULL;

	return newnode;
}
```



### queue_push

这里需要考虑单链表的形式，此处采用单向不带头不循环链表。究其原因，队列只能尾插，没有中间插入和头插的需求，则不带头完全够用（如果带头对于你更友好请自行改为带头）。

```cpp
void queue_push(queue* qu, queue_datatype data)
{
	assert(qu);
	
	queue_node* newnode = queue_node_create(data);
	
	if (qu->head == NULL && qu->tail == NULL)
	{
		qu->head = newnode;
		qu->tail = newnode;
	}
	else
	{
		qu->tail->next = newnode;
		qu->tail = newnode;
	}
}
```



###  queue_empty

在执行删除操作前，判空操作是必要的，单链表的判空更是轻而易举。

```cpp
bool queue_empty(queue* qu)
{
    assert(qu);
    
	return qu->head == NULL;
}
```



### queue_pop

队列的删除相当于执行单链表的头删操作，需要注意仅有一个元素的情况。

```cpp
void queue_pop(queue* qu)
{
    assert(qu && !queue_empty(qu)); //空链表不可删除
    
    queue* del = head;
    
    if(head->next == NULL)
    {
        qu->head = NULL;
        qu->tail = NULL; 
    }
    else
    {
        qu->head = qu->head->next;
	}
    
    free(head);
	del = NULL;
}
```



 ### queue_front

取队头元素，即链表的首元素。

```cpp
queue_datatype queue_front(queue* qu)
{
    assert(qu && !queue_empty(qu));
    
    return qu->head->val;
}
```



### queue_back

取队尾元素，即链表的尾元素。

```cpp
queue_datatype queue_back(queue* qu)
{
	assert(qu && !queue_empty(qu));

	return qu->tail->val;
}
```



### queue_destroy

有创建就应该有销毁，必须记得一一对应，以防内存泄漏。

```cpp
void queue_destroy(queue* qu)
{
	assert(qu);

	queue_node* cur = qu->head;
	queue_node* cur_next = NULL;

	while (cur)
	{
		cur_next = cur->next;

		free(cur);
		cur = cur_next;
	}
}
```



---

## 补充说明

* 对于本文使用的数据结构 “单向不带头不循环链表” 只是一种选择，任何链式结构在这里都是通用的，都可以实现（顺序表也不例外，已分析劣势），请依据效率和个人代码风格来酌情更改。
* 至于队尾的出现是依据 STL 而来的，STL 的 queue 中提供了 back 这个成员函数，故此处也一并实现。
