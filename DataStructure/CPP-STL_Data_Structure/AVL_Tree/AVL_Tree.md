# AVL_Tree

---

## 概述

AVL树得名于它的发明者 G. M. Adelson-Velsky 和 E. M. Landis，他们在1962年的论文《An algorithm for the organization of information》中发表了它。而 AVL 树是一个平衡二叉搜索树，即在完成二叉搜索树的基础上加上了平衡特性，而这一平衡特性将采用旋转的方式完成。AVL 树实际上采用了严格平衡，也就是说整棵树及其子树的左右子树高度相差不会超过 1（不可能保持绝对平衡，例如有两个节点，则必须会有一边高一边低）。控制树的平衡可以采用很多种方式，本篇全文采用 **平衡因子** 来控制高度保持平衡，这种方式具象且便于理解。

本篇文章的主要目的也仅仅限于对 AVL 树的了解，而并非深入和探知每个细节，所以该类的讲述并不是完整的，而更像是对某些重要步骤和细节的一种单独提取和讲述（不包括迭代器，不包括删除，不包括一些简单的函数）。若对此有更深入的研究想法，可以参考其它文章。

---

## 类的实现

### 成员变量

在二叉搜索树中，找父节点似乎永远都伴随着每一次插入删除等，由于平衡二叉搜索树依然是一个二叉搜索树，且加之有更为复杂的旋转细节控制，在这里将采用三叉链的形式辅助我们完成，即添加一个额外的成员变量指向自己的父节点，这样不必再单独完成对父节点的查找。且由于采用平衡因子控制平衡，故也需要一个成员变量来记录它，这便是 AVL 树的节点结构：

```cpp
template <typename T>
class avl_tree_node
{
public:
	avl_tree_node* _left;
	avl_tree_node* _right;
	avl_tree_node* _parent; //三叉链
	T _val;
	int _balance_factor; //平衡因子

public:
	avl_tree_node(const T& val)
		:_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_val(val)
		,_balance_factor(0)
	{}
};
```

在这其中，类模板仅仅使用了一个参数 T，而更为标准的话应该是一个 K V 结构，但这里便于理解仅以单参数讲解，在红黑树中将改用 K V 结构。

AVL 树的结构实际上只需要再次封装即可：

```cpp
template <typename T>
class avl_tree
{
private:
	typedef avl_tree_node<T> node; //方便使用

private:
	node* _root = nullptr; //省略了构造函数，初始化为 nullptr 即可
};
```



---

### 成员函数

#### find

查找实际上和搜索二叉树是一致的，这里不过多阐述：

```cpp
bool find(const T& val)
{
    node* cur = _root;
    while (cur)
    {
        if (val < cur->_val)
        {
            cur = cur->_left;
        }
        else if (val > cur->_val)
        {
            cur = cur->_right;
        }
        else
        {
            return cur;
        }
    }

    return nullptr;
}
```



#### rotate_left

需要向左旋转必然出现了右边较高的情况：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p1.png)

如上图所示，由于右边较高，将其向左旋转后，变成了平衡状态。而以上仅仅是针对三个节点进行了演示，而实际上，这三个节点可能都有其各自的子树，包括其自身也可能只是一棵子树。所以为了讨论更完整的情况，我们将以下图为例，这张图将泛型的展示所有需要左单选的情况：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p3.png)

这种情况将触发左单选，如果需要平衡，则需要把左边 “按下来”。由于 B 子树作为 32 节点的右边部分，则所有节点一定比 32 大，这样可以让 B 子树作为 32 的右子树，而 32 及其左右子树作为 64 的左子树，因为 B 在转移之前是 64 的左子树，一定比 64 小，64 作为 32 的右子树，也一定比 32 和 32 的左子树要大，故最后的结果是这样的：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p4.png)

如果这是一颗子树，则 64 节点的父节点也要随之更新。

