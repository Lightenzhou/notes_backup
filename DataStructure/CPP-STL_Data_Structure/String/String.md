# String

---

## 概述

string 是对字符串进行一系列操作的结构，由于 string 类中有各种编码类格式，例如：u16string，u32string，wstring 等，本篇将不会一一阐述，以 basic_string 为主进行模拟实现。由于 string 类也有部分功能存在一定程度的冗余，且些许功能使用次数极少，故仅针对频繁且有实际意义的成员函数提供实现。由于此部分由 C++ 进行实现，故在其中将进行一些格式变更。

---

## 类的实现

### 成员变量

管理字符串在底层依然是依靠 char* 指针，且辅佐以大小和容量，方便管理扩容等操作。故成员变量一共有以下三种：

```cpp
namespace Thepale
{
    class string
    {
    public:
        typedef char* iterator;
        typedef const char* const_iterator;
        
    private:
        char* _str;
        size_t _size;
        size_t _capacity;
    };
}
```

为了和 std 中的 string 类区分，故加上命名空间隔离（迭代器会在后续介绍）。

---

### 成员函数

#### 构造函数

 一般而言，string 类多以无参初始化或字符串初始化，故此处仅考虑这两种情况。当 string 以无参构造时，并非将指针置为 nullptr，这会给后续管理带来很大麻烦，例如插入数据需要额外考虑 nullptr 的情况，拷贝数据也需要考虑 nullptr 的情况等等，后续自然会见证。故就算是无参初始化，也默认附带一个 '\0'，这样很大程度上能进行统一管理。

```cpp
string(const char* str = "")
    :_str(nullptr)
    , _size(strlen(str))
    , _capacity(_size + 1)
{
    _str = new char[_capacity];
    strcpy(_str, str);
}
```

在本篇实现的代码中，\_capacity 表示的是实际开辟的容量，\_size表示的是字符串的实际大小（不包括结尾的 '\0'，这里仅指结尾，若是中间的 '\0' 是会被计算的）。在针对空间的开辟中，使用 strlen 和 strcpy，显然如果所传递的字符串在中间有 '\0' 出现，它将会被解析为字符串的结束，这里不做特殊处理，因为是直接的字符串传递，若遇 '\0' 则释义就是结束。若对此点有疑惑，请见：[^拷贝构造函数]。

这里恰好可以同时解决两种情况，若无参构造则传递 '\0'，在初始化列表中也可以直接 new 空间，但个人更习惯先给 nullptr，依据个人喜好即可。而这里需要注意 \_capacity 是依据 \_size 初始化的，这就表明在声明成员变量时 \_capacity 必须在 \_size 后被声明，必须注意这一点。



#### 拷贝构造函数

```cpp
string(const string& str)
{
    _size = str._size;
    _capacity = str._capacity;

    _str = new char[_capacity];
    memcpy(_str, str._str, str._capacity);
}
```

拷贝构造需要进行的是深拷贝，不能让两个指针管理同一块空间，故这里必须额外开辟空间。这里必须使用 memcpy，若在字符串 _size 为 3，字符串为：`A \0 B \0`，则中间的 '\0' 必须拷贝，若使用 strcpy 则会在遇到 '\0' 时停止，因为无法避免用户插入 '\0' 的情况。

额外手动开辟空间是可行的，但也可以复用构造函数：

```cpp
void swap(string& str)
{
    std::swap(_str, str._str);
    std::swap(_size, str._size);
    std::swap(_capacity, str._capacity);
}

string(const string& str)
    :_str(nullptr)
    ,_size(0)
    ,_capacity(0)
{

    string tmp(str._str);
	swap(tmp);
}
```

可以直接利用构造函数构造和原 string 字符串数据一样但空间不同的对象，（实质上是构造函数帮忙完成了手动开辟空间），然后交换它们的内容，将 tmp 的内容交给当前对象管理，而 tmp 在拷贝构造结束时又会进行销毁，且析构 nullptr 是无危害的，便借助构造函数完成了拷贝构造函数的功能。

