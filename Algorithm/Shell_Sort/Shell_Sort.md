# Shell_Sort

---

## 算法介绍

希尔排序是对插入排序的一种优化，其主要思想依然是插入排序。可知，当插入排序对有序数组进行排序时时间复杂度为：`O(N)`。

例如序列：`0 1 2 3 4 5 6 7 8 9`，插入排序仅会相邻俩俩比较，且无需移动数据，相当于遍历一遍即结束，故时间复杂度为`O(N)`，希尔排序在于使数据先趋于有序再进行插入排序，这样会使排序效率变高。趋于有序主要采用 **间隔分组插入排序**，以 gap 间隔为一组，把整个数据分成 gap 份排序，使其趋于有序。

以 gap 为 3，数据为：`9 8 7 6 5 4 3 2 1 0`为例：

![](D:\Program\Algorithm\Shell_Sort\Shell_Sort-p1.png)

此过程完成后，原数组由：`9 8 7 6 5 4 3 2 1 0` 变为了：`0 2 1 3 5 4 6 8 7 9`，可见其更趋于有序。

此过程完成后再进行插入排序即可以高效率的形式完成排序，有效降低时间复杂度。

---

## 代码实现

若以 gap 为 3 举例，则整个数据被分成三组，排序算法的关键依然是先完成单趟排序，如上图中我们需要先执行红色部分的排序工作：

```cpp
int end; //尾位置下标
int tmp; //储存需要插入的值

int gap = 3; //确定间隔

while (end >= 0)
{
    if (tmp > arr[end])
    {
        break;
    }
    else
    {
        arr[end + gap] = arr[end];
    }

    end -= gap;
}

arr[end + gap] = tmp;
```

此时将插入排序完成单趟排序的代码拿过来对比：

```cpp
int end;
int tmp;

// int gap = 1;

while (end >= 0)
{
	if (tmp > arr[end])
    {
        break;
    }
    else
    {
        arr[end + 1] = arr[end];
    }

    end--;
}

arr[end + 1] = tmp;
```

可以发现插入排序本质进行的就是 gap 为 1 的希尔排序。故当 gap 为 3 时，只需要进行对应的变化即可轻易得到希尔单趟排序的代码（这里的单趟排序指的是使数据趋于有序的过程，但由于排序过程也可通过改变 gap 为 1 来完成，所以才成趋于有序的过程也是单趟排序）。

于是希尔排序的单趟排序为：

```cpp
for (int i = gap; i < n; i += gap) //这里继承从第二个数据位置开始的思想（从第一个位置开始也是可行的，需要注意循环条件的变更）
{
    int end = i - gap;
    int tmp = arr[i];

    while (end >= 0)
    {
        if (tmp > arr[end])
        {
            break;
        }
        else
        {
            arr[end + gap] = arr[end];
        }

        end -= gap;
    }

    arr[end + gap] = tmp;
}
```

而希尔排序中分为 gap 组，则就要排 gap 次，再套上一层循环并对当前循环的循环条件进行变更（每一组的首元素位置不同）。

```cpp
for (int j = 0; j < gap; ++j)
{
    for (int i = j + gap; i < n; i += gap)
    {
        int end = i - gap;
        int tmp = arr[i];

        while (end >= 0)
        {
            if (tmp > arr[end])
            {
                break;
            }
            else
            {
                arr[end + gap] = arr[end];
            }

            end -= gap;
        }

        arr[end + gap] = tmp;
    }
}
```

这样就完成了趋于有序的过程。



还有一种更优的写法（仅代码上），因为并不是要把红色组排完了再去排绿色组，而是可以先排一次红色组，再排一次绿色组，再排一次蓝色组，三组交替进行，这样可以省略一层循环但效率并未改变。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Shell_Sort-p2.png)

