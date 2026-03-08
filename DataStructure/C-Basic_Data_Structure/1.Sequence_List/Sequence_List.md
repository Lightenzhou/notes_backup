# Sequence_List

---

## 结构类

顺序表本质是对数组的操控，是为了能够实现动态内存开辟的动态增长数组。于此，我们首先需要一个**指针变量**来维系动态数组；由于动态开辟数组不能计算元素个数，则需要一个 **size** 来记录数组元素个数；由于动态开辟的内存大小也是一定容量的，如果容量达到上限则需要扩容，则需要一个 **capacity** 记录开辟的容量大小。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Sequence-p1.png)

```cpp
typedef int sequence_datatype; //由于数组类型不确定，所以用 typedef 是更好的选择

struct sequence_list
{
    sequence_datatype* arr;
    int size;
    int capacity;
};

typedef struct sequence_list sequence; //为方便使用，将此就命名为 sequence
```

---

## 成员函数

### sequence_initialize

创建此对象时，无疑有两种方式：

```cpp
sequence my_seq; //1
sequence* my_seq = (sequence*) malloc(sizeof(sequence)); //2
```

如果你倾向于第一种，那你只需要对其成员赋值或取地址传给初始化函数即可；如果你倾向于第二种，那直接传给初始化函数即可，并且在更多情况下都更倾向于第二种，所以这个函数是有必要的，而其功能也是对成员赋值而已。

```cpp
void sequence_initialize(sequence* seq)
{
    assert(seq); //严格禁止传入空指针
    
    seq->arr = NULL;
    seq->size = 0;
    seq->capacity = 0;
}
```

返回值当然可以是 int 以 0 或 非0 来返回你是否初始化成功，这取决于你自己。



### sequence_reserve

先写出扩容操作很容易理解，不难得知你需要在插入操作中频繁的应用到它，为了防止代码冗余，我们先写出扩容将其单独封装为函数，以便于调用。

```cpp
void sequence_reserve(sequence* seq, int new_capacity)
{
    assert(seq && new_capacity > seq->capacity); //新容量必须大于旧容量且必须大于 0
    
    sequence_datatype* tmp = (sequence_datatype*)realloc(seq->arr, sizeof(sequence_datatype) * new_capacity);
    assert(tmp); //防止 realloc 失败
    
    seq->arr = tmp;
    seq->capacity = new_capacity;
}
```



### sequence_insert

相比于 push_back 和 push_front，先写出 insert 显然是更好的选择，因为头插和尾插都可以对 insert 进行复用（头插就是在 pos 为 0 的位置插入，尾插就是在 pos 为 size 的位置插入<*这里以前插为例，即 pos 及 pos 后的数据统一后移*>），这将极大的减少代码的冗余量。

```cpp
void sequence_insert(sequence* seq, int pos, sequence_datatype val)
{
    assert(seq && pos >= 0); //插入位置严格大于等于 0

    //在 pos 插入，把原 pos 及 pos 后的数据后移（这样头插无需传入 -1 复用，而是传入 0 复用）
    if (seq->size == seq->capacity) //容量满时先扩容
    {
        sequence_reserve(seq, seq->capacity == 0 ? 10 : seq->capacity * 2); //注意对空容量的处理
    }

    for (int i = 0; i < seq->size - pos; ++i) //总循环次数 = 总数据个数 - 插入下标处
    {
        seq->arr[seq->size - i] = seq->arr[seq->size - i - 1];
    }

    seq->arr[pos] = val;
    seq->size++;
}
```



### sequence_push_back

尾插就是在 pos 为 size 的位置插入。

```cpp
void sequence_push_back(sequence* seq, sequence_datatype val)
{
    sequence_insert(seq, seq->size, val); //无需断言，在 insert 中自会处理
}
```



### sequence_push_front

头插就是在 pos 为 0 的位置插入。

```cpp
void sequence_push_front(sequence* seq, sequence_datatype val)
{
    sequence_insert(seq, 0, val);
}
```



### sequence_erase

删除同理，相比之下，先实现 erase 可以更好的减少代码冗余，因为头删尾删都可对其进行复用。

```cpp
void sequence_erase(sequence* seq, int pos)
{
    assert(seq && seq->arr && pos >= 0 && pos < seq->size); //*保证 seq 存在、 arr 有实际空间且 pos 位置合法*

    for (int i = 0; i < seq->size - pos - 1; ++i) //总循环次数 = 总元素个数 - 删除下标处 - 1
    {
        seq->arr[pos + i] = seq->arr[pos + i + 1]; //覆盖删除
    }

    seq->size--;

    //对于 pos == seq->size - 1 的情况无需担心，尽管循环未进入，但 size 自减后仍然相当于删除
}
```



### sequence_pop_back

尾删可以直接使 size-- 或复用 erase，尾删就是在 size - 1位置的 erase。

