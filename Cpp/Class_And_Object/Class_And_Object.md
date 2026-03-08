# Class_And_Object

---

## 类和对象概述

### 区分面向过程和面向对象

面向过程数据和操作是分离的，每一个操作都是独立的，这里拿学生举例：

```cpp
//以下是面向过程所对应的学生操作：
struct Student
{
    char name[24];
    int age;
    char sex;
};

void init_student(struct Student* stu, char* str, int age, char sex);
void input_age(struct Student* stu, int age);
void output_age(struct Student* stu);
```

而面向对象中数据和操作是被封装在类中的：（封装是对数据和操作数据的方法的有机结合，隐藏对象的属性和实现细节的一种做法，仅对外公开接口来和对象进行交互，封装本质上是一种更严格的管理）

```cpp
class Student
{
private:
    char name[24];
    int age;
    char sex;
    
public:
	Student(char* str, int age, char sex);
	void input_age(int age);
	void output_age();
};
```

在面向过程的示例中，我们使用了结构体和函数来描述学生的属性和对信息进行操作。结构体用于封装学生的属性，而函数则用于对学生信息进行操作。函数是对数据的操作过程的封装，数据和操作是分离的。面向过程编程强调按照指定的步骤逐步解决问题，代码的流程直观，从上到下逐步执行。

在面向对象的示例中，我们定义了一个类 `Student`，其中封装了学生的属性和操作学生信息的函数。类将数据（属性）和操作函数（方法）封装在一起，对象是类的实例，它拥有类定义的属性和方法。面向对象编程强调对象的概念和封装性，通过创建对象来实现操作。类提供了一种模板，允许我们根据需要创建多个对象，并且每个对象都具有自己的属性和行为，这更类似于现实生活中的对象交互。



### 类的声明与定义

```cpp
class Thepale; //类的声明
class Thepale //类的定义
{
private:
//...
    
public:
//...
};
```

这里需要注意区分类和函数的使用方式，函数可以前向声明告诉编译器函数存在，通过符号表匹配后调用；但类的前向声明只能告诉编译器该类的类型存在，没有类似于函数的符号表最后可以找到类的定义中的具体内容，所以类的使用一般采用直接定义。故这样的代码可以运行：

```cpp
int func();

int main()
{
	printf("%d", func());

	return 0;
}

int func() { return 521; }
```

而这样的代码不可以运行：

```cpp
class Thepale;

int main()
{
	printf("%d", sizeof(Thepale));

	return 0;
}

class Thepale
{

};
```



### 类的访问限定符

public（公有的）、protected（保护的）、private（私有的）。

```cpp
class Thepale
{
public:
    int a;
    void func1() { std::cout << "Thepale" << std::endl; }
protected:
	int b;
    void func2() { std::cout << "Thepale" << std::endl; }
private:
    int c;
    void func3() { std::cout << "Thepale" << std::endl; }
};

int main()
{
    Thepale obj;
    
    //public:
	obj.a;
    obj.func1();
    
    //protected:
    obj.b;
    obj.func2();
    
    //private:
    obj.c;
    obj.func3();
    
    return 0;
}
```

访问权限**作用域从该访问限定符出现的位置开始直到下一个访问限定符出现时为止，若后续没有访问限定符出现，则到大括号结束处为止**。

在以上函数中，只有 public 的访问部分是合法的，其它部分均会报错，在这里 private 和 protected 是类似的（具体区别在继承中讲解）。对于公有部分可以被任意访问（函数和数据可以通过对象调用和修改），私有和保护部分只能在类中被访问（无法通过对象直接调用函数和修改对象数据）。故可知其体现了封装特性。

在 C++ 中，将 C 语言的 struct 升级为类，**若没有访问限定符，struct 的默认属性是 public（为了兼容 C），class 的默认属性是 private**。



### 类的成员变量和成员函数

```cpp
class Thepale
{
    //成员变量
    int a;
    int b;
    int c;
    
    //成员函数
    int func1() { return a; }
    int func2() { return b; }
    int func3() { return c; }
};
```

类实现数据和方法的封装，以上所定义的变量等为成员变量，成员变量可以是基本数据类型，自定义类型，指针类型（Date* / Student* 都是指针类型，请区分指针类型和指针所指向的类型），数组类型等所有类型，也可以有 const 或 static 属性，这里只是做一个统称。所定义的函数为成员函数，包括自定义成员函数，构造函数，析构函数等，这里同样只是做一个统称。



### 类的实例化

类一般作为图纸、框架存在，例如每个人都会有年龄，身高，体重，姓名，但每个人的这些属性并不一定相同，类可以实例化出各种各样不同的对象。类也可以比作房屋的图纸，告诉哪个地方需要怎么建造，它并没有实际空间，只有实例化出的对象（依据图纸建出的房子）才有实际空间。



### 类的作用域

在以下代码中，所采用的成员函数定义方式是声明和定义同行：

