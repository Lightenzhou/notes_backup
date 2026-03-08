# Binary_Tree

---

## 概述

本二叉树的文章设计是为了深度剖析 map 与 set 的底层数据结构，目的是实现红黑树，即平衡二叉搜索树。由于红黑树的体系过于复杂，需要系统解析关于二叉搜索树的部分内容，其将包括：普通二叉搜索树、AVL 树、红黑树。以便支撑后续数据结构的实现。且在本章中会讲述二叉树的前中后序遍历的非递归实现。由于本篇并非体系的数据结构，故排版并未遵循标准。

## 二叉搜索树的实现

二叉搜索树的特性是：**若左子树不为空则所有的左子树上的节点都比根节点小，若右子树不为空则所有右子树上的节点都比根节点大，且在子树中仍然满足这一特性**。下图为例：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Binary_Tree-p1.png)

### find

查找是二叉搜索树最基本的思想，在保证没有重复数据的情况下，通过比根大往右走，比根小往左走，可以非常容易的实现查找：

```cpp
node* find(const T& val)
{
    node* cur = _root;
    while (cur)
    {
        if (val > cur->_val)
        {
            cur = cur->_right;
        }
        else if (val < cur->_val)
        {
            cur = cur->_left;
        }
        else
        {
            return cur;
        }
    }

    return nullptr; //未找到
}
```

当然，二叉树的节点如下：

```cpp
template <typename T>
class binary_search_tree_node
{
public:
	binary_search_tree_node<T>* _left;
	binary_search_tree_node<T>* _right;
	T _val;

	binary_search_tree_node(const T& val = T())
		:_left(nullptr)
		,_right(nullptr)
		,_val(val)
	{}
};
```

上述是为了便捷，故在 binary_search_tree 中（后文简称 BST）定义为：

```cpp
typedef binary_search_tree_node<T> node;
```



### insert

插入的情况无非就是查找 + 创建节点 + 更改链接关系，查找和创建节点是十分容易的，而链接却有些特殊要求。在找到插入的特定位置时，需要同时记录父节点，否则无法完成链接关系的更改。并且此时该位置是父节点的左孩子还是右孩子并不知道，需要再次判断：

```cpp
bool insert(const T& val)
{
    // root 为空的插入情况
    if (_root == nullptr)
    {
        _root = new node(val);
        return true;
    }

    //root 不为空的插入情况
    node* parent = nullptr;
    node* cur = _root;
    while (cur) //找插入位置及该位置的父亲节点
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

    //找到后仍需判断是父亲的左还是右
    node* new_node = new node(val);
    if (val < parent->_val)
    {
        parent->_left = new_node;
    }
    else
    {
        parent->_right = new_node;
    }

    return true;
}
```



### erase

删除是二叉搜索树的难点，主要分为以下三种情况：1.删除的是叶子节点；2.删除的是单孩子节点；3.删除的是双孩子节点。

#### 删除叶子节点

删除叶子节点也需要进行查找 + 更改链接关系 + 删除，此时需要记录父节点，并和插入一致，由于并不知道此时需要删除的叶子节点是左孩子还是右孩子，需要额外判断且需要记录父节点来更改链接关系：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Binary_Tree-p2.png)

```cpp
	bool erase_test(const T& val)
	{
		node* parent = nullptr;
		node* cur = _root;

		while (cur) //查找
		{
			if (val > cur->_val)
			{
				parent = cur;
				cur = cur->_right;
			}
			else if (val < cur->_val)
			{
				parent = cur;
				cur = cur->_left;
			}
			else //找到了
			{
				//叶子节点的情况
				if (cur->_left == nullptr && cur->_right == nullptr)
				{
					if (cur == _root) //删除的是为叶子节点的根节点的情况
					{
						_root = nullptr;
					}
					else if (parent->_left == cur)
					{
						parent->_left = nullptr;
					}
					else
					{
						parent->_right = nullptr;
					}

					delete cur;
					cur = nullptr;

					return true;
				}
			}
		}
	}
```

erase 的查找和 insert 实际上有些不同，我们并不能把 `parent = cur;` 这一语句提出来，而是必须要分别写在 if 语句中，因为如果找到了，则这一语句实际上不会被执行，会直接跳转到第一个 if 的 else 中，这一点要注意区分。

