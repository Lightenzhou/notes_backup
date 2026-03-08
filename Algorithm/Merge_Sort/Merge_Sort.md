# Merge_Sort

---

## 算法简介

归并排序在于对递归分治思想的运用，具体思想是对于两个有序数组，依次从头开始比较元素，把小的往下放，从而让两个有序数组合并为一个有序数组的过程：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Merge_Sort-g1.gif)

而归并排序便是利用这一思想，将数据不断二分，直至分为一个数位置（一个数默认有序），然后逐步归并，最后合成有序数组的过程：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Merge_Sort-g2.gif)

---

## 代码实现

### 递归实现

归并的单趟排序如下：

```cpp
int mid; //需要以 mid 将有序数组划分为两个区间

//标记左右区间的起始和结束位置
int begin_l, begin_r;
int end_l, end_r;

int i;

//归并过程 - 由于归并不能在原数组中进行，所以开辟了和原数组大小一样的 tmp 数组来辅助排序然后拷贝到原数组
while (begin_l <= end_l && begin_r <= end_r) //开始逐个比对，把小数往下放
{
    if (arr[begin_l] < arr[begin_r])
    {
        tmp[i++] = arr[begin_l++];
    }
    else
    {
        tmp[i++] = arr[begin_r++];
    }
}

//两个数组不一定是一起放完的，若有一个数组没放完，需要继续放
while (begin_l <= end_l) //左边没放完放左边
{
    tmp[i++] = arr[begin_l++];
}

while (begin_r <= end_r) //右边没放完放右边
{
    tmp[i++] = arr[begin_r++];
}

memcpy(); //将 tmp 中排好序的数据放回 arr
```

再对归并排序进行一层封装：

```cpp
void merge_sort_portion(int* arr, int* tmp, int left, int right)
{
	if (left >= right) { return; } //递归结束条件

	int mid = (left + right) / 2;
    
    //递归分治
	merge_sort_portion(arr, tmp, left, mid);
	merge_sort_portion(arr, tmp, mid + 1, right);

    //划分左右边界
	int begin_l = left, begin_r = mid + 1;
	int end_l = mid, end_r = right;

	int i = left;
	while (begin_l <= end_l && begin_r <= end_r)
	{
		if (arr[begin_l] < arr[begin_r])
		{
			tmp[i++] = arr[begin_l++];
		}
		else
		{
			tmp[i++] = arr[begin_r++];
		}
	}

	while (begin_l <= end_l)
	{
		tmp[i++] = arr[begin_l++];
	}

	while (begin_r <= end_r)
	{
		tmp[i++] = arr[begin_r++];
	}

	memcpy(arr + left, tmp + left, 4 * (right - left + 1));
}

void merge_sort(int* arr, int n)
{
    //开辟临时空间
	int* tmp = (int*)malloc(sizeof(int) * n);
	if (tmp == NULL) { exit(-1); }

	merge_sort_portion(arr, tmp, 0, n - 1);

	free(tmp);
}
```



### 非递归版本

归并排序无论如何都是标准的对半划分，不会出现快速排序有序情况下只缩小一个区间导致栈溢出的情况。但依然实现非递归版本以供学习参考。

快速排序的递归方式更像一个前序遍历，划分区间先处理再划分；而归并排序的递归方式更像一个后序遍历，划分到最后再开始处理。故其和斐波那契的递归版本一样可以直接改为循环：

