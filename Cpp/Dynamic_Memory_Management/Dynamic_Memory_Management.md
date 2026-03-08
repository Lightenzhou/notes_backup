# Dynamic_Memory_Management

---

## 什么是动态内存管理

C / C++ 拥有多种数据类型，不同的数据类型将被分配在不同的内存区域：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Dynamic_Memory_Management-p1.png)

例如 int，指针类型数据被存放在栈区；static 和 全局数据存放在数据段。而动态内存开辟的空间存放在堆区。在 C 语言中，用于动态内存管理的库函数有：`malloc` `calloc` `realloc` `free`，而在 C++ 中，将引入：`new` `delete` `new[]` `delete[]`。具体原因是 C++ 引入了类和对象，仅用 C 语言的动态内存管理无法和类和对象完美契合，故有了它们的产生。

---

## new & delete - 内置类型

### new

当需要动态开辟内置类型时，new 的使用方法为：

```cpp
type* p = new type;
```

不需要进行强制类型转换，因为在开辟空间时已经指定了类型，利用模板即可返回对应类型，这意味着用 auto 接收也是可以的。

在开辟的同时可以进行初始化：

```cpp
int* p = new int(521);
```

内置类型初始化的方式为：`(val)`。

### delete

动态开辟的空间需要释放，而 delete 正是起到这一作用：

```cpp
delete name;
```

### new[]

用 new 同时也可以开辟数组类型的空间：

```cpp
type* parr = new type[size];
```

开辟数组类型同时也支持初始化：

```cpp
int* parr = new int[1024]{1,2,3};
```

内置类型的初始化的方式为：`{p1, p2, p3}`。

### delete[]

开辟的数组空间需要用 delete[] 释放：

```cpp
delete[] name;
```



可见对于内置类型而言，其与 C 语言的动态开辟并无太大区别，可以进行开辟时初始化，开辟方式更为简洁值得称赞。不过在 C 语言中，动态内存开辟使用的是函数，而 C++ 中则是操作符，这一点需要注意。更重要的是它们在自定义类型中的特性。

---

## new & delete - 自定义类型

以下内容基于此类讲解：

```cpp
class Thepale
{
private:
    int _data1;
    int _data2;

public:
    Thepale(int data1 = 0, int data2 = 0)
        :_data1(data1)
        ,_data2(data2)
    {
        cout << "构造函数被调用" << endl;
    }

    Thepale(const Thepale& e)
    {
        _data1 = e._data1;
        _data2 = e._data2;
        cout << "拷贝构造函数被调用" << endl;
    }

    ~Thepale()
    {
        cout << "析构函数被调用" << endl;
    }
};
```

可以通过 new 创建对象：

```cpp
int main()
{
    Thepale* p = new Thepale;
    
    return 0;
}
```

程序的输出结果为：

```cpp
构造函数被调用
```

可以发现 **new 创建对象会调用默认构造函数**。

可以通过 delete 销毁对象：

```cpp
int main()
{
    Thepale* p = new Thepale;
    delete p;
    
    return 0;
}
```

程序的输出结果为：

```cpp
构造函数被调用
析构函数被调用
```

可以发现 **delete 销毁对象会调用析构函数**。

在通过 new 创建对象时仍然可以进行传值构造对象：

```cpp
int main()
{
    Thepale* p = new Thepale(521, 1314);
    delete p;
    
    return 0;
}
```

可以使用 new[] 构建对象数组并初始化，并使用 delete[] 销毁对象数组：

```cpp
int main()
{
    Thepale* parr = new Thepale[10]{ {1, 2}, {3, 4} }; //若写成{(1, 2), (3, 4)} 小括号会被当做表达式
    delete[]  parr;
    return 0;
}
//--------------------------------------------------------------或者
int main()
{
    Thepale* parr = new Thepale[10]{ Thepale(1, 2), Thepale(3, 4) };
    delete[]  parr;
    return 0;
}
```

以上无论哪种方式在编译器优化后都是一次拷贝构造。

---

## 底层探究

用以下代码进行分析：

```cpp
int main()
{
    Thepale* p = new Thepale;
    delete p;

    return 0;
}
```

new 对应的汇编：

```assembly
    Thepale* p = new Thepale;
00007FF6C62623AB  mov         ecx,8  
;00007FF6C62623B0  call        operator new (07FF6C626104Bh)  
00007FF6C62623B5  mov         qword ptr [rbp+108h],rax  
00007FF6C62623BC  cmp         qword ptr [rbp+108h],0  
00007FF6C62623C4  je          main+50h (07FF6C62623E0h)  
00007FF6C62623C6  xor         r8d,r8d  
00007FF6C62623C9  xor         edx,edx  
00007FF6C62623CB  mov         rcx,qword ptr [rbp+108h]  
;00007FF6C62623D2  call        Thepale::Thepale (07FF6C62610C3h)  
00007FF6C62623D7  mov         qword ptr [rbp+138h],rax  
00007FF6C62623DE  jmp         main+5Bh (07FF6C62623EBh)  
00007FF6C62623E0  mov         qword ptr [rbp+138h],0  
00007FF6C62623EB  mov         rax,qword ptr [rbp+138h]  
00007FF6C62623F2  mov         qword ptr [rbp+0E8h],rax  
00007FF6C62623F9  mov         rax,qword ptr [rbp+0E8h]  
00007FF6C6262400  mov         qword ptr [p],rax
```