且找到后，需要判断这一节点是否是整棵树的根节点且同时是叶子节点，因为这种情况会导致 parent 未被赋值就直接进入判断，会导致对空指针的解引用，所以这一步一定要单独处理。



#### 删除单孩子节点

单孩子节点无非就是只有左孩子或只有右孩子，而这种情况很好处理，只需要将其具有的单支直接与父节点建立链接关系即可解决。若只有左孩子，则把左孩子链接给父节点，若只有右孩子，则把右孩子链接给父节点（这里的左后孩子并非指单一节点，也可能是一棵树）。而链接给父节点的左还是右节点和查找的原理一致，我们并不知道，故需要额外判断：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Binary_Tree-p3.png)

```cpp
if (cur->_left == nullptr) //只有右孩子
{
    if (_root == cur) //删除的是为单支节点的根节点的情况
    {
        _root = cur->_right;
    }
    else if (parent->_left == cur)
    {
        parent->_left = cur->_right;
    }
    else
    {
        parent->_right = cur->_right;
    }
}
else if (cur->_right == nullptr) //只有左孩子
{
    if (_root == cur) //删除的是为单支节点的根节点的情况
    {
        _root = cur->_left;
    }
    else if (parent->_left == cur)
    {
        parent->_left = cur->_left;
    }
    else
    {
        parent->_right = cur->_right;
    }
}

delete cur;
cur = nullptr;
```

由于部分查找判断过程已经给出，此时不再赘述。此时仍然需要考虑删除的是根节点而导致 parent 为 nullptr 的情况，这种单支情况实际上只需要实现对根节点的变更即可。

而巧合的是，在处理单支节点删除的情况同时，这一可能本身就包括了叶子节点的情况，因为叶子节点本身就是一种单支节点，所以前两种情况实际上可以融合为一种情况，即单孩子节点的删除。



#### 删除双孩子节点

删除双孩子节点无疑是最困难的情况，主要在于删除这一节点后无法通过简单的链接关系变更而维护二叉搜索树，故现在所采用的方法是找到可以替换当前节点的节点，而 **这一节点一定是被删除节点的左子树的最大节点或右子树的最小节点**，请思考。只有这两个节点可以在替换当前需要删除的节点后维护二叉搜索树不变，而衍生出了替换删除。此处将以找到被删除节点的左子树最大节点为例：

```cpp
if (cur->_left != nullptr && cur->_right != nullptr)
{
    node* left_max = cur->_left;
    while (left_max->_right)
    {
        left_max = left_max->_right;
    }

    std::swap(cur->_val, left_max->_val);
}
```

在完成这一交换后，删除双孩子节点的问题就变成了删除单孩子节点的问题或删除叶子节点的问题，因为左子树的最大节点或右子树的最小节点不可能有两个孩子，不然它就不是最大节点或最小节点。此时只需要再走一遍正常的删除逻辑即可，当然，这个时候需要记录 left_max 的父节点：

```cpp
if (cur->_left != nullptr && cur->_right != nullptr)
{
    node* left_max = cur->_left;
    node* left_max_parent = cur;

    while (left_max->_right)
    {
        left_max_parent = left_max;
        left_max = left_max->_right;
    }

    std::swap(cur->_val, left_max->_val);

    if (left_max->_left == nullptr) //叶子节点的情况
    {
        if (left_max_parent->_left == left_max)
        {
            left_max_parent->_left = nullptr;
        }
        else
        {
            left_max_parent->_right = nullptr;
        }
    }
    else //单孩子节点的情况
    {
        left_max_parent->_left = left_max->_left;
    }
}
```

同样，可能存在 `left_max->right` 直接为空的情况，此时 left_max_parent 是没有被更改的，这里将其默认值设为 nullptr 后续再判断是可行的，我个人认为两者的理解难度相当，故此处采用初始化为 cur 的方式。找左子树的最大节点，是一个一直向右走的过程，故最后找到的 left_max 节点不可能有右孩子，所以仅需判断是否存在左孩子就可知道需要删除的节点是叶子节点还是单孩子节点。此情况图示：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Binary_Tree-p4.png)

