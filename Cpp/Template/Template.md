# Template

---

## 什么是模板

模板是 C++ 于 C 语言十分重要的一种差异和提升，造就了 C++ 泛型编程的思维。通过函数模板实例化出不同的函数，类模板实例化出不同的类，以满足在处理不同数据类型时的种种要求。那 C 语言中的顺序表举例，当实现一份顺序表时往往仅能指定一种明确的数据类型，当需要对另一种数据类型进行处理时，则需要再实现一份特定类型的顺序表，而模板仅需要实现一份，通过编译器自行实例化得到。这两者从底层逻辑上来讲并无二至，只是将重新实现一份不同类型的类或函数的这个过程交给了编译器完成。但从代码上而言，增加了代码的可读性，由于编译器需要做更多的检查和处理，同时它也在一定程度上降低了编译器的速度。

---

## 模板的使用

### 函数模板

```cpp
template <class T1, typename T2>
void func(const T1& e1, const T2& e2) {}
```

模板需要使用模板关键字：`template` ，这里的 T1 与 T2 则是类型名，而定义类型名既可以用 class 也可以用 typename，个人更为推荐 typename，表意较明确。而这里的类型是根据调用时所传递的参数类型进行实例化，有多少种情况就会实例化出多少个 func 函数。

例如，swap 函数不必再命名为：`swap_int, swap_double`，而是可以直接这样实现：

```cpp
template <typename T>
void swap(T& e1, T& e2)
{
    T tmp = e1;
    e1 = e2;
    e2 = tmp;
}
```



#### 隐式实例化

当产生这样的调用时：

```cpp
int a = 10, b = 20;
swap(a, b);
```

编译器会自动实例化出（这一过程称之为隐式实例化，由编译器自行推导类型）：

```cpp
void swap(int& e1, int& e2)
{
    int tmp = e1;
    e1 = e2;
    e2 = tmp;
}
```

其它类型也会对应实例化，不做过多说明，但这也验证了上述：和 C 语言写出 swap_int，swap_double 并无二至，只是编译器代替了我们完成了这一工作。



#### 显式实例化

如果所传递的两个参数类型不一样，自然会报错，这时候采取强制类型转换可以解决问题，也可以采用显式实例化，这里并不能拿 swap 举例，因为不管是强制类型转换还是显式实例化所触发的隐式类型转换都将产生具有常性的临时变量，故需要有 const 属性才能接收，但这样并不能完成交换，故以 add 函数举例：

```cpp
template <typename T>
T add(const T& e1, const T& e2)
{
    return e1 + e2;
}
```

```cpp
int a = 10;
int b = 3.1415926;
add<int>(a, b); //显式实例化
```

这时 b 会隐式转换为 int 类型，结果为 13。

需要注意，用两个 int 调用的 add 函数和 两个 double 调用的 add 函数并不是一个函数，它们分别实例化成两个函数并构成函数重载。



#### 调用优先原则

当已经实现了特定类型的某函数和该函数的函数模板时，调用特定类型的该函数会优先调用已经实现的版本，而不是用编译器实例化。且若编译器再次实例化会导致函数名冲突。**效率至上原则**。

```cpp
template <typename T>
T add(const T& e1, const T& e2)
{
    return e1 + e2;
}

int add(int e1, int e2)
{
    return e1 + e2;
}
```

调用：`add(int(1), int(2))`，会调用后者而并非使用前者实例化。

编译器会倾向于最优匹配（**最优匹配原则**）：

```cpp
template <typename T>
T add(const T& e1, const T& e2)
{
    return e1 + e2;
}

int add(size_t e1, int e2)
{
    return e1 + e2;
}
```

调用：`add(size_t(1), size_t(2))`，会调用前者实例化出 T 为 size_t 的类型的函数调用，这相比于后者是更加匹配的版本。

调用：`add(int(1), size_t(2))`，这时会调用后者，因为前者产生了参数不匹配，无法编译通过，可以理解为后者仍然是仅存的最优匹配。



#### 函数模板的特化

