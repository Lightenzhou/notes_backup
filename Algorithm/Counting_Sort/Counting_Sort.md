# Counting_Sort

---

## 算法介绍

计数排序是一种非比较排序，排序不依靠数据间的相互比较，而是根据数据来对应下标存储数据个数，实现排序，类似于哈希，建立映射关系。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Counting_Sort-g1.gif)

---

##  代码实现

建立映射必须需要知道数据范围，这将决定映射数组的空间开辟，故需要先找出最大值再映射（不包括负数情况的话）。

首先，若数据量集中在 100 ~ 109 之间，若开辟 110 个空间则产生了浪费，故一般不建立绝对映射而是建立相对映射，故若只有 100 ~ 109 的数据，只需要开辟 10 个空间映射，在映射时，每个数据映射到减去 min 的位置即可，写回时加上 min 时覆盖即可（负数依旧适用）。

```cpp
void counting_sort(int* arr, int n)
{
	if (arr == NULL) { return; } //由于最大值最小值开始默认为第一位，所以数组不能为空

	int max = arr[0];
	int min = arr[0];
    //找出最大最小值
	for (int i = 1; i < n; ++i)
	{
		if (arr[i] > max) { max = arr[i]; }
		if (arr[i] < min) { min = arr[i]; }
	}

	int range = max - min + 1; //界定映射数组大小
	int* tmp = (int*)calloc(range, sizeof(int));
	if (tmp == NULL) { exit(-1); }

    //建立映射关系
	for (int i = 0; i < n; ++i)
	{
		tmp[arr[i] - min]++; //建立相对映射
	}

    //将数据写回原数组完成排序
	int pos = 0;
	for (int i = 0; i < range; ++i)
	{
		while (tmp[i]-- > 0)
		{
			arr[pos++] = i + min; //相对映射的写回方式
		}
	}

	free(tmp);
}
```

---

## 效率分析

计数排序时间复杂度：`O(N)`
计数排序空间复杂度：`O(range)` （range == max - min + 1）

找出最大数和最小数，需要遍历数组，则为：`N`

建立映射关系需要再次遍历数组，则为：`N`

写回数据到原数组相当于覆盖原数组的数据一次，则为：`N`

故时间复杂度为：`N + N + N == 3N`
省略常数项，故为：`O(N)`

由于需要建立映射数组，故空间复杂度为：`O(range)`

---

## 补充说明

* 计数排序是不可能保持稳定的，因为其仅仅统计数据个数而不在乎数据顺序。
* 计数排序仅适用于整数类型的排序，不适用于浮点数等。