但这种写法有两点需要注意：其中一点是，在进行拷贝构造时，当前对象的空间未被初始化，很有可能出现随机值，当交换后，随机空间被释放则会报错（在赋值运算符重载中会再次说明）；还有一点是若字符串中在中间出现 '\0'，使用这种写法在借助构造函数时传递的是字符串，故会截断字符串，导致拷贝不完全，出于严谨应使用第一种写法。



#### 析构函数

析构字符串即析构动态开辟的空间是十分简单的，不过多赘述：

```cpp
~string()
{
    delete[] _str;
    _str = nullptr;

    _size = 0;
    _capacity = 0;
}
```



#### 赋值运算符重载

赋值一定是针对两个已存在的对象的，若仔细划分，则会存在三种情况：赋值过来的空间比原空间小，赋值过来的空间和原空间相等，赋值过来的空间比原空间大。若比原空间小，一般不会执行缩容操作，故必然有一部分空间会被浪费；若和原空间相等，则需要替换其内容；若比原来的空间大，则需要执行扩容操作后再替换其内容。上述操作实则可以直接简化为释放原空间后再次申请空间完成赋值，则省略了冗余的判断和空间浪费的问题：

```cpp
 string& operator=(const string& str)
{
    if (this != &str) //防止自身拷贝的情况导致不必要的消耗
    {
        char* tmp = new char[str._capacity];
        memcpy(tmp, str._str, str._capacity);

        delete[] _str;
        _str = tmp;

        _size = str._size;
        _capacity = str._capacity;
    }

    return *this;
}
```

只需要开辟一个新的 tmp 来存放 str 中的内容，再将原空间释放，重新管理 tmp 的空间即可。当然，这一过程不仅仅可以复用构造函数，也可以复用拷贝构造，一般而言复用拷贝构造，可直接在传参部分完成拷贝：

```cpp
string& operator=(string str) //在传参时不使用引用，直接完成拷贝构造创建新对象
{
    if (this != &str)
    {
		swap(str);
    }

    return *this;
}
```

在传参时完成拷贝新对象，直接将拷贝的对象资源交给当前对象管理，将需要释放的资源交换给了 str，在函数结束时 str 又会自动销毁，则顺带销毁了原本需要释放的当前对象的空间，一举两得。

一般而言，部分编译器（以 VS 为例）会解决拷贝构造函数中未初始化当前对象的问题，就算不在拷贝构造函数中写明初始化列表，成员变量依然被初始化，重要的是指针被初始化为 nullptr，这将不会引起释放时的问题。但如果出现以下情况：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/String-p1.png)

两个对象执行赋值操作，在 e2 拷贝构造 str 时，str（红框）会出现随机值，此时将 str 和构造函数创建的 tmp 对象进行空间交换，拷贝构造函数执行完成后会释放 tmp 空间（即 str 未初始化的随机空间），会产生报错。故在复用构造函数实现拷贝构造函数时，初始化列表是必须的。

---

#### Iterators

迭代器只实现普通和 const 版本，一般而言反向迭代器和 const 反向迭代器的应用场景比较有限，实现较复杂，且易被替代，实现的意义不大。由于在 string 类中，迭代器实则就是对指针的直接封装，所以实现也非常简单，故做了合并处理。需要注意迭代器区间是左闭右开。这里同时实现了 cbegin 和 cend，它们是 C++ 11 中的内容，我认为 cbegin 和 cend 存在的意义是无论是否为 const 对象，都可以保证获取的迭代器为 const，无法通过迭代器更改对象内容。它或许只是为了方便更明确的表达意图，除了本篇出现外，在后续的容器模拟实现中，大概率不会再出现它们，除非有特殊场景。（后续仅使用非 const 版本的 begin 与 end，const 版本的 cbegin 与 cend 做一些演示）

```cpp
typedef char* iterator;
typedef const char* const_iterator;
```

```cpp
iterator begin()
{
    return _str;
}

iterator end()
{
    return _str + _size; 
}

const_iterator begin() const
{
    return _str;
}

const_iterator end() const
{
    return _str + _size; 
}

const_iterator cbegin()
{
    return _str;
}

const_iterator cend()
{
    return _str + _size;
}

const_iterator cbegin() const
{
    return _str;
}

const_iterator cend() const
{
    return _str + _size;
}
```

