# Priority_Queue

---

## 概述

优先级队列是一个容器适配器，但对封装的要求较高。优先级队列为其中的元素根据一定的规则分配优先级以实现最高优先级的元素在最前面的特性，在 C 语言基本的数据结构中，二叉堆是实现优先级队列的不二选择，故封装和复用逻辑将以二叉堆为主，默认的适配容器也是 vector。由于需要有大堆或小堆等需求，仿函数在这一节中也会被介绍。

---

## 类的实现

由于使用的是容器适配器，构造、析构、拷贝、赋值等函数的实现无需自己完成，依靠所传递的容器即可。

### 迭代器区间构造

这是需要额外说明的特殊情况，当已经确定了需要用哪些数据建堆时，调用 push 函数则会导致 adjust_up 被调用建堆，建堆的时间复杂度为：`O(N * logN)` ，使用 adjust_down 建堆可使时间复杂度降为：`O(N)`（时间复杂度的分析与证明请见：Algorithm-Heap_Sort）

```cpp
template <typename InputIterator>
priority_queue(InputIterator first, InputIterator last)
{
    while (first != last)
    {
        _con.push_back(*first);
        ++first;
    }

    for (int i = (_con.size() - 1 - 1) / 2; i >= 0; --i)
    {
        adjust_down(i);
    }
}
```

有了迭代器区间初始化后，编译器提供的默认构造函数不会再生成，这里需要手动生成。

```cpp
priority_queue() {}
```



### push

单个数据的插入需要插入后向上调整，即和双亲节点对比优先级，优先级高的要往上移动，一直比较到孩子比双亲节点的优先级低或到根节点为止。

```cpp
//adjust_up
void adjust_up(size_t pos)
{
    Compare com;

    int child = (int)pos;
    int parent = (int)(child - 1) / 2;

    while (child)
    {
        if(com(_con[parent], _con[child]))
        {
            std::swap(_con[parent], _con[child]);

            child = parent;
            parent = (child - 1) / 2;
        }
        else
        {
            break;
        }
    }
}
```

```cpp
void push(const T& val)
{
    _con.push_back(val);
    adjust_up(_con.size() - 1);
}
```



### pop

删除采用首尾元素交换后首元素向下调整，需要找左右孩子中优先级高的进行比较，使优先级高的元素上浮，且使自身到达合适的位置。循环到双亲节点的优先级大于两个孩子或到叶子节点为止。

```cpp
//adjust_down
void adjust_down(size_t pos)
{
    Compare com;

    int parent = (int)pos;
    int child = (int)(parent * 2 + 1);

    while (child < size())
    {
        //修正孩子
        if (child + 1 < size() && com(_con[child], _con[child + 1]))
        {
            ++child;
        }

        if (com(_con[parent], _con[child]))
        {
            std::swap(_con[child], _con[parent]);

            parent = child;
            child = parent * 2 + 1;
        }
        else
        {
            break;
        }
    }
```

```cpp
void pop()
{
    assert(!empty());

    std::swap(_con.front(), _con.back());
    _con.pop_back();
    adjust_down(0);
}
```



### 仿函数

在 adjust_up 和 adjust_down 中，仿函数被应用，它主要负责优先级的比较逻辑（控制大的优先级大还是小的优先级大以实现大堆或小堆）。其本质为一个类实例化的对象调用 operator() 进行特定的比较后返回需要的逻辑值，例如库中的 less 进行的是小于比较：

```cpp
template <typename T>
class Less
{
public:
    bool operator()(const T& e1, const T& e2)
    {
        return e1 < e2;
    }
};
```

故在后续的比较逻辑中，直接使用 Compare 类实例化出对象调用 operator() 进行比较即可，可以方便程序中逻辑的控制，减少代码冗余。

在一些情况中，自己实现仿函数是必要的，例如一个 vector 中储存的是自定义类型的指针，如果调用库中的仿函数则会识别成内置类型指针进行比较，而如果此时所希望的是比较对象的大小，则需要自己实现仿函数单独处理这一情况。