每次操作的是粉色框内的数据，一个颜色操作一次，依然可以完成趋于有序的工作。实际上仅仅是把三个整体步骤拆分了而已。
此时则可以去除最外层循环，将内层循环的 `i += gap` 条件改为：`++i` 即可。

```cpp
for (int i = gap; i < n; ++i) 
{
    int end = i - gap;
    int tmp = arr[i];

    while (end >= 0)
    {
        if (tmp > arr[end])
        {
            break;
        }
        else
        {
            arr[end + gap] = arr[end];
        }

        end -= gap;
    }

    arr[end + gap] = tmp;
}
```



**而对于 gap 而言，若 gap 越大，则趋于有序的程度越低，但数据跳跃的次数越少；若 gap 越小，则趋于有序的程度越高，但数据跳跃的次数越多。**

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Shell_Sort-p3.png)

可见 gap 为 2 时，9 要走到相对正确的位置需要调整 4 次，但 gap 为 4 时只需要调整 2 次；同时 gap 为 2 时调整了路径上的 5 个数，而 gap 为 4 时只调整了路径上的 3 个数，故上述结论得证。

而对于不同的数据大小，gap 的取值应是不尽相同的，若是 100w 个数据，gap 为 3 就显得有些小，若是 10 个数据 gap 为 8 则显得有些大，故 gap 不应该是定值，对 n 个数据而言，一般采用：`gap = n; gap /= 2;`的方式或 `gap = n; gap = gap / 3 + 1;` 的方式，这两种方法可以使得 gap 先大后小，且最后都为 1，顺便完成了插入排序（gap 为 1 时就是插入排序），无需单独再写一遍插入排序。

```cpp
void shell_sort(int* arr, int n)
{
	int gap = n;

	while (gap > 1) //采用 gap /= 2 的方式
	{
		gap /= 2;

		for (int i = gap; i < n; ++i) 
        {
            int end = i - gap;
            int tmp = arr[i];

            while (end >= 0)
            {
                if (tmp > arr[end])
                {
                    break;
                }
                else
                {
                    arr[end + gap] = arr[end];
                }

                end -= gap;
            }

            arr[end + gap] = tmp;
        }
	}
}
```



---

## 效率分析

希尔排序的效率分析一直以来是一个难题，我们以 gap 为：`n /= 2` 的情况为例，进行粗略分析：

第一次排序：`gap = n / 2;` 此时观察循环：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Shell_Sort-p4.png)

红色循环部分因为每次 gap 都要除以 `2`，故 `gap / 2 / 2 / 2 ... / 2 == 1;`设总共除了 `x` 个 `2`，即 `2^x == n`，则 `x == logN`。

绿色部分（对第一次分析）一共要进行 `n / 2` 次比较，每次最多交换 `1` 次，故最坏情况第一次需要进行 `n / 2` 次交换。



第二次排序：`gap = n / 4;` 此时却不能按最复杂的情况来计算，因为第一次已经将序列进行了一次趋于有序的过程，若第二次按最复杂情况计算，应该要进行：

`(3 + 2 + 1) * n / 4 == 3 / 2 * n` 次，而实际次数应比这个要小。这一特性放在最后一次最为明显，最后一次 `gap == 1`，若按照最复杂的情况计算即时间复杂度是插入排序的时间复杂度，为：`O(N^2)`，但完成了趋于排序的过程最后一次应极大程度上趋于有序，所以应该按最好的情况计算，即 `O(N)` 才对。



故这里的时间复杂度不再深入研究和计算，参考清华大学殷人昆教授的说明，希尔排序的时间复杂度取：`O(N^1.25 ~ 1.6N^1.25)`，或记为：`O(N^1.3)` 即可。


---

## 补充说明

* gap 为几，数据就会被分为几组（数据个数小于 gap 的情况除外），即 (n / gap)(每组的个数) * (gap)(组数) = n。
* 希尔排序不稳定，因为涉及到大幅度的数据交换，不能保证在不同组别相同数据不被交换位置。