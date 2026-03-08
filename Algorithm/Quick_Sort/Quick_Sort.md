# Quick_Sort

---

## 算法介绍

快速排序是通过选 key 的方式，让比 key 小的到 key  的左边，比 key 大的到 key 的右边，这样 key 就到了正确的位置，再对剩余左右区间分治处理，完成排序。既然也涉及到分治思想，首先将完成快速排序的单趟排序，而总共介绍三种方法：**hoare 法，挖坑法，前后指针法**。

### hoare 法

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Quick_Sort-g1.gif)

这里选取的左为 key，**则右边先走**（同理，右为 key 则左先走），左找大右找小，目的则是让小于 key 的都到左边，大于 key 的都到右边，当一个位置的左边都是比他小的，右边都是比它大的，则这个数所在的便是正确的位置。且最后相遇位置一定比 key 小。（关于左 key 右先走相遇位置一定比 key 小的解答：[^ 注释1]）



### 挖坑法

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Quick_Sort-g2.gif)

挖坑法是把 key 所在的值单独保存，创造一个 “坑”，如果 key 取左边，则右边先找小来填坑，再形成新的坑；左边再找大来填坑，再形成新的坑；最后两者一定在坑位相遇，此时便是 key 的正确位置。其思想和 hoare 版本大同小异，依然是在做一个将比 key 小的放到左边，比 key 大的放到右边的工作，不过其给人的感觉（感觉是指左选 key 留坑自然右边先走填坑）更加自然，更便于理解。不过需要注意的是，两种方法单趟排序出来大概率并不一样。



### 前后指针法

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Quick_Sort-g3.gif)

依然是取左边为 key，此时有两个指针，cur 和 prev。cur 默认在 prev 的后一个位置，prev 默认在 key 的位置（prev 也可以在 cur 的位置，个人控制即可，此文章所用方法可以让 prev 最后在 key 的正确位置，因为其保证 prev 后面的位置比 key 大<不包括prev>）。cur 负责找小，找到小的则和 prev 的后一个交换，最后 key 的位置再和 prev 的位置交换即可。（因为 cur 是找小的，所以走过的地方一定是比 key 大的值，所以让 cur 找到的小的和 prev 的后一个交换所做的事情是大致一致的，依旧是把小的往前扔，大的往后扔）。

---

## 代码实现

### hoare 法

由于快速排序需要的不再是元素个数，而是需要操作的区间，故传参方面采用了左右区间的传参方式。三种方式以下分别于数字 1 2 3 代替。

swap 的实现：

```cpp
void swap(int* e1, int* e2)
{
	int tmp = *e1;
	*e1 = *e2;
	*e2 = tmp;
}
```



```cpp
void quick_sort_1(int* arr, int left, int right)
{
	if (left >= right) { return; } //递归跳出条件

	int key = left; //选最左边的数为 key

	int begin = left, end = right; //必须记录左右区间的值在递归时使用

	while (left < right)
	{
		while (left < right && arr[right] >= arr[key]) //任何时候都要保证 left < right; 右找小
		{
			--right;
		}

		while (left < right && arr[left] <= arr[key])  //左找大
		{
			++left;
		}

		swap(&arr[left], &arr[right]); //交换右找到的小和左找到的大
	}

	swap(&arr[left], &arr[key]); //使 key 到达正确位置

    //递归分治
	quick_sort_1(arr, begin, left - 1); //此时 key 已经到达正确位置，区间被划分为：[begin, key_pos - 1] key_pos [key_pos + 1, end];
	quick_sort_1(arr, left + 1, end);
}
```



### 挖坑法

```cpp
void quick_sort_2(int* arr, int left, int right)
{
	if (left >= right) { return; } 

	int key = arr[left]; //取出 key 所在位置的数
	int hole = left; //key所在位置为坑位

	int begin = left, end = right;

	while(left < right)
	{
		while (left < right && arr[right] >= key) //右找小
		{
			--right;
		}
		arr[hole] = arr[right]; //填坑
		hole = right; //所在位置形成新的坑位

		while (left < right && arr[left] <= key)
		{
			++left;
		}
		arr[hole] = arr[left]; //填坑
		hole = left; //所在位置形成新的坑位
	}

	arr[hole] = key; //将 key 放入最后的坑中，到正确位置

	quick_sort_2(arr, begin, left - 1);
	quick_sort_2(arr, left + 1, end);
}
```



### 前后指针法

前后指针法是最常被应用的，因为它相比于前两种方法实现中更不容易出现问题，也更容易理解（仁者见仁智者见智）。

```cpp
void quick_sort_3(int* arr, int left, int right)
{
	if (left >= right) { return; }

	int key = left;
	
	int cur = left + 1, prev = left; //前后指针位置

	while(cur <= right) //由于 right 是右下标，故这里是小于等于而非小于
	{
		if(arr[cur] < arr[key]) //如果比 key 小则 ++prev 后交换
			swap(&arr[++prev], &arr[cur]); 
			
        cur++;
	}
    
    //以上逻辑注重理解，也可替换为如下代码：
    //while(cur <= right)
    //{
    //    if(arr[cur] < arr[key] && ++prev != cur)
    //    {
    //       swap(&arr[prev], &arr[cur]); 
	//	}
    //    
    //    ++cur;
	//}

	swap(&arr[key], &arr[prev]);

	quick_sort_3(arr, left, prev - 1);
	quick_sort_3(arr, prev + 1, right);
}
```