如果是叶子节点，会有两种情况出现，一种是 cur 碰巧和 left_max_parent 是相同的，这种情况下说明并没有进入循环，同样说明 cur 的左就是需要删除的节点，则此时删除的必然是 left_max_parent（cur） 的左孩子。而如果这两者并不相同，删除的一定是 left_max_parent 的右孩子，因为此时循环必然进入，找到的该叶子节点一定是 left_max_parent 的右孩子。此情况图示：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Binary_Tree-p5.png)

若 left_max_parent 初始化为空，则对应代码应为：	

```cpp
if (left_max->_left == nullptr) //叶子节点的情况
{
    if (left_max_parent == nullptr)
    {
        cur->_left = nullptr;
    }
    else
    {
        left_max_parent->_right = nullptr;
    }
}
else //单孩子节点情况
{
    if (left_max_parent == nullptr)
    {
        cur->_left = left_max->_left;
    }
    else
    {
        left_max_parent->_left = left_max->_left;
    }
}
```

如果是单孩子节点，则完成链接即可。这里需要注意，如果采用 nullptr 初始化 left_max_parent，则在单孩子节点的情况下也需要判断是否为空，如果为空要用 cur 代替 left_max_parent。

我认为，采用 nullptr 初始化 left_max_parent 的方式有利于理解删除叶子节点时 `cur == left_max_parent` 而导致的特殊情况，否则用 cur 初始化而衍生出来的判断是令人头疼的；而采用 cur 初始化 left_max_parent 更有利于理解单孩子节点的删除情况，无论如何都是简单的更新链接关系，去除了对 left_max_parent 的判断。故这两者我无法很好权衡，请自行择优而选。（以上说法皆针对删除双孩子节点的情况，请勿与之前删除叶子节点和单孩子节点的情况混淆）

---

### find 的递归版本

递归版本相对于循环而言是更容易理解的，但递归版本也有自己的弊端，例如存在栈溢出等问题，在实际应用中仍然更推荐使用循环版本，而阐述递归版本的目的是希望可以体会其中思想。

find 的递归是较为基本和简单的，根据大小判断从而决定递归左右子树，递归到空则未找到，找到了就返回当前节点：

```cpp
node* find(const T& val)
{
    return _find(_root, val);
}

node* _find(node* root, const T& val)
{
    if (root == nullptr)
    {
        return nullptr;
    }

    if (val > root->_val)
    {
        return _find(root->_right, val);
    }
    else if (val < root->_val)
    {
        return _find(root->_left, val);
    }
    else
    {
        return root;
    }

}
```

成员函数的递归一般都存在一些问题，例如我直接采用函数 `_find`，那么 `root` 必须要放为公有或需要提供 `get_root` 这样的接口，故一般采用子函数的方式递归。



### insert 的递归版本

insert 同 find 一样需要找到插入位置，而既然要插入节点，由上述循环版本可知，必然要同时查找其父节点，否则无法完成链接关系的变更。解决这一问题可以采用多种方法，例如可以多传一个参数用于记录 `parent`，本文采用指针的引用的方式完成这一过程，请体会这一过程，理解 `node*& root` 会带来什么变化。可知在找到合适的插入位置后，此时的 root 是上一层递归（即父节点）的 `root->right` 或 `root->left` 的引用。而修改当前递归层的 root 则相当于直接修改了上一层 root 的左孩子节点或右孩子节点，链接关系水到渠成。

```cpp
bool insert(const T& val)
{
    return _insert(_root, val);
}

bool _insert(node*& root, const T& val)
{
    if (root == nullptr)
    {
        root = new node(val); //此时的 root 是父节点的 左/右 孩子的引用
        return true;
    }

    if (val > root->_val)
    {
        return _insert(root->_right, val);
    }
    else if (val < root->_val)
    {
        return _insert(root->_left, val);
    }
    else
    {
        return false;
    }
}
```



### erase 的递归版本

erase 的整体思路是不变的，首先需要查找到需要删除的节点，同 insert，由于采用了指针的引用，在删除单孩子节点和叶子节点时链接关系的修改变得更加简单，可以直接借助 root 完成这一过程。而在删除双孩子节点的过程中，查找替换节点依旧是必要的，此时又可以复用 erase 进行删除，因为交换值后，需要删除的节点必然是单孩子节点或叶子节点，这可以复用之前的方法删除。这里唯一需要注意的是，交换值后，整棵树并非标准的二叉搜索树了，故在复用时所传递的是当前被替换节点的左子树，因为交换值后，它的左子树必然还是一颗标准的二叉搜索树，而在完成删除后，整棵树又成了标准的二叉搜索树。