```cpp
void rotate_left(node* cur)
{
    node* cur_parent = cur->_parent; 
    node* cur_right_child = cur->_right;

    cur->_right = cur_right_child->_left; //key 1：64 节点的左孩子作为 32 节点的右孩子
    if (cur_right_child->_left) //防止 cur_right_child->left 不存在，在双旋中这一点更好理解
    {
        cur_right_child->_left->_parent = cur;
    }

    cur_right_child->_left = cur; //key 2：32 节点及其左右孩子变为 64 的左孩子
    
    //更新三叉链
    cur->_parent = cur_right_child;
    cur_right_child->_parent = cur_parent;
    
    if (cur_parent == nullptr) //为空说明所操作的就是一整棵树，故需要更新根节点
    {
        _root = cur_right_child;
    }
    else //不为空说明所操作的是一颗子树，需要更新三叉链
    {
        if (cur_parent->_left == cur)
        {
            cur_parent->_left = cur_right_child;
        }
        else
        {
            cur_parent->_right = cur_right_child;
        }
    }

    //调整后 cur 和 cur_right_child 的 bf 都是 0
    cur->_balance_factor = 0;
    cur_right_child->_balance_factor = 0;
}
```



#### rotate_right

需要向右旋转必然出现了左边较高的情况：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p2.png)

同左旋，依然以下图为例：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p5.png)

这种情况将会引发右旋，同样，我们需要把右边 “按下来”，16 的右一定比 32 小，且比 16 大，把 16 的右给 32 的左，让 32 变为 16 的右，这种旋转方式便是向右旋转：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p6.png)

无论是左单旋还是右单旋，旋转之后会发现平衡因子都变为了 0，这一点需要注意。

```cpp
void rotate_right(node* cur)
{
    node* cur_parent = cur->_parent;
    node* cur_left_child = cur->_left;

    cur->_left = cur_left_child->_right; //key 1：16 节点的右孩子作为 32 节点的左孩子
    if (cur_left_child->_right) //防止 cur_left_child->right 不存在，在双旋中这一点更好理解
    {
        cur_left_child->_right->_parent = cur;
    }

    cur_left_child->_right = cur; //key 2：32 节点及其左右孩子变为 16 的右孩子
    
    //更新三叉链
    cur->_parent = cur_left_child;
    cur_left_child->_parent = cur_parent;
    
    if (cur_parent == nullptr) //为空说明所操作的就是一整棵树，故需要更新根节点
    {
        _root = cur_left_child;
    }
    else  //不为空说明所操作的是一颗子树，需要更新三叉链
    {
        if (cur_parent->_left == cur)
        {
            cur_parent->_left = cur_left_child;
        }
        else
        {
            cur_parent->_right = cur_left_child;
        }
    }

    //调整后 cur 和 cur_left_child 的 bf 都是 0
    cur->_balance_factor = 0;
    cur_left_child->_balance_factor = 0;
}
```

需要注意的是，如果出现的情况既不是左单选也不是右单旋，而是出现了 “折线” 形状，这将会触发双旋，即左单选和右单旋将被组合使用，在下面的左右双旋和右左双旋中将会解释它们。



#### rotate_left_right

左右双旋最简单的模型是这样的（这种情况旋转后平衡因子都为 0）：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p7.png)

而这一过程是对 16 节点先进行了左单旋，让树变为一边高；再整体右单旋，这样便完成了平衡。左单旋和右单旋的逻辑在之前已经详细叙述，它们的具体展示是这样的：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p9.png)

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p10.png)

只要在 24 节点的左子树或右子树任意插入就会引发左右双旋的情况，这个时候仅靠单旋无法解决问题。以上一图对 B 子树进行了拆分，实际上从抽象上理解，只要在 B 子树上插入就会引发双旋：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p11.png)

