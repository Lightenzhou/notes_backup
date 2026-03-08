# Bitset

---

```cpp
#pragma once

#include <vector>
#include <assert.h>

namespace Thepale
{
	template <size_t N = 32>
	class bitset
	{
	private:
		std::vector<char> _v;

	public:
		bitset()
		{
			_v.resize(N / 8 + 1, 0);
		}

		bool set(size_t pos)
		{
			if (pos >= N)
			{
				return false;
			}

			_v[pos / 8] |= 1 << (pos % 8);
			return true;
		}

		bool reset(size_t pos)
		{
			if (pos >= N)
			{
				return false;
			}

			_v[pos / 8] &= ~(1 << (pos % 8));
			return true;
		}

		bool test(size_t pos)
		{
			assert(pos < N);

			return _v[pos / 8] & (1 << (pos % 8));
		}
	};
}
```

