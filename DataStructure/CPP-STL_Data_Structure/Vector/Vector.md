# Vector

---

## 概述

vector 是严格意义上 STL 的容器之一，它是一个类模板[^类模板和模板类]，可以通过显式实例化来管理多种数据类型。在 C 语言中，它对应的数据结构是顺序表。该模板对于 bool 类型的实例化是特殊的，因为 bool 类型的管理往往和其它类型不太相同，这里的 vector 实现不包含 bool 类型的部分。

---

## 类的实现

### 成员变量

和 C 语言的顺序表类似，vector 管理的依然是一个特定类型的数组，但管理方式有所变化，需要 start，finish，end_of_storage 进行管理，且由于是类模板，需要添加模板参数且为了独立性和测试的方便这里封装在命名空间中。

以下为截取 STL 源码部分：

```cpp
template <class T, class Alloc = alloc> //关于 Alloc（空间配置器）的内容后续会有讲解
class vector {
public:
  typedef T value_type;
  typedef value_type* pointer;
  typedef const value_type* const_pointer;
  typedef value_type* iterator;
  typedef const value_type* const_iterator;
  typedef value_type& reference;
  typedef const value_type& const_reference;
  typedef size_t size_type;
  typedef ptrdiff_t difference_type;
    
protected:
  iterator start;
  iterator finish;
  iterator end_of_storage;
```

可知在 vector 中采用 start 和 end 代替了 _arr 和 _size，用 end_of_storage 代替了 _capacity，且采用迭代器的方式管理，由 typedef 可见 vector 的迭代器也是简单的对指针的封装，且由于是类模板的形式，最好将引用也封装一层方便理解。

```cpp
namespace Thepale
{
	template<typename T>
	class vector
	{
	public:
		typedef T* iterator;
		typedef const T* const_iterator;
		typedef T& reference;
		typedef const T& const_reference;


	private: //不涉及继承请认为这里的 private 和 protected 是一致的
		iterator start;
		iterator finish;
		iterator end_of_storage;
    };
}
```

---

### 成员函数

#### 构造函数

在 vector 中有三种主流的构造方式，分别是：无参构造，n 个 val 构造和迭代器区间构造。

```cpp
//无参构造
vector()
    :start(nullptr)
    ,finish(nullptr)
    ,end_of_storage(nullptr)
{}
```

是否需要在构造时就开空间取决于个人行为，可以开默认空间，也可以当使用时再进行检查是否开辟。

```cpp
//n 个 val 构造
vector(size_t n, const T& val)
    :start(nullptr)
    ,finish(nullptr)
    ,end_of_storage(nullptr)
{
    T* tmp = new T[n];
    for (size_t i = 0; i < n; ++i)
    {
    	tmp[i] = val;
    }

    start = tmp;
    finish = start + n;
    end_of_storage = start + n;
}
```

请注意，这里使用了 for 循环赋值而非 memcpy 等内存函数拷贝。当 T 类型中有动态管理的空间时，memcpy 会导致两个对象同时管理同一块空间（例如指针被复制），这在析构时将会报错。赋值往往是调用了 T 类型中的 operator= 从而再实现 T 类型的深拷贝。

```cpp
//迭代器区间构造
template<typename InputIterator>
vector(InputIterator first, InputIterator last)
    :start(nullptr)
    , finish(nullptr)
    , end_of_storage(nullptr)
{
    while (first != last)
    {
        push_back(*first);
        ++first;
    }
}
```

迭代器区间构造在类模板中再次使用了模板函数，首先这是被允许的，其次这样做的目的是为了让不同类型的迭代器都能用于构造 vector，例如 list 的迭代器非简单的指针封装，再次使用模板可以让 InputIterator 变为 list 的迭代器类型，在执行自增等相关操作时调用 list 的迭代器中的相关操作，而非 vector 中使指针的地址自增，因为并非所有的数据结构都是地址连续存储的，故再次使用函数模板是必要的。

而这往往会导致使用 int 初始化时：`vector<int> e(10, 1);` 错误的调用到迭代器区间构造。原因是 size_t 类型和 int 类型并未成功匹配，而 InputIterator 却可以正好识别为 int 满足需求，所以存在了抢占问题，一般需要进行重载来解决这种情况：

```cpp
vector(int n, const T& val) //更换为 int 类型，第一时间匹配
    :start(nullptr)
    ,finish(nullptr)
    ,end_of_storage(nullptr)
{
	resize(n, val); //可以提前预知 resize 刚好具有初始化 n 个 val 的情况，这里必然可以复用，详见 resize
}
```



#### 拷贝构造函数

