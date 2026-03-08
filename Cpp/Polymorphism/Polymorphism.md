# Polymorphism

---

## 多态概述

多态即为不同的对象调用同一方法时会表现出不同形态。例如在现实中，对于某景点的参观需要买票，成人买票是全价，而学生买票是半价。同样是买票这一行为，不同的对象调用同一方法但结果却不相同，这即为多态。

---

## 多态的使用方法及特性

### 多态的使用方法

多态的调用有两个必要条件：**1.必须使用基类的指针或引用进行调用；2.所调用的函数为虚拟函数（虚拟函数即被 virtual 修饰的函数）必须在子类中被重写（重写即函数名、参数、返回值和父类对应函数均相同，且在正常情况下，父类和子类中的该函数必须为虚拟函数）**。

```cpp
class Person
{
public:
    virtual void buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }
};

class Student : public Person
{
public:
    virtual void buy_ticket() //重写
    {
        cout << "学生 - 半价" << endl;
    }
};

int main()
{
    Person adult;
    Student student;

    Person* p; //使用父类指针进行调用（引用同理）

    p = &adult;
    p->buy_ticket();

    p = &student;
    p->buy_ticket();

    return 0;
}
```

程序的输出结果为：

```cpp
成人 - 全价
学生 - 半价
```

---

### 多态的特性

#### 协变

构成多态也可以有一个例外：**返回值不同，但为父子类的指针或引用**。例如：

```cpp
class Person
{
public:
    virtual Person* buy_ticket()
    //virtual Person& buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }
};

class Student : public Person
{
public:
    virtual Student* buy_ticket()
   	//virtual Student& buy_ticket()
    {
        cout << "学生 - 半价" << endl;
    }
};
```

它们甚至可以不需要是自己父子关系的指针或引用，其它类也可以实现协变：

```cpp
class A {};
class B : public A {};

class Person
{
public:
    virtual A* buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }
};

class Student : public Person
{
public:
    virtual B* buy_ticket()
    {
        cout << "学生 - 半价" << endl;
    }
};
```

但并不可以颠倒返回值的父子顺序，例如这样是不合法的：

```cpp
class Student;

class Person
{
public:
    virtual Student* buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }
};

class Student : public Person
{
public:
    virtual Person* buy_ticket()
    {
        cout << "学生 - 半价" << endl;
    }
};
```

即父类中必须返回的是父类指针或引用，子类中必须返回的是子类指针或引用。



#### 子类重写对 virtual 的省略

在父类已经声明了虚拟函数，子类需要对其重写时，可以不添加 virtual：

```cpp
class Person
{
public:
    virtual void buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }
};

class Student : public Person
{
public:
    void buy_ticket()
    {
        cout << "学生 - 半价" << endl;
    }
};
```

这一特性现认为应该是特意为析构函数准备的，因为在动态开辟空间时，若用父类指针指向子类对象，析构时无法完全释放而导致内存泄漏，为了解决这一问题必须让析构函数构成多态，完成 “是谁就析构谁” 的任务：

```cpp 
class Person
{
public:
    virtual void buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }

    ~Person()
    {
        cout << "~Person" << endl;
    }
};

class Student : public Person
{
public:
    void buy_ticket()
    {
        cout << "学生 - 半价" << endl;
    }

    ~Student()
    {
        cout << "~Student" << endl;
    }
};

int main()
{
    Person* stu = new Student;

    delete stu;

    return 0;
}
```

程序输出结果为：

```cpp
~Person
```

显然，若 Student 类实例化的对象中有动态开辟的空间则直接导致了内存泄漏。由于析构函数名称最后会被统一处理为 `destructor`，故在父类析构函数加上 virtual 修饰后，子类可以省略不写，个人认为这一特性是为析构函数所准备的，在一般的应用场景中，并不推荐这样做，严格按照标准实现更为合适。



#### 虚拟函数的重写和默认参数

```cpp
class Base
{
public:
    virtual void func(int data = 520)
    {
        cout << data << endl;
    }
};

class Derived : public Base
{
public:
    virtual void func(int data = 1314)
    {
        cout << data << endl;
    }
};

int main()
{
    Base* ptr = new Derived;
    ptr->func();
    
    Derived d;
    d.func();

    return 0;
}
```

以上代码的输出结果为：

```cpp
520
1314
```

用我本人的话来说，默认参数的确认取决于如何调用该函数和何时调用该函数，由于我对于此概念的研究程度并不深，在这里借用 chatGPT 的回答供作参考：

