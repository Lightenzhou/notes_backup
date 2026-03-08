# Binary_Heap

---

## 结构类

首先需要提及的是，堆是一个被排序的树结构（逻辑上），通常按照特定的顺序（通常为升序或降序）排列。在最小堆中，父节点的值小于或等于其子节点的值；而在最大堆中，父节点的值大于或等于其子节点的值。而平时所提及的堆基本上所指的都是二叉堆，其为一种满足完全二叉树性质的堆。但存在斐波那契堆、二项堆等不满足二叉树的结构和性质，所以首先需要区分堆和二叉堆。

其次，优先队列和二叉堆并非毫无二致，优先队列是一种队列，但其具有堆的特殊性质，也就是元素被赋予了优先级，能够保证最高优先级的元素在队头。优先队列可以采用不同的方式实现，二叉堆是最常用的方式，故二叉堆是实现优先队列的一种方式，且是最优方式。

考虑行文和理解方便，本文全部采用大堆举例和实现，并采用数组储存堆。

其数组和其所对应的逻辑树结构如下图所示：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-BinaryHeap-p1.png)

其总是堆的性质且符合完全二叉树的结构。

```cpp
typedef int heap_datatype;

struct heap
{
	heap_datatype* arr;
	int size;
	int capacity;
};

typedef struct heap heap;
```

可发现其结构和顺序表的结构一致，但随后涉及到的诸多操作却不尽相同。

---

## 成员函数

### heap_initialize

对于结构的初始化，是基本的固定操作。

```cpp
void heap_initialize(heap* hp)
{
	assert(hp);

	hp->arr = NULL;
	hp->size = 0;
	hp->capacity = 0;
}
```



### heap_swap

由于堆的调整是涉及到元素交换的，向上调整和向下调整都会进行频繁的交换，故单独独立交换函数。

```cpp
void heap_swap(heap_datatype* e1, heap_datatype* e2)
{
	assert(e1 && e2);

	heap_datatype tmp = *e1;
	*e1 = *e2;
	*e2 = tmp;
}
```



### heap_reverse

在进行 push 操作时，元素仍像顺序表般插入在数组末尾，则首先涉及到的问题是数组的扩容，故独立 reverse 函数完成这一工作。

```cpp
void heap_reverse(heap* hp, int new_capacity)
{
	assert(hp && new_capacity > hp->capacity);

	heap_datatype* tmp = (heap_datatype*)realloc(hp->arr, sizeof(heap_datatype) * new_capacity);
	assert(tmp);

	hp->arr = tmp;
	hp->capacity = new_capacity;
}
```



### adjust_up

进行顺序表一致的插入操作后，其重点在于如何保证堆的性质不发生变化，经过怎样的变换可以维持大堆的性质和状态是重点。其通过向上调整，即不断比较孩子与双亲节点的大小关系，若优先级顺序不对则进行交换，保证双亲节点的元素必须比孩子的优先级高，一直调整到满足优先级的状态或根节点为止。

```cpp
void adjust_up(heap* hp, int pos) //pos并不是必须的，绝大多数情况下，向上调整都是负责调整最后一个元素
{
	assert(hp && pos >= 0);

	int parents = 0;
	int child = pos;

	while (child) //调整到根节点结束
	{
		parents = (child - 1) / 2; //通过孩子找双亲节点的方式：减 1 除 2 做的是整数除法，故必然可以得到双亲节点下标

		if (hp->arr[parents] < hp->arr[child])
		{
			heap_swap(&hp->arr[parents], &hp->arr[child]);

			child = parents;
		}
		else
		{
			break; //满足优先级条件后结束
		}
	}
}
```



### heap_push

如此便可构建一个堆，其包括：顺序表的基本插入和扩容 + 向上调整 构成。

```cpp
void heap_push(heap* hp, heap_datatype val)
{
	assert(hp);

	if (hp->size == hp->capacity)
	{
		heap_reverse(hp, hp->capacity == 0 ? 8 : hp->capacity * 2);
	}

	hp->arr[hp->size] = val;

	adjust_up(hp, hp->size++); //简化代码处理
}
```



### adjust_down

向下调整是为删除做准备的，删除数组的尾元素是毫无意义的，大堆保证了最上面的元素优先级最高，所以总是取出最上面的元素并调整使其仍然满足堆的性质，其频繁用于 TOPK 问题，所以可知所取出的元素必然是堆顶元素。单独的取出堆顶元素是不可取的，其会导致优先级关系紊乱，一般采用与最后元素交换删除后将最后换上来的元素向下调整，是优先级关系正确。

```cpp
void adjust_down(heap* hp, int pos)
{
	assert(hp && pos >= 0);

	int parents = pos;
	int child = parents * 2 + 1;

	while (child < hp->size)
	{
		if (child + 1 != hp->size) //说明不是数组尾元素（未越界）
		{
			child = hp->arr[child] > hp->arr[child + 1] ? child : child + 1; //找大孩子（不可随便找左或者找右）
		}

		if (hp->arr[parents] < hp->arr[child])
		{
			heap_swap(&hp->arr[child], &hp->arr[parents]);

			parents = child;
			child = parents * 2 + 1;
		}
		else
		{
			break;
		}
	}
}
```



### heap_pop

删除操作需要完成的是：交换首尾元素 + 向下调整。

```cpp
void heap_pop(heap* hp)
{
	assert(hp);

	heap_swap(&hp->arr[0], &hp->arr[--hp->size]);

	adjust_down(hp, 0);
}
```



### heap_destroy

有动态开辟的空间则必须对应释放。

```cpp
void heap_destroy(heap* hp)
{
    assert(hp);
    
    free(hp->arr);
    hp->arr = NULL;
    hp->size = 0;
    hp->capacity = 0;
}
```

---

## 效率分析

* 插入：O(`logN`)，顺序表正常插入时间复杂度为 O(1)，向上调整最多调整高度次，也就是 `logN` 次。
* 删除：O(`logN`)，顺序表正常删除尾元素的时间复杂度为 O(1)，向下调整最多调整高度此，也就是 `logN` 次。
* 建堆：O(`N * logN`)，证明详见`Algorithm-HeapSort`。

---

## 补充说明

* 堆是一种数据结构，其承载在数组之上，堆逻辑上是一种完全二叉树，且是一种特殊的优先队列。
* 堆和顺序表的结构一致，是因为其存储方式都采用数组，但其逻辑结构不同，需要进行区分。
* 此文章不作为初学者理解用途，仅适用于复习巩固堆的相关知识，故文章先阐述了 向上调整 向下调整等概念，而不是在 push 和 pop 的时候引出他们。