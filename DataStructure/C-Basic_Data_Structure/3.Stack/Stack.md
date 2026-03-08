# Stack

---

## 结构类

栈是一个后进先出，或者说是先进后出的结构。其结构是一个上端开口下端闭口的容器，先放入的只能后取出来，具体逻辑结构如下：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Stack-p1.png)

如果把 A B C 三个元素放入，则会变成这样：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Stack-p2.png)

则如果此时要取数据，栈底是封闭式的，无法取出数据，则只能从栈顶取，而此时的 C 即是栈顶。（*注意，栈顶是随时改变的，例如取出 C 后，栈顶就是 B*）

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Stack-p3.png)

由此，后进先出的特性得以体现。

栈可以以很多方式实现，例如顺序表，链表等都可用来实现栈，但此处采用顺序表的方式实现，其具体原因以及原理，可参考补充说明。而既然是顺序表管理，也容易得知其结构为：

```cpp
typedef int stack_datatype;

struct stack
{
    stack_datatype* arr;
    int size; //由于 size 所在的位置是栈顶，一些地方也直接命名为 top
    int capacity;
};

typedef struct stack stack;
```

其结构与顺序表并无二样，只是对于数据操作有些大同小异。例如根据其特性可以得知，栈只能实现尾插尾删等。

---

## 成员函数

### stack_initialize

和顺序表一致，其结构类型需要初始化函数。

```cpp
void stack_initialize(stack* st)
{
    assert(st); //结构体必须存在
    
    st->arr = NULL;
    st->size = 0;
    st->capacity = 0;
}
```

有一点忘记提及的是，size 从 0 开始则意味着 size 做下标位置则永远指向有效数据的下一个，当然，你也可以让 size 初始化为 -1 而保证其指向尾部元素，依据习惯而定，不做区分。且一旦更改这一点，插入等操作的具体实现步骤会有轻微变化，本文仅以 size 为 0 行文。



### stack_reverse

扩容操作仍然是必须的，因为插入操作仍然必不可少，这同时也取决于顺序表的特性，如果你用链表实现则不需要这一步，因为链表会在插入时自动扩容。

```cpp
void stack_reverse(stack* st, int new_capacity)
{
    assert(st && new_capacity > st->capacity); //新容量要严格大于就容量，且这一步已经包含了其必须大于 0 的条件
    
    stack_datatype* tmp = (stack_datatype*)realloc(st->arr, sizeof(stack_datatype) * new_capacity);
    assert(tmp); //防止 realloc 失败
    
    st->arr = tmp;
    st->capacity = new_capacity;
}
```



### stack_push

对于栈的特性而言，不存在头插和插入的概念，所有插入均只能尾插，即**入栈**。所以名称的命名为 push，当然，遵循 STL 命名规则。

```cpp	
void stack_push(stack* st, stack_datatype val)
{
    assert(st);
    
    if(st->size == st->capacity)
    {
        int new_capacity = st->capacity == 0 ? 10 : st->capacity * 2; //考虑栈为空的情况
        stack_reverse(st, new_capacity);
    }
    
    //入栈
    st->arr[st->size++] = val;
}
```



### stack_empty

为了保证代码的封装性和易读性，单独实现 empty 判空栈。在上面已叙述过的问题，关于 size 初始化为 0 还是 -1 的问题，如果没有 empty 函数判空，则容易导致错误。（*如果对方不清楚你的具体实现*）

```cpp
bool stack_empty(stack* st)
{
    return st->size == 0;
}
```



### stack_pop

有入栈的概念，则必然有**出栈**的概念，其也就对应顺序表的尾删。

```cpp
void stack_pop(stack* st)
{
    assert(st && !stack_empty(st)); //保证栈存在且不为空
    
    //出栈
    st->size--;
}
```



### stack_top

入栈出栈固然重要，但入和出的操作隐于其后，栈取出数据只能从栈顶拿取，则取数据的操作也是单一的，采用单独的函数执行。

```cpp
stack_datatype stack_top(stack* st)
{
    assert(st && !stack_empty(st));
    
    return st->arr[st->size - 1];
}
```



### stack_destroy

对于动态申请的空间，不要忘了释放。

```cpp
void stack_destroy(stack* st)
{
    assert(st);
    
    free(st->arr);
    st->arr = NULL;
    st->size = 0;
    st->capacity = 0;
}
```



---

## 效率分析

* 入栈：O (1)
* 出栈：O (1)
* 取栈顶数据：O (1)

---

## 补充说明

* 代码的凝练和可读性都是十分重要的，如 stack_push 中的入栈操作：`st->arr[st->size++] = val;` 如果阅读困难，可自行拆解。
* 通过效率分析，链表和顺序表实现栈都是可行的，选择顺序表的原因要从 CPU 说起。CPU 总是从它的三级缓存中拿取数据，其原因是为了保证运行速度，所以可知 CPU 不会直接去内存条或者硬盘中拿数据，所有数据都必须先加载到缓存。顺序表究其优点是因为其为顺序结构，保证了数据的连续性，这样 CPU 缓存加载数据同时会把所拿数据的周围数据同时拿走，进行多次内存操作，顺序表因为数据连续，所以大概率都被拿到了缓存中，这样会被 CPU 直接 “命中”，而链表则数据离散比较大，可能面临缓存找不到导致从下级再次查找的可能。总的来说，**顺序表保证了 CPU 的命中率高，故理论上速度更快**。
* 这里所述的栈是一种数据结构，而操作系统的栈是一种被划分出的空间，负责存储局部变量或为函数调用分配空间等，请注意区分。