由于考虑到非递归的实现，并不希望调用一次快速排序直接完成整个排序，而是希望可以一步步的来，则需要将代码再套一层，如下：

```cpp
int quick_sort_portion(int* arr, int left, int right)
{
    if (left >= right) { return; }

	int key = left;
	
	int cur = left + 1, prev = left;

	while(cur <= right)
	{
		if(arr[cur] < arr[key])
			swap(&arr[++prev], &arr[cur]); 
			
        cur++;
	}
    
    swap(&arr[key], &arr[prev]);
}

void quick_sort(int* arr, int left, int right)
{
    int sep = quick_sort_portion(arr, left, right);
    
    quick_sort(arr, left, sep - 1);
    quick_sort(arr, sep + 1, right);
}
```



如果快速排序遇到有序的情况，那么每次递归仅能将区间缩小 1，在数据量较大时，必然会导致**栈溢出**，且此时时间复杂度也退化到：`O(N^2)`。一般采用 **三数取中法** 或 **随机 key 法** 来解决这一问题。（诱发问题的本因是因为每次取到的都是最大或最小的数）

#### 三数取中法

```cpp
int mid_of_three(int* arr, int left, int right)
{
	int mid = (left + right) / 2;
    
	if (arr[left] < arr[mid])
	{
		if (arr[mid] < arr[right])
		{
			return mid;
		}
		else if (arr[left] > arr[right])
		{
			return left;
		}
		else
		{
			return right;
		}
	}
	else
	{
		if (arr[mid] > arr[right])
		{
			return mid;
		}
		else if (arr[left] < arr[right])
		{
			return left;
		}
		else
		{
			return right;
		}
	}
}
```



#### 随机 key 法

```cpp
int randkey(int left, int right)
{
	return left + rand() % (right - left); //由于 rand 取模后的数据相对于下标来说是从 0 开始的，所以必须加上 left
}
```



而这两种方法并非让循环从所取得的 key 的位置开始，这会让快速排序的实现十分棘手和不解，而是让最左元素于所取得的位置的元素交换，仅交换数据即可达到解决问题的目的。

```cpp
int quick_sort_portion(int* arr, int left, int right)
{
    if (left >= right) { return; }

	int key = left;
	swap(&arr[key], &arr[mid_of_three(arr, left, right)]); //三数取中法相对而言是使用更多的
    
	int cur = left + 1, prev = left;

	while(cur <= right)
	{
		if(arr[cur] < arr[key])
			swap(&arr[++prev], &arr[cur++]); 
			
        cur++;
	}
    
    swap(&arr[key], &arr[prev]);
}

void quick_sort(int* arr, int left, int right)
{
    int sep = quick_sort_portion(arr, left, right);
    
    quick_sort(arr, left, sep - 1);
    quick_sort(arr, sep + 1, right);
}
```



#### 小区间优化

用递归实现快排时，如果只有少量的数需要排序，使用递归会建立较多的栈空间产生较多的消耗，以下以有五个数排序为例：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Quick_Sort-p1.png)

可见递归建立栈帧较多，故可以采用小区间优化的方式，由于小数据量时 `O(N) 和 O(N^2)` 的效率差距不大，大数据量时，当经过快排处理后小区间是大致有序的，这时插入排序的性能较佳，故界定一个区间，若小于此区间直接用插入排序完成排序，可略微提升效率：

```cpp
int quick_sort_portion(int* arr, int left, int right)
{
	int key = left;
	int mid = mid_of_three(arr, left, right);

	int cur = left + 1, prev = left;

	while (cur <= right)
	{
		if (arr[cur] < arr[key])
			swap(&arr[++prev], &arr[cur]);

		cur++;
	}

	swap(&arr[key], &arr[prev]);

	return prev;
}

void quick_sort(int* arr, int left, int right)
{
	if (left >= right) { return; }

    //小区间优化
	if (right - left + 1 < 10)
	{
		Insertion_Sort(arr + left, right - left + 1);
	}
	else
	{
		int sep = quick_sort_portion(arr, left, right);
		quick_sort(arr, left, sep - 1);
		quick_sort(arr, sep + 1, right);
	}
}
```

这里取 10 作为标准。



#### 三路划分

快速排序在面对大量重复数据时比较乏力，不管采用上述怎样的优化方式都无法解决这种问题，故采用三路划分的方法解决。（在讲述三路划分方法时，着重点在三路划分而非其他优化，故不一定会带上其它优化方式）

在普通的划分方式中，key 的左边是小于等于 key 的数，key 的右边是大于等于 key 的数。而三路划分的方式是将比 key 小的划分到左边，比 key 大的划分到右边，和 key 相等的位于中间。