```cpp
bool erase(const T& val)
{
    return _erase(_root, val);
}	

bool _erase(node*& root, const T& val)
{
    if (root == nullptr)
    {
        return false;
    }
	
    //查找需要删除的节点
    if (val > root->_val)
    {
        return _erase(root->_right, val);
    }
    else if (val < root->_val)
    {
        return _erase(root->_left, val);
    }
    else //找到了
    {
        //处理单孩子节点和叶子节点的情况
        if (root->_left == nullptr)
        {
            node* del = root;
            root = root->_right;
            delete del;
        }
        else if (root->_right == nullptr)
        {
            node* del = root;
            root = root->_left;
            delete del;
        }
        else //双孩子节点的情况
        {
            node* left_max = root->_left;
            while (left_max->_right) //找替换节点
            {
                left_max = left_max->_right;
            }

            std::swap(root->_val, left_max->_val); //交换

            _erase(root->_left, val); //复用删除
        }

        return true;
    }
}
```

---

### 拷贝构造函数

树的拷贝构造和析构函数仅提供递归版本，相对容易理解：

```cpp
binary_search_tree(const binary_search_tree& bst)
{
    _root = _copy_tree(bst._root);
}

node* _copy_tree(node* root)
{
    if (root == nullptr)
    {
        return nullptr;
    }

    node* copy_root = new node(root->_val);
    copy_root->_left = _copy_tree(root->_left);
    copy_root->_right = _copy_tree(root->_right);

    return copy_root;
}
```



### 析构函数

```cpp
~binary_search_tree()
{
    _destroy_tree(_root);
}

void _destroy_tree(node* root)
{
    if (root == nullptr)
    {
        return;
    }

    node* del = root;
    _destroy_tree(root->_left);
    _destroy_tree(root->_right);

    delete del;
    del = nullptr;

    return;
}
```

---

### in_order

二叉搜索树在存储数据的过程中，由于其特性默认完成了排序 + 去重，只要输入 BST 的中序遍历即可：

```cpp
void in_order()
{
    _in_order(_root);
    std::cout << std::endl;
}

void _in_order(node* cur)
{
    if (cur == nullptr) { return; }
    _in_order(cur->_left);
    printf("%d ", cur->_val);
    _in_order(cur->_right);
}
```



### 整体实现