---

### 整体实现



```cpp
#pragma once
#include <iostream>
#include <vector>
#include <assert.h>

namespace Thepale
{
	template <typename T, typename Container = std::vector<T>, typename Compare = std::less<T>>
	class priority_queue
	{
	private:
		Container _con;

	private:
		void adjust_up(size_t pos)
		{
			Compare com;

			int child = (int)pos;
			int parent = (int)(child - 1) / 2;

			while (child)
			{
				if(com(_con[parent], _con[child]))
				{
					std::swap(_con[parent], _con[child]);

					child = parent;
					parent = (child - 1) / 2;
				}
				else
				{
					break;
				}
			}
		}

		void adjust_down(size_t pos)
		{
			Compare com;

			int parent = (int)pos;
			int child = (int)(parent * 2 + 1);

			while (child < size())
			{
				//修正孩子
				if (child + 1 < size() && com(_con[child], _con[child + 1]))
				{
					++child;
				}

				if (com(_con[parent], _con[child]))
				{
					std::swap(_con[child], _con[parent]);

					parent = child;
					child = parent * 2 + 1;
				}
				else
				{
					break;
				}
			}

		}

	public:
		priority_queue() {}

		template <typename InputIterator>
		priority_queue(InputIterator first, InputIterator last)
		{
			while (first != last)
			{
				_con.push_back(*first);
				++first;
			}

			for (int i = (_con.size() - 1 - 1) / 2; i >= 0; --i)
			{
				adjust_down(i);
			}
		}

		bool empty() const
		{
			return _con.empty();
		}

		size_t size() const
		{
			return _con.size();
		}

		T& top()
		{
			return _con.front();
		}

		const T& top() const
		{
			return _con.front();
		}

		void push(const T& val)
		{
			_con.push_back(val);
			adjust_up(_con.size() - 1);
		}

		void pop()
		{
			assert(!empty());

			std::swap(_con.front(), _con.back());
			_con.pop_back();
			adjust_down(0);
		}
	};
}
```

以下为测试代码（非严谨测试）：

```cpp
#include "priority_queue.h"
using std::cout; using std::endl;

int main()
{
	//小堆测试
	cout << "小堆测试" << endl;
	Thepale::priority_queue<int, std::vector<int>, std::greater<int>> pq1;
	cout << pq1.empty() << endl;
	pq1.push(4);
	pq1.push(7);
	pq1.push(2);
	pq1.push(6);
	pq1.push(8);
	pq1.push(3);
	pq1.push(5);
	pq1.push(9);
	cout << pq1.empty() << endl;
	cout << pq1.size() << endl;

	while (!pq1.empty())
	{
		cout << pq1.top() << " ";
		pq1.pop();
	}
	cout << endl;

	//大堆测试
	cout << "大堆测试" << endl;
	Thepale::priority_queue<int, std::vector<int>, std::less<int>> pq2;
	cout << pq2.empty() << endl;
	pq2.push(4);
	pq2.push(7);
	pq2.push(2);
	pq2.push(6);
	pq2.push(8);
	pq2.push(3);
	pq2.push(5);
	pq2.push(9);
	cout << pq2.empty() << endl;
	cout << pq2.size() << endl;

	while (!pq2.empty())
	{
		cout << pq2.top() << " ";
		pq2.pop();
	}
	cout << endl;

	//迭代器区间构造测试
	cout << "迭代器区间构造测试" << endl;
	std::vector<int> v;
	v.push_back(6);
	v.push_back(2);
	v.push_back(9);
	v.push_back(8);
	v.push_back(3);
	v.push_back(1);

	Thepale::priority_queue<int, std::vector<int>, std::less<int>> pq3(v.begin(), v.end());
	while (!pq3.empty())
	{
		cout << pq3.top() << " ";
		pq3.pop();
	}
	cout << endl;

	return 0;
}
```