首先需要说明，函数模板的实例化和特化是不同的概念，尽管它们都涉及到函数模板的实例化过程。函数模板的实例化是根据推导类型实例化出对应类型的函数，而特化则是针对特定类型进行特殊处理的一种逻辑。

##### 函数模板的全特化

```cpp
template <typename T1, typename T2>
void func(const T1& e1, const T2& e2)
{
    cout << "void func(const T1& e1, const T2& e2)" << endl;
}

template <> //全特化
void func<int, double>(const int& e1, const double& e2)
{
    cout << "void func<int, double>(const int& e1, const double& e2)" << endl;
}
```

此时 对于 int 和 double 类型的调用会被特殊处理，调用全特化部分的内容。

**请注意，函数模板是不存在偏特化的。**

但这样可以实现类似偏特化的效果：

```cpp
template <typename T1, typename T2>
void func(const T1& e1, const T2& e2)
{
    cout << "void func(const T1& e1, const T2& e2)" << endl;
}

template <typename T2>
void func(const int& e1, const T2& e2)
{
    std::cout << "void func<int, T2>(const int& e1, const T2& e2)" << std::endl;
}
```



我认为函数模板的特化其实意义不大。利用函数重载，由于优先原则的存在就可以在绝大部分情况下替代函数模板的特化，这里也只是点明这一特性的存在，在实际应用中这一特性的应用较少。

---

### 类模板

#### 类模板必须显式实例化

与函数模板类似，使用类模板也可以通过不同的数据类型实例化出不同的类，但类仅能显式实例化：
```cpp
template <typename T>
class Thepale
{
private:
    T _val;
};
```

当需要使用类模板时，必须显式告知类型：

```cpp
Thepale<int> e;
```

因为编译器在创建类时无法通过推导完成这一过程。



#### 类的声明与定义不可分离

假设在 thepale.h 中：
```cpp
template <typename T>
class Thepale
{
public:
    Thepale(const T& val = T())
        :_val(val)
    {}

    void func(const T& val);

private:
    T _val;
};
```

在 thepale.cpp 中：

```cpp
template <typename T>
void Thepale<T>::func(const T& val)
{
    cout << "void Thepale<T>::func()" << endl;
    cout << "The val is:" << val << endl;
}
```

在 test.cpp 中调用：`Thepale e; e.func(1);` 会出现：

```cpp
error LNK2019: 无法解析的外部符号 "public: void __cdecl Thepale<int>::func(int const &)" (?func@?$Thepale@H@@QEAAXAEBH@Z)，函数 main 中引用了该符号
```

的错误。

<img src="https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Template-p1.png" style="zoom:80%;" />

原因是定义的 func 函数在符号表中没有产生对应的符号，因为 T 类型是不确定的，这无法生成可匹配的函数名。而在 main 函数中调用 func 函数时已知传递参数是 int 类型，则按照 int 类型和特定的函数名修饰规则生成的函数名去符号表中通过该名称匹配对应地址，但在 thepale.cpp 中所定义的函数是一个模板，链接阶段无法利用函数名修饰规则生成对应匹配的名称，因为它并不知道 T 是什么类型，所以导致了链接错误。

细致点来说，thepale.h 分别在 test.cpp 和 thepale.cpp 中展开，而 Thepale 在调用 func 函数前已经在 main 函数所在文件以特定类型（这里是int）实例化出了类，而调用 func 时由于只有函数声明，故需要在符号表中匹配 int 类型的该函数名。而在 thepale.cpp 中展开的 thepale.h 和在 test.cpp 中展开的不一样，thepale.cpp 中没有途径知道 T 的类型，便导致了这一错误。

解决方案也十分简单，只需要在 thepale.cpp 中显式实例化出一个 int 类型的 Thepale 类即可：

```cpp
template
class Thepale<int>;
```

但这种解决方法显然是不明智的，对于多个类型就需要多次显式实例化，治标不治本。故在同一文件内实现声明和定义分离即可（例如仅在 thepale.h 中分离）。



#### 非类型模板参数

在定义模板时不仅仅只能定义类型的模板，也可以定义非类型模板：

```cpp
template<typename T, size_t N>
class Thepale
{
private:
    T _arr[N];
};
```