实际上，如果仅仅是为了分析双旋的发生条件，那么在 24 节点的左或右插入并不重要，因为 24 的平衡因子并不是关键点。而是只要在 24 的任意子树插入，16 和 32 的平衡因子会呈现出确定的规律，这就是左右双旋的信号。但后续平衡因子的更新需要明确是在左边还是右边插入。双旋需要经历两步：1. 对 16 节点进行左单旋；2.对 32 节点进行右单旋。

16 节点左单旋后：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p12.png)

32 节点进行右单旋后：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p13.png)

具体的单旋逻辑请参考前文。这里虽然是拿在 B 子树上插入为例，但仍然可以进行统一的抽象分析：经过两次旋转后，B 子树变成了 16 的右子树、C 子树变成了 32 的左子树。故在 B 子树或 C 子树插入节点最终会导致平衡因子的变化不同，这一点请自行分析或结合以下代码分析：

```cpp
void rotate_left_right(node* cur)
{
    node* c_lc = cur->_left;
    node* c_lc_rc = c_lc->_right; //判断插入位置的关键
    int bf = c_lc_rc->_balance_factor;

    rotate_left(c_lc);
    rotate_right(cur);

    if (bf == 1) //说明是在 24 右插入引发的双旋
    {
        c_lc_rc->_balance_factor = 0;
        c_lc->_balance_factor = -1;
        cur->_balance_factor = 0;
    }
    else if (bf == -1) //说明是在 24 左插入引发的双旋
    {
        c_lc_rc->_balance_factor = 0;
        c_lc->_balance_factor = 0;
        cur->_balance_factor = 1;
    }
    else if (bf == 0) //特殊情况
    {
        c_lc_rc->_balance_factor = 0;
        c_lc->_balance_factor = 0;
        cur->_balance_factor = 0;
    }
    else
    {
        assert(0);
    }
}
```

这里需要注意一点，对最基本模型进行分析时，双旋后平衡因子全部为 0，故代码中一共有三种情况出现。



#### rotate_right_left

右左双旋最简单的模型是这样的：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p8.png)

而这一过程是对 64 节点先进行了右单旋，让树变为一边高；再整体左单旋，这样便完成了平衡。它的具体形式应该是这样的：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p14.png)

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p15.png)

原理和左右双旋一致，完全可以借助左右双旋推导有做双旋的实现，这里仍然阐述全部实现和图解，希望可以于读者自己的图解形成参考和对比。

对 64 节点进行右单旋：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p16.png)

对 32 节点进行左单旋：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/AVL_Tree-p17.png)

```cpp
	void rotate_right_left(node* cur)
	{
		node* c_rc = cur->_right;
		node* c_rc_lc = c_rc->_left; //判断插入位置的关键
		int bf = c_rc_lc->_balance_factor;

		rotate_right(c_rc);
		rotate_left(cur);

		if (bf == 1) //说明是在 48 右插入引发的双旋
		{
			c_rc_lc->_balance_factor = 0;
			c_rc->_balance_factor = 0;
			cur->_balance_factor = -1;
		}
		else if (bf == -1) //说明是在 48 左插入引发的双旋
		{
			c_rc_lc->_balance_factor = 0;
			c_rc->_balance_factor = 1;
			cur->_balance_factor = 0;
		}
		else if (bf == 0) //特殊情况
		{
			c_rc_lc->_balance_factor = 0;
			c_rc->_balance_factor = 0;
			cur->_balance_factor = 0;
		}
		else
		{
			assert(0);
		}
	}
```



#### insert

当分析并实现了所有的旋转操作，同时也分析出了每一种旋转所对于的判断条件，这样 insert 就可以顺利执行了：

