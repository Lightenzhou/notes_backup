# Inheritance

---

## 继承概述

继承是针对类而言的，派生类（子类）可以通过继承基类（父类）来继承基类中的成员变量和调用基类中的成员函数，即一个类继承另一个类的属性和方法。继承也是面向对象中的重要思想，它实现了对代码的复用和对逻辑的有效梳理，降低了维护成本和加强了类之间的层次关系。

继承分为：公有继承、保护继承、私有继承，它们分别将基类成员的访问权限在派生类中变更，总有九种组合方式：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Inheritance-p1.png)

而最常见的往往是 public 继承方式，且本篇继承也主要以 public 继承展开，主要是为了完整的展示继承特性，其它的继承方式可能在本篇中无法得到同样结论或导致报错。

---

## 继承的使用方式及特性

### 继承的使用方式

#### 单继承

```cpp
class Person
{
public:
    char* _name;
    int _age;
};

class Student : public Person //Student 类继承 Person 类
{
public:
    int _id;
    char* _major;
};
```

由于学生、老师、辅导员等身份信息都会包含最基本的人的信息，例如姓名、年龄、性别等，所以将 Person 用于储存人的基本信息，每个身份都应该包括它。而对于特定身份的特定信息，例如学生的学号，老师的教学专业，辅导员的管理学院等，采用继承公有信息后再单独定义特定信息的方式，可以减少代码冗余和增强可读性。

此时 Person 中有四个成员，分别是 Person 的两个成员和 Student 的两个成员。



#### 多继承

```cpp
class A
{
public:
    int _a;
};

class B
{
public:
    int _b;
};

class C : public A, public B
{
public: 
    int _c;
};
```

C 类同时继承了 A 类和 B 类的属性和方法，C 类中一共有三个成员，分别是 A 类中的 _a，B 类中的 _b，C 类中的 _c。（没有举例是因为多继承的例子实在难以脱口而出，正如此，多继承的运用场景其实不多，在某些语言中，例如 Java 直接取消了多继承）

---

### 赋值兼容

当派生类和基类进行赋值操作时，派生类赋值给基类往往是天然支持的，这一过程将会把派生类中的基类部分对所赋值的基类进行赋值操作，这一操作称之为切割或切片。而将基类赋值给派生类往往是不允许的，因为派生类中是包含基类的，而除了基类以外的成员变量无法被赋值。

```cpp
class Parent
{
public:
    int _a = 520;
};

class Son : public Parent
{
public:
    int _b = 1314;
};

int main()
{
    Parent p;
    Son s;

    p = s; //切片

    return 0;
}
```

指针同理，可以使用父类指针管理子类：

```cpp
int main()
{
    Parent p;
    Son s;

    Parent* pp = &s;

    return 0;
}
```

引用同理，可以使用父类引用管理子类：

```cpp
int main()
{
    Parent p;
    Son s;

    Parent& rp = s;

    return 0;
}
```

通过引用的直接管理可知，这中间并没有产生临时对象，否则需要加 const 才可管理，故这一种管理是天然支持的行为。而如果将父子颠倒，则均会报错（在特定的情况下可以实现基类指针或引用强制转换给派生类的指针或引用，需要使用 `dynamic_cast` 保证转换安全，此处不予讲解）。它们的管理类似于下图：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Inheritance-p2.png)



---

### 子类的默认成员函数

#### 构造函数

在子类的构造函数中，父类会优于一切进行初始化（包括子类初始化列表），而没有对父类进行显式初始化时，会自动调用父类的默认构造函数，没有默认构造函数则报错：

```cpp
class Parent
{
public:
    Parent(int a)
        :_a(a)
    {}

protected:
    int _a;
};

class Son : public Parent
{
public:
    Son(int b)
        :_b(b)
    {}

    int _b = 1314;
};
```

编译器报错：`error C2512: “Parent”: 没有合适的默认构造函数可用`

若需要显式初始化，则仅需在子类的初始化列表中完成，且都是首先被执行（若是多继承则按照继承顺序依次执行）：

