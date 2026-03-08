# List

---

## 概述

list 是 STL 的容器之一，它是一个类模板。在 STL 中 list 采用双向带头循环链表实现，故本次模拟实现依然采用这种方式（若有疑惑请参考 C 语言基础数据结构）。一般而言，需要对 list 的节点和迭代器进行单独的封装，这将会展现出一个相对复杂的结构，详见下文。

---

## 类的实现

### 成员变量

由于是双向带头循环链表，在类中只需要存储头节点的指针即可，通过头节点即可进行所有链表的访问操作。这里还会储存一个 _size 方便记录容量而非通过遍历链表的方式读取。一般而言节点定义为结构体更为合适，因为默认就是 public，方便使用。用类定义也无伤大雅。

```cpp
namespace Thepale
{
	template <typename T>
	class list_node //链表节点
	{
	public:
		list_node* _prev;
		list_node* _next;
		T _val;
		
	public:
		list_node(const T& val = T())
			:_prev(nullptr)
			,_next(nullptr)
			,_val(val)
		{}
	};
    
    template <typename T>
	class list
	{
	private:
		typedef list_node<T> list_node;

	private:
		list_node* _head;
		size_t _size;
}
```

这里对 `list_node<T>` 进行 typedef 是有意义的，否则头节点的定义应该是：`list_node<T>* _head;` typedef 后我自认为可读性是更强的，也更不容易出现漏写模板参数等错误。

---

### 迭代器

迭代器在链表这一节中被单独拎出来，首先以 vector 为例，来谈谈迭代器。vector 中迭代器是原生指针，所以对迭代器只是简单的 typedef，指针的自增自减操作对编译器而言都是已知操作，例如 int 类型的指针自增就是增加四个字节。（vector 使用原生指针直接封装是一种方法，同样也可以单独封装迭代器）vector 能使用原生指针的很大原因是它从物理空间上是连续存储的，和本身的数据结构有很大关系。而 list 在物理空间上并不连续，也不支持随机访问，当进行自增操作时，若采用原生指针封装必然出现不可预料的错误。故 list 的自增操作我们所希望的是从当前节点移动到下一节点，需要通过运算符重载完成，故 list 的迭代器必须是被单独封装的，否则无法实现运算符重载。

---

#### 成员变量

```cpp
template <typename T, typename Ref, typename Ptr>
class list_iterator
{
public:
    typedef list_node<T> list_node;
    typedef list_iterator<T, Ref, Ptr> self;
    
public:
    list_node* _node;
}
```

