---
layout: single
title: "DFS의 구현(Recursive/Iterative)"
categories:
  - PS
tags:
  - DFS
  - Implementation
---

<div markdown="1" style="font-size:18px;font-family:'Consolas', 맑은 고딕;">
## 두 구현 방법의 차이
· DFS는 자식 노드 중 방문하지 않은 노드가 있으면 현재 노드에서의 탐색을 멈추고 자식 노드로 옮겨가서 DFS를 완료한 후에 다시 돌아오는 구조이다. 이것을 보기 좋게 함수로 구현한 것이 recursive하게 구현하는 방법이고, stack을 이용해 main안에 한 번에 구현하는 것이 Iterative하게 구현하는 방법이다.  
아래 두 문제는 모두 트리의 지름을 구하는 문제이다. 트리의 지름은 문제를 읽어보면 쉽게 이해할 수 있다.
(내가 푼 방법 말고도 임의의 노드에서 dfs로 가장 멀리 떨어진 노드를 구한 후 그 노드에서 다시 가장 멀리 떨어진 노드를 구하는 방법도 있다. 직관이 필요한 것 같다.)

· [1167 - 트리의 지름](https://www.acmicpc.net/problem/1167){: target="_blank"} - 
[https://www.acmicpc.net/source/...](https://www.acmicpc.net/source/23798904){: target="_blank"}

· [1967 - 트리의 지름](https://www.acmicpc.net/problem/1967){: target="_blank"} - 
[https://www.acmicpc.net/source/...](https://www.acmicpc.net/source/23814837){: target="_blank"}

1167번에서는 dfs를 iterative로 구현했고 1967번에서는 recursive하게 구현했다. 소비한 메모리와 시간은 아래와 같다.

| 구현 방식 | 메모리 | 시간 |
| :--- | :--- | :--- |
| Iterative | `8052KB` | `112ms` |
| Recursive | `2068KB` | `4ms` |


· Recursive가 메모리, 시간 모두 효율적인 것으로 보인다. 하지만 Recursive가 함수를 호출하는 과정때문에 시간이 더 걸리고 stack영역을 사용해서 재귀 호출의 수가 많아지면 터질 수도 있다고 알고있다. 즉 입력이 커질수록 Iterative가 안정적인 것이다.

· 컴파일러에서 [꼬리재귀(tail call recursion)](https://velog.io/@ljinsk3/%EB%B0%98%EB%B3%B5%EB%AC%B8Iteration-vs-%EC%9E%AC%EA%B7%80Recursion){: target="_blank"}를 지원하는 경우 return문에서 연산을 하지 않고 리턴값을 parameter에 포함시켜 최적화시킬 수 있다고 한다.

· recursive도 가독성이 좋다는 장점이 있기 때문에 앞으로 dfs는 recursive, bfs는 iterative하게 구현해야겠다.

</div>