```cpp
class Thepale
{
    int a;
    
    int func() { return a; } //声明和定义
};
```

在更多的应用场景中，将声明和定义分离（更多是在不同文件中）：

```cpp
class Thepale
{
    int a;
    
    int func();
};

int Thepale::func() { return a; } //需要用域作用限定符指定函数所在的类域，且该函数可以使用类中的成员变量
```

类似于这样的操作是不允许的：

```cpp
Thepale::a;
```

因为类仅仅是图纸，无法在图纸中找到房子的实体，以上操作是完全错误的。

```cpp
Thepale::func();
```

以上代码也是不允许的，涉及到 this 指针，因为成员函数无法显式传递 this 指针，故这样的调用会导致参数缺失，编译器通常所报出的是：调用非静态成员函数需要一个对象，道理一致，本质需要的是 [this 指针](#This 指针)





### 类的存储方式与大小计算

```cpp
class Thepale
{
    int a = 0;
    char b = 0;
    
    void func1() {}
    void func2() {}
};
```

采用 sizeof 计算上述类的大小，结果是：8 byte

其和结构体占用内存的计算方式一致，遵循结构体内存对齐原则。[^结构体内存对齐]

成员函数是被放在公共代码段的，如果实例化一个对象就需要创建一份成员函数，将会导致大量的代码冗余，故类中真实存储的只有成员变量，所以计算大小也是计算成员变量的大小。

```cpp
class Thepale
{};
```

对于空类而言，其 sizeof 所计算的大小仍有 1 byte，因为必须让类所实例化出的对象要占据具体的内存空间，否则 `Thepale e; &e;` 这里 e 的地址就不复存在了。但这并不是 C++ 的要求，而是编译器的选择。

```cpp
class Thepale
{
    void func1() {}    
};
```

故对于这种情况，大小仍是 1 byte，原因已叙述，这里只是做再次证明。



### This 指针

```cpp
class Thepale
{
public:
    int a;
    int func() { return a; }
};

int main()
{
    Thepale o1; o1.a = 521;
    Thepale o2; o2.a = 1314;

    std::cout << o1.func() << " ";
    std::cout << o2.func();

    return 0;
}
```

以上程序输出结果为：`521 1314`

但它们调用的是同一个函数，但是它们返回的 a 值并不一样。在[类的作用域](#类的作用域)中就已提到 this 指针。对于 Thepale 类中的 func 函数，还可以这样实现：

```cpp
int func() { return this->a; }
```

本质就是这个函数通过哪个对象调用，就会传递该对象的地址，在底层实现中，调用和实现应是这样的：

```cpp
//调用
func(&o1);
func(&o2);

//实现
int func(Thepale* const this) { return this->a; }
```

**this 指针是不允许显式传递和接收的，但可以显式使用**，可以通过 this 指针访问对象中的成员变量。这也解释了 `Thepale::func();` 是不可行的。从 this 指针的类型也可以知道，this 指针是不能被修改的。

若和 C 语言对比，会发现其实本质是一样的：

```cpp
struct Thepale
{
    int a;
};

int func(struct Thepale* const p) { return p->a; }
```

只是 C++ 省略了用户传递的过程，都由编译器完成，提升了可读性，使代码更加简洁，也是封装必不可少的一环。



#### This 指针为空

this 指针是可以为空的：

```cpp
class Thepale
{
public:
    int a;
    void func() { std::cout << "hello" << std::endl; }
};

int main()
{
    Thepale* p = nullptr;
    p->func();
    
	return 0;
}
```

以上程序正常输出：`hello`

相当于这样调用：`func(nullptr);` 虽然 this 指针为空，但 func 函数中并没有任何地方使用到 this 指针（或者说没有访问成员变量），故并不会报错。且这里并不能看作对空指针的解引用，因为其汇编底层是这样的：

```assembly
    Thepale* p = nullptr;
00007FF6D78F1EAB  mov         qword ptr [p],0  
    p->func();
00007FF6D78F1EB3  mov         rcx,qword ptr [p]  ;rcx 中装的是 this 指针的地址
00007FF6D78F1EB7  call        Thepale::func (07FF6D78F1465h) ;以空的 this 指针调用 func 函数
```

不可凭借字面意思理解代码。



This 指针严格来说是存在于栈区中，因为其作为函数形参存在，但有些编译器存放在寄存器中，例如 visual studio。

---

## 六大默认成员函数

### 构造函数

在一个类实例化出对象时，一般都需要进行初始化，在 C 语言阶段，一般通过 xxx_initialize 来完成，但在 C++ 中，构造函数承担了这一工作，它会在对象被创建时自动调用实现初始化，以保证每个对象都被初始化，防止未初始化所带来的错误。（注意，构造函数的意思是完成初始化而非分配空间）

构造函数具有以下特性：

* 函数名与类名相同。
* 没有返回值。（并非指返回值为 void，而是没有返回值的选项）
* 对象实例化时自己调用。
* 可以被重载，满足不同的初始化情况。
* 若未显式定义，系统会自动创建。对自动创建的构造函数对内置类型不处理，对自定义类型调用自定义类型的默认构造函数。

以下构造函数以日期类举例：

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    void initialize(size_t year, size_t month, size_t day)
    {
        _year = year;
        _month = month;
        _day = day;
    }

    void print()
    {
        cout << _year << "_" << _month << "_" << _day << endl;
    }
};
```

如果日期类是这样，那实例化对象后要调用初始化函数进行初始化：

```
Date d1;
d1.initialize(2022, 05, 21);
```

这和 C 语言并无二至，且也没有实际解决忘记调用、初始化不规范等问题，故构造函数便由此诞生：

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    Date(int year = 1, int month = 1, int day = 1) //这里添加了缺省值
        :_year(year) //这里使用了初始化列表，具体讲解请见构造函数讲解结尾
        ,_month(month)
        ,_day(day)
    {}
    
    void print()
    {
        cout << _year << "_" << _month << "_" << _day << endl;
    }
};
```

可以看到构造函数的名称和类名一致，且没有返回值。若需要实例化对象，这样即可：

```cpp
Date d1(2022, 05, 21);
```

但如果出现：

```cpp
Date d1;
d1.Date(2022, 05, 21); //error：不允许使用类型名
```

可知编译器是不允许构造函数被显式调用的，它会在对象构造时自动调用。

如果想使用缺省值初始化对象，应该这样写：

```cpp
Date d1;
```

而不是：

```
Date d1();
```

如果写成：`Date d1();` 最大的原因是会被编译器识别为函数声明，无法完成对象的构造。故如果实现无参调用，直接构造即可，不需要加括号，这算是因为冲突而不得已的做法吧。



构造函数是可以被重载的，满足不同的初始化情况：

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    Date(int year, int month, int day) //带参构造
        :_year(year)
        ,_month(month)
        ,_day(day)
    {}

    Date() //无参构造
        :_year(1946)
        , _month(2)
        , _day(15)
    {}

    void print()
    {
        cout << _year << "_" << _month << "_" << _day << endl;
    }
};
```

以上两个构造函数构成函数重载，函数重载定义具体见对应文章。



如果日期类不写构造函数：

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
	Date()
    {
        _year = 1946;
        _month = 2;
        _day = 15;
    }

    void print()
    {
        cout << _year << "_" << _month << "_" << _day << endl;
    }
};

int main()
{
    Date d1;
    d1.print();
    
    return 0;
}
```