```cpp
bool insert(const T& val)
{
    if (_root == nullptr) //处理根为空的情况
    {
        _root = new node(val);
        return true;
    }

    node* parent = nullptr;
    node* cur = _root;
    while (cur) //找到合适的插入位置
    {
        parent = cur;

        if (val < cur->_val)
        {
            cur = cur->_left;
        }
        else if (val > cur->_val)
        {
            cur = cur->_right;
        }
        else
        {
            return false;
        }
    }

    //插入节点
    node* new_node = new node(val);
    if (val < parent->_val)
    {
        parent->_left = new_node;
    }
    else
    {
        parent->_right = new_node;
    }
    new_node->_parent = parent;

    //初始化一下
    cur = new_node;
    parent = cur->_parent;

    //调节平衡因子（走到这里说明插入成功，平衡因子必须调节）
    while (parent) //结束条件 1：调节到根，即 parent 为空
    {
        if (parent->_left == cur) //调节（是右孩子则增，左孩子则减）（第一次判断的是插入节点对父亲的影响，后面的判断都是相对于整条路径上的）
        {
            --parent->_balance_factor;
        }
        else
        {
            ++parent->_balance_factor;
        }

        if (parent->_balance_factor == 0) //结束条件 2：parent 的 bf 为 0，无需再次调节，健康状态
        {
            break;
        }
        else if(parent->_balance_factor == 1 || parent->_balance_factor == -1) //亚健康，不需要旋转但需要继续影响上层
        {
            cur = parent;
            parent = parent->_parent;
        }
        else if (parent->_balance_factor == 2 || parent->_balance_factor == -2) //生病，需要旋转
        {
            if (parent->_balance_factor == 2 && cur->_balance_factor == 1) //左单旋
            {
                rotate_left(parent);
            }
            else if (parent->_balance_factor == -2 && cur->_balance_factor == -1) //右单旋
            {
                rotate_right(parent);
            }
            else if (parent->_balance_factor == -2 && cur->_balance_factor == 1) //左右双旋
            {
                rotate_left_right(parent);
            }
            else if (parent->_balance_factor == 2 && cur->_balance_factor == -1) //右左双旋
            {
                rotate_right_left(parent);
            }
            else //这是不可能出现的情况，用来杜绝错误
            {
                assert(0);
            }

            break;
        }
        else //这是不可能出现的情况，用来杜绝错误
        {
            assert(0);
        }
    }

    return true;
}
```

在 insert 的实现中，调节平衡因子是关键之一。在左插入则减一，在右插入则加一，然后依次向上更新。请注意，对于该树的父节点来说，是否需要调整父节点的平衡因子取决于子树的总高度是否变化，故当此子树的平衡因子变为 0 时，说明刚开始是单边高的情况，而现在变成一样高了，但这两种情况这颗树的总高度是没有变化的，所以无需再向上调整；反之如果这颗树的平衡因子插入之后平衡因子变成了 -1 或 1，这说明树原本是平衡的，结果变得单边高了，这样树的总高度发生了变化，故必然要继续向上调整。

旋转则是关键之二，之前已经系统剖析了旋转逻辑，故在这里直接调用即可。有一点有趣的是，本来以上已经包揽了所有情况，但并没有用 else 处理最后一种情况。这里需要提醒的是，没有人能保证自己的代码不会出错，这往往是一种随附的检测手段，用来提示是否有意料之外的错误发生。



#### is_balance

这是一个用于检测的函数，检测实现的数据结构是否严格遵守了 AVL 树的标准，而这里我们必须硬性判断树的高度，并和平衡因子比较，以此纠错。在很多的程序设计之中这一步是必要的，用于测试自己的代码是否正确：

```cpp
void is_balance()
{
    _is_balance(_root);
}

int _is_balance(node* cur)
{
    if (cur == nullptr)
    {
        return 0;
    }

    int left_height = _is_balance(cur->_left);
    int right_height = _is_balance(cur->_right);

    if (right_height - left_height != cur->_balance_factor)
    {
        printf("val:%d 平衡因子错误，正确的平衡因子为：%d\n", cur->_val, right_height - left_height);
    }

    return std::max(left_height, right_height) + 1;
}
```



### 整体实现