---

#### Capacity

在这里不会实现 max_size，在各个编译器中它的值都是不确定的，这取决于编译器行为。同时也不会实现 length，它和 size 的含义实际是一样的，由于 string 早于 STL 问世，所以才会有 length 的出现，size 做到了和 STL 标准一致。shrink_to_fit 由于不考虑缩容操作，它不仅会产生性能开销且存在释放的不确定性（缩容是一个请求而非强制要求）。

##### reserve

reserve 一般主要负责空间的开辟，后续有多个函数可以对 reserve 进行复用完成扩容操作。

```cpp
void reserve(size_t new_capacity)
{
    if (new_capacity > _capacity) //检查新容量是否大于当前容量，一般不进行缩容操作
    {
        char* tmp = new char[new_capacity];

        memcpy(tmp, _str, _size + 1);
        delete[] _str;

        _str = tmp;

        _capacity = new_capacity;
    }
}
```

这里先开辟了空间，转移数据后就释放了原空间，因为 C++ 并没有类似 C 语言中 realloc 的操作，只能释放后重新开辟。且这里只能使用 memcpy，若使用 strcpy  在字符串中间出现 '\0' 时则会导致数据拷贝时丢失。这里仅需拷贝 _size + 1 的空间，而非大小为 _capacity 的所有空间。



##### resize

resize 在完成空间开辟的同时也会实现赋值操作，一般为 n 个 char，若不指明 char 则初始化为 '\0' ：

```cpp
void resize(size_t n, char c = '\0')
{
    if (n < _size)
    {
        _size = n;
        _str[_size] = '\0';
    }
    else
    {
        reserve(n + 1); //两种情况一起处理

        for (size_t i = _size; i < n; ++i)
        {
            _str[i] = c;
        }

        _size = n;
        _str[_size] = '\0';
    }
}
```

对于 n < _size 的情况，一般认为是删除数据，仅需添加 '\0' 表示字符串结束即可。（这里并不会导致进行拷贝时拷贝 '\0' 后的多余数据，因为 _size 也改变了）等于和大于的情况可以一起处理，等于情况没有新开的空间，所以无需处理，大于的情况将新开的空间赋值为对应 char 即可。



##### size

需要返回 size 和 capacity 的情况都十分简单，不过多赘述，不过别忘了 const 对象：

```cpp
size_t size() const
{
    return _size;
}
```



##### capacity

```cpp
size_t capacity() const
{
    return _capacity;
}
```



##### empty

判空操作是必要的，它在很多地方都能派上用场，而 string 的判空也是极为简单：

```cpp
bool empty() const
{
    return _size == 0;
}
```



##### clear

清除操作的目的在于清空所有元素，当然它也不会涉及缩容：

```cpp
void clear()
{
    _str[0] = '\0';
    _size = 0;
}
```

始终注意字符串的实际大小是由 _size 控制的，所以即使不清除此字符串 '\0' 后的内容，在计算机看来它依然是结尾的 '\0' 而非中间的。

---

#### Element Access

对于元素的访问仅实现 operator[]，它和 at 的差别在于 at 是抛异常而 operator[] 是断言检查。front 和 back 的实现意义真的微乎其微，有这个功夫为何不直接使用 str[0] 和 str[str.size() - 1] 呢？

##### operator[]

这样访问到的元素是可以进行修改的（如果不是 const 对象的话），故需要返回引用：

```cpp
char& operator[](size_t pos)
{
    assert(pos < _size);

    return _str[pos];
}

const char& operator[](size_t pos) const 
{
    assert(pos < _size);

    return _str[pos];
}
```

---

#### Modifiers

assign 用于对字符串进行删除重写，即全部替换；replace 实现对字符串的部分替换。它们的工作更像是对 insert、clear、erase 的进一步封装，这里不予实现。

##### push_back

尾插负责完成单个字符的插入工作：

```cpp
void push_back(char c)
{
    if (_size + 1 >= _capacity)
    {
        reserve(_capacity * 2);
    }

    _str[_size++] = c;
    _str[_size] = '\0';
}
```