程序的输出结果为：`14757395258967641292_14757395258967641292_14757395258967641292`（随机值）

但实际上在没有显式写出构造函数时，编译器会自动生成无参默认构造函数，这个构造函数对于内置类型不处理，对于自定义类型调用则调用该类型的默认构造函数，故默认生成的构造函数并非一无是处。故在一些特殊情况，例如用双队列实现栈时，可以只实现队列的无参默认构造函数，栈可以不实现：

```cpp
class Queue
{
private:
    int _a; //假设，仅做演示

public:
    Queue()
        :_a(0)
    {}
};

class Stack
{
private:
    Queue q1;
    Queue q2;
};
```

Stack 不写构造函数也是可以正常构造的。



这里需要捋清概念，默认构造函数指的是不需要传参就能调用的构造函数（全部使用缺省参数的构造函数不是严格意义上的默认构造函数，但也可当作默认构造函数），所以如果自己写一个不需要参数就可以调用的构造函数称为默认构造函数，编译器自动生成的也是默认构造函数。（请区分不需要参数和无参）显然，默认构造函数只能存在一个，否则会存在函数调用不明确的问题，且如果类中有自己写的构造函数，则编译器不会再默认生成，否则将引发调用不明确的问题：

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    Date(int year, int month, int day)
        :_year(year)
        ,_month(month)
        ,_day(day)
    {}

    void print()
    {
        cout << _year << "_" << _month << "_" << _day << endl;
    }
};
```

如果类中的构造函数定义如上，但调用方式如下：

```cpp
Date d1;
```

编译器会报错，因为有了构造函数之后编译器不会默认生成，而这又是一个无参构造，并没有满足条件的构造函数，故报错。	



#### 初始化列表

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    Date(int year, int month, int day) //初始化列表初始化
        :_year(year)
        , _month(month)
        , _day(day)
    {}
};

class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    Date(int year, int month, int day) //赋初值
    {
        _year = year;
        _month = month;
        _day = day;
    }
};
```

以上代码中，第一个类采用的是初始化列表初始化，**初始化列表初始化的阶段是对象内成员被定义的阶段**，故**初始化列表完成的是初始化**，任何成员在其中最多出现一次。而**第二个类采用的是赋初值**的方式，**这种方式不能称之为初始化**，因为在这一阶段对于成员变量可以多次赋值，且这一阶段无法完成一些特殊成员变量的初始化。

以下是必须使用初始化列表初始化的三种情况：

