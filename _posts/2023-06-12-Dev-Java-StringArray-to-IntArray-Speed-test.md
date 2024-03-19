---
layout: single
title: "Java String[], int[] 변환 방법별 속도 측정"
categories:
  - Dev
tags:
  - Java
  - Stream
  - Python
  - plot
---

```java
public class Main {
    public static void main(String[] args) {
        int rep = 100, size = 10000000;
        int[] res = {0, 0, 0};
        long tinit, t1, t2, t3, tter;
        List<String> lstr = new ArrayList<>();
        for (int i = 0; i < 10000000; i++) {
            lstr.add(String.valueOf(i % 10));
        }
        String[] strings = lstr.toArray(new String[0]);

        for (int i = 0; i < rep; i++) {
            int tmp1, tmp2, tmp3;
            tinit = System.currentTimeMillis();
            int[] nums1 = Arrays.stream(strings).mapToInt(Integer::parseInt).toArray();
            t1 = System.currentTimeMillis();
            int[] nums2 = Arrays.asList(strings).stream().mapToInt(Integer::parseInt).toArray();
            t2 = System.currentTimeMillis();
            int[] nums3 = new int[strings.length];
            for (int j = 0; j < strings.length; j++) {
                nums3[j] = Integer.parseInt(strings[j]);
            }
            tter = System.currentTimeMillis();
            tmp1 = (int)(t1 - tinit);
            tmp2 = (int)(t2 - t1);
            tmp3 = (int)(tter - t2);
            System.out.println("rep : " + i + " \tnums1 = " + tmp1 + "\tnum2 = " + tmp2 + "\tnum3 = " + tmp3);
            res[0] += tmp1;
            res[1] += tmp2;
            res[2] += tmp3;
        }
        System.out.println("Size : " + size + " Iteration : " + rep);
        System.out.println("num1 : " + (res[0] / rep) + "ms");
        System.out.println("num2 : " + (res[1] / rep) + "ms");
        System.out.println("num3 : " + (res[2] / rep) + "ms");
    }
}
```

- `nums1` : `String[]` → stream 처리
- `nums2` : `String[]` → `List<>` → stream 처리
- `nums3` : `String[]` → 원소 하나씩 `Integer.parseInt()`
- `lstr.toArray(new String[0])`은 `lstr.size()`, 파라미터 Array의 크기를 비교해서 큰 쪽을 리턴 Array의 크기로 설정함

## 실행 결과

```
rep : 0 	nums1 = 100	num2 = 70	num3 = 43
rep : 1 	nums1 = 52	num2 = 44	num3 = 64
rep : 2 	nums1 = 43	num2 = 55	num3 = 47
rep : 3 	nums1 = 43	num2 = 43	num3 = 41
rep : 4 	nums1 = 42	num2 = 42	num3 = 40
rep : 5 	nums1 = 43	num2 = 41	num3 = 40
rep : 6 	nums1 = 42	num2 = 41	num3 = 41
rep : 7 	nums1 = 42	num2 = 40	num3 = 40
rep : 8 	nums1 = 41	num2 = 41	num3 = 39
rep : 9 	nums1 = 41	num2 = 41	num3 = 40
...
rep : 90 	nums1 = 41	num2 = 41	num3 = 41
rep : 91 	nums1 = 42	num2 = 43	num3 = 41
rep : 92 	nums1 = 42	num2 = 40	num3 = 40
rep : 93 	nums1 = 41	num2 = 41	num3 = 41
rep : 94 	nums1 = 42	num2 = 41	num3 = 40
rep : 95 	nums1 = 42	num2 = 42	num3 = 42
rep : 96 	nums1 = 41	num2 = 40	num3 = 41
rep : 97 	nums1 = 42	num2 = 41	num3 = 40
rep : 98 	nums1 = 44	num2 = 40	num3 = 40
rep : 99 	nums1 = 41	num2 = 41	num3 = 39
Size : 10000000 Iteration : 100
num1 : 43ms
num2 : 41ms
num3 : 41ms
```

### plot

```python
import matplotlib.pyplot as plt
ref="RESULT"
rep=100
l=ref.strip().split('\n')
x=[]
y1=[]
y2=[]
y3=[]
for i in range(rep):
    tstr=l[i].split(",")
    x.append(int(tstr[0]))
    y1.append(int(tstr[1]))
    y2.append(int(tstr[2]))
    y3.append(int(tstr[3]))
plt.plot(x, y1, 'r', label='num1')
plt.plot(x, y2, 'g', label='num2')
plt.plot(x, y3, 'b', label='num3')
plt.xlabel('iteration')
plt.ylabel('ms')
plt.legend(loc='upper right')
plt.show()
```

- 자바에서 csv 형식으로 바꿔서 처리함

![string-intarray-speed-test](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/string-intarray-speed-test.png)

- 위 그래프는 `nums1`, `nums2`, `nums3`의 순서대로 실행한 결과임

## Discussion

- (`nums2`, `nums1`, `nums3`), (`nums1`, `nums2`, `nums2`, `nums3`)의 순서대로 속도를 측정해도 항상 `stream`을 사용하는 첫 번째 실행만 속도가 느리고 나머지는 40ms 대로 비슷하게 측정됨
  - `stream`으로 변환한 다음 `toArray()`로 종료해줘도 어딘가에 레퍼런스가 남아있어서 그걸 계속 사용하는듯
- 변환 작업을 여러 번 수행할 때는 어떤 것을 사용하든 모두 실행 속도가 결국 비슷해짐
- 변환 작업을 한 번만 수행할 때는 `parseInt()`로 하나씩 변환해주는게 속도가 빠른 듯
