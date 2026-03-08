# Reference

---

## 什么是引用

在 C++ 中，引用即取别名，例如：孙悟空、齐天大圣、美猴王、孙行者都指的是孙悟空；本质都表示一个已存在变量或对象的别名，它们都访问同一块内存空间。引用底层仍然采用指针实现，在一些情况下优化了代码质量。

---

## 引用的性质

```cpp
int a = 0;
int& b = a;
```

以上代码表示 b 是 a 的引用，即 b 是 a 的别名。

```cpp
int a = 0;
int& b = a;

a = 1;
std::cout << a << " ";

b = 2;
std::cout << a;
```

以上代码输出结果为：`1 2`
可知更改别名也可更改变量本身，即如果孙悟空吃饭了，那么齐天大圣也吃饭了，美猴王也吃饭了，是一个道理。



```cpp
int a = 0;
int& b = a;
int& c = b;
int& d = b;
```

多次引用也是允许的，别名是可以起多个的。



```cpp
double a = 0;
int& ra = a;
```

不同类型是不可以引用的（a 生成临时变量给 ra 时存在权限放大，相当于把 const int& 给 int&，这里因为已经编译错误，不深究），当然如果你愿意可以强制类型转换，但请尽量杜绝这种操作：

```cpp
char a = 0;
int& ra = (int&)a;

ra = 10000;
```

这样会导致程序崩溃，非法内存访问。



### 引用在定义时必须初始化

```cpp
int a = 0;

//error
int& ra;
ra = a;

//correct
int* pa;
pa = &a;
```

指针可以先定义不初始化，后续再赋值。但引用必须在定义时就初始化，不允许赋值，这也就是下一条性质。



### 引用不允许改变引用对象

```cpp
int a = 0;
int b = 1;

int& ra = a;
ra = b; //1.赋值 //2.改变引用对象

int* pa = &a;
pa = &b;
```

引用在定义时必须初始化，换言之即引用在定义时必须确定引用对象，一旦确定就不可更改，所以这一步是给  ra 别名赋值为 1。但指针是可以改变指向的。

---

## 函数中的引用

### 引用做函数形式参数

由于引用的特性，它可以在一定程度上替代指针，拿 swap 函数举例：

```cpp
void swap(int* e1, int* e2) //指针版本
{
	int tmp = *e1;
	*e1 = *e2;
	*e2 = tmp;
}

void swap(int& e1, int& e2) //引用版本
{
    int tmp = e1;
    e1 = e2;
    e2 = tmp;
}
```

可知引用省去了指针繁琐的解引用操作。



```cpp
void func(int a);
void func(int& a);
```

以上两个函数是构成函数重载的，一个是 int 类型，一个是 int 引用类型，类型不同而构成重载，但如果有：`func(1)` 这样的调用会导致调用不明确。



### 引用做函数返回参数

对比引用返回和传值返回：

```cpp
int func()
{
    static int a = 0;
    ++a;
    
    return a;
}

int& func()
{
    static int a = 0;
    ++a;
    
    return a;
}
```

对于传值返回，a 只会随着调用次数的增加而变大：

```cpp
std::cout << func() << " ";
std::cout << func() << " ";
std::cout << func() << " ";
```

输出结果为：`1 2 3 `，无法通过其它手段更改 a 的值。



在传引用返回时，可以直接对 a 进行更改：

```cpp
func() += 100;

std::cout << func() << " ";
std::cout << func() << " ";
std::cout << func() << " ";
```

输出结果为：`102 103 104 `



首先观测传值返回的汇编代码：

```cpp
//程序代码如下：
int func()
{
    static int a = 0;
    ++a;

    return a;
}

int main()
{
    int ret = func();

	return 0;
}
```



```assembly
int func()
{
;函数入栈操作
00007FF7BE232250  push        rbp  
00007FF7BE232252  push        rdi  
00007FF7BE232253  sub         rsp,0E8h  
00007FF7BE23225A  lea         rbp,[rsp+20h]  
00007FF7BE23225F  lea         rcx,[__385AFBA2_test@cpp (07FF7BE243068h)]  
00007FF7BE232266  call        __CheckForDebuggerJustMyCode (07FF7BE2313E8h)  
;入栈完成

    static int a = 0;
    ++a;
00007FF7BE23226B  mov         eax,dword ptr [a (07FF7BE23D170h)]  
00007FF7BE232271  inc         eax  
00007FF7BE232273  mov         dword ptr [a (07FF7BE23D170h)],eax  

    return a;

00007FF7BE232279  mov         eax,dword ptr [a (07FF7BE23D170h)]  ;将 a 的值移入寄存器 eax
}
;函数出栈操作
00007FF7BE23227F  lea         rsp,[rbp+0C8h]  
00007FF7BE232286  pop         rdi  
00007FF7BE232287  pop         rbp  
00007FF7BE232288  ret 
;出栈完成

int main()
{
00007FF7BE230FD2  push        rdi  
00007FF7BE230FD3  sub         rsp,0E8h  
00007FF7BE230FDA  lea         rbp,[rsp+20h]  
00007FF7BE230FDF  lea         rcx,[__385AFBA2_test@cpp (07FF7BE243068h)]  
00007FF7BE230FE6  call        __CheckForDebuggerJustMyCode (07FF7BE2313E8h)  
    int ret = func();
00007FF7BE230FEB  call        func (07FF7BE231253h)  
00007FF7BE230FF0  mov         dword ptr [ret],eax ;将 eax 的值给 ret 完成函数调用

	return 0;
00007FF7BE230FF3  xor         eax,eax  
}
00007FF7BE230FF5  lea         rsp,[rbp+0C8h]  
00007FF7BE230FFC  pop         rdi  
00007FF7BE230FFD  pop         rbp  
00007FF7BE230FFE  ret  
```

