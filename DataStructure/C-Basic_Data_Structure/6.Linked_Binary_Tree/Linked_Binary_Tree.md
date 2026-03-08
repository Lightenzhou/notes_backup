# Linked_Binary _Tree

---

## 结构类

单独的链式二叉树仅仅是一种树型结构，对其进行增删查改的操作没有实际意义，采用树的存储方式，其目的并不在于单纯的存储数据，而更多的是需要完成对数据的特殊要求（例如查找、排序等）。故此文并非对增删查改进行具体提分析，而是对二叉树的基本操作进行分析和解读。

逻辑结构图（这同时是一颗链式二叉搜索树，暂不讨论链式二叉搜索树）：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Linked_Binary_Tree-p1.png)

这里采用链式结构存储，因为无法保证树的结构是完全二叉树，如果遇到的是这样的树，其对应的数组表示如下所示：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Linked_Binary_Tree-p2.png)

会导致大量的空间浪费，却不存数据的位置还需要附带 bool 等变量进一步判断（因为仅靠所存储的数据类型无法表示数据是否存在，例如存储的是整型数据，若规定 -1 是无数据，则 -1 这个数据无法存储，所以需要借助另一个变量才可判断）。并且链式结构能够清晰的表述逻辑关系。

故二叉树节点的结构如下：

```cpp
typedef int tree_datatype;

struct tree_node
{
    tree_datatype val;
    struct tree_node* left;
    struct tree_node* right;
};

typedef struct tree_node tree_node;
```

二叉树仅需要一个头节点就可以对整个树进行一系列操作，故无需单独的结构体封装。

---

## 成员函数

### binary_tree_prev_order

对于二叉树，三种遍历方式是需要首先掌握的，它们分别是：前序遍历，中序遍历，后序遍历。前序遍历所对应的是 **根 左 右**，中序遍历对应的是 **左根右**，后序遍历对应的是 **左 右 根**。其意义在于访问根节点的时机不同。

```cpp
void binary_tree_prev_order(tree_node* root)
{
	if (root == NULL)
	{
		printf("NULL ");
		return;
	}

	printf("%c ", root->val);
	binary_tree_prev_order(root->left);
	binary_tree_prev_order(root->right);
}
```

所有函数都统一对此树进行分析：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Linked_Binary_Tree-p1.png)

其打印的结果为：`4 2 1 NULL NULL 3 NULL NULL 5 NULL 6 NULL NULL`

递归展开图如下（仅展示单路递归）：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-Linked_Binary_Tree-p3.png)



### binary_tree_in_order

中序遍历的顺序为：**左 根 右**

```cpp
void binary_tree_in_order(tree_node* root)
{
	if (root == NULL)
	{
		printf("NULL ");
		return;
	}

	binary_tree_in_order(root->left);
	printf("%c ", root->val);
	binary_tree_in_order(root->right);
}
```

其打印结果为：`NULL 1 NULL 2 NULL 3 NULL 4 NULL 5 NULL 6 NULL`
（这里发现数据有序实则是搜索二叉树的性质，暂不深入研究）

递归展开图如下（仅展示单路递归）：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/QQ%E5%9B%BE%E7%89%8720230702154925.png)

需要注意，由于前序遍历总是先打印根节点，故节点的打印对应着步骤的先后，但中序遍历和后序遍历不是按步骤打印，故图中单独说明了打印的时机，切勿混淆。



### binary_tree_post_order

后序遍历的顺序为：**左 右 根**

```cpp
void binary_tree_post_order(tree_node* root)
{
	if (root == NULL)
	{
		printf("NULL ");
		return;
	}

	binary_tree_post_order(root->left);
	binary_tree_post_order(root->right);
	printf("%c ", root->val);
}
```

其打印结果为：`NULL NULL 1 NULL NULL 3 2 NULL NULL NULL 6 5 4`

递归展开图如下（仅展示单路递归）：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-LinkedBinaryTree-p5.png)

综上三图所述，可知其对于单路的递归路径完全一致（完全展开图请自行画图），仅仅是访问（打印）根节点的顺序不一样。而正是对根节点的访问时机不同，在访问的路径完全相同的情况下却得到的三种序列的遍历方式和特性，在整个二叉树的的递归中也是如此。



### binary_tree_size

此函数的目的是统计整棵树中的节点个数，而对于二叉树的此类问题，多考虑用递归解决，递归的本质就是分治，若把这个问题分解成子问题则为：**树的总节点个数 = 左子树节点个数 + 右子树节点个数 + 1**，而每一个树都可以这样拆解直至无法拆解为止，即把一个问题有限次的拆解成许多最小子问题，则为分治，也同时是递归。

```cpp
int binary_tree_size(tree_node* root)
{
    if(root == NULL) {return 0;}
    
    return binary_tree_size(root->left) + binary_tree_size(root->right) + 1;
}
```

如果把代码拆分成这样：

```cpp
int binary_tree_size(tree_node* root)
{
    if(root == NULL) {return 0;}
    
    int left_size = binary_tree_size(root->left);
    int right_size = binary_tree_size(root->right);
    
    return left_size + right_size + 1;
}
```

会发现递归的路径与前中后序遍历并无二至（如果你遵循先左后右的顺序的话），递归和分治本就息息相关，前中后序遍历也可看做是对二叉树进行一定顺序的有限次拆分和访问，故前中后序遍历也属于分治，只是相比具体问题不够具象。