```cpp
int* tmp = (int*)malloc(sizeof(int) * n);
if (tmp == NULL) { exit(-1); }

for (int i = 0; i < n; i += gap * 2) //因为一次要跳过的是两个区间，故 i += gap * 2
{
    //左右区间控制
    int begin_l = i, end_l = begin_l + gap - 1;
	int begin_r = end_l + 1, end_r = begin_r + gap - 1;
    
    int pos = begin_l;
    
    while (begin_l <= end_l && begin_r <= end_r)
    {
        if (arr[begin_l] < arr[begin_r])
        {
            tmp[pos++] = arr[begin_l++];
        }
        else
        {
            tmp[pos++] = arr[begin_r++];
        }
    }

    while (begin_l <= end_l)
    {
        tmp[pos++] = arr[begin_l++];
    }

    while (begin_r <= end_r)
    {
        tmp[pos++] = arr[begin_r++];
    }
}

memcpy(arr, tmp, sizeof(int) * n); //这里是等所有数据都做完排序后一次性拷贝，后续会详细讲解

```

整个排序需要手动控制 gap，来确定几几为一组，由归并排序的二分思想可知，gap 应成 2 的指数次增长：

```cpp
void merge_sort_non_recursive(int* arr, int n)
{
    int* tmp = (int*)malloc(sizeof(int) * n);
    if (tmp == NULL) { exit(-1); }

    int gap = 1; //gap 默认从 1 开始
    while (gap < n)
    {
        for (int i = 0; i < n; i += gap * 2)
        {
            int begin_l = i, end_l = begin_l + gap - 1;
            int begin_r = end_l + 1, end_r = begin_r + gap - 1;

            int pos = begin_l;
            while (begin_l <= end_l && begin_r <= end_r)
            {
                if (arr[begin_l] < arr[begin_r])
                {
                    tmp[pos++] = arr[begin_l++];
                }
                else
                {
                    tmp[pos++] = arr[begin_r++];
                }
            }

            while (begin_l <= end_l)
            {
                tmp[pos++] = arr[begin_l++];
            }

            while (begin_r <= end_r)
            {
                tmp[pos++] = arr[begin_r++];
            }
        }

        memcpy(arr, tmp, sizeof(int) * n);

        gap *= 2; //以 2 的指数增长
    }
}
```

当数据个数为 2 的指数时，一切划分都是正常的，这里对区间进行打印输出：

```cpp
int arr[8] = {3,2,7,5,1,4,9,6};
```

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Merge_Sort-p1.png)

若将数组增加一个元素：

```cpp
int arr[9] = {3,2,7,5,1,4,9,6,8};
```

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Merge_Sort-p2.png)

程序崩溃出错，可见其区间在：`end_l; begin_r; end_r;` 皆出现了越界，故非递归主要针对这三种情况进行调整（以下仍采用整个排序完后一次性拷贝）：

#### end_l 越界

这说明 begin_r 和 end_r 也一定越界了，由于数据需要拷贝到 tmp 中，故需要将 end_l 修正为 `n - 1`，防止访问越界，使数据正常拷贝到 tmp 中（但这里其实数据不需要处理，因为右区间不存在，左区间做的事其实是拷贝到 tmp 然后再拷贝到原数组）；而 begin_r 和 end_r 需要变成一个不存在的区间以便不进行循环，故修正为 `n 和 n - 1` 即可，这样 `begin_r > end_r` 循环自然不会进行。

```cpp
if (end_l >= n) 
{
	end_l = n - 1;
	begin_r = n;
	end_r = n - 1;
}
```



#### begin_r 越界

若 end_l 未越界，begin_r 越界说明 end_r 也一定越界了，依然是需要通过修正让此区间不存在。而左区间的数据正常拷贝 tmp 即可。（此时左区间依然是拷贝到 tmp 然后再拷贝回原数组）

```cpp
if (begin_r >= n) 
{
	begin_r = n;
	end_r = n - 1;
}
```



#### end_r 越界

若只有 end_r 越界，说明右区间有数据但并没有达到 2 的指数次，这时左右区间仍需归并，需要修正 end_r 到正确的位置，即 `n - 1`，让归并工作正常进行。

```cpp
if(end_r >= n)
{
    end_r = n - 1;
}
```



故代码如下：

