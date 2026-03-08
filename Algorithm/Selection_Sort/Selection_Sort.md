# Selection_Sort

---

## 算法介绍

选择排序也和扑克牌一般，只不过此时扑克牌采用发牌的方式，所有牌发完了你才拿起来看，这个时候先找到最小的牌，再找次小的牌，如此使牌有序的过程便是选择排序。（以升序整型数据为例）

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Selection_Sort-g1.gif)

---

## 代码实现

选择排序的单趟排序就是在数组中找最小数的过程：

```cpp
int min_pos = 0; //最小数的下标位置
for (int i = 0; i < n; ++i)
{
    if (arr[i] < arr[min_pos])
    {
        min_pos = i;
    }
 }
```

这趟排序走完之后便可找到最小数，让它回到正确的位置，即第一位，交换即可。而整个插入排序就是不断在找小的过程：

```cpp
void swap(int* e1, int* e2)
{
	int temp = *e1;
	*e1 = *e2;
	*e2 = temp;
}

void selection_sort(int* arr, int n)
{
	for (int i = 0; i < n; ++i)
	{
		int min_pos = i;
		for (int j = i; j < n; ++j)
		{
			if (arr[j] < arr[min_pos])
			{
				min_pos = j;
			}
		 }

		swap(&arr[i], &arr[min_pos]);
	}
}
```

在一趟排序中，找小的同时也可以找大，这样可以略微提升效率，而此时由于是两头找，我更推荐用 left/right 的形式：

```cpp
void swap(int* e1, int* e2)
{
	int temp = *e1;
	*e1 = *e2;
	*e2 = temp;
}

void selection_sort(int* arr, int n)
{
	int left = 0;
	int right = n - 1;

	while(left < right)
	{
		int min_pos = left;
		int max_pos = left;
		for (int j = left; j <= right; ++j)
		{
			if (arr[j] < arr[min_pos])
			{
				min_pos = j;
			}

			if (arr[j] > arr[max_pos])
			{
				max_pos = j;
			}
		 }

		swap(&arr[left], &arr[min_pos]);
		swap(&arr[right], &arr[max_pos]);

		left++, right--;
	}
}
```

但此处代码依然隐藏一些问题，具体体现在若交换时 `left == max_pos`，则会出现这样的情况：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Selection_Sort-p1.png)

交换（2）这一步骤中，`max_pos` 已经被换走了，所以这一步会导致错误发生，可知 `max_pos` 被换到了 `min_pos` 的位置，做如下修正即可：

```cpp
void swap(int* e1, int* e2)
{
	int temp = *e1;
	*e1 = *e2;
	*e2 = temp;
}

void selection_sort(int* arr, int n)
{
	int left = 0;
	int right = n - 1;

	while(left < right)
	{
		int min_pos = left;
		int max_pos = left;
		for (int j = left; j <= right; ++j)
		{
			if (arr[j] < arr[min_pos])
			{
				min_pos = j;
			}

			if (arr[j] > arr[max_pos])
			{
				max_pos = j;
			}
		 }

		swap(&arr[left], &arr[min_pos]);
        if(left == max_pos)
        {
            max_pos = min_pos;
        }
          
		swap(&arr[right], &arr[max_pos]);

		left++, right--;
	}
}
```

---

## 效率分析

选择排序时间复杂度：`O(N^2)`

若按每次只选一个数来计算：

第一次需要遍历整个数组，共 `n` 次；
第二次需要遍历除了第一个数之外的数组，共 `n - 1` 次；
第三次需要遍历除了第一个和第二个数之外的数组，共 `n - 2` 次；
......
第 `n - 1`  次需要遍历最后两个数，共 `2` 次；
第 `n` 次需要遍历最后一个数（也可不遍历，因为序列已经确定），共 `1` 次。

故整体是一个等差数列：`n + (n - 1) + (n - 2) + ...... + 2 + 1 == (n + 1) * n / 2 ≈ (n^2 + n) / 2`
省略常数和低次项，故时间复杂度为：`O(N^2)`



若按每次选两个数来计算：

易证其为：`n + (n - 2) + (n - 4) + ...... + 2 + 0 == n^2 / 4`
省略常数项，故时间复杂度为：`O(N^2)`



综上所述，时间复杂度为：`O(N^2)`

---

## 补充说明

* 选择排序为不稳定排序，因为排序中数据交换时可能导致相同数据乱序。
* 选择排序不受有序或无序影响，即一个排序就算接近有序选择排序也必须走完 `N^2 `次，而非如插入排序一般，在接近有序时效率会提升。