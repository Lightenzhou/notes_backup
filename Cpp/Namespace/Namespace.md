# Namespace

---

## 什么是命名空间

命名空间界定了一种特殊的域：**命名空间域**。在 C++ 中，域一共分为四种：局部域，全局域，命名空间域，类域。而对变量、常量、函数等进行访问和使用，就受域的限制，以下将结合除类域之外的域来探讨。

---

## 命名空间的性质

观察以下代码：

```cpp
int a = 0;

int main()
{
	int a = 1;

	printf("a = %d\n", a);

	return 0;
}
```

程序的输出结果为：`a = 1`

可知，**在全局域中和局部域中同时存在 a，则会遵循局部优先原则，优先访问局部的 a**。此时若想访问全局的 a，将局部的 a 去除即可：

```cpp
int a = 0;

int main()
{
	printf("a = %d\n", a);

	return 0;
}
```

程序的输出结果为：`a = 0`

**当局部找不到 a 变量时，就会在全局域中查找**。



若一个项目需要多个人负责，并合并代码，无法保证所有人使用的变量名、常量名、函数名等名称不相同，这就是命名空间需要解决的问题。在命名空间中，可以定义需要的变量、常量、函数等，将其封装在特定的命名空间内，使用时声明命名空间。以下是定义方法：

```cpp
namespace Thepale
{
	int a = 2022; //定义变量

	static int b = 5201314; //定义常量

	int func() { return 0; } //定义函数

	typedef int my_datatype; //定义类型
}
```

请观察如下代码：

```cpp
namespace Thepale
{
	int a = 2022;
}

int a = 0;

int main()
{
	printf("a = %d\n", a);

	return 0;
}
```

程序的输出结果为：`a = 0`

同理，在全局域找到 a，若此时屏蔽全局的 a：

```cpp
namespace Thepale
{
	int a = 2022;
}

int main()
{
	printf("a = %d\n", a);

	return 0;
}
```

程序报错：未定义标识符 “a”

可知，**在命名空间域中的变量无法直接访问（除变量外其它套用结论即可），必须声明域** ：

```cpp
namespace Thepale
{
	int a = 2022;
}

int main()
{
	printf("a = %d\n", Thepale::a);

	return 0;
}
```

程序的输出结果为：`a = 2022`

此处的：`::` 名为 **域作用限定符**，这里声明了域是 Thepale，故访问的是 Thepale 中的 a。



**当域作用限定符前什么都不跟时即代表全局域**，故可以直接让其不访问局部，只访问全局：

```cpp
int a = 0;

int main()
{
	int a = 1;

	printf("a = %d\n", ::a);

	return 0;
}
```

程序的输出结果为：`a = 0`



### 命名空间域的合并

当出现两个名称相同的命名空间域，其中的内容会被合并在一起：

```cpp
namespace Thepale
{ 
	int a = 2022;
}

namespace Thepale
{
	int b = 2023;
}

using namespace Thepale;

int main()
{
	printf("a = %d ", Thepale::a);
	printf("b = %d ", Thepale::b);

	return 0;
}
```

程序的输出结果为：`a = 2022 b = 2023 `



### 命名空间的嵌套

命名空间是允许嵌套的：

```cpp
namespace scope1
{
	int a = 1;

	namespace scope2
	{
		int a = 2;
	}
}

int main()
{
	printf("a = %d ", scope1::a);
	printf("a = %d ", scope1::scope2::a);

	return 0;
}
```

程序的输出结果为：`a = 1 a = 2 ` 



### 命名空间域的展开

当看到：`using namespace std;` 时，本质是展开了 std 这个域，如果不展开，则例如 `cout cin` 等函数无法直接使用，而是需要声明域才能只用：`std::cout`。同理，对自己声明的命名空间也可以进行展开操作，并且**展开操作是将命名空间域中的所有内容展开到了全局**，展开后可以当作全局的来使用。

```cpp
namespace Thepale
{
	int a = 2022;
}

using namespace Thepale;

int main()
{
	printf("a = %d\n", a);

	return 0;
}
```

程序的输出结果为：`a = 2022`

*当然，在全局中同时定义两个同名变量是不允许的，但却可以在命名空间和全局中各定义一个，并将命名空间展开，这样只要不直接使用这个变量，就不会报错。例如仍然声明作用域的话，依然是可以正常使用的*：

```cpp
namespace Thepale
{
	int a = 2022;
}

using namespace Thepale;

int a = 0;

int main()
{
	printf("a = %d\n", ::a);

	return 0;
}
```

程序的输出结果为：`a = 0`

命名空间的展开决定了编译器是否会到命名空间中去搜索，虽然此处二者同名，但值为 2022 的 a 永远隶属于 Thepale，这是编译器所明白的，故声明具体的作用域是仍不会报错，但请尽量杜绝这种做法。



#### 特定的展开

一般在进行大型项目开发时，不会将命名空间全部展开，对于需要特定使用的函数等，可以单独展开：

```cpp
using std::cout;
using std::cin;
```

这样可以保证在使用 `cin 和 cout` 时不必声明域。



### 关于类域

当定义一个类时：

```cpp
class Thepale
{
    int func(int e)
    {
        e = a;
        return e;
    }
    
private:
	int a;   
};
```

即同时也产生类域，在成员函数中，访问遵循：先局部域，后类域，再全局域的搜索规则。

---

## 总结

* 在使用变量时，若不指定域会自动在局部域搜索，如果局部域没有，则到全局域搜索。
* 名称相同的命名空间域会自动合并。
* 命名空间域可以嵌套定义。
* 使用域作用限定符可以指定域访问。
* 展开命名空间可以将其中的内容释放到全局。 

---

## 补充说明

* 在 std 命名空间中，并非包含所有内容（例如 vector list 等容器），而是在定义 vector list 等容器时，套在 std 命名空间内，这样同时包含 vector 和 list，它们的命名空间名称一致会被合并，达成了用什么有什么的目的，提高了效率。
* 只有全局和局部可以影响变量或对象的生命周期，类域和命名空间域主要用于代码的组织和命名，不能影响变量或对象的生命周期。