### binary_tree_leaf_size

此函数的目的是统计整棵树的叶子节点个数，相比于上一个函数，仅仅是多了一个条件判断是否为叶子节点。

```cpp
int binary_tree_leaf_size(tree_node* root)
{
    if(root == NULL) {return 0;}
    
    if(root->left == NULL && root->right == NULL)
    {
        return 1;
	}
    
    return binary_tree_leaf_size(root->left) + binary_tree_leaf_size(root->right);
}
```



### binary_tree_height

此函数的目的是求出树有多高，即二叉树有多少层，仍然采用分治思想：**二叉树的高度 = max(左子树高度，右子树高度) + 1**，以此拆分分治即可。

```cpp
int binary_tree_height(tree_node* root)
{
    if(root == NULL) {return 0;}
    
    int left_height = binary_tree_height(root->left);
    int right_height = binary_tree_height(root->right);
    
    return (left_height > right_height ? left_height : right_height) + 1;
}
```

切记不可图省事而写成这样：

```cpp
int binary_tree_height(tree_node* root)
{
    if(root == NULL) {return 0;}
    
    return (binary_tree_height(root->left) > binary_tree_height(root->right) ? binary_tree_height(root->left) : binary_tree_height(root->right)) + 1;
}
```

这样会导致 "失忆性递归"（我喜欢这样称呼），也就是递归没有记忆性，第一次只判断谁大，之后就忘记了它是多少，则又需要回去算一遍，这不仅仅是简单的算了两遍，复杂度并非倍数增长。若对于倒数第二层而言，倒数第一层被多调用了一次，但对于倒数第三层而言，第二层也被多调用了一次，这样对于倒数第一层就多调用了三次，以`2^n`增长，具体可画递归展开图分析。

### binary_tree_klevel_size

此函数的目的是求出二叉树特定层数（第 k 层）的节点个数，实现方法依然是分治思想：**第 k 层的节点个数相当于子树的第 k - 1 层的节点个数，直至 k 为 1**，（层数是从 1 开始的，因为为了更好的用 0 表示空树）

```cpp
int binary_tree_klevel_size(tree_node* root, int k)
{
    if(root == NULL) {return 0;}
    
    if(k == 1)
    {
        return 1;
	}
    
    return binary_tree_klevel_size(root->left, k - 1) + binary_tree_klevel_size(root->right, k - 1);
}
```



### binary_tree_find

此函数负责找出特定值的节点并返回指针，面临两个问题：第一个是正常的分治，有没有分别看左子树有没有和右子树有没有，第二个是负责把找到的节点指针送上来，这也就是 return 的重要操作。

```cpp
tree_node* binary_tree_find(tree_node* root, tree_datatype data)
{
    if(root == NULL) {return NULL;}
    
    if(root->val == data) {return root;}
    
    tree_node* left_find = binary_tree_find(root->left, data);
    tree_node* right_find = binary_tree_find(root->right, data);
    
    return left_find == NULL ? right_find : left_find;
}
```

这里的 return 具体展开应该是这样的，上面为简写方式，但所表达的意思是一致的：

```cpp
tree_node* binary_tree_find(tree_node* root, tree_datatype data)
{
    if(root == NULL) {return NULL;}
    
    if(root->val == data) {return root;}
    
    tree_node* left_find = binary_tree_find(root->left, data);
    tree_node* right_find = binary_tree_find(root->right, data);
    
    
    if(left_find == NULL && right_find == NULL) {return NULL;} //左右子树都没有
    if(left_find == NULL) {return right_find;} //已经确定左右不同时为空，若左为空说明右子树找到了，右为空说明左子树找到了
    return left_find;
}
```



### binary_tree_level_order

层序遍历需要借助队列，所以这里使用之前所写的队列。其主要思路是，先把根节点放进去，出一个节点则把它的左孩子和右孩子入队列，以此可以做到一层一层的从左往右的遍历方式。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/DataStructure-LinkedBinaryTree-p6.png)

```cpp
void binary_tree_level_order(tree_node* root) //此代码不入 NULL，有需求可以自行修改
{
    if(root == NULL) {return;}
    
    queue qu; //queue_datatype 是 tree_node*，通过指针才能找到左右孩子，而不是数值
    if(queue_empty(&qu)) {queue_push(&qu, root);}
    
    while(!queue_empty(&qu)) //队列出完了才算结束
    {
        tree_node* node = queue_front(&qu);
        queue_pop(&qu); //这里的 pop 释放的是队列的节点，不会把二叉树的节点释放掉：即二叉树在队列中相当于队列的 val，队列是再开了一块空间来存储这个 					  //val，而这块空间的释放不影响 val，即不影响二叉树。
        
        printf("%d ", node->val);
        
        if(node->left != NULL) {queue_push(&qu, node->left);}
        if(node->right != NULL) {queue_push(&qu, node->right);}
	}
}
```



---

## 补充说明

* 对于任何关于递归分治问题无法很好理解的地方，务必画递归展开图。
* 本文章旨在通过二叉树的基本操作函数强化对递归分治的理解，并能一举两得运用此类思想解决类似问题。（虽然有部分操作，但并不包括类似于：判断二叉树是否为完全二叉树、是否为对称二叉树，一棵树是否为另一棵树的子树等类似问题，无法做到全部涵盖，但思想互通，授人以渔为关键）