```cpp
void merge_sort_non_recursive(int* arr, int n)
{
	int* tmp = (int*)malloc(sizeof(int) * n);
	if (tmp == NULL) { exit(-1); }

	int gap = 1;
	while (gap < n)
	{
		for (int i = 0; i < n; i += gap * 2)
		{
			int begin_l = i, end_l = begin_l + gap - 1;
			int begin_r = end_l + 1, end_r = begin_r + gap - 1;

            //以下代码可以优化，但为了逻辑清晰而这样表述
			if (end_l >= n) 
			{
				end_l = n - 1;
				begin_r = n;
				end_r = n - 1;
			}
			else if (begin_r >= n)
			{
				begin_r = n;
				end_r = n - 1;
			}
			else if (end_r >= n)
			{
				end_r = n - 1;
			}

			int pos = begin_l;
			while (begin_l <= end_l && begin_r <= end_r)
			{
				if (arr[begin_l] < arr[begin_r])
				{
					tmp[pos++] = arr[begin_l++];
				}
				else
				{
					tmp[pos++] = arr[begin_r++];
				}
			}

			while (begin_l <= end_l)
			{
				tmp[pos++] = arr[begin_l++];
			}

			while (begin_r <= end_r)
			{
				tmp[pos++] = arr[begin_r++];
			}
		}

		memcpy(arr, tmp, sizeof(int) * n);

		gap *= 2;
	}
	
	free(tmp);
}
```

以上代码以可以解决非递归的归并排序，但存在一些情况原数组的数据被拷贝到 tmp 数组中又拷贝回原数组（end_l 越界和 begin_r 越界的情况，如果直接跳出则 tmp 中的数据缺失，还会把原数组数据覆盖），多了一些消耗，故一次性的拷贝看似方便但却不如片段式的拷贝来的有效：

```cpp
void merge_sort_non_recursive(int* arr, int n)
{
	int* tmp = (int*)malloc(sizeof(int) * n);
	if (tmp == NULL) { exit(-1); }

	int gap = 1;
	while (gap < n)
	{
		for (int i = 0; i < n; i += gap * 2)
		{
			int begin_l = i, end_l = begin_l + gap - 1;
			int begin_r = end_l + 1, end_r = begin_r + gap - 1;

			if (end_l >= n || begin_l >= n) //当出现这两种情况时直接跳出，不需要拷贝
			{
				break;
			}

			if (end_r >= n)
			{
				end_r = n - 1;
			}

			int pos = begin_l;
			while (begin_l <= end_l && begin_r <= end_r)
			{
				if (arr[begin_l] < arr[begin_r])
				{
					tmp[pos++] = arr[begin_l++];
				}
				else
				{
					tmp[pos++] = arr[begin_r++];
				}
			}

			while (begin_l <= end_l)
			{
				tmp[pos++] = arr[begin_l++];
			}

			while (begin_r <= end_r)
			{
				tmp[pos++] = arr[begin_r++];
			}

            // + i 的意思其实是 + begin_l，但 begin_l 已经在归并中改变，所以这里是 + i
			memcpy(arr + i, tmp + i, sizeof(int) * (end_r - i + 1)); //片段式处理 - 排完一次就拷贝一次
		}

		gap *= 2;
	}
	
	free(tmp);
}
```

---

## 效率分析

归并排序时间复杂度：`O(N * logN)`

归并排序空间复杂度：`O(N)`

归并排序是标准的二分思想，时间复杂度稳定在 `N * logN`：
一直二分则高度为：`logN` 层，而每一层都需要对左右区间数据归并，即：`O(N)`
故总的时间复杂度为：`O(N * logN)`



由于归并需要开辟额外的原数组大小的空间存储区间归并的内容，故空间复杂度为：`O(N)`

---

## 补充说明

* 以上所写的归并排序并不稳定，但可以将其变为稳定的。其保证稳定的必要条件就是将左右区间归并，数据相等时，让左区间的数据先下来，故只要在比较数据的位置添加上等号即可。