```cpp
#pragma once

#include <iostream>
#include <algorithm>
#include <assert.h>


template <typename T>
class avl_tree_node
{
public:
	avl_tree_node* _left;
	avl_tree_node* _right;
	avl_tree_node* _parent; //三叉链
	T _val;
	int _balance_factor; //平衡因子

public:
	avl_tree_node(const T& val)
		:_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_val(val)
		,_balance_factor(0)
	{}
};

template <typename T>
class avl_tree
{
private:
	typedef avl_tree_node<T> node;

private:
	node* _root = nullptr;

public:
	bool find(const T& val)
	{
		node* cur = _root;
		while (cur)
		{
			if (val < cur->_val)
			{
				cur = cur->_left;
			}
			else if (val > cur->_val)
			{
				cur = cur->_right;
			}
			else
			{
				return cur;
			}
		}

		return nullptr;
	}

	bool insert(const T& val)
	{
		if (_root == nullptr) //处理根为空的情况
		{
			_root = new node(val);
			return true;
		}

		node* parent = nullptr;
		node* cur = _root;
		while (cur) //找到合适的插入位置
		{
			parent = cur;

			if (val < cur->_val)
			{
				cur = cur->_left;
			}
			else if (val > cur->_val)
			{
				cur = cur->_right;
			}
			else
			{
				return false;
			}
		}

		node* new_node = new node(val);
		if (val < parent->_val)
		{
			parent->_left = new_node;
		}
		else
		{
			parent->_right = new_node;
		}
		new_node->_parent = parent;
		
		//初始化一下
		cur = new_node;
		parent = cur->_parent;

		//调节平衡因子（走到这里说明插入成功，平衡因子必须调节）
		while (parent) //结束条件 1：调节到根，即 parent 为空
		{
			if (parent->_left == cur) //调节（是右孩子则增，左孩子则减）（第一次判断的是插入节点对父亲的影响，后面的判断都是相对于整条路径上的）
			{
				--parent->_balance_factor;
			}
			else
			{
				++parent->_balance_factor;
			}

			if (parent->_balance_factor == 0) //结束条件 2：parent 的 bf 为 0，无需再次调节，健康状态
			{
				break;
			}
			else if(parent->_balance_factor == 1 || parent->_balance_factor == -1) //亚健康，不需要旋转但需要继续影响上层
			{
				cur = parent;
				parent = parent->_parent;
			}
			else if (parent->_balance_factor == 2 || parent->_balance_factor == -2) //生病，需要旋转
			{
				if (parent->_balance_factor == 2 && cur->_balance_factor == 1) //左单旋
				{
					rotate_left(parent);
				}
				else if (parent->_balance_factor == -2 && cur->_balance_factor == -1) //右单旋
				{
					rotate_right(parent);
				}
				else if (parent->_balance_factor == -2 && cur->_balance_factor == 1) //左右双旋
				{
					rotate_left_right(parent);
				}
				else if (parent->_balance_factor == 2 && cur->_balance_factor == -1) //右左双旋
				{
					rotate_right_left(parent);
				}
				else
				{
					assert(0);
				}

				break;
			}
			else //这是不可能出现的情况，用来杜绝错误
			{
				assert(0);
			}
		}

		return true;
	}

		void is_balance()
		{
			_is_balance(_root);
		}

private:
	void rotate_left(node* cur)
	{
		node* cur_parent = cur->_parent;
		node* cur_right_child = cur->_right;

		cur->_right = cur_right_child->_left; //key 1：生病节点的右孩子的左给生病节点的右
		if (cur_right_child->_left) //可能不存在？
		{
			cur_right_child->_left->_parent = cur;
		}

		cur_right_child->_left = cur; //key 2：生病节点的右孩子的左变为生病节点
		cur->_parent = cur_right_child;

		cur_right_child->_parent = cur_parent;
		if (cur_parent == nullptr) //空的时候需要更新 _root
		{
			_root = cur_right_child;
		}
		else
		{
			if (cur_parent->_left == cur)
			{
				cur_parent->_left = cur_right_child;
			}
			else
			{
				cur_parent->_right = cur_right_child;
			}
		}

		//调整后 cur 和 cur_right_child 的 bf 都是 0
		cur->_balance_factor = 0;
		cur_right_child->_balance_factor = 0;
	}
	
	void rotate_right(node* cur)
	{
		node* cur_parent = cur->_parent;
		node* cur_left_child = cur->_left;

		cur->_left = cur_left_child->_right;
		if (cur_left_child->_right) //可能不存在？
		{
			cur_left_child->_right->_parent = cur;
		}

		cur_left_child->_right = cur;
		cur->_parent = cur_left_child;

		cur_left_child->_parent = cur_parent;
		if (cur_parent == nullptr) //空的时候需要更新 _root
		{
			_root = cur_left_child;
		}
		else
		{
			if (cur_parent->_left == cur)
			{
				cur_parent->_left = cur_left_child;
			}
			else
			{
				cur_parent->_right = cur_left_child;
			}
		}

		//调整后 cur 和 cur_right_child 的 bf 都是 0
		cur->_balance_factor = 0;
		cur_left_child->_balance_factor = 0;
	}

	void rotate_left_right(node* cur)
	{
		node* c_lc = cur->_left;
		node* c_lc_rc = c_lc->_right; //判断插入位置的关键
		int bf = c_lc_rc->_balance_factor;

		rotate_left(c_lc);
		rotate_right(cur);

		if (bf == 1) //说明是在右插入引发的双旋
		{
			c_lc_rc->_balance_factor = 0;
			c_lc->_balance_factor = -1;
			cur->_balance_factor = 0;
		}
		else if (bf == -1) //说明实在左插入引发的双旋
		{
			c_lc_rc->_balance_factor = 0;
			c_lc->_balance_factor = 0;
			cur->_balance_factor = 1;
		}
		else if (bf == 0) //特殊情况
		{
			c_lc_rc->_balance_factor = 0;
			c_lc->_balance_factor = 0;
			cur->_balance_factor = 0;
		}
		else
		{
			assert(0);
		}
	}

	void rotate_right_left(node* cur)
	{
		node* c_rc = cur->_right;
		node* c_rc_lc = c_rc->_left; //判断插入位置的关键
		int bf = c_rc_lc->_balance_factor;

		rotate_right(c_rc);
		rotate_left(cur);

		if (bf == 1) //说明是在右插入引发的双旋
		{
			c_rc_lc->_balance_factor = 0;
			c_rc->_balance_factor = 0;
			cur->_balance_factor = -1;
		}
		else if (bf == -1) //说明实在左插入引发的双旋
		{
			c_rc_lc->_balance_factor = 0;
			c_rc->_balance_factor = 1;
			cur->_balance_factor = 0;
		}
		else if (bf == 0) //特殊情况
		{
			c_rc_lc->_balance_factor = 0;
			c_rc->_balance_factor = 0;
			cur->_balance_factor = 0;
		}
		else
		{
			assert(0);
		}
	}

	int _is_balance(node* cur)
	{
		if (cur == nullptr)
		{
			return 0;
		}

		int left_height = _is_balance(cur->_left);
		int right_height = _is_balance(cur->_right);

		if (right_height - left_height != cur->_balance_factor)
		{
			printf("val:%d 平衡因子错误，正确的平衡因子为：%d\n", cur->_val, right_height - left_height);
		}

		return std::max(left_height, right_height) + 1;
	}
};
```



## 补充说明

* AVL 树是一种相对复杂的结构，捋清或许不是一时半会的功夫，且这里仅仅是对 insert 进行了系统分析，旨在了解 AVL 的基本底层逻辑。erase 的实现和 insert 有极大的相似之处，只不过很多不步骤或许是相反的，感兴趣可以自行探究。