迭代器的成员仅需要节点指针即可，在迭代器中对节点指针进行操作即是目的。typedef self 仅仅是为了在返回参数时可以简单明了，维持代码可读性。至于传递三个模板参数是为了解决 const 迭代器的相关问题，具体见：[Iterators](# Iterators)

---

#### 成员函数

##### 构造函数

构造函数仅需要完成将节点的指针构造为迭代器对象的工作即可。

```cpp
list_iterator(list_node* node)
	:_node(node)
{}
```

而对于迭代器的拷贝构造、赋值运算符重载，仅完成浅拷贝即可。由于管理的空间并非迭代器所有，迭代器仅是对其进行一系列的运算操作，故不具有释放权限，析构函数也是不必要的。

---

##### operator++

自增运算符对应的是迭代器自增的过程，需要完成从当前节点指向下一节点的过程：

```cpp
self& operator++()
{
    _node = _node->_next;

    return *this;
}

self operator++(int)
{
    self ret = *this;
    _node = _node->_next;

    return ret;
}
```

这时 self 的价值便体现出来，否则返回值将要写成：`list_iterator<T, Ref, Ptr>& ` 和 `list_iterator<T, Ref, Ptr>`。



##### operator--

```cpp
self& operator--()
{
    _node = _node->_prev;

    return *this;
}

self operator--(int)
{
    self ret = *this;
    _node = _node->_prev;

    return ret;
}
```

---

##### operator*

此操作对应着迭代器的解引用，而这一步的目的是取出数据并返回引用：

```cpp
Ref operator*()
{
    return _node->_val;
}
```

而这里的返回值或许写成：`T&` 即可，因为有了 T 即有了 T& 和 T*，但这一做并不合适，Ref 和 Ptr 的存在也并非冗余，详见：[Iterators](# Iterators)



##### operator->

箭头的操作返回的是数据的地址：

```cpp
Ptr operator->()
{
    return &(_node->_val);
}
```

这一操作可能会让人觉得有些怪异，但当 T 类型为自定义类型时，通过箭头可以获得 T 类型对象的地址，虽然通过解引用操作也可以直接获取 T 类型对象，但它们访问 T 类型对象的成员（假设有 T 类，成员为 public 的 int _a;）的操作分别是这样的：

解引用：`(*it)._a;`
箭头：`it->_a`

但其实这是编译器优化后的结果，箭头的实际操作为：`it->->_a`，故箭头的重载更像是为了符合这一特殊情况而产生的，不必深究。

---

##### operator!=

迭代器需要判断是否等于 end() 来判断是否结束循环，故不等于的判断是必要的，直接比较节点地址即可。

```cpp
bool operator!=(const self& it) const
{
    return _node != it._node;
}
```

---

### 成员函数

#### 构造函数

链表在构造时必须完成的是生成头节点，并完成头节点的链接。

```cpp
//无参构造
list()
    :_head(new list_node())
    ,_size(0)
{
    _head->_prev = _head;
    _head->_next = _head;
}
```

用 n 个 val 构造会复用 resize，具体见：[resize](# resize)

```cpp
//n 个 val 构造
list(size_t n, const T& val = T())
    :_head(new list_node())
    ,_size(0)
{
    _head->_prev = _head;
    _head->_next = _head;

    resize(n, val);
}

list(int n, const T& val = T()) //防止迭代器区间构造的误识别
    :_head(new list_node())
    ,_size(0)
{
    _head->_prev = _head;
    _head->_next = _head;

    resize(n, val);
}
```

迭代器区间构造会复用 push_back，详见：[push_back](# push_back)

```cpp
//迭代器区间构造
template <typename InputIterator>
list(InputIterator first, InputIterator last)
    :_head(new list_node())
    , _size(0)
{
    _head->_prev = _head;
    _head->_next = _head;

    while (first != last)
    {
        push_back(*first);
        ++first;
    }
}
```



#### 拷贝构造函数

拷贝构造函数仅需要将需要拷贝的对象的元素一个个取出来，将值一个个 push_back 进新对象即可。

```cpp
list(const list<T>& ls)
    :_head(new list_node())
    , _size(0)
{
    _head->_prev = _head;
    _head->_next = _head;

    for (auto& e : ls)
    {
        push_back(e);
    }
}
```

可以发现构造函数初始化头节点的操作是必要的，且需要进行多次，这里进行单独封装也是可以的，本文并没有这样做。



#### 析构函数

clear() 的工作是清除除了头节点的所有节点，故析构函数仅需清除所有节点后再释放头节点即可，关于 clear 详见：[clear](# clear)

```cpp
~list()
{
    clear();
    delete _head;
}
```



#### 赋值运算符重载

仍然采用现代写法，调用拷贝构造交换空间即可。

```cpp
void swap(list<T>& ls)
{
    std::swap(_head, ls._head);
    std::swap(_size, ls._size);
}

list<T>& operator=(list<T> ls)
{
    swap(ls);

    return *this;
}
```

---

#### Iterators

```cpp
typedef list_iterator<T, T&, T*> iterator;
typedef list_iterator<T, const T&, const T*> const_iterator;
```

由于迭代器的使用统一规定是 iterator（例如 `list<int>::iterator` 是合法的，而并非 `list_iterator<int, int&, int*>`），虽然链表的迭代器被单独封装，但在链表中将其 typedef 为 iterator 反而是必要的，一是为了遵循标准，二是为了方便使用。这里通过传递 T& 和 const T&，正好区分了普通迭代器和 const 迭代器。因为若传递 const T& 则迭代器中的 Ref 参数为 const T&，在返回 Ref 时，正好返回的时 const T&，从而实现了 const 迭代器的目的。这就是需要传递三个模板参数的原因，这样避免了重新实现一个 const_list_iterator 的冗余操作，因为它们仅仅是在一些函数的返回值上有区别。

从底层而言，不同的参数传递会实例化出不同的对象，以上写法用于区分 const 迭代器和普通迭代器实际上和实现两个迭代器并无二至，只是将这一工作交给了编译器完成。

请注意这两个 typedef 必须是 public 的，因为在类外对迭代器的访问是被允许的。

typedef 也为迭代器的使用提供了便利性和可读性。

```cpp
iterator begin()
{
    return _head->_next; //第一个有效节点是头节点的下一个位置
}

iterator end()
{
    return _head; //尾节点即最后一个有效节点的下一个位置，即头节点
}

const_iterator begin() const
{
    return _head->_next;
}

const_iterator end() const
{
    return _head;
}
```

---

#### Capacity

##### size

由于在成员变量中记录了 _size 的值，则在返回 _size 时极其方便，时间复杂度为 O(1)，若采用遍历链表的方式读取节点个数，则时间复杂度为 O(N)。

```cpp
size_t size() const
{
    return _size;
}
```



##### empty

判空操作使用 _size 也能方便的完成。

```cpp
bool empty() const
{
    return _size == 0;
}
```

---

#### Element Access

由于链表不支持随机访问，一般通过遍历访问，库中只提供了两个意义不大的接口：

##### front

```cpp
T& front()
{
    assert(!empty());

    return _head->_next->_val;
}

const T& front() const
{
    assert(!empty());

    return _head->_next->_val;
}
```



##### back

```cpp
T& back()
{
    assert(!empty());

    return _head->_prev->_val;
}

const T& back() const
{
    assert(!empty());

    return _head->_prev->_val;
}
```

---

#### Modifiers

##### insert

insert 的重载较多，而大部分会进行复用，在本篇中会尽量展现这一复用过程，因为往往它们是实际中使用频繁的接口：

```cpp
iterator insert(iterator pos, size_t n, const T& val)
{
    assert(pos != nullptr);

    list_node* pos_prev = pos._node->_prev;
    list_node* cur = pos._node;
    list_node* new_node = nullptr;

    while (n--)
    {
        new_node = new list_node(val);
		
        //链接
        new_node->_prev = pos_prev;
        new_node->_next = cur;
        pos_prev->_next = new_node;
        cur->_prev = new_node;

        cur = new_node;
        
        _size++;
    }

    return cur; //迭代器返回新插入的元素 - 且这里是单参数的构造触发隐式类型转换
}
```

这里的 _size 是在循环内自增，若在循环外采用 `_size += n;` 会因为 size_t 类型导致 _size 错误，需要进一步处理，故直接在循环内处理较为方便。

```cpp
iterator insert(iterator pos, const T& val)
{
    return insert(pos, 1, val);
}
```

需要注意 insert 不会存在迭代器失效，在 C++ 官方文档中，返回值是新插入节点位置的迭代器。



##### push_front

头插即在 begin 位置插入。本质是对 [insert](# insert) 的复用。

```cpp
void push_front(const T& val)
{
    insert(begin(), val);
}
```



##### push_back

尾插即在 end 位置插入。本质是对 [insert](# insert) 的复用。

```cpp
void push_back(const T& val)
{
    insert(end(), val);
}
```



##### erase

```cpp
iterator erase(iterator first, iterator last)
{
    assert(!empty()); //为空不能删除

    list_node* del = first._node;
    list_node* del_prev = nullptr;
    list_node* del_next = nullptr;

    while (del != last._node) //删除节点
    {
        del_prev = del->_prev;
        del_next = del->_next;

        del_prev->_next = del_next;
        del_next->_prev = del_prev;

        delete del;

        del = del_next; //记录下一个要删除的节点
        _size--;
    }

    return del;
}
```

erase 会导致迭代器失效，返回值为被删除节点的下一个位置。

```cpp
iterator erase(iterator pos)
{
    iterator first = pos;
    iterator last = ++pos;
    return erase(first, last);
}
```



##### pop_front

头删即删除 begin 位置的节点。本质是对 [erase](# erase) 的复用。

```cpp
void pop_front()
{
    erase(begin());
}
```



##### pop_back

尾删即删除 end 位置的节点。本质是对 [erase](# erase) 的复用。

```cpp
void pop_back()
{
    erase(--end());
}
```



##### clear

clear 实则是对 [erase](# erase) 的复用，删除 begin 到 end 区间的所有节点即可。但不包括头节点。

```cpp
void clear()
{
    if (!empty())
    {
        erase(begin(), end());
    }
}
```



##### resize

resize 是对 [push_back](# push_back) 或 [pop_back](# pop_back) 的复用。

```cpp
void resize(size_t n, const T& val = T())
{
    while (n < _size) { pop_back(); }
    while (n > _size) { push_back(val); }
}
```

---

### 整体实现

```cpp
#pragma once

#include <iostream>
#include <assert.h>

namespace Thepale
{
    //list_node
	template <typename T>
	class list_node
	{
	public:
		list_node* _prev;
		list_node* _next;
		T _val;
		
	public:
		list_node(const T& val = T())
			:_prev(nullptr)
			,_next(nullptr)
			,_val(val)
		{}
	};

	//list_iterator
	template <typename T, typename Ref, typename Ptr>
	class list_iterator
	{
	public:
		typedef list_node<T> list_node;

	public:
		list_node* _node;
		typedef list_iterator<T, Ref, Ptr> self;
	public:
		list_iterator(list_node* node)
			:_node(node)
		{}

		Ref operator*()
		{
			return _node->_val;
		}

		Ptr operator->()
		{
			return &(_node->_val);
		}

		self& operator++()
		{
			_node = _node->_next;

			return *this;
		}

		self operator++(int)
		{
			self ret = *this;
			_node = _node->_next;

			return ret;
		}

		self& operator--()
		{
			_node = _node->_prev;
			
			return *this;
		}

		self operator--(int)
		{
			self ret = *this;
			_node = _node->_prev;

			return ret;
		}

		bool operator!=(const self& it) const
		{
			return _node != it._node;
		}
	};

	//list
	template <typename T>
	class list
	{
	private:
		typedef list_node<T> list_node; //私有

	public:
		typedef list_iterator<T, T&, T*> iterator; //公有
		typedef list_iterator<T, const T&, const T*> const_iterator;


	private:
		list_node* _head;
		size_t _size;

	public:
		//构造函数
		list()
			:_head(new list_node())
			,_size(0)
		{
			_head->_prev = _head;
			_head->_next = _head;
		}

		list(size_t n, const T& val = T())
			:_head(new list_node())
			,_size(0)
		{
			_head->_prev = _head;
			_head->_next = _head;

			resize(n, val);
		}

		list(int n, const T& val = T())
			:_head(new list_node())
			, _size(0) //要复用 resize _size必须是 0，不可以直接初始化 _size
		{
			_head->_prev = _head;
			_head->_next = _head;

			resize(n, val);
		} 

		template <typename InputIterator>
		list(InputIterator first, InputIterator last)
			:_head(new list_node())
			, _size(0)
		{
			_head->_prev = _head;
			_head->_next = _head;

			while (first != last)
			{
				push_back(*first);
				++first;
			}
		}

		//拷贝构造函数
		list(const list<T>& ls)
			:_head(new list_node())
			, _size(0)
		{
			_head->_prev = _head;
			_head->_next = _head;

			for (auto& e : ls)
			{
				push_back(e);
			}
		}

		//析构函数
		~list()
		{
			clear();
			delete _head;
		}

		//赋值运算符重载
		list<T>& operator=(list<T> ls)
		{
			swap(ls);

			return *this;
		}

		//Iterators
		iterator begin()
		{
			return _head->_next;
		}

		iterator end()
		{
			return _head;
		}

		const_iterator begin() const
		{
			return _head->_next;
		}

		const_iterator end() const
		{
			return _head;
		}

		//Capacity
		bool empty() const
		{
			return _size == 0;
		}

		size_t size() const
		{
			return _size;
		}

		//Element Access
		T& front()
		{
			assert(!empty());

			return _head->_next->_val;
		}

		const T& front() const
		{
			assert(!empty());

			return _head->_next->_val;
		}

		T& back()
		{
			assert(!empty());

			return _head->_prev->_val;
		}

		const T& back() const
		{
			assert(!empty());

			return _head->_prev->_val;
		}

		//Modifiers
		iterator insert(iterator pos, size_t n, const T& val)
		{
			assert(pos != nullptr);

			list_node* pos_prev = pos._node->_prev;
			list_node* cur = pos._node;
			list_node* new_node = nullptr;

			while (n--)
			{
				new_node = new list_node(val);

				new_node->_prev = pos_prev;
				new_node->_next = cur;
				pos_prev->_next = new_node;
				cur->_prev = new_node;

				cur = new_node;

				_size++;
			}


 			return cur; //迭代器返回新插入的元素 - 且这里是单参数的构造触发隐式类型转换
		}

		iterator insert(iterator pos, const T& val)
		{
			return insert(pos, 1, val);
		}

		void push_front(const T& val)
		{
			insert(begin(), val);
		}

		void push_back(const T& val)
		{
			insert(end(), val);
		}

		iterator erase(iterator first, iterator last)
		{
			assert(!empty());

			list_node* del = first._node;
			list_node* del_prev = nullptr;
			list_node* del_next = nullptr;

			while (del != last._node)
			{
				del_prev = del->_prev;
				del_next = del->_next;

				del_prev->_next = del_next;
				del_next->_prev = del_prev;

				delete del;

				del = del_next;
				_size--;
			}

			return del;
		}

		iterator erase(iterator pos)
		{
			iterator first = pos;
			iterator last = ++pos;
			return erase(first, last);
		}

		void pop_front()
		{
			erase(begin());
		}

		void pop_back()
		{
			erase(--end());
		}

		void clear()
		{
			if (!empty())
			{
				erase(begin(), end());
			}
		}

		void resize(size_t n, const T& val = T())
		{
			while (n < _size) { pop_back(); }
			while (n > _size) { push_back(val); }
		}
		
        //Non-Member Function Overloads
		void swap(list<T>& ls)
		{
			std::swap(_head, ls._head);
			std::swap(_size, ls._size);
		}
	};
}
```

以下为测试代码（非严谨测试）：

```cpp
#include "list.h"
using namespace std;

#include <list>
#include <vector>

int main()
{
	//构造函数测试
	cout << "构造函数测试" << endl;
	Thepale::list<int> e1; for (auto& e : e1) { cout << e << " "; } cout << endl;
	Thepale::list<int> e2(5, 1314); for (auto& e : e2) { cout << e << " "; } cout << endl;
	std::vector<int> v1(10, 666);
	Thepale::list<int> e3(v1.begin() + 2, v1.end()); for (auto& e : e3) { cout << e << " "; } cout << endl;

	//拷贝构造函数测试
	cout << "拷贝构造函数测试" << endl;
	Thepale::list<int> e4(6, 912);
	Thepale::list<int> e5(e4);  for (auto& e : e5) { cout << e << " "; } cout << endl;

	//赋值运算符重载测试
	cout << "赋值运算符重载测试" << endl;
	Thepale::list<int> e6(5, 716);
	Thepale::list<int> e7(8, 888); for (auto& e : e7) { cout << e << " "; } cout << endl;
	e7 = e6; for (auto& e : e7) { cout << e << " "; } cout << endl;

	//size 测试
	cout << "size 测试" << endl;
	Thepale::list<int> e8(666, 1); cout << e8.size() << endl;

	//empty 测试
	cout << "empty 测试" << endl;
	Thepale::list<int> e9; cout << e9.empty() << endl;
	Thepale::list<int> e10(1, 1); cout << e10.empty() << endl;

	//front、back 测试
	cout << "front、back 测试" << endl;
	Thepale::list<int> e11;
	e11.push_back(1);
	e11.push_back(2);
	e11.push_back(3);
	e11.push_back(4);
	cout << e11.front() << endl; cout << e11.back() << endl;

	//insert 测试
	cout << "insert 测试" << endl;
	Thepale::list<int> e12;
	e12.push_back(1);
	e12.push_back(2);
	e12.push_back(3); for (auto& e : e12) { cout << e << " "; } cout << endl;
	e12.insert(++e12.begin(), 3, 888); for (auto& e : e12) { cout << e << " "; } cout << endl;
	e12.insert(e12.end(), 0); for (auto& e : e12) { cout << e << " "; } cout << endl;
	e12.push_front(10000); for (auto& e : e12) { cout << e << " "; } cout << endl;

	//erase 测试
	cout << "erase 测试" << endl;
	Thepale::list<int> e13(8, 888);
	e13.push_back(666);
	e13.push_back(666);
	e13.push_back(666);
	e13.push_front(111);
	e13.push_front(111);
	e13.push_front(111);
	e13.erase(++e13.begin()); for (auto& e : e13) { cout << e << " "; } cout << endl;
	e13.pop_back(); for (auto& e : e13) { cout << e << " "; } cout << endl;
	e13.pop_front(); for (auto& e : e13) { cout << e << " "; } cout << endl;
	e13.erase(++e13.begin(), --e13.end()); for (auto& e : e13) { cout << e << " "; } cout << endl;
	e13.clear(); for (auto& e : e13) { cout << e << " "; } cout << endl;

	//resize 测试
	cout << "resize 测试" << endl;
	Thepale::list<int> e14;
	e14.resize(10, 102938); for (auto& e : e14) { cout << e << " "; } cout << endl;
		
 	return 0;
}
```

---

## 补充说明

* 数据结构类型的文章并不具备顺序阅读的可行性，或许在中途可能会遇到还未实现的函数，你可以选择跳转查看或暂时忽略，等掌握了该函数调用的函数再回头查看。
* C++11 的内容暂时未被添加，例如 emplace，它涉及右值引用和可变模板参数，它会在 C++ 语法篇中被讲解和实现，而这里的数据结构追求可读性强，耦合性低，简洁易懂。故反向迭代器的内容也未被添加，它一般通过封装正向迭代器实现。后续内容将会涉及到它，但它并不适合直接出现在 vector 或 list 中来追求复杂度和理解难度的上升。