```cpp
template <typename T>
class binary_search_tree_node
{
public:
	binary_search_tree_node<T>* _left;
	binary_search_tree_node<T>* _right;
	T _val;

	binary_search_tree_node(const T& val = T())
		:_left(nullptr)
		,_right(nullptr)
		,_val(val)
	{}
};

template <typename T>
class binary_search_tree
{
private:
	typedef binary_search_tree_node<T> node;

private:
	node* _root;

public:
	binary_search_tree()
		:_root(nullptr)
	{}

	binary_search_tree(const binary_search_tree& bst)
	{
		_root = _copy_tree(bst._root);
	}

	~binary_search_tree()
	{
		_destroy_tree(_root);
	}

	binary_search_tree<T>& operator=(binary_search_tree<T> bst)
	{
		std::swap(_root, bst._root);

		return *this;
	}

	bool insert(const T& val)
	{
		// root 为空的插入情况
		if (_root == nullptr)
		{
			_root = new node(val);
			return true;
		}
	
		//root 不为空的插入情况
		node* parent = nullptr;
		node* cur = _root;
		while (cur) //找插入位置及该位置的父亲节点
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
				return false; //直接返回 false，如果 break 后续还需判断
			}
		}
	
		//找到后仍需判断是父亲的左还是右
		node* new_node = new node(val);
		if (val < parent->_val)
		{
			parent->_left = new_node;
		}
		else
		{
			parent->_right = new_node;
		}
	
		return true;
	}

	bool insert_r(const T& val)
	{
		return _insert(_root, val);
	}

	node* find(const T& val)
	{
		node* cur = _root;
		while (cur)
		{
			if (val > cur->_val)
			{
				cur = cur->_right;
			}
			else if (val < cur->_val)
			{
				cur = cur->_left;
			}
			else
			{
				return cur;
			}
		}

		return nullptr;
	}

	node* find_r(const T& val)
	{
		return _find(_root, val);
	}

	bool erase(const T& val)
	{
		//找到需要删除的位置与父节点
		node* parent = nullptr;
		node* cur = _root;
	
		while (cur)
		{
			if (val < cur->_val)
			{
				parent = cur;
				cur = cur->_left;
			}
			else if (val > cur->_val)
			{
				parent = cur;
				cur = cur->_right;
			}
			else //找到了
			{
				if (cur->_left == nullptr) //叶子节点和单孩子节点情况可一并处理
				{
					if (_root == cur)
					{
						_root = cur->_right;
					}
					else if (parent->_left == cur)
					{
						parent->_left = cur->_right;
					}
					else
					{
						parent->_right = cur->_right;
					}
				}
				else if (cur->_right == nullptr)
				{
					if (_root == cur) //解决删除节点是 root 的情况
					{ 
						_root = cur->_left;
					}
					else if (parent->_left == cur)
					{
						parent->_left = cur->_left;
					}
					else
					{
						parent->_right = cur->_left;
					}
	
					delete cur;
					cur = nullptr;
				}
				else //双孩子节点的情况
				{
					//找替换节点（这里找被删除节点左子树最大节点，也可找被删除接待你右子树最小节点）
					node* left_max_parent = cur;
					node* left_max = cur->_left;
					while (left_max->_right)
					{
						left_max_parent = left_max;
						left_max = left_max->_right;
					}
	
					std::swap(cur->_val, left_max->_val); //找到后进行替换
					
	
					//理解一下，left_max 不可能有右节点
					if (left_max->_left == nullptr) //叶子节点的情况
					{
						if (left_max_parent->_left == left_max)
						{
							left_max_parent->_left = nullptr;
						}
						else
						{
							left_max_parent->_right = nullptr;
						}
					}
					else //单孩子节点的情况
					{
						left_max_parent->_left = left_max->_left;
					}
				}
	
				return true;
			}
		}
	
		return false; //循环结束则未找到
	}

	bool erase_r(const T& val)
	{
		return _erase(_root, val);
	}

	void in_order()
	{
		_in_order(_root);
		std::cout << std::endl;
	}

private:
	void _in_order(node* cur)
	{
		if (cur == nullptr) { return; }
		_in_order(cur->_left);
		printf("%d ", cur->_val);
		_in_order(cur->_right);
	}

	node* _copy_tree(node* root)
	{
		if (root == nullptr)
		{
			return nullptr;
		}

		node* copy_root = new node(root->_val);
		copy_root->_left = _copy_tree(root->_left);
		copy_root->_right = _copy_tree(root->_right);

		return copy_root;
	}

	void _destroy_tree(node* root)
	{
		if (root == nullptr)
		{
			return;
		}

		node* del = root;
		_destroy_tree(root->_left);
		_destroy_tree(root->_right);

		delete del;
		del = nullptr;

		return;
	}

	node* _find(node* root, const T& val)
	{
		if (root == nullptr)
		{
			return nullptr;
		}

		if (val > root->_val)
		{
			return _find(root->_right, val);
		}
		else if (val < root->_val)
		{
			return _find(root->_left, val);
		}
		else
		{
			return root;
		}

	}

	bool _insert(node*& root, const T& val)
	{
		if (root == nullptr)
		{
			root = new node(val);
			return true;
		}

		if (val > root->_val)
		{
			return _insert(root->_right, val);
		}
		else if (val < root->_val)
		{
			return _insert(root->_left, val);
		}
		else
		{
			return false;
		}
	}

	bool _erase(node*& root, const T& val)
	{
		if (root == nullptr)
		{
			return false;
		}

		if (val > root->_val)
		{
			return _erase(root->_right, val);
		}
		else if (val < root->_val)
		{
			return _erase(root->_left, val);
		}
		else
		{
			if (root->_left == nullptr)
			{
				node* del = root;
				root = root->_right;
				delete del;
			}
			else if (root->_right == nullptr)
			{
				node* del = root;
				root = root->_left;
				delete del;
			}
			else
			{
				node* left_max = root->_left;
				while (left_max->_right)
				{
					left_max = left_max->_right;
				}

				std::swap(root->_val, left_max->_val);

				_erase(root->_left, val);
			}

			return true;
		}
	}
};
```



