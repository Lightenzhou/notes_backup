# Unordered_Set & Unordered_Map

---

```cpp
#pragma once
#include <iostream>
#include <vector>
using namespace std;

namespace Thepale
{
	template <typename K, typename V, typename Key_of_Value, typename Hash_Table_Func>
	class hash_table; //声明，迭代器需要使用

	template <typename T>
	class hash_table_node
	{
	public:
		T _val;
		hash_table_node<T>* _next;

		hash_table_node(const T& val)
			:_val(val)
			,_next(nullptr)
		{}
	};

	template <typename T>
	class Default_Hash_Table_Func
	{
	public:
		size_t operator()(const T& val)
		{
			return size_t(val);
		}
	};

	template <typename K, typename V, typename Ref, typename Ptr, typename Key_of_Value, typename Hash_Table_Func>
	class hash_table_iterator
	{
	private:
		typedef hash_table_node<V> node;
		typedef hash_table<K, V, Key_of_Value, Hash_Table_Func> hash_table;
		typedef hash_table_iterator<K, V, Ref, Ptr, Key_of_Value, Hash_Table_Func> self;
		typedef hash_table_iterator<K, V, V&, V*, Key_of_Value, Hash_Table_Func> iterator;

	public:
		node* _cur;
		const hash_table* _pht; //这里的哈希表指针必须是 const 指针，因为面对 const 对象，this 指针就是 const 的
		//无法用普通类型接收

		Key_of_Value kov;
		Hash_Table_Func htf;
	public:
		hash_table_iterator(node* cur, const hash_table* pht)
			:_cur(cur)
			,_pht(pht)
		{}

		hash_table_iterator(const iterator& it) //这里无法访问 iterator 的私有，把 iterator 的成员声明为公有即可
			:_cur(it._cur)
			,_pht(it._pht)
		{}

		Ref operator*()
		{
			return _cur->_val;
		}

		Ptr operator->()
		{
			return &(_cur->val);
		}

		self& operator++()
		{
			if (_cur->_next) //_cur 为空就不会进来
			{
				_cur = _cur->_next;
			}
			else
			{
				size_t pos = htf(kov(_cur->_val)) % _pht->_table.size();
				node* cur = nullptr;
				for (size_t i = pos + 1; i < _pht->_table.size(); ++i)
				{
					cur = _pht->_table[i];
					if (cur)
					{
						break;
					}
				}

				_cur = cur;
			}

			return *this;
		}

		bool operator!=(const self& s)
		{
			return _cur != s._cur;
		}
	};

	template <typename K, typename V, typename Key_of_Value, typename Hash_Table_Func = Default_Hash_Table_Func<K>>
	class hash_table
	{
		template <typename K, typename V, typename Ref, typename Ptr, typename Key_of_Value, typename Hash_Table_Func>
		friend class hash_table_iterator; //深入探究回顾一下友元类

	private:
		typedef hash_table_node<V> node;

	public:
		typedef hash_table_iterator<K, V, V&, V*, Key_of_Value, Hash_Table_Func> iterator;
		typedef hash_table_iterator<K, V, const V&, const V*, Key_of_Value, Hash_Table_Func> const_iterator;

	private:
		std::vector<node*> _table;
		size_t _size;

		Key_of_Value kov;
		Hash_Table_Func htf;

	public:
		hash_table()
			:_size(0)
		{
			_table.resize(4, nullptr);
		}

		iterator find(const K& key)
		{
			size_t pos = htf(key) % _table.size();

			node* cur = _table[pos];
			while (cur)
			{
				if (cur->_val == key)
				{
					return iterator(cur);
				}
				else
				{
					cur = cur->_next;
				}
			}

			return iterator(nullptr);
		}
		
		std::pair<iterator, bool> insert(const V& val)
		{
			if (_size == _table.size())
			{
				vector<node*> new_table;
				new_table.resize(_table.size() * 2, nullptr);

				for (size_t i = 0; i < _table.size(); ++i)
				{
					node* cur = _table[i];
					node* next = nullptr;
					while (cur)
					{
						next = cur->_next;

						size_t pos = htf(kov(cur->_val)) % new_table.size();
						cur->_next = new_table[pos];
						new_table[pos] = cur;

						cur = next;
					}

					_table[i] = nullptr;
				}

				std::swap(_table, new_table);
			}

			size_t pos = htf(kov(val)) % _table.size();

			node* new_node = new node(val);
			new_node->_next = _table[pos];
			_table[pos] = new_node;

			++_size;

			return std::pair<iterator, bool>(make_pair(iterator(new_node, this), true));
		}

		iterator begin()
		{
			node* cur = nullptr;

			for (size_t i = 0; i < _table.size(); ++i)
			{
				cur = _table[i];
				if (cur)
				{
					break;
				}
			}

			return iterator(cur, this);
		}

		iterator end() 
		{
			return iterator(nullptr, this);
		}

		const_iterator begin() const
		{
			node* cur = nullptr;

			for (size_t i = 0; i < _table.size(); ++i)
			{
				cur = _table[i];
				if (cur)
				{
					break;
				}
			}

			return const_iterator(cur, this);
		}

		const_iterator end() const
		{
			return const_iterator(nullptr, this);
		}

		void print()
		{
			for (size_t i = 0; i < _table.size(); ++i)
			{
				node* cur = _table[i];

				while (cur)
				{
					//cout << cur->_val << "->";
					cout << cur->_val.first << ":" << cur->_val.second << "->";
					cur = cur->_next;
				}

				cout << "nullptr" << endl;
			}

			cout << endl;
		}
	};
}
```



