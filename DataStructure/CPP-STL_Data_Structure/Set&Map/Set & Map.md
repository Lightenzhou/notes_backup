# Set & Map

---

```cpp
//rb_tree
#pragma once

#include <iostream>
#include <assert.h>

enum COLOR
{
    BLACK = 0,
    RED = 1
};

template <typename T>
class rb_tree_node
{
public:
    rb_tree_node<T>* _left;
    rb_tree_node<T>* _right;
    rb_tree_node<T>* _parent;
    bool _color;
    T _val;

public:
    rb_tree_node(const T& val)
		:_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_color(RED)
		,_val(val)
    {}
};

template <typename T, typename Ref, typename Ptr>
class rb_tree_iterator
{
private:
	typedef rb_tree_node<T> node;
	typedef rb_tree_iterator<T, Ref, Ptr> self;
	typedef rb_tree_iterator<T, T&, T*> iterator; //着重区分 rb_tree_iterator<T, Ref, Ptr> 和 rb_tree_iterator<T, T&, T*>

public: //这里必须是公有，否则构造函数访问不了
	node* _cur;

public:
	rb_tree_iterator(node* cur)
		:_cur(cur)
	{}

	rb_tree_iterator(const iterator& s) //同时兼具拷贝构造（用普通迭代器拷贝构造普通迭代器）和构造（用普通迭代器构造const迭代器）的特性
		:_cur(s._cur)
	{}

	bool operator!=(const self& s)
	{
		return _cur != s._cur;
	}

	Ref operator*()
	{
		return _cur->_val;
	}

	Ptr operator->()
	{
		return &(_cur->_val);
	}

	self& operator++()
	{
		node* cur = _cur;
		node* cur_parent = cur->_parent;
		node* cur_right_child = cur->_right;

		if (cur_right_child == nullptr) //右孩子为空
		{
			if (cur_parent->_left == cur)
			{
				_cur = cur_parent;
			}
			else
			{
				while (cur_parent && cur_parent->_right == cur)
				{
					cur = cur_parent;
					cur_parent = cur->_parent;
				}

				_cur = cur_parent;
			}
		}
		else //右孩子不为空
		{
			if (cur_right_child->_left == nullptr)
			{
				_cur = cur_right_child;
			}
			else
			{
				node* cur_rc_max_l = cur_right_child;
				while (cur_rc_max_l->_left != nullptr)
				{
					cur_rc_max_l = cur_rc_max_l->_left;
				}

				_cur = cur_rc_max_l;
			}
		}

		return *this;
	}
};

//红黑树的五个特性：
//1.根节点必须是黑色
//2.不允许出现连续的红节点
//3.树中最长路径不超过最短路径的两倍
//4.所有的叶子节点的空节点都算作黑色节点
//5.每条路径上的黑色节点数量是一样的

template <typename K, typename V, typename Key_of_Value> //这里的 V 在 map 时是 pair
class rb_tree
{
private:
    typedef rb_tree_node<V> node;

public:
	typedef rb_tree_iterator<V, V&, V*> iterator;
	typedef rb_tree_iterator<V, const V&, const V*> const_iterator;

private:
    node* _root;
	Key_of_Value kov;

public:
    rb_tree()
        :_root(nullptr)
    {}

	bool find(const K& key)
	{
		node* cur = _root;

		while (cur)
		{
			if (key > kov(cur->_val))
			{
				cur = cur->_right;
			}
			else if (key < kov(cur->_val))
			{
				cur = cur->_left;
			}
			else
			{
				return true;
			}
		}

		return false;
	}

    pair<iterator, bool> insert(const V& val)
    {
		if (_root == nullptr) //处理根为空的情况
		{
			_root = new node(val);
			_root->_color = BLACK;
			return make_pair(iterator(_root), true);
			//return pair<iterator, bool>(iterator(_root), true);
		}

		node* parent = nullptr;
		node* cur = _root;
		while (cur) //找到合适的插入位置
		{
			parent = cur;

			if (kov(val) < kov(cur->_val))
			{
				cur = cur->_left;
			}
			else if (kov(val) > kov(cur->_val))
			{
				cur = cur->_right;
			}
			else
			{
				return make_pair(iterator(nullptr), false);
				//return pair<iterator, bool>(iterator(nullptr), false);
			}
		}

		node* new_node = new node(val);
		if (kov(val) < kov(parent->_val))
		{
			parent->_left = new_node;
		}
		else
		{
			parent->_right = new_node;
		}
		new_node->_parent = parent;

		//判别颜色
		cur = new_node;
		parent = cur->_parent;

		while (parent && parent->_color == RED)
		{
			node* grandfather = parent->_parent;
			if (grandfather == nullptr)
			{
				assert(0); //grandfather 不可能为空
			}

			node* uncle = grandfather->_left == parent ? grandfather->_right : grandfather->_left;

			if(uncle == nullptr || uncle->_color == BLACK) //uncle 为空或不存在
			{
				if (grandfather->_left == parent && parent->_left == cur)
				{
					rotate_right(grandfather);

					grandfather->_color = RED;
					parent->_color = BLACK;
					cur->_color = RED;
				}
				else if(grandfather->_right == parent && parent->_right == cur)
				{
					rotate_left(grandfather);

					grandfather->_color = RED;
					parent->_color = BLACK;
					cur->_color = RED;
				}
				else if(grandfather->_left == parent && parent->_right == cur)
				{
					rotate_left(parent);
					rotate_right(grandfather);

					grandfather->_color = RED;
					parent->_color = RED;
					cur->_color = BLACK;

				}
				else if(grandfather->_right == parent && parent->_left == cur)
				{
					rotate_right(parent);
					rotate_left(grandfather);

					grandfather->_color = RED;
					parent->_color = RED;
					cur->_color = BLACK;
				}
				else
				{
					assert(0);
				}

				break;
			}
			else //uncle 为红
			{
				grandfather->_color = RED;
				uncle->_color = BLACK;
				parent->_color = BLACK;
				cur->_color = RED;

				cur = grandfather;
				parent = cur->_parent;
			}
		}
		_root->_color = BLACK;

		return make_pair(iterator(new_node), true);
		//return pair<iterator, bool>(iterator(new_node), true);
    }

	iterator begin()
	{
		node* cur = _root;
		while (cur && cur->_left)
		{
			cur = cur->_left;
		}

		return iterator(cur);
	}

	iterator end()
	{
		return iterator(nullptr);
	}

	const_iterator begin() const
	{
		node* cur = _root;
		while (cur && cur->_left)
		{
			cur = cur->_left;
		}

		return const_iterator(cur);
	}

	const_iterator end() const
	{
		return const_iterator(nullptr);
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
	}
};
```



