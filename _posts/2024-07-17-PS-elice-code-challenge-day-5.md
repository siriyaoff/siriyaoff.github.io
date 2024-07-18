---
layout: single
title: "엘리스 코드 챌린지 - 수열 복원"
categories:
  - PS
tags:
  - Multiset
  - elice code challenge
  - Implementation
---

## 문제

양의 정수로 이루어진 수열 $$a_n$$에 대해, $$n(1\le n \le 15)$$, 모든 부분 수열의 합인 $$2^n$$개의 정수가 $$s_1, \cdots s_{2^n}$$으로 주어질 때, 원래 수열의 원소를 구해야 한다.

## 풀이
- $$s_i$$에서 0을 제외한 최소값을 원래 수열의 원소로 추가하면서, 추가한 원소를 사용하여 만들 수 있는 부분 수열의 합을 $$s_i$$에서 제거하며 n개의 원소를 구하면 된다.

<details markdown="1">
<summary>소스코드</summary>

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

int pow(int x, int p) {
    int r = 1;
    while (p > 0) {
        if (p % 2) r *= x;
        x *= x;
        p /= 2;
    }
    return r;
}

int main() {
    int n;
    cin >> n;
    vector<int> s(pow(2, n), 0), org, pre, sub;
    for (int i = 0; i < pow(2, n); i++) {
        cin >> s[i];
    }
    sort(s.begin(), s.end());
    s.erase(s.begin());
    while (org.size() != n) {
        int elem = s[0];
        sub.push_back(elem);
        for (int i = 0; i < pre.size(); i++)
            sub.push_back(elem + pre[i]);
        sub.erase(unique(sub.begin(), sub.end()), sub.end());
        for (int i = 0; i < sub.size(); i++) {
            int idx = lower_bound(s.begin(), s.end(), sub[i]) - s.begin();
            if (idx == s.size()) continue;
            if (s[idx] == sub[i]) s.erase(s.begin() + idx);
        }
        org.push_back(elem);
        pre.insert(pre.begin(), sub.begin(), sub.end());
        pre.erase(unique(pre.begin(), pre.end()), pre.end());
        sub.clear();
    }
    for (int i = 0; i < org.size(); i++) cout << org[i] << ' ';
    return 0;
}
```

</details>

## 풀고나서

- 해설지에 나온 코드는 아래와 같다.

<details markdown="1">
<summary>소스코드</summary>

```cpp
#include<bits/stdc++.h>
using namespace std;
using ll = long long;
ll n, m, a, ans;
vector<ll>v, res;
multiset<int>now;
void dfs(ll x, ll sum) {
    if (x == res.size()) {
        now.insert(sum + m);
        return;
    }
    dfs(x + 1, sum);
    dfs(x + 1, sum + res[x]);
}
void solve() {
    cin >> n;
    for (int i = 0; i < (1 << n); i++) {
        cin >> a;
        v.push_back(a);
    }
    sort(v.begin(), v.end());
    for (int i =1; i < v.size(); i++) {

		if (now.find(v[i]) == now.end()) {
			m = v[i];
			dfs(0, 0);
			res.push_back(v[i]);
		}
		now.erase(now.find(v[i]));
    }
    for (auto nxt : res)cout << nxt << ' ';
}
int main() {
    ios_base::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    solve();
}
```

</details>

- `multiset`이라는 자료형에 대하여 알게 되었다.
  - always sorted, Associative container 특성을 가지고 있다.
  - `vector`보다 탐색이 훨씬 빠르다(logN)
- `multiset`을 사용하여 중복 제거 없이 가능한 모든 부분 수열의 합을 넣고 $$s_i$$를 정리한다.
  - 로직은 같지만 예시 코드가 가독성이 훨씬 높은 것 같다.
- 사실 $$\sum_1^{n}{2^i}=2^{n+1}-1$$이기 때문에, 중복 제거를 하지 않아도 TLE는 나지 않을 것 같다.