```cpp
class Parent
{
public:
    Parent(int a)
        :_a(a)
    {}

protected:
    int _a;
};

class Son : public Parent
{
public:
    Son(int a, int b)
        :Parent(a) //优先级最高
        ,_b(b)
    {}

    int _b;
};
```

这有些类似于手动调用构造函数并传参。



#### 拷贝构造函数

子类进行拷贝构造时，会优先调用父类的拷贝构造函数，然后再拷贝除父类外的子类内容，而这一步较于构造函数的不同是必须手动调用：

```cpp
class Parent
{
public:
    Parent(int a)
        :_a(a)
    {}

    Parent(const Parent& p) //父类的拷贝构造
    {
        _a = p._a;
    }

protected:
    int _a;
};

class Son : public Parent
{
public:
    Son(int a, int b)
        :Parent(a)
        ,_b(b)
    {}
       
    Son(const Son& s) //子类的拷贝构造
        :Parent(s) //显式调用
    {
        _b = s._b;
    }

    int _b;
};
```

这类似于显式调用父类的拷贝构造函数，且这里的传参为对象 s，而默认会触发引用切片，仅管理子类对象 s 中的父类成分。

如果不手动调用父类拷贝构造函数，则编译器会默认调用父类的默认构造函数。在不手动调用时，编译器报错为：`“Parent”: 没有合适的默认构造函数可用`，若调用了默认构造函数实现拷贝构造，往往会带有一定风险：

```cpp
class Parent
{
public:
    Parent(int a = 520) //为父类提供默认构造函数
        :_a(a)
    {}

protected:
    int _a;
};

class Son : public Parent
{
public:
    Son(int a, int b)
        :Parent(a)
        ,_b(b)
    {}
       
    Son(const Son& s) //不显式调用父类的拷贝构造函数
    {
        _b = s._b;
    }

    int _b;
};

int main()
{
    Son s1(1, 2);
    Son s2(s1);

    return 0;
}
```

此时 s1 的 `_a` 为 1，`_b` 为 2；s2 的 `_a` 为 520，`_b` 为 2。和预期不符，就是因为调用了父类默认构造初始化了父类部分。故请尽量避免这一情况，尽管父类没有提供拷贝构造函数，也可以显式调用父类拷贝构造函数从而调用编译器自动生成的默认拷贝构造完成浅拷贝。



#### 析构函数

在子类创建时，必须先创建父类再创建子类；在子类析构时，必须先析构子类再析构父类。这有些类似于栈的先进后出操作。但放在实际应用中来看，不排除子类对象会使用父类对象的部分资源和数据，若先析构父类，子类中的访问和使用必然出现错误，故先析构子类是必要的。而在子类的析构函数中，并不必显式调用父类的析构函数，编译器会在子类析构调用结束后自动调用父类析构，这也是为了防止人为的使父类先于子类析构。

```cpp
class Parent
{
public:
    ~Parent()
    {
        std::cout << "~Parent()" << std::endl;
    }

protected:
    int _a;
};

class Son : public Parent
{
public:
    ~Son()
    {
        std::cout << "~Son()" << std::endl;
    }

private:
    int _b;
};

int main()
{
    Son s;

    return 0;
}
```

程序输出为：
```cpp
~Son()
~Parent()
```



#### 赋值运算符重载

赋值运算符重载没有初始化列表，故必须在函数体内自行手动调用父类的 `operator=`，当然这里涉及到函数隐藏（重定义），故需要指明类域访问：

```cpp
class Parent
{
public:
    Parent(int a)
        :_a(a)
    {}

    Parent& operator=(const Parent& p)
    {
        if(this != &p)
        {
            _a = p._a;
        }

        return *this;
    }

protected:
    int _a;
};

class Son : public Parent
{
public:
    Son(int a, int b)
        :Parent(a)
        ,_b(b)
    {}

    Son& operator=(const Son& s)
    {
        if (this != &s)
        {
            Parent::operator=(s); //父类对象赋值，传递 s 自动切片
            _b = s._b; //子类对象赋值
        }

        return *this;
    }

private:
    int _b;
};
```

