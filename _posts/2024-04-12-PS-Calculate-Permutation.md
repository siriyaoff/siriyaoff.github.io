---
layout: single
title: "C++에서 permutation 구하기"
categories:
  - PS
tags:
  - C++
  - Permutation
---

Implementation, Simulation 문제를 풀 때는 순열을 사용해야 하는 경우가 종종 있다.  
`<algorithm>` 헤더에 있는 `next_permutation()`을 사용해서 아래와 같이 편리하게 구할 수 있다.

```cpp
#include <algorithm>
#include <iostream>
#include <string>

int main()
{
    std::string s = "aba";
 
    do
    {
        std::cout << s << '\n';
    }
    while (std::next_permutation(s.begin(), s.end()));
 
    std::cout << s << '\n';
}
```
- `first`, `last`있는 모든 자료형에서 사용 가능(`string` 등)

STL을 사용하지 못하는 경우 아래와 같이 dfs를 사용하여 구현할 수 있다.

```cpp
#include <iostream>
#include <vector>

using namespace std;

int n = 4;
vector<int> a{1, 2, 3, 4}, visited(4, 0), res(4, 0);

void permutation(int depth)
{
	if (depth == n)
	{
		for (int i = 0; i < n; i++)
			cout << res[i] << ' ';
		cout << '\n';
		return;
	}
	for (int i = 0; i < n; i++)
	{
		if (!visited[i])
		{
			visited[i] = 1;
			res[depth] = a[i];
			permutation(depth + 1);
			visited[i] = 0;
		}
	}
}

int main()
{
	permutation(0);
}
```
- 사전 순으로 출력할 필요가 없으면 depth번째에 `[depth, n]`을 모두 대입하며 팩토리얼의 정의대로 재귀를 돌리면 코드의 가독성이 좀 더 높아짐