##### append

append 负责完成字符串的拼接：

```cpp
void append(const char* str)
{
    size_t len = strlen(str);

    if (_size + len >= _capacity)
    {
        reserve(len + _size + 1);
    }

    strcpy(_str + _size, str);
    _size = _size + len;
}
```

这里容量不够的处理操作是将容量扩大到刚好能装下，当然不同的编译器会有不同的处理方式，也可能会扩的比当前空间大（例如 VS）。strcat 作为 C 语言的字符串操作函数在此处复用让我觉得有些不合适，它强调的是字符串拼接，但如果出现字符串中有 '\0' 的情况则会导致后续内容被拼接的字符串覆盖，但若加到 _size 后的地址空间又失去了拼接的意义，故采用 strcpy 将需要拼接的内容拷贝到后面即可（由于 append 接收到的是原生字符串，故可以不采用 memcpy，将 '\0' 解释为字符串结束即可）。



##### insert

插入操作的重载也很多，这里主要实现两种以完成大部分功能（插入 n 个字符或字符串）：

```cpp
void insert(size_t pos, size_t n, char c)
{
    assert(pos <= _size);

    if (_size + n >= _capacity)
    {
        reserve(_size + n + 1);
    }

    for (size_t i = pos; i < _size + 1 - pos; ++i) //挪动数据
    {
        _str[_size + n - i] = _str[_size - i];
    }

    for (size_t i = 0; i < n; ++i) //完成插入
    {
        _str[pos + i] = c;
    }

    _size += n;
}
```

```cpp
void insert(size_t pos, const char* str)
{
    assert(pos <= _size);

    size_t len = strlen(str);
    if (_size + len >= _capacity)
    {
        reserve(_size + len + 1);
    }

    for (size_t i = 0; i < _size + 1 - pos; ++i) //挪动数据
    {
        _str[_size + len - i] = _str[_size - i];
    }

    strncpy(_str + pos, str, len); //完成插入
    _size += len;
}
```

因为 str 是原生字符串，故可以不使用 memcpy。



##### operator+=

这是 string 类中及其方便的一个运算符重载，它可以直接实现对字符串的尾插（尾插字符或字符串），事先实现好的 push_back 和 append 也将在此时被复用：

```cpp
string& operator+=(char c)
{
    push_back(c);

    return *this;
}
```

```cpp
string& operator+=(const char* str)
{
    append(str);

    return *this;
}
```

考虑连续加等的情况，请使用引用返回。

    string& operator+=(const string& str)
    {
        append(str._str);
    
        return *this;
    }

当然，加一个对象也是允许的。



##### pop_back

尾删负责完成单个字符的删除工作：

```cpp
void pop_back()
{
    assert(_size > 0);

    --_size;
}
```



##### erase

删除的重载也比较多，删除对应区间是比较常用且覆盖较广的一个，仅实现这一个：

这里涉及到了 npos，它是无符号形式的 -1（在 x86 环境下，它是 4294967295，整型的最大值，一般用于表示字符串的结尾）

```cpp
size_t string::npos = -1; //这里是定义部分，请自行于类中声明
```

```cpp
void erase(size_t pos, size_t len = npos)
{
    assert(pos <= _size);

    if (len == npos || len > _size - pos) //pos 后全部删除的情况
    {
        _str[pos] = 0;
        _size = pos;
    }
    else
    {
        for (size_t i = 0; i <= _size - pos - len; ++i) //删除中间的情况
        {
            _str[pos + i] = _str[pos + len + i];
        }

        _size -= len;
    }
}
```

---

#### String Operations

在这里仍然会挑选使用频率高、覆盖范围大的函数进行实现。例如我认为 find_first_of、find_last_of 等函数大可用 find 代替等。

##### c_str

它用于直接取出对象所管理的指针，由名称可知这个接口更多程度是为了契合 C 语言，在 vector 中提供了 data() 函数来完成这一点，尽管 string 中也提供了 data()，但 c_str 或许仍然是更为公认的表示方法。

```cpp
char* c_str() const
{
    return _str;
}
```