```cpp
#pragma once

#include "hash_table.h"

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
	class unordered_set
	{
	private:
		typedef hash_table<T, T, Set_Key_of_Value<T>> hash_table;
		
	public:
		typedef typename Thepale::hash_table<T, T, Set_Key_of_Value<T>>::const_iterator iterator;
		typedef typename Thepale::hash_table<T, T, Set_Key_of_Value<T>>::const_iterator const_iterator; //都不能修改，底层都是 const_iterators

	private:
		hash_table _ht;

	public:
		iterator find(const T& val) const
		{
			return _ht.find(val);
		}

		std::pair<iterator, bool> insert(const T& val)
		{
			std::pair<Thepale::hash_table<T, T, Set_Key_of_Value<T>>::iterator, bool> ret = _ht.insert(val);
			return std::pair<iterator, bool>(make_pair(iterator(ret.first), ret.second));
		}

		iterator begin() const //可以同时支持普通迭代器和 const 迭代器，反正底层都是 const 迭代器
		{
			return _ht.begin();
		}

		iterator end() const
		{
			return _ht.end();
		}

		void print()
		{
			_ht.print();
		}
	};
}	
```

```cpp
#pragma once

#include "hash_table.h"

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
	class unordered_map
	{
	private:
		typedef hash_table<K, std::pair<const K, V>, Map_Key_of_Value<K, V>> hash_table; //注意这里的 const K，必须匹配

	public:
		typedef typename Thepale::hash_table<K, std::pair<const K, V>, Map_Key_of_Value<K, V>>::iterator iterator;
		typedef typename Thepale::hash_table<K, std::pair<const K, V>, Map_Key_of_Value<K, V>>::const_iterator const_iterator;

	private:
		hash_table _ht;

	public:
		iterator find(const K& val)
		{
			return _ht.find(val);
		}

		std::pair<iterator, bool> insert(const std::pair<K, V>& val)
		{
			return _ht.insert(val);
		}

		iterator begin()
		{
			return _ht.begin();
		}

		iterator end()
		{
			return _ht.end();
		}

		const_iterator begin() const
		{
			return _ht.begin();
		}

		const_iterator end() const
		{
			return _ht.end();
		}

		void print()
		{
			_ht.print();
		}
	};
}
```