以上操作可知，在传值返回时，返回的是 a 的临时拷贝 eax，也称之为临时变量。



传引用返回时（去除了函数入栈和出栈的代码，若有需求见传值返回情况）：

```cpp
//程序代码如下：
int& func()
{
    static int a = 0;
    ++a;

    return a;
}

int main()
{
    int& ret = func();

	return 0;
}
```



```assembly
int& func()
{
    static int a = 0;
    ++a;
00007FF6A1D81E5B  mov         eax,dword ptr [a (07FF6A1D8D170h)]  
00007FF6A1D81E61  inc         eax  
00007FF6A1D81E63  mov         dword ptr [a (07FF6A1D8D170h)],eax  

    return a;
00007FF6A1D81E69  lea         rax,[a (07FF6A1D8D170h)]  ;将 a 的地址给了 rax (lea = load effective address)
}

int main()
{ 
    int& ret = func();
00007FF7374D204B  call        func (07FF7374D145Bh)  
00007FF7374D2050  mov         qword ptr [ret],rax  ;直接将 rax 存储的地址给了 ret，使得 ret 可以更改这个地址中的值

	return 0;
00007FF6A1D82055  xor         eax,eax  
}
```

以上操作可知，传引用返回没有发生临时拷贝，而是直接把 a 的地址传回来（如果表层来讲，就是把 a 的引用传回来给了 ret 这个别名），ret 接收到这个地址并可对其值实施更改（如果表层来讲，ret 就是 a 的别名），这也揭示了引用的底层实现是靠指针完成的。

如果将以上代码替换为：

```cpp
int* func()
{
    static int a = 0;
    ++a;

    return &a;
}

int main()
{
    int* ret = func();

	return 0;
}
```

**会发现其汇编代码和传引用返回一模一样**：

```assembly
int* func()
{
    static int a = 0;
    ++a;
00007FF757AA1E5B  mov         eax,dword ptr [a (07FF757AAD170h)]  
00007FF757AA1E61  inc         eax  
00007FF757AA1E63  mov         dword ptr [a (07FF757AAD170h)],eax  

    return &a;
00007FF757AA1E69  lea         rax,[a (07FF757AAD170h)]  
}

int main()
{
    int* ret = func();
00007FF757AA204B  call        func (07FF757AA1460h)  
00007FF757AA2050  mov         qword ptr [ret],rax  

	return 0;
00007FF757AA2054  xor         eax,eax  
}
```

故更进一步揭示了指针和引用的紧密关系。同时，会发现返回值为指针时和传引用返回一样没有发生临时拷贝，故**C++ 一旦发现返回值是指针或引用便不会生成临时拷贝**，哪怕需要的正确返回是 int\*\*，而你返回了 int\* 也不会发生临时拷贝。



当然，以此衍生出的问题需要提点一些，例如：

传引用返回后用 int 类型接收会如何？这个过程相当于：

```cpp
int a = 0;
int& ra = a;
int b = ra;
```

很明显，b所得到的是 a 的值，进行了赋值操作而已。而 int& 和 int 不是同一类型，编译器会发生隐式类型转换，在这一过程中，底层实际上是对 ra 地址解引用获得所指向的值，然后赋值给临时变量 eax（或许，反正是某一寄存器），再由 eax 赋值给 b。

在进行类似这样的比较时：

```cpp
int a = 1;
double b = 1.11;

if(b > a) {std::cout << "1";}
```

比较时 a 就会发生隐式类型转换，和 b 比较的实际上是 a 所赋值的 double 类型的临时变量。



如果：

```cpp
int func() 
{
    int a = 521;
    return a;
}

int main()
{
    //int& ra = func(); //error
    const int& ra = func();
    
    return 0;
}
```

尽管这是一个错误代码（func 结束后 a 已销毁），但可以揭示：**临时变量具有常性**。

---

## 常引用权限问题

```cpp
const int a = 0;
int& b = a; // error
```

此代码编译不通过，a 是一个常变量，而 b 是对变量的引用，这样会导致权限放大，是禁止的。

```cpp
const int a = 0;
const int& b = a;
```

此代码编译通过，a 是常变量，b 是对常变量的引用，这属于权限的平移，是允许的。

```cpp
int a = 0;
const int& b = a;
```

此代码编译通过，a 是变量，b 是对常变量的引用，这属于权限的缩小，是允许的。（更改 a 的值也会更改 b，只是通过 b 无法更改 a）

**权限是可以被平移和缩小的，权限不能放大。**

---

## 总结

* 引用不可以改变引用对象。
* 一个变量或对象可以有多个引用。
* 引用在定义时必须初始化。
* 不同类型不可以引用。
* 权限不允许放大，可以缩小和平移。
* 临时变量具有常性。

---

## 补充说明

* 不对比指针和引用的区别，总结引用相关特性后即可明白，无需赘述。