##### find

find 分为查找字符和查找字符串两类：

```cpp
size_t find(char c, size_t pos = 0) const
{
    assert(pos < _size);

    for (size_t i = pos; i < _size; ++i)
    {
        if (_str[i] == c)
        {
            return i;
        }
    }

    return npos; //表示未查找到字符串
}
```

```cpp
size_t find(const char* str, size_t pos = 0) const
{
    assert(pos < _size);

    const char* p = strstr(_str + pos, str); //字符串的匹配正好使用 strstr
    if (p != nullptr)
    {
        return p - _str;
    }

    return npos;
}
```

可见在官方所提供的标准中，find 返回的仍然是下标位置，而非迭代器，迭代器在 string 中不太常用，所以在之前的构造函数中，我思索一番依然没有写出迭代器构造的版本。对于大多数程序员而言，直接通过指针和下标索引等操作字符串是更方便舒适的。



##### substr

substr 的功能是取出子串：

```cpp
string substr(size_t pos = 0, size_t len = npos)
{
    assert(pos < _size);

    string ret;
    if (len > _size - pos) 
    {
        len = _size - pos;
    }

    ret.reserve(len + 1);
    ret._size = len;

    memcpy(ret._str, _str + pos, len); //不能使用 strcpy，原因已说明

    (ret._str)[ret._size] = '\0'; //这一步要注意（单纯因为我不小心写错了调试了很久）

    return ret;
}
```

---

#### Non-Member Function Overloads

##### swap

用于对象的交换。

```cpp
void swap(string& str)
{
    std::swap(_str, str._str);
    std::swap(_size, str._size);
    std::swap(_capacity, str._capacity);
}
```



##### Relational Operators

字符串比较时，使用 C 语言的 strcmp 无法应对字符串中间出现 '\0' 的情况，这里务必自己实现。

```cpp
bool operator<(const string& str) const
{
    int ret = memcmp(_str, str._str, _size < str._size ? _size : str._size);

    return ret == 0 ? _size < str._size : ret < 0;
}

bool operator==(const string& str) const
{
    return _size == str._size && memcmp(_str, str._str, _size) == 0;
}

bool operator<=(const string& str) const
{
    return *this < str || *this == str;
}

bool operator>(const string& str) const
{
    return !(*this <= str);
}

bool operator>=(const string& str) const
{
    return !(*this < str);
}

bool operator!=(const string& str) const
{
    return !(*this == str);
}
```



##### operator<<

由于可以通过对象直接取到 size 和 capacity，故不必为友元函数。

```cpp
std::ostream& operator<<(std::ostream& out, const string& str)
{
    for (size_t i = 0; i < str.size(); ++i)
    {
        std::cout << str[i];
    }

    return out;
}
```

这里并没有通过 c_str 直接输出，原因也是为了防止中间出现 '\0' 的情况，会导致字符串提前截至，应严格按照 _size 的大小输出。



##### operator>>

字符串的流插入重载并非如想象中简单，因为要输入的字符串长度是未知的，无法通过提前开空间储存字符串，所以一般采用逐个字符读取后尾插的方式完成：

```cpp
std::istream& operator>>(std::istream& in, string& str)
{
    str.clear(); //插入前会清空字符串的内容

    char c;
    c = in.get();

    //清理字符串前的空格和换行
    while (c == ' ' || c == '\n')
    {
        c = in.get(); 
    }

    size_t i = 0;
    while (c != ' ' && c != '\n')
    {
        str += c;
        c = in.get();   
    }

    return in;
}
```

清理字符串前的空格和换行时，不会导致进入 while 时 c 为空格或换行，因为 c 接收到的永远是清除时 while 判断的后一个字符。

但进行逐个字符插入时，str 将进行多次扩容操作带来额外的消耗，故可以添加一个固定的 buffer 数组来事先储存输入的字符串，然后一次推入 buffer 数组大小的数据量，以减少扩容次数：