##### 1.const 成员变量

```cpp
class Thepale
{
private:
    const int _data;

public:
    Thepale(int data = 0)
        :_data(data)
    {}
};
```

因为 const 成员变量只有一次初始化的机会，若错过了初始化，则不能在对其值进行修改。而赋初值的方式实际上是已经经历了初始化阶段后的（不明确提供初始化列表则会自动生成默认初始化列表，默认初始化行为取决于成员变量的类型以及它们是否有默认构造函数，也就是我们所说的对内置内心不处理，对自定义类型调用它的默认构造函数），因为初始化阶段对内置类型不处理，故赋初值无法更改 const 成员变量。

##### 2.引用成员变量

```cpp
class Thepale
{
private:
    int& _data;

public:
    Thepale(int& data)
        :_data(data)
    {}
};
```

同理，引用在定义时必须初始化，不可以有空引用出现，若不在初始化列表初始化，则对象会无法构造。

##### 3.没有默认构造函数的自定义类型

```cpp
class A
{
private:
    int _data;

public:
    A(int data)
        :_data(data)
    {}
};

class Thepale
{
private:
    A _a;

public:
    Thepale(int a)
        :_a(a)
    {}
};
```

由于自定义类型默认会调用它的默认构造函数，若没有在初始化列表传参初始化，没有默认构造函数的对象会无法得到初始化，对象会无法调用它的构造函数，故必须在初始化列表中传参进行初始化。

**成员变量的初始化顺序和声明顺序有关，和初始化列表中的初始化顺序无关。**

在更多情况下，初始化列表和构造函数是相辅相成的，例如动态内存开辟的空间赋对应值等，有些部分必须在函数体内完成，初始化列表也并非全知全能。

C++11 中添加了类内初始化的特性：

```cpp
class Date
{
private:
    size_t _year = 1;
    size_t _month = 1;
    size_t _day = 1;

public:
    void print()
    {
        cout << _year << "_" << _month << "_" << _day << endl;
    }
};
```

这种情况实际上是在初始化列表没有明确提供时充当默认初始化列表中的参数使用，当初始化列表中提供了实际参数则类内初始化的值将没有实际用处。如果是使用赋初值的方式而不明确写明初始化列表，则会在调试途中发现成员变量会被多次赋值，一次是初始化列表利用内类初始化的默认值初始化，一次是构造函数体内的赋值，切勿混淆。

----

### 析构函数

在 C 语言中，若使用动态开辟的空间，最后一定要 free，在 C++ 的 new 和 delete 也是一样，这个操作需要手动进行，故容易引发内存泄漏。析构函数便由此而来，它负责在对象要被销毁时自动调用，负责释放资源。

析构函数具有以下特性：

* 函数名和是类名前加上 `~` 号。
* 没有返回值。
* 对象被销毁时自动调用。
* 析构函数不可重载，仅存在一个。
* 若未显式定义，系统会自动创建。自动创建的析构函数对内置类型不处理，对自定义类型调用自定义类型的析构函数。

```cpp
class Thepale
{
private:
    int* _arr;

public:
    Thepale()
    {
        _arr = new int[1024]; //动态开辟的空间
    }

    ~Thepale() //析构命名方式
    {
        delete[] _arr; //析构时释放
    }
};

int main()
{
    Thepale e;

    return 0;
}
```



默认生成的析构函数，和构造函数一样，对内置类型不处理，例如如果是日期类可以不同显式写明析构函数，对象生命周期结束内置类型会自动回收；对自定义类型则去调用自定义类型的析构函数，应用场景依然是与双栈实现队列等类似的，这里不过多说明。

---

### 拷贝构造函数

拷贝构造函数绝大多数的情况下在拷贝初始化[^拷贝初始化]中被调用，常见场景有三种：1.使用已存在的对象创建新对象（这种情况一定是调用拷贝构造进行初始化而非 operator=，只要是用已有对象创建新对象就是拷贝构造的场景，切勿通过符号表象解释）；2.函数参数类型为类类型对象；3.函数返回值类型为类类型对象。拷贝构造是一种特殊的构造函数，也是构造函数的重载形式，和构造函数一样具有大部分相同的特性：

* 拷贝构造是构造函数的重载。
* 函数名和类名相同。
* 没有返回值。
* 形参只有一个且类型必须是对象的引用（如果是指针则成为了构造函数的重载形式，而非拷贝构造函数）。
* 未显式调用会自动生成默认拷贝构造函数，对内置类型完成逐字节拷贝（浅拷贝 / 值拷贝），对自定义类型调用自定义类型的拷贝构造函数。

```cpp
class Thepale
{
private:
    int _a;
    int _b;

public:
    Thepale(int a = 0, int b = 0)
        :_a(a)
        ,_b(b)
    {}

    Thepale(const Thepale& e) //拷贝构造函数
    {
        _a = e._a;
        _b = e._b;
    }

    ~Thepale()
    {}
};

int main()
{
    Thepale e1;
    Thepale e2(e1); //用 e1 拷贝构造 e2
    
    return 0;
}
```