---

## 二叉树的非递归遍历

### 二叉树的前序遍历

首先需要明确一点，不管是前序遍历、中序遍历还是后序遍历，它们的顺序必然是`_ 左 _ 右 _`，仅需要调整根在何处，就可以完成三种遍历。而先左后右的顺序是不会变的，变的只是访问根的顺序。

故面对二叉树的非递归遍历，我们需要将其拆分为：左 + 右。即先将一棵树的所有左节点入栈，那剩余需要处理的就是栈中所有节点的右节点，这就简单完成了拆分。而最后一个问题就是什么时候访问根，即可解决此类问题。

由于前序遍历是：根左右，故我们在访问左节点入栈的同时，就应该直接读取根节点数据写入 ans，因为根是最先被访问的。（言语无法完全表达，可以结合递归理解）

```cpp
vector<int> preorderTraversal(TreeNode* root) 
{
    vector<int> ans; //负责存放遍历的结果
    stack<TreeNode*> st; //辅助栈

    while(1)
    {
        while(root) //存放所有左节点
        {
            st.push(root);
            
            ans.push_back(root->val); //根的读取时机

            root = root->left;
        }

        if(!st.empty()) //栈不为空则继续处理
        {
            root = st.top();
            st.pop();

            root = root->right; //继续处理右树数据（继续被拆分为左右）
        }
        else //栈为空则代表所有数据处理完毕
        {
            break;
        }
    }

    return ans;
}
```



### 二叉树的中序遍历

由于中序遍历是：左根右，故在我们访问完了所有左节点后，就应该开始读取根节点，故中序遍历根节点的读取时机便是入完了所有左节点后开始出栈的时候。

```cpp
vector<int> inorderTraversal(TreeNode* root) 
{
    vector<int> ans;
    stack<TreeNode*> st;

    while(1)
    {
        while(root)
        {
            st.push(root);
            root = root->left;
        }

        if(!st.empty())
        {
            root = st.top();
            st.pop();

            ans.push_back(root->val); //仅仅改变根的读取时机即可
            
            root = root->right;
        }
        else
        {
            break;
        }
    }

    return ans;
}
```



### 二叉树的后序遍历

后序遍历和前两者都不相同，因为后序遍历必须要在同时访问完了左子树和右子树的时候才能访问根节点，故必须判断该根节点是否已经访问完了它的右节点，若没有就要去访问它的右节点，若访问了就读取该根节点。

```cpp
vector<int> postorderTraversal(TreeNode* root) 
{
    vector<int> ans;
    stack<TreeNode*> st;

    TreeNode* cur = root;
    while(1)
    {
        while(cur)
        {
            st.push(cur);
            cur = cur->left;
        }

        while(!st.empty())
        {
            cur = st.top(); //这里取栈顶后不能随即出栈，必须等到左右节点访问完，读取了根节点才能出栈。

            //读取根节点细分为两种情况：1.根节点没有右节点，则直接读取；2.根节点的右节点的值已经存在于读取过的根节点中，说明当前根节点的左右节点皆被访
            //问，则该根节点可以被读取。
            if(cur->right == nullptr || (!ans.empty() && cur->right->val == ans.back()))
            {
                ans.push_back(cur->val); //根的读取时机
                st.pop(); //读取后才可出栈
            }
            else
            {
                cur = cur->right;
                break;
            }
        }

        if(st.empty()) { break; }
    }

    return ans;
}
```

从以上的非递归中，我希望你可以理解，除了根节点的任何节点都可以是根节点，左节点或右节点，它们永远是一个相对的概念。就好像在前序遍历中，之所以在访问所有的 “左节点” 时需要同时读取根节点，是因为这里所说的左节点，它们同时也是自己的根节点。“根左右” 所表述的，你当然可以认为是先访问根节点再访问左子树最后访问右子树，但是我更希望你所理解的是，根的位置是一种访问的时机，而并非一个具象的根。 例如 “根左右” “左根右” “左右根”，左右是不变的，访问一个树是先左后右，而正是因为决定了何时访问节点的数据才有了这三种遍历方式。