```cpp
vector(const vector<T>& v)
    :start(nullptr)
    ,finish(nullptr)
    ,end_of_storage(nullptr)
{
    size_t _size = v.size(); //既可以扩容到 size，也可以扩容到 capacity
    T* tmp = new T[_size];

    for (size_t i = 0; i < _size; ++i)
    {
        tmp[i] = v.start[i];
    }

    start = tmp;
    finish = start + _size;
    end_of_storage = start + _size;
}
```

拷贝构造函数同时也需要注意必须使用循环赋值而非 memcpy，防止出现 **深拷贝中的浅拷贝**（我再次冗余的提起这一点，拷贝构造函数实实在在的开辟了新空间，这是 vector 的深拷贝，但如果直接移动数据则导致开辟了动态空间的类型发生两个指针管理一块空间的情况，在析构时，delete[] start 又会调用 T 类型的析构函数，导致同一块空间二次析构而发生错误。时刻注意，你需要保证 T 类型也执行深拷贝，而非仅完成 vector 的深拷贝）。

当然，后续实现了 reserve 和 push_back 后，甚至可以这样实现拷贝构造函数：

```cpp
vector(const vector<T>& v)
    :start(nullptr)
    ,finish(nullptr)
    ,end_of_storage(nullptr)
{
    size_t _size = v.size();
    reserve(_size);

    for (auto& e : v)
    {
        push_back(e);
    }
}
```