拷贝构造函数即对一个对象中成员变量的拷贝，对于上述情况，仅需要完成浅拷贝时，甚至可以不显式写明拷贝构造函数，使用默认生成的即可。



```cpp
class Thepale
{
private:
    int* _arr;
    int _size;

public:
    Thepale(int size = 128) //仅作演示，不考虑细节（代码功能就是开辟特定大小的空间）
    {
        _arr = new int[size] {0};
        _size = size;
    }

    Thepale(const Thepale& e) //拷贝构造
    {
        _arr = new int[e._size] {0}; //必须重新开辟空间
        memcpy(_arr, e._arr, 4 * e._size);
        _size = e._size;
    }

    ~Thepale()
    {
        delete[] _arr;
        _size = 0;
        _arr = nullptr;
    }
};

int main()
{
    Thepale e1;
    Thepale e2(e1); //用 e1 深拷贝构造 e2

    return 0;
}
```

以上场景只能使用深拷贝，若使用浅拷贝，则两个对象指向同一块空间（指针是内置类型，会被拷贝），析构两次（实际在 memcpy 时就出现空指针错误），单个对象的数据修改同时影响两个或多个对象，造成严重问题。



拷贝构造函数严禁传值调用，以上述代码举例，如果拷贝构造函数写成：`Thepale(const Thepale e);` 这时 e2 进行拷贝构造，即相当于需要将 e1 传给 e，而这个实参拷贝给形参的过程又会调用拷贝构造，导致无限调用。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Class_And_Object-p1.png)

---

### 赋值运算符重载

这里重载的意义和函数重载不同，它更多的意义是偏向于对自定义对象进行运算的运算符意义的重新定义，故称之为运算符重载。赋值运算符重载，它通常发生在两个已存在的对象之间进行赋值。

赋值运算符的特性：

* 用户没有显示写明时，会生成默认的赋值运算符重载，内置类型完成逐字节拷贝，自定义类型调用自定义类型的赋值运算符重载。

```cpp
class Date
{
private:
    size_t _year;
    size_t _month;
    size_t _day;

public:
    Date(int year = 1, int month = 1, int day = 1)
        :_year(year)
        ,_month(month)
        ,_day(day)
    {}
    
    Date& operator=(const Date& d)
    {
        if(this != &d) //如果对象为本身则不需要进行赋值
        {
    		_year = d._year;
    		_month = d._month;
        	_day = d._day;
		}

        return *this;
    }
};

int main()
{
    Date d1(2023, 5, 21);
    Date d2(2023, 7, 16);
    
    d1 = d2; //调用赋值运算符重载
}
```

当然，其也存在深拷贝的情况，不再举例。

一般而言，赋值需要完成连续赋值的特性需要返回引用，这也就是代码中返回值为 `Date&` 的原因。而一般传参也使用引用传参以提高效率，会进行检查是否为自己给自己赋值，避免不必要的资源消耗。注意，前面所说的并不是必须执行，只是给了一个常用的合理的解。

---

### 取地址操作符重载和 const 取地址操作符重载

```cpp
class Date
{ 
private:
 	int _year;
 	int _month;
	 int _day;
    
public :
     Date* operator&() //取地址操作符重载，一般不显示写明，编译器默认生成的即返回 this 指针，除非想让取地址能获得特定的内容
     {
     	return this;
     }
    
     const Date* operator&() const //对 const 匹配，const 对象取地址返回 const 的指针，防止内容被改变
     {
     	return this;
     }
};
```

（在成员函数后加上 const，即类似于 `const Date* const this`，为前一个 const，防止 Date 内的数据被修改；后一个 const 是 this 指针自带的属性，不能改变 this 指针的指向。const 不能用于修饰构造或析构函数，const 也起了对调用的匹配作用，即 const 的 this 指针的匹配：const 对象取地址是 const 的 this 指针，会调用 const 的函数，因为 const 修饰的函数隐藏的形参 this 指针的参数类型是 const *）

这两者不过多叙述，一般由编译器默认生成，且很少有需要显示写明的必要。

---

## 类和对象的其它特性

### explicit

```cpp
class Thepale
{
private:
    int _data;

public:
    Thepale(int data = 0)
        :_data(data)
    {}
};

int main()
{
    Thepale e = 1; //隐式类型转换

    return 0;
}
```

对象也可以用以上注释行的方式被构造，其主要原因是存在隐式类型转换，将 1 转换成 Thepale 类型的临时对象，这一转换过程会调用构造函数以 1 构造临时对象，然后再将这个临时对象拷贝构造给 e。不过现今编译器绝大多数都会优化这一情况，会直接使用 1 去构造对象 e，这里是去除优化谈原理。而这个隐式类型转换，可以被人为禁止，即给对应函数加上 `explicit` 关键字：