```cpp
void sequence_pop_back(sequence* seq)
{
    sequence_erase(seq, seq->size - 1);
}

//你完全可以写成这样，且少一次函数调用，保持复用仅为了格式一致
void sequence_pop_back(sequence* seq)
{
    assert(seq && seq->arr && seq->size > 0); //保证有数据存在（arr 有实际空间且 size 大于 0）
    seq->size--;
}
```



### sequence_pop_front

头删就是在 0 位置的 erase。

```cpp
void sequence_pop_front(sequence* seq)
{
    sequence_erase(seq, 0);
}
```



### sequence_destroy

对于 malloc、realloc 等动态开辟的空间，最后不再使用时需要手动释放防止内存泄漏。

```cpp
void sequence_destroy(sequence* seq)
{
    assert(seq);
    
    free(seq->arr);
    seq->size = 0;
    seq->capacity = 0;
}
```



至于例如：查找、更改、获取 size 大小等操作可自行实现，以上仅实现最关键的成员函数功能。



### 整体实现

```cpp
#pragma once
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

typedef int sequence_datatype; //由于数组类型不确定，所以用 typedef 是更好的选择

struct sequence_list
{
    sequence_datatype* arr;
    int size;
    int capacity;
};

typedef struct sequence_list sequence; //为方便使用，将此就命名为 sequence

void sequence_initialize(sequence* seq)
{
    assert(seq); //严格禁止传入空指针

    seq->arr = NULL;
    seq->size = 0;
    seq->capacity = 0;
}

void sequence_reserve(sequence* seq, int new_capacity)
{
    assert(seq && new_capacity > seq->capacity); //新容量必须大于旧容量且必须大于 0

    sequence_datatype* tmp = (sequence_datatype*)realloc(seq->arr, sizeof(sequence_datatype) * new_capacity);
    assert(tmp); //防止 realloc 失败

    seq->arr = tmp;
    seq->capacity = new_capacity;
}

void sequence_insert(sequence* seq, int pos, sequence_datatype val)
{
    assert(seq && pos >= 0); //插入位置严格大于等于 0

    //在 pos 插入，把原 pos 及 pos 后的数据后移（这样头插无需传入 -1 复用，而是传入 0 复用）
    if (seq->size == seq->capacity) //容量满时先扩容
    {
        sequence_reserve(seq, seq->capacity == 0 ? 10 : seq->capacity * 2); //注意对空容量的处理
    }

    for (int i = 0; i < seq->size - pos; ++i) //总循环次数 = 总数据个数 - 插入下标处
    {
        seq->arr[seq->size - i] = seq->arr[seq->size - i - 1];
    }

    seq->arr[pos] = val;
    seq->size++;
}

void sequence_push_back(sequence* seq, sequence_datatype val)
{
    sequence_insert(seq, seq->size, val); //无需断言，在 insert 中自会处理
}

void sequence_push_front(sequence* seq, sequence_datatype val)
{
    sequence_insert(seq, 0, val);
}

void sequence_erase(sequence* seq, int pos)
{
    assert(seq && seq->arr && pos >= 0 && pos < seq->size); //*保证 seq 存在、 arr 有实际空间且 pos 位置合法*

    for (int i = 0; i < seq->size - pos - 1; ++i) //总循环次数 = 总元素个数 - 删除下标处 - 1
    {
        seq->arr[pos + i] = seq->arr[pos + i + 1]; //覆盖删除
    }

    seq->size--;

    //对于 pos == seq->size - 1 的情况无需担心，尽管循环未进入，但 size 自减后仍然相当于删除
}

void sequence_pop_back(sequence* seq)
{
    sequence_erase(seq, seq->size - 1);
}

void sequence_pop_front(sequence* seq)
{
    sequence_erase(seq, 0);
}

void sequence_destroy(sequence* seq)
{
    assert(seq);

    free(seq->arr);
    seq->size = 0;
    seq->capacity = 0;
}
```



---

## 效率分析

- 头插：O(N) - 需要整体移动数据腾出空间
- 尾插：O(1) - 直接在 size 处插入
- 插入：O(N) - 需要在 pos 处插入，仍需整体移动数据
- 头删：O(N) - 需要整体移动数据覆盖删除
- 尾删：O(1) - 直接将 size 自减
- 删除：O(N) - 需要在 pos 处删除，仍需整体移动数据

---

## 补充说明

- 顺序表本是 C++ STL 中 vector 的前身，所以函数命名规则统一遵循了 C++ 命名规则。
- 本代码一律采用 “无冗余解释” 的风格，所以你不会在第二个 assert 中看到 “严格禁止传入空指针” 的字眼。
- 本代码一律采用 “结果最优化” 的写法，或许在初学时应该先去理解头插和尾插，而不是一上来就接触 insert 插入，但其理解并未造成大的困难，故先理解 insert 也无影响。故本文未阐述头插尾插等是因理解 insert 同时也可以理解头插尾插，这既是 “结果最优化”，其并非一味的追求代码量冗余量少而不考虑理解难度。
