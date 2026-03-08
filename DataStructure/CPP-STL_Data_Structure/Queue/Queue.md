# Queue

---

## 概述

queue 是一个容器适配器，默认采用 deque 进行适配。由于队列先进先出的特性，deque 和 list 等容器能较好适配。队列更大程度上仅仅是对适配容器的封装和复用，故实现过程不具体分析。

---

## 类的实现

```cpp
#pragma once

#include <iostream>
#include <deque>

namespace Thepale
{
	template <typename T, typename Container = std::deque<T>>
	class queue
	{
	private:
		Container _con;

	public:
		bool empty()
		{
			return _con.empty();
		}

		size_t size()
		{
			return _con.size();
		}

		T& front()
		{
			return _con.front();
		}

		const T& front() const
		{
			return _con.front();
		}

		T& back()
		{
			return _con.back();
		}

		const T& back() const
		{
			return _con.back();
		}

		void push(const T& val)
		{
			_con.push_back(val);
		}

		void pop()
		{
			_con.pop_front();
		}
	};
}
```

以下为测试代码（非严谨测试）：

```cpp
#include "queue.h"
#include <list>
using std::cout; using std::endl;

int main()
{
	//以 list 为 container
	cout << "以 list 为 container" << endl;
	Thepale::queue<int, std::list<int>> qu1;
	cout << qu1.empty() << endl;
	qu1.push(1);
	qu1.push(2);
	qu1.push(3);
	qu1.push(4);
	cout << qu1.size() << endl;
	cout << qu1.empty() << endl;
	cout << qu1.front() << endl;
	cout << qu1.back() << endl;
	qu1.pop();
	qu1.pop();
	cout << qu1.front() << endl;

	//以 deque 为 container
	cout << "以 deque 为 container" << endl;
	Thepale::queue<int> qu2;
	cout << qu2.empty() << endl;
	qu2.push(1);
	qu2.push(2);
	qu2.push(3);
	qu2.push(4);
	cout << qu2.size() << endl;
	cout << qu2.empty() << endl;
	cout << qu2.front() << endl;
	cout << qu2.back() << endl;
	qu2.pop();
	qu2.pop();
	cout << qu2.front() << endl;

	return 0;
}
```