```cpp
class Thepale
{
private:
    int _data;

public:
    explicit Thepale(int data = 0)
        :_data(data)
    {}
};

int main()
{
    Thepale e = 1; //隐式类型转换

    return 0;
}
```

编译无法通过：`不存在从 "int" 转换到 "Thepale" 的适当构造函数`

当然，隐式类型转换有很多应用场景，例如在 string 中就有应用，直接用字符串初始化对象等；explicit 在智能指针中有应用，不允许隐式类型转换防止导致内存泄漏等问题。

---

### static 成员变量和成员函数

#### static 成员变量

当需要类似于统计现有效对象个数等场景时，使用全局变量没有保证封装和安全性，定义类内的 static 成员则可以保证：

```cpp
class Thepale
{
private:
    int _data;
    static int _count;

public:
    Thepale(int data = 0) //构造时自增
        :_data(data)
    {
        _count++;
    }

    Thepale(const Thepale& t) //拷贝构造时自增
    {
        _data = t._data;
        _count++;
    }

    ~Thepale() //析构时自减
    {
        _count--;
    }
};

int Thepale::_count = 0; //类外定义
```

**成员变量属于对象，储存在对象中；而静态成员变量属于类，对象可以共享，但存在静态区中。**

当然，静态成员变量不属于对象成员，不可以在初始化列表中被定义，一般在类外定义；且不可以使用类内初始化值，因为初始值是需要给初始化列表的，静态成员变量不在其中，故也没有在类内定义的方法，只能类外定义。（静态成员只需要突破类域和访问限定符就可以被访问，不需要通过对象，例如假设 _count 是公有的，则可以直接通过指定类域：`Thepale::_count = 1;` 进行访问）

如果变成：`const static int _count = 0;` 则可以直接在类内初始化，这种方法仅限于常整型静态成员，据我个人所能查到的资料和消息：*这种做法是因为这样的初始化表达式是编译时常量，可以在编译时被直接计算替换。*具体目的是为了优化效率，我个人对这种做法表示不解和困惑，也不推荐去深入了解和剖析，按常规情况处理即可，若有理解困难，强烈建议忽略此点。

#### static 成员函数

```cpp
class Thepale
{
private:
    int _data;
    static int _count;

public:
    Thepale(int data = 0)
        :_data(data)
    {
        _count++;
    }

    Thepale(const Thepale& t)
    {
        _data = t._data;
        _count++;
    }

    ~Thepale()
    {
        _count--;
    }

    static int get_count() //静态成员函数
    {
        return _count;
    }
};

int Thepale::_count = 0;
```

静态成员函数没有 this 指针，故在静态成员函数中不可以访问其它成员变量，这也间接说明静态成员变量的储存位置和成员变量不同，其不属于某一个对象，而是属于类，储存在静态区。静态成员函数也只需要指定类域和突破访问限定符限制即可访问：`Thepale::get_count();`。一般配合静态成员变量使用。当然，在类中静态成员变量和函数都是可以被使用的（类内不受访问限定符和类域限制）。

---

### friend

#### 友元函数

**友元函数可以访问类中的私有和保护成员，且友元函数必须是定义在类外的普通函数**（因为友元函数不属于成员函数），但**需要在类内声明**，声明可以放在类内的任何地方，它仅仅起一种告知作用，不受访问限定符等影响。

```cpp
class Date
{
private:
    int _year;
    int _month;
    int _day;

public:
    Date(int year = 1, int month = 1, int day = 1)
        :_year(year)
        ,_month(month)
        ,_day(day)
    {}

    friend ostream& operator<<(std::ostream& out, const Date& d);
};

ostream& operator<<(std::ostream& out, const Date& d) //友元函数
{
    out << d._year << "年" << d._month << "月" << d._day << "日";
    return out;
}
```

例如在实现自定义类型的输出重载时，如果不采用友元，则第一个参数默认是 this 指针，这种方式可以完成输出，但会是这样：`d << cout;` 不符合使用习惯和语法逻辑，故输出输入类的操作符重载会考虑采用友元方式实现，这样可以手动控制参数个数和顺序，且可以访问类内私有和保护成员。

当然，一般情况下不太推荐使用友元，友元会在一定程度上破坏代码封装性（直接访问私有和保护成员），破坏继承关系（子类不会继承友元），增加耦合（紧密性增加，一个部分的更改影响多个部分）等。

**友元不可以传递**，若 B 是 A 的友元，C 是 B 的友元，C 并不是 A 的友元。

#### 友元类

```cpp
class Time
{
private:
    int _hour;
    int _minute;
    int _second;

public:
    Time(int hour = 0, int minute = 0, int second = 0)
        :_hour(hour)
        , _minute(minute)
        , _second(second)
    {}

    friend class Date; //友元类
};

class Date
{
private:
    int _year;
    int _month;
    int _day;
    Time t;

public:
    Date(int year = 1, int month = 1, int day = 1)
        :_year(year)
        ,_month(month)
        ,_day(day)
    {}

    void set_time(int hour = 0, int minute = 0, int second = 0) //可以设置 Time 类生成对象中的成员
    {
        t._hour = hour;
        t._minute = minute;
        t._second = second;
    }
};
```