这里的 N 是一个常量，不可修改。在该类中，传递不同的 N，就可以生成大小固定的 T 类型数组 _arr：

```cpp
Thepale<int, 100> e1;
Thepale<double, 20> e2;
Thepale<double, 1> e3;
```

而非类型模板参数的类型只能是整型或整型家族的类型，因为这个参数设计的目的仅是为了开辟对于大小的数组空间。例如在位图中：

```cpp
template <size_t N> class bitset;
```

array 中（顺便吐槽这个容器真的毫无价值）：

```cpp
template <class T, size_t N> class array;
```



#### typename

在这种情况下：

```cpp
template<typename T>
class Thepale
{
public:
    typedef T* iterator;

    static int a;
};

template<class T>
int Thepale<T>::a = 1;

template<typename Container>
void func(const Container& con)
{
    Container::iterator it;
}

int main()
{
    Thepale<int> e;
    func(e);

    return 0;
}
```

调用 func 函数时，`Container::iterator` 编译器会不知道这一步是找的类型还是类似于 a 这样的静态变量，如果是静态变量则会产生语法错误，如果是类型则没有问题，故一般需要指明这里是类型，否则编译将不通过：`typename Container::iterator it;` **用 typename 告诉编译器这是一个类型名**。



#### 类模板的特化

##### 类模板的全特化

```cpp
template<typename T1, typename T2>
class Thepale
{
private:
    T1 _a = T1();
    T2 _b = T2();

public:
    void print()
    {
        cout << _a << _b << endl;
    }
};

template <>
class Thepale<int, double> //对 int 和 double 类型做全特化
{
private:
    int _a = 520;
    double _b = 1314;

public:
    void print()
    {
        cout << "Full Specialization" << endl;
        cout << _a << _b << endl;
    }
};
```

当需要实例化出 `Thepale<int, double>` 的对象时，则会转而实例化全特化的对象，且通过实例化对象调用的函数都是全特化中单独实现的：

```cpp
int main()
{
    Thepale<int, double> e;
    e.print();

    return 0;
}
```

程序输出为：
```cpp
Full Specialization
5201314
```

故全特化可以对 int 和 double 类型的 Thepale 对象做特殊处理。



在实现优先级队列时，仿函数针对类对象的指针（例如日期类对象的指针）进行比较往往会进行地址比较，而这时使用类模板特化，让所有的指针类型进行特殊处理，对对象进行比较而非指针，则是一种不错的优化方式：

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

template <typename T>
class Less<T*>
{
public:
    bool operator()(const T* e1, const T* e2)
    {
        return *e1 < *e2;
    }
};
```

这是一种偏特化，具体目的是为了对类型进行进一步的限制（例如限制为指针类型）。且特化版本的模板参数 T 是按照原始模板参数列表进行推导的，也就是说，首先指针匹配的是 `Less<T*>` 这个偏特化类型，而 `<typename T>` 中的 T 是根据 T* 推导的。故 T 的类型是对象的类型而非指针，这里较容易混淆。 



##### 类模板的偏特化

```cpp
template<typename T1, typename T2>
class Thepale
{
private:
    T1 _a = T1();
    T2 _b = T2();

public:
    void print()
    {
        cout << _a << _b << endl;
    }
};

template <typename T2>
class Thepale<int, T2> //对 int 类型实现偏特化
{
private:
    int _a = 52;
    T2 _b = T2();

public:
    void print()
    {
        cout << "Partial Specialization" << endl;
        cout << _a << _b << endl;
    }
};
```

只要是需要实例化第一个参数为 int 的 Thepale 对象，则会调用该偏特化，原理和全特化一致。

```cpp
int main()
{
    Thepale<int, double> e1;
    e1.print();

    Thepale<int, char> e2;
    e2.print();

    return 0;
}
```

程序输出为：

```cpp
Partial Specialization
520
Partial Specialization
520
```

---

## 总结

* 函数模板的实例化遵循优先和效率至上原则。
* 函数模板只能全特化而不能偏特化，函数模板的特化较为鸡肋，用重载可以替代。
* 类模板必须显式实例化。
* 类模板不适合声明定义分离。