> 当调用 `ptr->func()` 时，以下步骤会发生：
>
> 1. 编译器看到 `ptr` 的类型是 `Base*`，所以它会假设 `ptr` 指向一个 `Base` 类的对象，因为它只知道 `ptr` 的静态类型。
> 2. 在编译时，编译器会查找 `Base` 类中的 `func` 函数，并根据默认参数值来确定参数是什么。在 `Base` 类中，`func` 函数的默认参数值是 520，所以编译器将此调用视为 `ptr->func(520)`。
> 3. 当程序运行时，由于 `ptr` 实际指向了一个 `Derived` 类的对象，虚拟函数机制会介入，确保调用的是 `Derived` 类中的 `func` 函数，而不是 `Base` 类中的。这就是运行时多态性的作用。
> 4. 因此，虽然编译器在编译时使用了 `Base` 类的默认参数值，但实际上在运行时调用了 `Derived` 类的函数。这导致了输出520，而不是1314。

> 当调用 `d.func()` 时，以下步骤会发生：
>
> 1. 创建了一个名为 `d` 的 `Derived` 类对象。
> 2. 然后，直接调用 `d` 对象的 `func` 函数，而不是通过指针。
> 3. 编译器会根据对象 `d` 的类型来选择要调用的函数版本。在这个情况下，对象 `d` 的类型是 `Derived`。
> 4. 因此，编译器知道要调用的函数是 `Derived` 类中的 `func` 函数，因为它是 `d` 对象的真实类型。
> 5. 在 `Derived` 类中，`func` 函数的默认参数值是 1314。
> 6. 所以，当你调用 `d.func()` 时，它会直接调用 `Derived` 类中的 `func` 函数，并且使用默认参数值1314。
> 7. 这就导致了输出结果为1314。

---

### C++11 关键字与多态的配合

#### final

C++11 中添加了 final 关键字，它可以加在虚拟函数声明后方使得该函数无法被重写，使其变为最终函数：

```cpp
class Base
{
public:
    virtual void func() final
    {
        cout << "Base_func" << endl;
    }
};

class Derived : public Base
{
public:
    virtual void func() //error: 无法重写 final 函数
    {
        cout << "Derived_func" << endl;
    }
};
```

同样，它也可以使类变为最终类，让该类无法被继承：

```cpp
class Base final {};

class Derived : public Base {}; //error: 不能将 final 类类型作为基类
```



#### override

override 保证被修饰的虚拟函数必须重写了基类的某个函数，否则会报错：

```cpp
class Base
{
public:
    virtual void func()
    {
        cout << "Base_func" << endl;
    }
};

class Derived : public Base
{
public:
    virtual void func() override
    {
        cout << "Derived_func" << endl;
    }
};
```

---

### 抽象类

在虚拟函数后加上 `= 0` 则可让该虚拟函数变为纯虚拟函数：

```cpp
class Base
{
public:
    virtual void func() = 0; //纯虚拟函数
};
```

且可以发现，该函数并不需要定义函数实现。有纯虚拟函数的类被称为抽象类（只要存在一个或更多纯虚拟函数即为抽象类），抽象类不能被实例化（或称之为接口类）：

```cpp
Base b;	//error: 不允许使用抽象类类型 "Base" 的对象
```

而抽象类被继承后，派生类也同样无法实例化出对象，除非重写了所有的纯虚拟函数。这同时也体现了抽象类存在的意义，即保证纯虚拟函数被重写，极大程度上强调了 “接口继承" 的概念。

```cpp
class Base
{
public:
    virtual void func() = 0;
};

class Derived : public Base
{
public:
    virtual void func()
    {
        cout << "纯虚拟函数必须被重写" << endl;
    }
};
```

---

## 多态的原理

### 虚拟函数表

当一个类中没有虚拟函数时，这个类实例化出的对象也不会有虚拟函数表；但如果这个类中有一个或一个以上的虚拟函数，该类实例化出的对象则会有一个虚拟函数表（采用 16 进制显示）：**（记得上传图片改路径）**

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/polymorphism-p1.png)

可以发现被 virtual 修饰的函数放入了虚拟函数表（__vfptr），而在该对象中，存放的并非虚拟函数表，而是虚拟函数表的地址。真正的虚拟函数表存放在代码段（常量区），可通过打印地址对比验证。

---

### 重写

对于这两个类：
```cpp
class A
{
protected:
    int _a = 1;
    int _b = 2;

public:
    virtual void func1() {}
    virtual void func2() {}
    virtual void func3() {}
};

class B : public A
{
protected:
    int _a = 3;
    int _b = 4;

public:
    virtual void func1() {}
    virtual void func2() {}
    void func4() {}
    virtual void func5() {}
};
```

B 类会对 A 类的 func1 函数和 func2 函数完成重写。它们实例化出的对象是这样的：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/polymorphism-p2.png)