同理，当一个类变成另一个类的友元，则这个类可以访问另一个类的私有或保护的成员变量。**友元是单向的**（A 是 B 的友元类仅仅是 B 可以访问 A，A 不能访问 B，除非是相互为友元类）

---

### 内部类

内部类在 C++ 中并不常用（以我浅薄的知识储备是这样认为的）。内部类就是定义在类中的类，例如这样：

```cpp
class A
{
    int _a;
    
    class B
    {
        int _b;
    };
};
```

这里的 B 类就是内部类。这里有一点需要提及，A 类的大小是 4，而不是 8：B 类在 A 类中仅仅是声明，没有实例化出对象，声明是不需要占用空间的，所以这里计算的仅仅是 A 类的大小。形象的理解为，A 类是一张图纸，B 类是 A 类图纸中的图纸，或许这样说有些奇怪，但就是 B 的声明在 A 的声明中。所以就算去除所有变量，A 类的大小也是一张图纸的大小，为 1 byte。

**外部类天生是内部类的友元类**，就拿上述日期和时间类举例：

```cpp
class Time
{
private:
    int _hour;
    int _minute;
    int _second;

public:
    Time(int hour = 0, int minute = 0, int second = 0)
        : _hour(hour)
        , _minute(minute)
        , _second(second)
    {}

    class Date; //先声明
};

class Time::Date
{
private:
    int _year;
    int _month;
    int _day;
    Time t;

public:
    Date(int year = 1, int month = 1, int day = 1)
        : _year(year)
        , _month(month)
        , _day(day)
    {}

    void set_time(int hour = 0, int minute = 0, int second = 0) //在内部类 Date 中可以随意访问 Time 类的成员
    {
        t._hour = hour;
        t._minute = minute;
        t._second = second;
    }
};
```

至于不直接嵌套进去的原因也很简单，若写成：

```cpp
class Time
{
private:
    int _hour;
    int _minute;
    int _second;

public:
    Time(int hour = 0, int minute = 0, int second = 0)
        :_hour(hour)
        , _minute(minute)
        , _second(second)
    {}

    class Date
    {
    private:
        int _year;
        int _month;
        int _day;
        Time t;

    public:
        Date(int year = 1, int month = 1, int day = 1)
            :_year(year)
            , _month(month)
            , _day(day)
        {}

        void set_time(int hour = 0, int minute = 0, int second = 0)
        {
            t._hour = hour;
            t._minute = minute;
            t._second = second;
        }
    };
};
```

由于 **在内部类未创建完成时，外部类就不会创建完成**，所以在内部类内使用外部类创建对象会出现：`不允许使用不完成的类型` 的报错。故采用先声明后定义的方法才是正确的。且内部类会受访问限定符限制，例如将内部类用私有限制，则正常情况下在外部不可以构建内部类对象，这里不再举例。

---

### 匿名对象

以此类举例：

```cpp
class Thepale
{
private:
    int _data;

public:
    Thepale(int data = 0)
        :_data(data)
    {
        cout << "构造函数被调用" << endl;
    }

    ~Thepale()
    {
        cout << "析构函数被调用" << endl;
    }
};
```

若正常构造一个对象，则会在 return 时被销毁；但如果构造**一个匿名对象，则它的生命周期仅在这一行**。故匿名对象会经常在临时计算、参数传递等场景下被使用。现采用如下方式调用：

```cpp
int main()
{
    Thepale e;

    cout << "hello" << endl;
    Thepale();
    cout << "hello" << endl;

    return 0;
}
```

程序的输出结果为：

```cpp
构造函数被调用
hello
构造函数被调用
析构函数被调用
hello
析构函数被调用
```

即匿名对象的生命周期仅在这一行，得证。

**匿名对象和临时对象一样具有常性**，故必须使用 const 类型接收或引用：

```cpp
Thepale re = Thepale(); //error
const Thepale& re = Thepale(); //correct
```

但当**匿名对象被施加引用时，生命周期会被延长**：

```cpp
int main()
{
    Thepale e;

    cout << "hello" << endl;
    const Thepale& re = Thepale();
    cout << "hello" << endl;

    return 0;
}
```

程序的输出结果为：

```cpp
构造函数被调用
hello
构造函数被调用
hello
析构函数被调用
析构函数被调用
```

其原因是匿名对象被起别名，这导致它不再是一个没有名字的对象，被赋予了生命，直到引用销毁时它才会销毁。也有一部分的情况是为了防止悬空引用的出现，避免无效访问。

---

### 编译器对拷贝构造的优化（拷贝省略）

一般而言，编译器在进行连续的构造和拷贝构造时，会进行拷贝省略，以以下类举例：