```cpp
#pragma once

#include <iostream>
#include "rb_tree.h"

namespace Thepale
{
	template <typename T>
	class Set_Key_of_Value
	{
	public:
		const T& operator()(const T& val)
		{
			return val;
		}
	};

	template <typename T>
	class set
	{
	public:
		typedef typename rb_tree<T, T, Set_Key_of_Value<T>>::const_iterator iterator;
		typedef typename rb_tree<T, T, Set_Key_of_Value<T>>::const_iterator const_iterator;
	private:
		typedef rb_tree<T, T, Set_Key_of_Value<T>> rb_tree;

	private:
		rb_tree rbt;

	public:
		set() {}

		bool find(const T& val) const
		{
			return rbt.find(val);
		}

		pair<iterator, bool> insert(const T& val) //为什么可以通过编译？已经支持了普通迭代器到const迭代器的转换
		{
			//return rbt.insert(val); //直接返回是有风险的，不一定所有的编译器都会通过

			pair<typename rb_tree::iterator, bool> ret = rbt.insert(val);
			return pair<iterator, bool>(ret.first, ret.second);
		}

		iterator begin() const
		{
			return rbt.begin();
		}

		iterator end() const
		{
			return rbt.end();
		}
	};
}
```



```cpp
// map
#pragma once

#include <iostream>
#include "rb_tree.h"

namespace Thepale
{
	template <typename K, typename V>
	class Map_Key_of_Value
	{
	public:
		const K& operator()(const std::pair<K, V>& val)
		{
			return val.first;
		}
	};

	template <typename K, typename V>
	class map
	{
	public:
		typedef typename rb_tree<K ,std::pair<const K, V>, Map_Key_of_Value<K, V>>::iterator iterator;
		typedef typename rb_tree<K, std::pair<const K, V>, Map_Key_of_Value<K, V>>::const_iterator const_iterator;

	private:
		typedef rb_tree<K, std::pair<const K, V>, Map_Key_of_Value<K, V>> rb_tree; //这里必须匹配 std::pair<const K, V>
		//否则无法完成 iterator<K, std::pair<K, V>, Map_Key_of_Value<K, V>> 到 iterator<K, std::pair<const K, V>, Map_Key_of_Value<K, V>> 的转换

	private:
		rb_tree rbt;

	public:
		map() {}

		bool find(const K& key)
		{
			return rbt.find(key);
		}

		pair<iterator, bool> insert(const std::pair<K, V>& val)
		{

			return rbt.insert(val);
		}

		V& operator[](const K& key)
		{
			pair<iterator, bool> ret = insert(make_pair(key, V()));	//这里调用的是 map 的 insert，故注意返回值
			return ret.first->second;
		}

		iterator begin()
		{
			return rbt.begin();
		}

		iterator end()
		{
			return rbt.end();
		}

		const_iterator begin() const
		{
			return rbt.begin();
		}

		const_iterator end() const
		{
			return rbt.end();
		}
	};
}

```