已用注释标出，发现 **new 先调用了 operator new，然后调用构造函数**。

delete 对应的汇编：

```assembly
    delete p;
00007FF6C6262404  mov         rax,qword ptr [p]  
00007FF6C6262408  mov         qword ptr [rbp+128h],rax  
00007FF6C626240F  cmp         qword ptr [rbp+128h],0  
00007FF6C6262417  je          main+0A3h (07FF6C6262433h)  
00007FF6C6262419  mov         edx,1  
00007FF6C626241E  mov         rcx,qword ptr [rbp+128h]  
;00007FF6C6262425  call        Thepale::`scalar deleting destructor' (07FF6C626153Ch)  这一层是封装，进入查看
00007FF6C626242A  mov         qword ptr [rbp+138h],rax  
00007FF6C6262431  jmp         main+0AEh (07FF6C626243Eh)  
00007FF6C6262433  mov         qword ptr [rbp+138h],0 

ForCppTest.exe!Thepale::`scalar deleting destructor'(unsigned int):
00007FF6C62624F0  mov         dword ptr [rsp+10h],edx  
00007FF6C62624F4  mov         qword ptr [rsp+8],rcx  
00007FF6C62624F9  push        rbp  
00007FF6C62624FA  push        rdi  
00007FF6C62624FB  sub         rsp,0E8h  
00007FF6C6262502  lea         rbp,[rsp+20h]  
00007FF6C6262507  mov         rcx,qword ptr [this]  
;00007FF6C626250E  call        Thepale::~Thepale (07FF6C62613C5h)  
00007FF6C6262513  mov         eax,dword ptr [rbp+0E8h]  
00007FF6C6262519  and         eax,1  
00007FF6C626251C  test        eax,eax  
00007FF6C626251E  je          Thepale::`scalar deleting destructor'+41h (07FF6C6262531h)  
00007FF6C6262520  mov         edx,8  
00007FF6C6262525  mov         rcx,qword ptr [this]  
;00007FF6C626252C  call        operator delete (07FF6C6261401h)  
00007FF6C6262531  mov         rax,qword ptr [this]  
00007FF6C6262538  lea         rsp,[rbp+0C8h]  
00007FF6C626253F  pop         rdi  
00007FF6C6262540  pop         rbp  
00007FF6C6262541  ret  
```

已用注释标出，发现 **delete 先调用了析构函数，然后调用 operator delete**。

关于 operator new 和 operator delete 在较新的编译器中已找不到相关定义代码，故进行了部分借鉴和参考：

```cpp
//operator new
void *__CRTDECL operator new(size_t size) _THROW1(_STD bad_alloc)
{
// try to allocate size bytes
void *p;
while ((p = malloc(size)) == 0)
     if (_callnewh(size) == 0)
     {
         static const std::bad_alloc nomem;
         _RAISE(nomem);
     }
    
	return (p);
}
```

operator new 尝试使用 malloc 开空间，若开辟失败尝试执行空间不足的应对措施，如果再次失败则抛异常。

```cpp
void operator delete(void *pUserData)
{
     _CrtMemBlockHeader * pHead;
     RTCCALLBACK(_RTC_Free_hook, (pUserData, 0));
     if (pUserData == NULL)
         return;
     _mlock(_HEAP_LOCK);  /* block other threads */
     __TRY
         /* get a pointer to memory block header */
         pHead = pHdr(pUserData);
          /* verify block type */
         _ASSERTE(_BLOCK_TYPE_IS_VALID(pHead->nBlockUse));
         _free_dbg( pUserData, pHead->nBlockUse );
     __FINALLY
         _munlock(_HEAP_LOCK);  /* release other threads */
     __END_TRY_FINALLY
     return;
}

#define   free(p)               _free_dbg(p, _NORMAL_BLOCK)
```

释放空间则是通过 free 来完成的。

简言之，new 和 delete 将被拆分为以下部分：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Dynamic_Memory_Management-p2.png)

若是 new[] 和 delete[] 也就是依次重复以上步骤。调用顺序是从左至右的（地址从低到高）。

---

## Placement New

定位 new 是一种调用构造函数的方式，若对象被创建但没有调用构造函数初始化则可以借助定位 new 调用构造函数（在内存池中会应用到）。

```cpp
class Thepale
{
private:
    int _data1;
    int _data2;

public:
    Thepale(int data1 = 0, int data2 = 0)
        :_data1(data1)
        , _data2(data2)
    {
        cout << "构造函数被调用"<< data1 << endl;
    }

    Thepale(const Thepale& e)
    {
        _data1 = e._data1;
        _data2 = e._data2;
        cout << "拷贝构造函数被调用" << endl;
    }

    ~Thepale()
    {              
        cout << "析构函数被调用" << endl;
    }
};


int main()
{
    Thepale* e = (Thepale*)malloc(sizeof(Thepale));
    new(e)Thepale(1, 1); //主动调用构造函数

    e->~Thepale(); //主动调用析构函数

    return 0;
}
```