```cpp
class Thepale
{
private:
    int _data;

public:
    Thepale(int data = 0)
        :_data(data)
    {
        cout << "构造函数被调用" << endl;
    }

    Thepale(const Thepale& e)
    {
        _data = e._data;
        cout << "拷贝构造函数被调用" << endl;
    }

    ~Thepale()
    {
        cout << "析构函数被调用" << endl;
    }
};
```

```cpp
void func(Thepale e){}

int main()
{
    func(1);
	//func(Thepale()); - 同理
    
    return 0;
}
```

以上代码，1 通过隐式类型转换调用构造函数构造对象，构造完成后传递给形参 e，总共是构造 + 拷贝构造，但经过编译器的优化，输出结果为：

```cpp
构造函数被调用
析构函数被调用
```

仅调用一次构造函数，直接用 1 构造 e。

```cpp
Thepale func()
{
    Thepale tmp;
    return tmp;
}

int main()
{
    Thepale e = func();

    return 0;
}
```

以上代码，e 通过 func() 进行拷贝构造，而 func() 在内部构造 tmp 后返回，tmp 拷贝构造给临时变量再拷贝构造给 e，总体过程为构造 + 拷贝构造 + 拷贝构造，但经过编译器的优化，输出结果为：

```cpp
构造函数被调用
析构函数被调用
```

从汇编代码可以看出是直接对 e 进行了构造：

```assembly
    Thepale e = func();
00007FF7A44F352D  lea         rcx,[e] ;取 e 的地址，在 func() 中构造  
00007FF7A44F3531  call        func (07FF7A44F14BFh)  
```

**这里并非 C++ 的特性，而是编译器做的优化，编译器各不相同，优化方式也有些差异，本举例以 Visual Studio 2022 为准，且目的在于告知编译器会有优化步骤，并非说明了优化标准。**

---

## 补充说明

* 对于以下代码（剖析拷贝构造在没有优化下的调用形式以及展示拷贝省略的一个场景）：

```cpp
class Thepale
{
private:
	int _a;

public:
    Thepale(int a = 0)
    {
        cout << "Thepale(int a = 0) -- 构造函数被调用" << endl;
		_a = a;
    }

    Thepale(const Thepale& e)
    {
        cout << "Thepale(const Thepale& e) -- 拷贝构造函数被调用" << endl;
		_a = e._a;
    }

    ~Thepale()
    {
		cout << "~Thepale() -- 析构函数被调用" << endl;
    }
};

Thepale func()
{
    Thepale tmp;
    return tmp;
}

int main()
{
    Thepale ret = func(); //请注意此行
    
    return 0;
}
```

在运行后，程序的执行结果是：

```cpp
Thepale(int a = 0) -- 构造函数被调用
~Thepale() -- 析构函数被调用
```

这实际上是进行了返回值优化和拷贝省略之后的结果，如果不考虑这两步，注释行逻辑应该为：func 函数被调用，在 func 函数中，创建 tmp 对象，此时 **调用构造函数**；执行到 return 语句，需要返回 tmp，于是将 tmp 拷贝给临时变量，拷贝时调用了 **拷贝构造函数** 完成拷贝；函数调用完成，函数栈帧销毁，此时 tmp 对象也 **调用析构函数** 销毁；然后回到 main 函数，将临时变量赋值给 ret 对象，由于此时是对 ret 对象的初始化（拷贝初始化），所以依然 **调用拷贝构造函数**；注释行执行完后，临时变量销毁，**调用析构函数**；（临时变量的生命周期是当前行）main 函数执行到 return，函数结束，ret 被回收，**调用析构函数**。（以上情况在执行时可以看出明显使用了拷贝省略减少拷贝，其实此种情况就是拷贝初始化，但并没有调用拷贝构造函数）



* 所有的默认成员函数只能在类中声明，实现位置没有具体要求。
* 以下五个操作符不可重载：**::**（域作用限定符）、**:?**（三目操作符）、**.**（成员访问操作符）、**sizeof**（计算对象或数据类型大小的操作符）、**.***（成员指针运算符）



[^结构体内存对齐]: 1.第一个成员从 0 偏移量开始存储。2.其它成员变量对齐到对齐数[^对齐数]的整数倍处。3.结构体的总大小为最大对齐数[^最大对齐数]的整数倍。（结构体内存对齐一定程度上也取决于编译器，嵌套结构体的情况也按此规则处理）
[^对齐数]: 编译器的默认对齐数和该成员所占大小两者取较小值。
[^ 最大对齐数]: 在存储元素时，每一个元素的存储都会产生一个对齐数，取这些对齐数中的最大值。

[^拷贝初始化]: 拷贝初始化 = 拷贝构造函数 + 拷贝省略（返回值优化和右值引用）。拷贝构造函数绝大部分情况下会调用拷贝构造函数，但在一些情况下会触发拷贝省略则不调用拷贝构造函数，但这两者都叫做拷贝初始化。