```cpp
std::istream& operator>>(std::istream& in, string& str)
{
    str.clear();

    char c;
    c = in.get();

    while (c == ' ' || c == '\n')
    {
        c = in.get();
    }

    char buffer[128] = { 0 };

    size_t i = 0;
    while (c != ' ' && c != '\n')
    {
        buffer[i++] = c;
        c = in.get();

        if (i == 127)
        {
            buffer[i] = '\0';
            str += buffer;
            i = 0;
        }
    }

    if (i != 0)
    {
        buffer[i] = '\0';
        str += buffer;
    }

    return in;
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
	class string
	{
    public:
        static const size_t npos;

        typedef char* iterator;
        typedef const char* const_iterator;
        
	private:
		char* _str;
		size_t _size;
		size_t _capacity;

	public:
        //构造函数
		string(const char* str = "")
			:_str(nullptr)
			, _size(strlen(str))
			, _capacity(_size + 1)
		{
			_str = new char[_capacity];
			strcpy(_str, str);
		}
		
        //拷贝构造函数
		string(const string& str)
			:_str(nullptr)
			, _size(0)
			, _capacity(0)
		{

			string tmp(str._str);
			swap(tmp);
		}
		
        //析构函数
		~string()
		{
			delete[] _str;
			_str = nullptr;

			_size = 0;
			_capacity = 0;
		}

        //赋值运算符重载
		string& operator=(string str)
		{
			if (this != &str)
			{
				swap(str);
			}

			return *this;
		}
		
        //Iterators
		iterator begin()
		{
			return _str;
		}

		iterator end()
		{
			return _str + _size;
		}

		const_iterator cbegin() const
		{
			return _str;
		}

		const_iterator cend() const
		{
			return _str + _size;
		}

        //Capacity
		void reserve(size_t new_capacity)
		{
			if (new_capacity > _capacity)
			{
				char* tmp = new char[new_capacity];

				memcpy(tmp, _str, _size + 1);
				delete[] _str;

				_str = tmp;

				_capacity = new_capacity;
			}
		}

		void resize(size_t n, char c = '\0')
		{
			if (n < _size)
			{
				_size = n;
				_str[_size] = '\0';
			}
			else
			{
				reserve(n + 1);

				for (size_t i = _size; i < n; ++i)
				{
					_str[i] = c;
				}

				_size = n;
				_str[_size] = '\0';
			}
		}

		size_t size() const
		{
			return _size;
		}

		size_t capacity() const
		{
			return _capacity;
		}

		bool empty() const
		{
			return _size == 0;
		}

		void clear()
		{
			_str[0] = '\0';
			_size = 0;
		}

        //Element Access
		char& operator[](size_t pos)
		{
			assert(pos < _size);

			return _str[pos];
		}

		const char& operator[](size_t pos) const
		{
			assert(pos < _size);

			return _str[pos];
		}
		
        //Modifiers
		void push_back(char c)
		{
			if (_size + 1 >= _capacity)
			{
				reserve(_capacity * 2);
			}

			_str[_size++] = c;
			_str[_size] = '\0';
		}

		void append(const char* str)
		{
			size_t len = strlen(str);

			if (len + _size >= _capacity)
			{
				reserve(len + _size + 1);
			}

			strcpy(_str + _size, str);
			_size = _size + len;
		}

		void insert(size_t pos, size_t n, char c)
		{
			assert(pos <= _size);

			if (_size + n >= _capacity)
			{
				reserve(_size + n + 1);
			}

			for (size_t i = pos; i < _size + 1 - pos; ++i)
			{
				_str[_size + n - i] = _str[_size - i];
			}

			for (size_t i = 0; i < n; ++i)
			{
				_str[pos + i] = c;
			}

			_size += n;
		}

		void insert(size_t pos, const char* str)
		{
			assert(pos <= _size);

			size_t len = strlen(str);
			if (_size + len >= _capacity)
			{
				reserve(_size + len + 1);
			}

			for (size_t i = 0; i < _size + 1 - pos; ++i)
			{
				_str[_size + len - i] = _str[_size - i];
			}

			strncpy(_str + pos, str, len);
			_size += len;
		}

		string& operator+=(char c)
		{
			push_back(c);

			return *this;
		}

		string& operator+=(const char* str)
		{
			append(str);

			return *this;
		}

		string& operator+=(const string& str)
		{
			append(str._str);

			return *this;
		}

		void pop_back()
		{
			assert(_size > 0);

			--_size;
		}


		void erase(size_t pos, size_t len = npos)
		{
			assert(pos <= _size);

			if (len == npos || len > _size - pos)
			{
				_str[pos] = 0;
				_size = pos;
			}
			else
			{
				for (size_t i = 0; i <= _size - pos - len; ++i)
				{
					_str[pos + i] = _str[pos + len + i];
				}

				_size -= len;
			}
		}
		
        //String Operations
		char* c_str() const
		{
			return _str;
		}

		size_t find(char c, size_t pos = 0) const
		{
			assert(pos < _size);

			for (size_t i = pos; i < _size; ++i)
			{
				if (_str[i] == c)
				{
					return i;
				}
			}

			return npos;
		}

		size_t find(const char* str, size_t pos = 0) const
		{
			assert(pos < _size);

			const char* p = strstr(_str + pos, str);
			if (p != nullptr)
			{
				return p - _str;
			}

			return npos;
		}

		string substr(size_t pos = 0, size_t len = npos)
		{
			assert(pos < _size);

			string ret;
			if (len > _size - pos)
			{
				len = _size - pos;
			}

			ret.reserve(len + 1);
			ret._size = len;

			memcpy(ret._str, _str + pos, len);

			(ret._str)[ret._size] = '\0';

			return ret;
		}
		
        //No-Member Function Overloads
        void swap(string& str)
        {
            std::swap(_str, str._str);
            std::swap(_size, str._size);
            std::swap(_capacity, str._capacity);
        }
        
		bool operator<(const string& str) const
		{
			int ret = memcmp(_str, str._str, _size < str._size ? _size : str._size);

			return ret == 0 ? _size < str._size : ret < 0;
		}

		bool operator==(const string& str) const
		{
			return _size == str._size && memcmp(_str, str._str, _size) == 0;
		}

		bool operator<=(const string& str) const
		{
			return *this < str || *this == str;
		}

		bool operator>(const string& str) const
		{
			return !(*this <= str);
		}

		bool operator>=(const string& str) const
		{
			return !(*this < str);
		}

		bool operator!=(const string& str) const
		{
			return !(*this == str);
		}
	};

	std::ostream& operator<<(std::ostream& out, const string& str)
	{
		for (size_t i = 0; i < str.size(); ++i)
		{
			std::cout << str[i];
		}

		return out;
	}

	std::istream& operator>>(std::istream& in, string& str)
	{
		str.clear();

		char c;
		c = in.get();

		while (c == ' ' || c == '\n')
		{
			c = in.get();
		}

		char buffer[128] = { 0 };

		size_t i = 0;
		while (c != ' ' && c != '\n')
		{
			buffer[i++] = c;
			c = in.get();

			if (i == 127)
			{
				buffer[i] = '\0';
				str += buffer;
				i = 0;
			}
		}

		if (i != 0)
		{
			buffer[i] = '\0';
			str += buffer;
		}

		return in;
	}

	const size_t string::npos = -1;

}
```