B 类继承了 A 类，实际上也将虚拟函数表单独拷贝后为自己所有（虽然显示在 A 类中，实际上是 B 类的虚拟函数表，这是 VS 所做的处理）。由于在 B 类中重写了 A 类的 func1 函数和 func2 函数，可以发现 B 类的虚拟函数表中下标 0 所对应的 func1 和下标 1 所对应的 func2 地址与 A 类的虚拟函数表中的地址不同，但由于下标 2 对应的 func3 函数没有被重写，故直接被 B 类继承，两个类的虚拟函数表中 func3 的地址是一样的。

所以重写的本质就是将父类的虚拟函数表拷贝，若发现有匹配的被重写的虚拟函数，则会覆盖掉子类虚拟函数表中的对应函数。故在语法层面上，我们称之为重写，从原理层来说，它被称之为覆盖。

在 B 类中同时还有一个虚拟函数 func5，在 B 类的虚拟函数表中并没有它的身影，实际上这是我所使用的编译器（VS2022）的一些处理不当，当使用内存查看时更为清晰：

```assembly
0x00AD9B54  8c 15 ad 00  ?.?.
0x00AD9B58  87 15 ad 00  ?.?.
0x00AD9B5C  aa 15 ad 00  ?.?.
0x00AD9B60  a5 15 ad 00  ?.?.
0x00AD9B64  00 00 00 00  ....
```

当然，这个以 0 结尾的特性在不同的环境下可能会不同，例如在陈皓老师的博客中，提及到：

> 这个结束标志的值在不同的编译器下是不同的。在WinXP+VS2003下，这个值是NULL。而在Ubuntu 7.10 + Linux 2.6.22 + GCC 4.1.3下，这个值是如果1，表示还有下一个虚函数表，如果值是0，表示是最后一个虚函数表。

但考虑到 VS 可能使用增量编译，当修改一些内容后，这个特性或许也会消失，重新生成解决方案即可解决。

---

### 实现继承和接口继承

当谈及这些原理时，会发现一个函数从父类继承到子类，若不是虚拟函数，则会直接继承其实现，当使用子类调用该函数时会对子类指针进行切片后调用父类中的该函数，这便是实现继承。但若是虚拟函数，子类从父类继承的是虚拟函数表，这是一种接口继承，其最终目的是为了重写而实现多态，这称之为接口继承。故不需要实现多态时，尽量不要把函数定义为虚拟函数。

---

### 多态的实现

在实现多态时，前面提出了两个必要条件：1.必须通过父类指针或引用调用；2.调用的函数在父类中必须是虚拟函数且在子类中被重写。

#### 必须是父类指针或引用调用