在子类调用父类的赋值运算符重载可以增强代码的可读性和封装性，且在父类成员变量是私有时，在子类中无法对它们完成直接赋值。故在子类中直接对父类成员赋值是不明智的，调用父类的赋值运算符重载显然是更优解。

---

### 派生类中的隐藏

#### 成员变量隐藏

当父类和子类的成员变量中有名称是一样的成员时，它们则构成成员变量的隐藏（重定义）[^ 隐藏和重定义]，子类的重名成员会隐藏父类的成员，通过子类访问或修改该成员时默认访问子类成员，指定类域则可访问父类成员：

```cpp
class Parent
{
public:
    int _a = 520;
};

class Son : public Parent
{
public:
    int _a = 1314;
};

int main()
{
    Son s;
    std::cout << s._a << std::endl;
    std::cout << s.Parent::_a << std::endl;

    return 0;
}
```

程序输出结果为：

```cpp
1314
520
```

访问的顺序一般为：`域作用限定符所指定域 > 局部域 > 子类域 > 父类域 > 全局域`



#### 成员函数隐藏

当子类和父类中的函数名相同（注意，仅函数名相同即可，不包括参数和返回值等）即构成函数隐藏，此时默认调用子类函数，通过域作用限定符可调用父类函数：

```cpp
class Parent
{
public:
    void print()
    {
        std::cout << _a << std::endl;
    }

    int _a = 520;
};

class Son : public Parent
{
public:
    void print()
    {
        std::cout << _b << std::endl;
    }

    int _b = 1314;
};

int main()
{
    Son s;

    s.print();
    s.Parent::print();

    return 0;
}
```

程序的输出结果为：

```cpp
1314
520
```



无论是成员变量隐藏还是成员函数隐藏，都会降低代码的可读性和增加维护难度，故请尽量避免该情况出现。

---

### 继承的其它特性

#### 友元

基类中的友元派生类无法继承，无论是友元函数还是友元类。



#### 静态成员

静态成员会被继承，但继承的是使用权而并没有额外开空间。

---

### 菱形继承

#### 产生原因

菱形继承是多继承的一种特殊情况：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Inheritance-p3.png)

B 和 C 分别继承 A，而 D 分别继承 B 和 C，则会导致 D 中有两份 A 对象：

```cpp
class A
{
public:
    int _a = 1;
};

class B : public A
{
public:
    int _b = 2;
};

class C : public A
{
public:
    int _c = 3;
};

class D : public B, public C
{
public:
    int _d = 4;
};
```

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Inheritance-p4.png)

可以发现对于 A 类中的数据产生了冗余，当访问 D 类实例化出的对象中的 _a 时，会出现：`error C2385: 对“_a”的访问不明确` 的错误。通过指定类域可以解决二义性问题，例如：`D d; d.B::_a; d.C::_a;` 但这并不能解决数据冗余问题。 C++ 采用了虚拟继承来解决。



#### 解决方案

采用虚拟继承解决，虚拟继承使用到关键字：`virtual`

```cpp
class A
{
public:
    int _a = 1;
};

class B : virtual public A
{
public:
    int _b = 2;
};

class C : virtual public A
{
public:
    int _c = 3;
};

class D : public B, public C
{
public:
    int _d = 4;
};
```

且虚拟继承需要应用于腰部位置，此时的 A 类称为虚基类。



```cpp
int main()
{
    D d;
    
    std::cout << &d.B::_a << std::endl;
    std::cout << &d.C::_a << std::endl;

    return 0;
}
```

使用前， 程序运行结果为：

```cpp
00F3FDF8
00F3FE00
```

使用后，程序运行结果为：

```cpp
008FFAA0
008FFAA0
```

二义性和数据冗余的问题都被解决，现在也可以不加域作用限定符直接访问 `_a`。



#### 底层原理

在未进行虚拟继承时，&d 所看到的内存空间是这样的：