这里的范围 for 使用了引用，在 [insert](#insert) 会有详细说明。



#### 析构函数

```cpp
~vector()
{
    delete[] start;

    start = nullptr;
    finish = nullptr;
    end_of_storage = nullptr;
}
```



#### 赋值运算符重载

```cpp
void swap(const vector<T>& v)
{
    std::swap(start, v.start);
    std::swap(finish, v.finish);
    std::swap(end_of_storage, v.end_of_storage);
}
```

```cpp
vector<T>& operator=(vector<T> v)
{
    swap(v);

    return *this;
}
```

赋值运算符重载仍然可以通过参数不传引用的方式直接调用拷贝构造实现拷贝，交换数据后让 v 对象销毁顺便删除原对象数据，一举两得。

---

#### Iterators

注意迭代器区间是左闭右开的，finish 实际上指向的是最后一个数据的下一个位置。

```cpp
iterator begin()
{
    return start;
}

iterator end()
{
    return finish;
}

const_iterator begin() const
{
    return start;
}

const_iterator end() const
{
    return finish;
}
```

---

#### capacity

##### size

size 和 capacity 的计算都用到了指针相减，会得出两个指针间的元素个数。在空指针的情况下是可以判断的，因为 0 - 0 == 0

```cpp
size_t size() const
{
    return finish - start;
}
```



##### capacity

```cpp
size_t capacity() const
{
    return end_of_storage - start;
}
```



##### empty

empty 可以通过判断 size 大小或直接判断指针是否为空，两者无本质区别。

```cpp
bool empty() const
{
    return size();
}
```



##### reserve

```cpp
void reserve(size_t new_capacity)
{
    if (new_capacity > capacity())
    {
        size_t _size = size();

        T* tmp = new T[new_capacity];

        if (start != nullptr) //若是空指针无需执行下列步骤
        {
            for (size_t i = 0; i < _size; ++i)
            {
                tmp[i] = start[i];
            }

            delete[] start;
        }

        start = tmp;
        finish = start + _size;
        end_of_storage = start + new_capacity;
    }
}
```

涉及到扩容操作时，因为是异地扩容，需要将原数据拷贝到开辟的新空间。这里仍然不能使用 memcpy，后续会将 vector 中的数据 delete[]，这将导致新开的空间装载了一堆被释放了的指针，再三强调，不要出现深拷贝中的浅拷贝，使用 for 循环赋值即可（前提是 T 类型实现了赋值运算符重载的深拷贝，这是基本要求）。



##### resize

resize 依然分为两种情况，n 比 size 要小，n 比 size 大但比 capacity 小，n 比 capacity 大。后两种情况可以一并处理。

```cpp
void resize(size_t n, const T& val = T())
{
    size_t _size = size(); //这是为了防止 size() 被多次调用，故用变量储存

    if (n < _size)
    {
        finish = start + n; //直接定义新尾
    }
    else
    {
        reserve(n); //n 比 capacity 小时这一步是没有影响的

        for (size_t i = 0; i < n - _size; ++i)
        {
            *(finish) = val; //从末尾开始拷贝数据
            ++finish;
        }
    }
}
```

---

#### Element access

##### operator[]

返回值可以写为 T& 和 const T&，这里方便理解 typedef 做了一层封装。

```cpp
reference operator[](size_t pos)
{
    assert(pos < size());

    return start[pos];
}

const_reference operator[](size_t pos) const
{
    assert(pos < size());

    return start[pos];
}
```

---

#### Modifiers

##### insert

insert 的版本也比较多，其中在固定的迭代器位置插入一个 val，我认为可以用在固定的迭代器位置插入 n 个 val 代替，并没有再次实现，实则也只是一个复用而已。在 C++11 中，这里的形参类型应是 `const_iterator pos`，可以理解不希望传入的 pos 位置被改变，但这又会需要创建额外的迭代器，因为插入会基于 pos 位置进行修改，故这里仅使用 C++98 提供的普通迭代器，便于理解。

```cpp
iterator insert(iterator pos, size_t n, const T& val = T())
{
    assert(pos >= start && pos <= finish);

    size_t _capacity = capacity();
    size_t _size = size();
    size_t fix = pos - start; //计算 pos 位置基于 start 位置的偏移

    if (n > _capacity - _size)
    {
        reserve(_capacity == 0 ? 4 : _capacity + n); //reserve 后 迭代器位置均改变
        pos = start + fix; //修正迭代器失效
    }

    for (size_t i = 0; i < (size_t)(finish - pos); ++i) //移动数据
    {
        *(finish + n - i - 1) = *(finish - i - 1);
    }

    for (size_t i = 0; i < n; ++i) //填数据
    {
        *(pos + i) = val; //直接基于 pos 位置赋值，使用 const 迭代器则不可以这样做
    }

    finish += n;

    return pos;
}
```

在进行 reserve 操作后，**由于是异地扩容，会导致原指针失效（迭代器失效[^迭代器失效]）**，故 **需要记录 pos 的相对位置，扩容后进行复原**，否则将访问已释放的空间。这里填数据的操作是：`*(pos + i) = val;` 这一步类似于引用对象给另一对象赋值，会调用赋值运算符重载，在拷贝构造函数的范围 for 中，传引用也会在这里实现深拷贝（前提是 T 类型实现了赋值运算符重载的深拷贝），故并无大碍。

insert 是具有返回值的，且返回插入位置更新后的的迭代器。原因是在 insert 后可能进行扩容操作，会导致迭代器失效。返回值的意义在于可以使接收对象更新迭代器，防止迭代器失效。



##### push_back

尾插即可直接复用 insert。

```cpp
void push_back(const T& val = T())
{
insert(finish, 1, val);
}
```



##### erase

删除采用了迭代器范围删除，对单个迭代器的位置删除也可直接复用迭代器范围删除，故仅实现该版本。同样，对于形参不使用 const 迭代器，这仅仅是为了方便理解。erase 也返回了删除位置的迭代器，但仍然是被删除数据（或区间）的下一个位置，这样返回在 VS 中是有意义的，因为它会被认为是有效的迭代器，在 g++ 中或许意义不大。（但如果不排除有的编译器 erase 操作触发了缩容，返回迭代器位置便是有意义的）

```cpp
iterator erase(iterator first, iterator last)
{
    assert(first < last && first >= start && first <= finish && last >= start && last <= finish);

    size_t n = (size_t)(finish - last); //计算需要移动的数据个数
    size_t distance = (size_t)(last - first); //计算需要移动的距离
    for (size_t i = 0; i < n; ++i) //移动并覆盖数据
    {
        *(last - distance + i) = *(last + i);
    }

    finish -= distance;

    return first;
}
```



##### pop_back

尾删复用 erase 即可。

```cpp
void pop_back()
{
    erase(end() - 1, end());
}
```



##### clear

删除所有数据修正 finish 位置即可，在本篇实现中不包括缩容。

```cpp
void clear()
{
    finish = start;
}
```

---

### 整体实现

```cpp
#define _CRT_SECURE_NO_WARNINGS 1
#pragma once

#include <iostream>
#include <assert.h>

namespace Thepale
{
	template<typename T>
	class vector
	{
	public:
		typedef T* iterator;
		typedef const T* const_iterator;
		typedef T& reference;
		typedef const T& const_reference;


	private:
		iterator start;
		iterator finish;
		iterator end_of_storage;

	public:
		void swap(const vector<T>& v)
		{
			std::swap(start, v.start);
			std::swap(finish, v.finish);
			std::swap(end_of_storage, v.end_of_storage);
		}

		//构造函数
		vector()
			:start(nullptr)
			,finish(nullptr)
			,end_of_storage(nullptr)
		{}

		vector(size_t n, const T& val)
			:start(nullptr)
			,finish(nullptr)
			,end_of_storage(nullptr)
		{
			resize(n, val);
		}

		vector(int n, const T& val) //防止 InputIterator 抢走类似于 (10, 1) 的初始化情况
			:start(nullptr)
			, finish(nullptr)
			, end_of_storage(nullptr)
		{
			resize(n, val);
		}

		template<typename InputIterator>
		vector(InputIterator first, InputIterator last)
			:start(nullptr)
			, finish(nullptr)
			, end_of_storage(nullptr)
		{
			while (first != last)
			{
				push_back(*first);
				++first;
			}
		}

		//拷贝构造函数
		vector(const vector<T>& v)
			:start(nullptr)
			,finish(nullptr)
			,end_of_storage(nullptr)
		{
			size_t _size = v.size();
			reserve(_size);

			for (auto& e : v)
			{
				push_back(e);
			}
		}

		//析构函数
		~vector()
		{
			delete[] start;

			start = nullptr;
			finish = nullptr;
			end_of_storage = nullptr;
		}

		//赋值运算符重载
		vector<T>& operator=(vector<T> v)
		{
			swap(v);

			return *this;
		}


		//Iterators
		iterator begin()
		{
			return start;
		}

		iterator end()
		{
			return finish;
		}

		const_iterator begin() const
		{
			return start;
		}

		const_iterator end() const
		{
			return finish;
		}

		//Capacity
		size_t size() const
		{
			return finish - start;
		}

		size_t capacity() const
		{
			return end_of_storage - start;
		}

		bool empty() const
		{
			return size();
		}

		void reserve(size_t new_capacity)
		{
			if (new_capacity > capacity())
			{
				size_t _size = size();

				T* tmp = new T[new_capacity];

				if (start != nullptr)
				{
					//delete 后自定义类型也会删除，这样会导致扩容后仍然在管理删除后的空间，这里必须深拷贝
					for (size_t i = 0; i < _size; ++i)
					{
						tmp[i] = start[i];
					}

					delete[] start;
				}

				start = tmp;
				finish = start + _size;
				end_of_storage = start + new_capacity;
			}
		}

		void resize(size_t n, const T& val = T())
		{
			size_t _size = size();

			if (n < _size)
			{
				finish = start + n;
			}
			else
			{
				reserve(n);

				for (size_t i = 0; i < n - _size; ++i)
				{
					*(finish) = val;
					++finish;
				}
			}
		}

		//Element Access
		reference operator[](size_t pos)
		{
			assert(pos < size());

			return start[pos];
		}

		const_reference operator[](size_t pos) const
		{
			assert(pos < size());

			return start[pos];
		}

		//Modifiers
		iterator insert(iterator pos, size_t n, const T& val = T())
		{
			assert(pos >= start && pos <= finish);

			size_t _capacity = capacity();
			size_t _size = size();
			size_t fix = pos - start;

			if (n > _capacity - _size)
			{
				reserve(_capacity == 0 ? 4 : _capacity + n); //reserve 后 迭代器位置均改变
				pos = start + fix; //修正迭代器失效
			}

			for (size_t i = 0; i < (size_t)(finish - pos); ++i)
			{
				*(finish + n - i - 1) = *(finish - i - 1);
			}

			for (size_t i = 0; i < n; ++i)
			{
				*(pos + i) = val;
			}

			finish += n;

			return pos;
		}

		void push_back(const T& val = T())
		{
			insert(finish, 1, val);
		}

		iterator erase(iterator first, iterator last)
		{
			assert(first < last && first >= start && first <= finish && last >= start && last <= finish);

			size_t n = (size_t)(finish - last);
			size_t distance = (size_t)(last - first);
			for (size_t i = 0; i < n; ++i)
			{
				*(last - distance + i) = *(last + i);
			}

			finish -= distance;

			return first;
		}

		void pop_back()
		{
			erase(end() - 1, end());
		}

		void clear()
		{
			finish = start;
		}
        
        //Non-Member Function Overloads
        void swap(const vector<T>& v)
        {
            std::swap(start, v.start);
            std::swap(finish, v.finish);
            std::swap(end_of_storage, v.end_of_storage);
        }
	};
}

```

以下为测试代码（非严谨测试）：

```cpp
#include "vector.h"
using namespace std;
#include <string>
#include <vector>


int main()
{
	//构造函数测试
	cout << "构造函数测试" << endl;
	Thepale::vector<int> e1; for (auto& e : e1) { cout << e << " "; } cout << endl;
	Thepale::vector<int> e2(4, 520); for (auto& e : e2) { cout << e << " "; } cout << endl;

	Thepale::vector<std::string> e3(4, "ILOVEYOU"); for (auto& e : e3) { cout << e << " "; } cout << endl;

	//拷贝构造函数测试
	cout << "拷贝构造函数测试" << endl;
	Thepale::vector<std::string> e4(e3); for (auto& e : e4) { cout << e << " "; } cout << endl;

	//迭代器测试略

	//reserve、capacity 测试
	cout << "reserve、capacity 测试" << endl;
	Thepale::vector<std::string> e5;
	e5.reserve(1024); cout << e5.capacity() << endl;

	//resize、size 测试
	cout << "resize、size 测试" << endl;
	Thepale::vector<int> e6;
	e6.resize(10, 666); cout << e6.size() << endl; for (auto& e : e6) { cout << e << " "; } cout << endl;

	//empty 测试
	cout << "empty 测试" << endl;
	Thepale::vector<std::string> e7; cout << e7.empty() << endl;
	e7.push_back("I'M PROGRAMMER"); cout << e7.empty() << endl;

	//operator[] 测试
	cout << "operator[] 测试" << endl;
	Thepale::vector<std::string> e8(10, "DontGo");
	e8[5] = "GOOOO!!!!";
	for (size_t i = 0; i < e8.size(); ++i) { cout << e8[i] << " "; } cout << endl;

	//insert 测试
	cout << "insert 测试" << endl;
	Thepale::vector<std::string> e9(5, "WTF");
	e9.insert(e9.begin(), 2, "DontSay"); for (auto& e : e9) { cout << e << " "; } cout << endl;
	e9.insert(e9.end(), 2, "DoIt"); for (auto& e : e9) { cout << e << " "; } cout << endl;
	e9.insert(e9.begin() + 4, 4, "666"); for (auto& e : e9) { cout << e << " "; } cout << endl;

	//push_back 测试
	cout << "push_back 测试" << endl;
	Thepale::vector<std::string> e10(6, "PEACE");
	e10.push_back("Forever"); for (auto& e : e10) { cout << e << " "; } cout << endl;

	//erase 测试
	cout << "erase 测试" << endl;
	Thepale::vector<std::string> e11(8, "Love");
	e11.erase(e11.begin() + 2, e11.begin() + 4); for (auto& e : e11) { cout << e << " "; } cout << endl;

	//pop_back 测试
	cout << "pop_back 测试" << endl;
	Thepale::vector<std::string> e12;
	e12.push_back("I");
	e12.push_back("Will");
	e12.push_back("Never");
	e12.push_back("Back");
	e12.pop_back(); for (auto& e : e12) { cout << e << " "; } cout << endl;
	e12.pop_back(); for (auto& e : e12) { cout << e << " "; } cout << endl;

	return 0;
}
```



---

## 补充说明

* 在构造函数、拷贝构造函数等部分，都需要将三个成员变量置空进行初始化，这一操作可以用 C++11 中的类内初始化代替。

  ```cpp
  namespace Thepale
  {
  	template<typename T>
  	class vector
  	{
  	private:
  		iterator start = nullptr;
  		iterator finish = nullptr;
  		iterator end_of_storage = nullptr;
      };
  }
  ```

  这样做只是实现了代码上的省略，并没有实质性的逻辑变化和简略。

* 本篇没有实现 vector 的比较运算符重载，一般认为 vector 的比较是没有实际意义的。但如果使用排序，sort 也会自动比较 vector 内的元素，内置类型的比较行为是已经定义好了的，对于自定义类型，编译器会尝试使用该类型的默认比较运算符，这可能会根据类型的成员变量等来执行比较。（仅代表我的浅薄理解，可能出现错误）



[^类模板和模板类]: 这两者通常用于指代同一对象，但类模板更强调的是原始模板本身，而模板类强调的是基于模板而生成的类。

[^迭代器失效]: 在第一种情况中，以 insert 举例，在 insert 时有概率会导致扩容操作的发生，异地扩容将导致原迭代器失效，即指针指向被释放的空间，这一操作是十分危险的；在第二中情况中，以 erase 举例（单个数据删除举例），在 erase 后，通常需要删除的迭代器位置已经被删除，这时迭代器指向的是删除数据的下一个位置（不考虑有编译器实现 erase 后缩容的情况），这种情况下没有发生指针失效，但仍然认为是迭代器失效，因为它失去了它存在的意义（它的意义是指向被删除的元素，元素删除后它就没有意义了）。故不要出现先记录迭代器位置，insert 或 erase 后再使用该迭代器位置的情况，可以采用接收 insert 或 erase 的返回值更新迭代器，以防止迭代器失效。当然，不同编译器对迭代器失效的处理方式不同，例如 VS 会强制检查，而 g++ 不会。

