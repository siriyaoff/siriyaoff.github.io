---
layout: single
title: "엘리스 코드 챌린지 - 트리 위의 게임"
categories:
  - PS
tags:
  - Dp_tree
  - elice code challenge
---

## 문제

정점이 $$N(1\le N \le 100000)$$개인 트리의 간선 정보 $$N-1$$개가 주어진다.  
루트 노드는 1번이다.  
이때 각 정점에서 시작했을 때 아래 조건에 맞춰 게임을 진행한 후, 결과를 출력해야 한다.  

- 선공, 후공 모두 자신의 차례에 아래 동작을 수행한다.
  - 현재 말이 놓여 있는 정점의 번호를 자신의 점수에 더한다.
  - 현재 말이 놓여 있는 정점의 자식이 없다면 게임을 종료한다.
  - 자식 정점이 있다면, 자식 정점 중 원하는 자식 정점으로 말을 옮긴다.
- 두 사람 모두 최적으로 플레이한다고 가정
- 게임이 종료되었을 때 선공의 점수가 후공의 점수 이상이면 선공 승리, 아니면 후공 승리
- 선공 승리면 1, 후공 승리면 0 출력

$$N$$개의 줄에, 시작 위치가 $$i$$번째일 때의 결과를 각각 출력한다.

## 풀이
- 리프노드일 경우 선공이 승리이기 때문에, $$dp_i$$를 $$i$$번째 노드에서의 선공의 점수 - 후공의 점수라고 정의하면 아래와 같은 점화식을 세울 수 있다.

$$
dp_i = i - min(dp_{childs})
$$

- 자식 노드에서 시작할 때와 현재 노드에서 시작할 때는 선공, 후공이 바뀌기 때문에, 자식 노드 중 dp 값이 가장 작은 것을 선택하는 것이 최적으로 플레이하는 것이다.
- 루트 노드부터 BFS로 탐색해서 순서대로 스택에 담아, 리프노드부터 점수차를 구할 수 있다.

<details markdown="1">
<summary>소스코드</summary>

```cpp
#include <iostream>
#include <queue>
#include <stack>
#include <vector>

using namespace std;
using ll = long long;

int main() {
    ios::sync_with_stdio(0);
    cin.tie(0), cout.tie(0);
    int n;
    ll inf = 99999999999;
    cin >> n;
    vector<int> vis(n, 0);
    vector<ll> dp(n, inf);
    vector<vector<int>> adj(n, vector<int>(0));
    for (int i = 1; i < n; i++) {
        int s, e;
        cin >> s >> e;
        s--, e--;
        adj[s].push_back(e);
        adj[e].push_back(s);
    }
    stack<int> st;
    queue<int> q;
    q.push(0);
    while (!q.empty()) {
        int cur = q.front(), isleaf = 1;
        st.push(cur);
        vis[cur] = 1;
        q.pop();
        for (int i = 0; i < adj[cur].size(); i++) {
            int t = adj[cur][i];
            if (vis[t]) continue;
            q.push(t);
            isleaf = 0;
        }
        if (isleaf) dp[cur] = cur + 1;
    }
    while (!st.empty()) {
        int cur = st.top();
        ll min = inf;
        st.pop();
        if (dp[cur] != inf) continue;
        for (int i = 0; i < adj[cur].size(); i++) {
            int tcur = adj[cur][i];
            if (dp[tcur] != inf && min > dp[tcur]) min = dp[tcur];
        }
        dp[cur] = cur + 1 - min;
    }
    for (int i = 0; i < n; i++) {
        if (dp[i] >= 0)
            cout << 1 << '\n';
        else
            cout << 0 << '\n';
    }
    return 0;
}

```

</details>

## 풀고나서

- 처음에는 top-down 백트래킹을 시도했고, TLE를 받았다.