```assembly
0x005FF6F8  01 00 00 00  ....
0x005FF6FC  02 00 00 00  ....
0x005FF700  01 00 00 00  ....
0x005FF704  03 00 00 00  ....
0x005FF708  04 00 00 00  ....
```

可以发现依次存储的是：`_a _b _a _c _d`，这分别是所继承的 B 类对象，C 类对象和除此之外的 D 类对象的成员变量。

而在进行了虚拟继承后，&d 所看到的内存空间是这样的：

```assembly
0x004FFBFC  dc 7b 58 00  ?{X.
0x004FFC00  02 00 00 00  ....
0x004FFC04  e4 7b 58 00  ?{X.
0x004FFC08  03 00 00 00  ....
0x004FFC0C  04 00 00 00  ....
0x004FFC10  01 00 00 00  ....
```

可知原本存储 `_a` 的位置变成了一串地址，而 `_a` 储存在了 `0x004FFC10` 的位置。

在地址 `0x004FFBFC` 处，所存储的地址 `0x00587BDC` 指向的内存空间为：

```assembly
0x00587BDC  00 00 00 00  ....
0x00587BE0  14 00 00 00  ....
```

在地址 `0x004FFC04` 处，所储存的地址 `0x00587BE4` 指向的内存空间为：

```assembly
0x00587BE4  00 00 00 00  ....
0x00587BE8  0c 00 00 00  ....
```

它们分别在所指向地址偏移量 +4 的地方对应的是数字 20 和 数字 12。

而 `0x004FFC10 - 0x004FFBFC == 20`；`0x004FFC10 - 0x004FFC04 == 12 `。

这两个数字正好是储存 `_a` 的地址处与原本不加虚拟继承时储存 `_a` 地址处的差值，也称为偏移量。

它们的逻辑是这样的：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Inheritance-p5.png)

采用特定地址存储偏移量的做法有很多好处：从所指向的地址来看，并不仅仅存储了偏移量，还存储了偏移量外的其它内容，或许之后会存储更多其它内容，这样通过存储到地址所对应的特定空间显然是更优解，而非直接将偏移量存储在对象中。在实例化多份对象后，所有对象访问的对应地址和偏移量只有一份，这也是单独储存的好处，若直接存储在对象中，实例化一个对象就会多一份这样的数据，会导致数据冗余。

一个类进行虚拟继承后，例如 B 类虚拟继承了 A 类，A 类中的数据在 B 类中就会转换为以上所阐述的储存方式而并非直接存储，所以：

```cpp
B b;
D d;

int tmp1 = b._a;
int tmp2 = d._a;
```

这两行代码的逻辑在底层甚至是一模一样的：

```assembly
    int tmp1 = b._a;
00B41A83  mov         eax,dword ptr [b]  
00B41A86  mov         ecx,dword ptr [eax+4]  
00B41A89  mov         edx,dword ptr b[ecx]  
00B41A8D  mov         dword ptr [tmp1],edx  
    int tmp2 = d._a;
00B41A90  mov         eax,dword ptr [d]  
00B41A93  mov         ecx,dword ptr [eax+4]  
00B41A96  mov         edx,dword ptr d[ecx]  
00B41A9A  mov         dword ptr [tmp2],edx 
```

这种统一的解决方法对于计算机而言是友好的。

---

## 补充说明

* 继承和组合：继承的耦合度较高，但内部细节可见，可提供更高价值的白盒测试；组合的耦合度较低，但内部细节不可见，只能进行黑盒测试。典型的继承关系有：植物和花，人和学生；典型的组合关系有：汽车和引擎，电脑和 CPU。两者的使用需要看具体场景，但如果既符合继承又符合组合，更推荐使用组合来解决问题。

[^隐藏和重定义]: 隐藏和重定义在使用中往往认为是等价的。但隐藏是面向对象编程的专属概念，涉及派生类和基类间的关系。而重定义往往使用场景更为广泛，不仅限于面向对象，且不一定涉及继承。
