# Stack

---

## 概述

stack 是一个容器适配器。由栈的后进先出的特性可知，其可使用 vector、list 等多种容器实现，故栈仅需要对特定容器进行进一步封装即可。栈的实现不会展开叙述，其功能一般都复用和封装了其它容器。

---

## 类的实现

```cpp
#pragma once

#include <iostream>
#include <deque>

namespace Thepale
{	
	template <typename T, typename Container = std::deque<T>>
	class stack
	{
	private:
		Container _con;

	public:
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
			return _con.back();
		}

		const T& top() const
		{
			return _con.back();
		}

		void push(const T& val)
		{
			_con.push_back(val);
		}

		void pop()
		{
			_con.pop_back();
		}
	};
}

```

以下为测试代码（非严谨测试）：

```cpp
#include "stack.h"
#include <vector>
#include <list>
using std::cout; using std::endl;

int main()
{
	//以 vector 为 container
	cout << "以 vector 为 container" << endl;
	Thepale::stack<int, std::vector<int>> st1;
	cout << st1.empty() << endl;
	st1.push(1);
	st1.push(2);
	st1.push(3);
	st1.push(4);
	cout << st1.size() << endl;
	cout << st1.empty() << endl;
	cout << st1.top() << endl;
	st1.pop();
	st1.pop();
	cout << st1.top() << endl;

	//以 list 为 container
	cout << "以 list 为 container" << endl;
	Thepale::stack<int, std::list<int>> st2;
	cout << st2.empty() << endl;
	st2.push(1);
	st2.push(2);
	st2.push(3);
	st2.push(4);
	cout << st2.size() << endl;
	cout << st2.empty() << endl;
	cout << st2.top() << endl;
	st2.pop();
	st2.pop();
	cout << st2.top() << endl;

	//以 deque 为 container
	cout << "以 deque 为 container" << endl;
	Thepale::stack<int> st3;
	cout << st3.empty() << endl;
	st3.push(1);
	st3.push(2);
	st3.push(3);
	st3.push(4);
	cout << st3.size() << endl;
	cout << st3.empty() << endl;
	cout << st3.top() << endl;
	st3.pop();
	st3.pop();
	cout << st3.top() << endl;
	return 0;
}
```

---

## 补充说明

* deque 是双端队列，一般作为栈和队列的默认容器进行适配。它采用中控数组的方式控制多段 buffer，既做到了随机访问又采用了链式链接减少了扩容开销，但由于需要两者兼顾，导致随机访问效率较低，中间插入代价较大，用作栈和队列的默认容器较好，其它场景无法代替 vector 和 list 的位置。