以下为测试代码（非严谨测试）：

```cpp
#include "string.h"

int main()
{
	//构造函数测试
	cout << "构造函数测试" << endl;
	Thepale::string e1; cout << e1 << endl;
	Thepale::string e2("Thepale 5201314"); cout << e2 << endl;

	//拷贝构造函数测试
	cout << "拷贝构造函数测试" << endl;
	Thepale::string e3(e2); cout << e3 << endl;

	//赋值运算符重载测试
	cout << "赋值运算符重载测试" << endl;
	Thepale::string e4("hello");
	e4 = e2; cout << e4 << endl;

	//迭代器测试
	cout << "迭代器测试" << endl;
	Thepale::string e5("I am a iterator tester");
	Thepale::string::iterator it = e5.begin();
	while (it != e5.end())
	{
		cout << *it << " ";
		++it;
	}
	cout << endl;
	const Thepale::string e6("I am a const_iterator tester");
	Thepale::string::const_iterator cit = e6.cbegin();
	while (cit != e6.cend())
	{
		cout << *cit << " ";
		++cit;
	}
	cout << endl;


	//reserve 测试
	cout << "reserve测试" << endl;
	Thepale::string e7("1"); cout << e7.size() << " " << e7.capacity() << endl;
	e7.reserve(100); cout << e7.size() << " " << e7.capacity() << endl;

	//resize 测试
	cout << "resize测试" << endl;
	Thepale::string e8("hello"); cout << e8 << " " <<  e7.size() << " " << e7.capacity() << endl;
	e8.resize(10, 'x'); cout << e8 << " " << e8.size() << " " << e8.capacity() << endl;

	//size capacity 测试略
	
	//empty 测试
	cout << "empty 测试" << endl;
	Thepale::string e9; cout << e9.empty() << endl;
	Thepale::string e10("1"); cout << e10.empty() << endl;

	//clear 测试
	cout << "clear 测试" << endl;
	Thepale::string e11("5201314"); cout << e11 << endl;
	e11.clear(); cout << e11 << endl;

	//operator[] 测试
	cout << "operator[] 测试" << endl;
	Thepale::string e12("Love is Gone");
	for (size_t i = 0; i < e12.size(); ++i)
	{
		cout << e12[i] << " ";
	}
	cout << endl;
	const Thepale::string e13("Love is Gone(const)");
	for (size_t i = 0; i < e13.size(); ++i)
	{
		cout << e13[i] << " ";
	}
	cout << endl;

	//push_back 测试
	cout << "push_back 测试" << endl;
	Thepale::string e14("hello");
	e14.push_back('6');
	e14.push_back('6');
	e14.push_back('6');
	e14.push_back('6');
	e14.push_back('6');
	e14.push_back('6');
	cout << e14 << endl;

	//append 测试
	cout << "append 测试" << endl;
	Thepale::string e15("hello");
	e15.append("world"); cout << e15 << endl;

	//insert 测试
	cout << "insert 测试" << endl;
	Thepale::string e16("whatever");
	//e16.insert(3, "MMMM");
	e16.insert(0, 10, 'O');
	e16.insert(e16.size(), "END");
	e16.insert(e16.size(), 20, 'T');
	cout << e16 << endl;

	//operator+= 测试略

	//pop_back 测试
	cout << "pop_back 测试" << endl;
	Thepale::string e17("AA"); cout << e17 << endl;
	e17.pop_back(); cout << e17 << endl;
	e17.pop_back(); cout << e17 << endl;

	//erase 测试
	cout << "erase 测试" << endl;
	Thepale::string e18("ILoveYouLikeTheJoker");
	e18.erase(2, 7); cout << e18 << endl;
	e18.erase(0, 10000); cout << e18 << endl;

	//c_str 测试略

	//find 测试
	cout << "find 测试" << endl;
	Thepale::string e19("ABCDEFGHIJKLMNOPQRSTUVWXYZ");
	size_t pos1 = e19.find("ABC"); cout << pos1 << endl;
	size_t pos2 = e19.find("JKM"); cout << pos2 << endl;
	size_t pos3 = e19.find('X'); cout << pos3 << endl;
	size_t pos4 = e19.find('E', 10); cout << pos4 << endl;

	//subser 测试
	cout << "subser 测试" << endl;
	Thepale::string e20("I am a substr tester");
	cout << e20.substr(0, 4) << endl;
	cout << e20.substr(7, 6) << endl;
	cout << e20.substr(0) << endl;
	cout << e20.substr(14) << endl;

	return 0;
}
```



---

## 补充说明

* string 类不属于类模板，它就是 basic_string，严格来说它不属于 STL，所以在容器中甚至都没有它的身影。在 STL 中类将会以类模板的形式展现，以兼容多种不同的数据类型。
* 在进行拷贝构造时，有些场景会出现写时拷贝。即先进行代价较小的浅拷贝，并使计数器自增，当需要对空间进行修改等操作时，检测计数器数值，若有多个对象同时管理，则再单独开辟空间并自减计数器。这种情况可以很好的应对只读不写的场景，减少了性能消耗。