核心思想为定义三个指针，left right cur。三种情况：1.cur 位置比 key 小，则与 left 位置交换后都向后走；2.cur 位置和 key 相等，则只有 cur 向后走；3.cur 位置比 key 大，则与 right 交换后 right 向前走，cur 不动。

其具体保证，left 的位置永远是和 key 相等区间的开头，保证 left 的左边都是比 key 小的值。cur 负责遍历数组。right 最终的位置一定是和 key 相等区间的结尾，保证 right 的右边都是比 key 大的值。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Quick_Sort-p2.png)

此图可验证上述规律。

```cpp
void three_way_partition(int* arr, int left, int right)
{
	if (left >= right) { return; } //递归跳出条件

	int begin = left, end = right; //保存起始和结束位置

	int key = arr[left]; //确定 key 的值
	int cur = left + 1;

    //三路划分
	while (cur <= right)
	{
		if (arr[cur] < key) //比 key 小则交换后向后走
		{
			swap(&arr[cur++], &arr[left++]);
		}
		else if (arr[cur] == key) //相等只有 cur 向后走，因为 cur 找的是不相等的值，让它们到正确的位置
		{
			cur++;
		}
		else //比 key 大和 right 交换后 cur 不动，是因为不清楚交换过来的值是否比 key 小，等下一次循环处理即可
		{
			swap(&arr[cur], &arr[right--]);
		}
	}

    //区间被划分为：[begin, left - 1] [left, right] [right + 1, end];
	three_way_partition(arr, begin, left - 1);
	three_way_partition(arr, right + 1, end);
}
```



### 非递归版本

快速排序是选出一个区间，处理后再选出另外的区间，其递归过程类似于前序遍历。每一次递归所存储的也只是所需要处理的区间，这利用栈可以完美解决：

```cpp
int quick_sort_partition(int* arr, int left, int right)
{
	int key = left;
	int mid = mid_of_three(arr, left, right);

	int cur = left + 1, prev = left;

	while (cur <= right)
	{
		if (arr[cur] < arr[key])
			swap(&arr[++prev], &arr[cur]);

		cur++;
	}

	swap(&arr[key], &arr[prev]);

	return prev;
}

void quick_sort_non_recursion(int* arr, int n)
{
	stack st;
	stack_initialize(&st);

	stack_push(&st, n - 1);
	stack_push(&st, 0);

	while (!stack_empty(&st))
	{
		int left = stack_top(&st); stack_pop(&st);
		int right = stack_top(&st); stack_pop(&st);

		if (left >= right) { continue; }

		int sep = quick_sort_partition(arr, left, right);

		stack_push(&st, right);
		stack_push(&st, sep + 1);
		stack_push(&st, sep - 1);
		stack_push(&st, left);
	}

	stack_destroy(&st);
}
```



---

## 效率分析

快速排序时间复杂度：`O(N * logN)`

快速排序空间复杂度：`O(logN)`

本效率分析不考虑任何极端情况。

若每一次的 key 都位于中间，将区间划分为均匀的两份，可知快排的单趟遍历数组，时间复杂度为：`O(N)`，若每次二分区间，则总共会有：`logN` 层，但每次二分都会使 key 到达正确位置，故在这 `logN` 层中，它们的时间复杂度不都是 `O(N)`，而是：

第一层：`N`
第二层：`N - 1`
第三层：`N - 2`
......
第 `logN` 层：`N - logN + 1`

具体而言，其时间复杂度为：`N + (N - 1) + (N - 2) + ...... + (N - logN + 1) == N * logN - (logN - 1) * logN / 2`
但减号后续内容可忽略，且快速排序并非每一次都能均匀二分，故时间复杂度也不一定比 `O(N * logN)` 要小，故大致取：`O(N * logN)`



在递归时一直二分最大深度为：`logN`，故占用的栈空间为：`O(logN)`

---

## 补充说明

[^注释1]: 左为 key 右先走会有两种情况：1.右遇到左。一种情况是左在没有动的情况下被右遇到，这种情况说明右边全是比 key 大的，没有找到小所以没有让左出发，直接和左相遇则是在 key 位置，此时值和 key 相等，说明 key 所在位置已经是正确位置；第二种情况则是经历过交换后，右再次遇到左，此时左所在的位置是上一个交换过的位置，这个位置刚好被交换过来右所找到的一个比 key 小的值，而此时右在找小的过程中和左相遇，这说明左的右边全是比 key 大的（同时也说明左的左边全是比 key 小的，因为左在找大，所以走过的地方一定是比 key 小的），且左所在的位置刚好比 key 小，交换后 key 刚好到了正确位置。   2.左遇到右。在经历了一系列交换后，右找到了比 key 小的，此时左要找比 key 大的，找不到，和右相遇，说明右的左边全是比 key 小的（当然右的右边也全是比 key 大的，因为右在找小，能走到这里说明走过的地方都是比 key 大的），且右所在的位置是比 key 小的值，交换后让 key 到达正确的位置。