父类对象可以同时接收父类对象和子类对象，且会对子类对象进行切片，故使用父类对象保证了它可以同时接收两类对象。而使用父类对象的指针或引用时，父类对象不需要额外处理，子类对象会进行切片，让该指针或引用所指向的是子类中的父类部分，这一部分同时包含虚拟函数表。也就是说，父类的指针或引用接收到子类时，虚拟函数表也会变为子类的，故这个时候进行函数调用，若构成多态，则会通过虚拟函数表查找到对应函数调用，在父类虚拟函数表中查找到的就是父类的函数，在子类虚拟函数表中查找到的就是子类重写的函数。你会发现这套执行逻辑是一模一样的，而不一样的只是函数地址罢了（也就是上述对 [重写](# 重写) 的阐述中提到的重写后子类的虚拟函数表中地址会变化，即发生了覆盖），以下提供验证：

```cpp
class Person
{
public:
    virtual void buy_ticket()
    {
        cout << "成人 - 全价" << endl;
    }
};

class Student : public Person
{
public:
    virtual void buy_ticket()
    {
        cout << "学生 - 半价" << endl;
    }
};

int main()
{
    Person adult;
    Student student;

    Person* ptr_p = &adult;
    Person* ptr_s = &student;

    ptr_p->buy_ticket();
    ptr_s->buy_ticket();

    return 0;
}
```

该代码中，`ptr_p->buy_ticket();` 和 `ptr_s->buy_ticket()` 的执行逻辑是一模一样的：

```assembly
    ptr_p->buy_ticket();
002C5E3B  mov         eax,dword ptr [ptr_p]  
002C5E3E  mov         edx,dword ptr [eax]  
002C5E40  mov         esi,esp  
002C5E42  mov         ecx,dword ptr [ptr_p]  
002C5E45  mov         eax,dword ptr [edx]  
002C5E47  call        eax  
002C5E49  cmp         esi,esp  
002C5E4B  call        __RTC_CheckEsp (02C130Ch)  
    ptr_s->buy_ticket();
002C5E50  mov         eax,dword ptr [ptr_s]  
002C5E53  mov         edx,dword ptr [eax]  
002C5E55  mov         esi,esp  
002C5E57  mov         ecx,dword ptr [ptr_s]  
002C5E5A  mov         eax,dword ptr [edx]  
002C5E5C  call        eax  
002C5E5E  cmp         esi,esp  
002C5E60  call        __RTC_CheckEsp (02C130Ch) 
```

假如不使用父类指针或引用，则必然会产生拷贝，而**虚拟函数表是不会被拷贝的**（若虚拟函数表被拷贝，则父类的虚拟函数表是子类的，错误显而易见），故产生拷贝时，仅是将子类对象的父类部分进行拷贝，虚拟函数表不变，自然无法构成多态，无论如何都会调用父类函数。



#### 子类必须对父类的虚拟函数完成重写

显而易见，必须是虚拟函数是为了让该函数进入虚拟函数表。子类必须完成重写是为了让重写的函数覆盖掉原本从父类中继承的虚拟函数表的中该函数，实现多态。



#### 动态绑定和静态绑定

实际上函数重载也是一种多态形式，可以根据不同的数据类型决定调用哪个函数，而编译器完成对程序行为的确认是在编译阶段或链接阶段（若声明定义分离就是在链接阶段）完成的，编译器可以在程序运行前确认这一行为（前期绑定），称之为静态多态；而在通过虚拟函数表调用函数实现多态时，编译器在程序运行前根本无法确认程序行为（在 [必须是父类指针或引用调用](# 必须是父类指针或引用调用) 中已经证明，它们的执行逻辑是固定的），在运行是才知道自己调用的是谁（后期绑定），这称之为动态多态。

---

### 多继承的虚拟函数表

进行多继承时，子类会继承多个虚表：

```cpp
class Base1
{
public:
	int _data1 = 1;

	virtual void func1() {}
	virtual void func2() {}
};

class Base2
{
public:
	int _data2 = 2;

	virtual void func1() {}
	virtual void func3() {}
};

class Derived : public Base1, public Base2
{
public:
	int _data3 = 3;

	virtual void func1() {}
	virtual void func4() {}
};
```

这里我采用内存查看，并加以批注，因为 VS 无疑又要不显示某些虚拟函数了：

```assembly
; Base1 的虚拟函数表
0x00ED9B44  0e 16 ed 00 ; Base1::func1(void)
0x00ED9B48  1d 16 ed 00 ; Base1::func2(void)
```

```assembly
; Base2 的虚拟函数表
0x00ED9B54  22 16 ed 00 ; Base2::func1(void)
0x00ED9B58  09 16 ed 00 ; Base2::func3(void)
```

```assembly
; Derived 的虚拟函数表
; Base1 部分
0x00ED9B34  13 16 ed 00 ; 被重写的 func1
0x00ED9B38  1d 16 ed 00 ; 原本 Base1::func2(void)
0x00ED9B3C  ff 15 ed 00 ; 这是 Derived::func4(void)

; Base2 部分
0x00ED9B60  fa 15 ed 00 ; 被重写的 func1
0x00ED9B64  09 16 ed 00 ; 原本 Base2::func3(void)
```

可以发现若重写，会对每个虚拟函数表中的匹配函数都进行重写，且在派生类中的虚拟函数会被添加至第一张虚拟函数表的末尾。

在这里的 func1 被重写后，地址也各不相同，实际上是 VS2022 进行了多重跳转最后到达正确的位置。而本分析也主要以 VS2022 举例，其它编译器肯定会有不同的情况出现。

```cpp
Derived d;

Base1* ptr1 = &d;
Base2* ptr2 = &d;
Derived* ptr3 = &d;

ptr1->func1();
ptr2->func1();
ptr3->func1();
```

无疑，它们都能正确调用到 Derived 中的 func1，但实现逻辑不尽相同，例如在 VS2022 中，通过 `Base2*` 去调用 func1 时，由于需要的是 Derived 的 this 指针，故 VS2022 进行了多重跳转和对储存 this 指针的 ecx 寄存器进行了偏移操作，使其传递正确的 this 指针：

```assembly
006A1B75  sub         ecx,8  ; 偏移到正确的 this 位置
006A1B78  jmp         Derived::func1 (06A1613h)
```

当然，不同的编译器处理方式不同，所以我并不详细展开这些。

至于菱形继承和菱形虚拟继承的虚拟函数表的处理，我认为首先在实际中会杜绝这种场景的出现，其次这属于 C++ 比较令人诟病的部分，学习的价值并不大。若有学习需求或对此感兴趣，请参考陈皓老师文章：[C++ 对象的内存布局](https://coolshell.cn/articles/12176.html)

---



