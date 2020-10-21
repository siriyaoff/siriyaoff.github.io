---
layout: single
title: "C++ I/O 헷갈리는 내용 정리2"
categories:
  - Miscellaneous
tags:
  - C++
---
<div markdown="1" style="font-size:18px;font-family:'Consolas', 맑은 고딕;">
원래는 문제별로 적으려 했는데 한번에 정리해놓고 보는게 좋을 것 같다.

- [`long int strtol (const char* str, char** endptr, int base)`](http://www.cplusplus.com/reference/cstdlib/strtol/){: target="_blank"}
	* `<cstdlib>`에 정의되어 있다.
	* base진법의 수 str이 들어오면 10진법으로 바꾼 값을 리턴해준다.
	* str이 여러 개의 수로 이루어져있다면 첫 번째 수만 바꾸고 두 번재 수가 시작하는 위치를 endptr에 저장해주기 때문에 endptr을 이용해서 여러 개의 수를 한 번에 입력받아 바꿀 수 있다.

<details>
<summary>예시</summary>
<div markdown="1" style="font-size:20px;font-family:'Consolas', 맑은 고딕;">
**[BOJ 2745](https://www.acmicpc.net/problem/2745){: target="_blank"}**  
	* B(B<=36)진법 수 N이 들어오면 10진수로 바꿔 출력하는 문제이다.

```cpp
#include<cstdio>
#include<cstdlib>
int main()
{
	int b;
	char s[100];
	scanf("%s %d", s, &b);
	printf("%d", strtol(s, NULL, b));
}

// #include<iostream>
// #include<cstdio>
// #include<string>
// using namespace std;

// int t(char c){
// 	if(c-'0'<10) return c-'0';
// 	return c-'A'+10;
// }

// int main()
// {
// 	int b, n=0;
// 	string s;
// 	cin>>s;
// 	scanf("%d", &b);
// 	for(int i=1;!s.empty();i*=b){
// 		n+=i*t(s.back());
// 		if(s.empty()) break;
// 		s.pop_back();
// 	}
// 	printf("%d", n);
// }
```
	* `string`은 pop_back()으로 마지막 문자를 지우는게 편하다.

</div>
</details>

- strlen의 경우 시간복잡도가 O(N)이다.(O(1)이 아니다!)

<details>
<summary>BOJ 1373을 풀면서 알게 되었다.</summary>
<div markdown="1" style="font-size:20px;font-family:'Consolas', 맑은 고딕;">
**[BOJ 1373](https://www.acmicpc.net/problem/1373){: target="_blank"}**  
	* 2진수 N이 들어오면 8진수로 바꿔 출력하는 문제이다.

```cpp
#include<cstdio>
#include<cstdlib>
#include<cstring>
int main()
{
	char s[1000001];
	scanf("%s", s);
	int p=strlen(s)%3;
	if(p==1) printf("%d", s[0]-'0');
	if(p==2) printf("%d", (s[0]-'0')*2+s[1]-'0');
	int l=strlen(s);
	for(int i=p+2;i<l;i+=3)//i<strlen(s)로 했을 때 TLE
		printf("%d", (s[i-2]-'0')*4+(s[i-1]-'0')*2+(s[i]-'0'));
}
```

	* 처음에 시간초과가 뜨길래 내 눈을 의심했다. 문자열도 10^6이고 시간초과가 뜰 구석이 없었기 때문이다. 계속 `vector`에서 v.size()를 쓰다가 생각없이 strlen도 동일하게 사용하고있었다. `vector`의 경우 v.end()-v.begin()을 하면 바로 크기가 나오기 때문에 size()가 O(1)이지만 strlen은 고정된 크기의 배열에서 안에 값이 들어있는 칸의 개수를 구하는 것이기 때문에 O(n)인 것 같다.(cplusplus에도 time complexity가 나오지 않아서 확실하지는 않다.)
	* `s[i]-'0'`으로 계산하지 않고 ternary operator을 이용해서 계산하는 코드도 있었다.(`(s[i]==0)?0:4`와 같은 식으로) 이거도 제출해서 비교해보니 내 코드보다 10%정도 더 빨랐다. 그리고 ternary operator가 +보다 우선순위가 낮다는 것도 알게 되었다.

</div>
</details>

- 출력도 시간이 많이 걸리는 연산이기 때문에 입출력이 많은 문제의 경우 신경써야 한다.

<details>
<summary>BOJ 1212</summary>
<div markdown="1" style="font-size:20px;font-family:'Consolas', 맑은 고딕;">
**[BOJ 1212](https://www.acmicpc.net/problem/1212){: target="_blank"}**  
	* 8진수를 2진수로 변환하는 문제이다.

```cpp
//24ms짜리 코드
#include<cstdio>
#include<cstring>
void pprint(char c){
	if(c=='0') ;
	else if(c=='1') printf("1");
	else if(c=='2') printf("10");
	else if(c=='3') printf("11");
	else if(c=='4') printf("100");
	else if(c=='5') printf("101");
	else if(c=='6') printf("110");
	else if(c=='7') printf("111");
}
void print(char c){
	if(c=='0') printf("000");
	else if(c=='1') printf("001");
	else if(c=='2') printf("010");
	else if(c=='3') printf("011");
	else if(c=='4') printf("100");
	else if(c=='5') printf("101");
	else if(c=='6') printf("110");
	else if(c=='7') printf("111");
}
int main()
{
	char s[1000001];
	scanf("%s", s);
	int l=strlen(s);
	if(s[0]=='0'){
		printf("0");
		return 0;
	}
	pprint(s[0]);
	for(int i=1;i<l;i++)
		print(s[i]);
}
```

* 출력 횟수를 조절해서 시간을 더 줄일 수 있다. `printf`로 바로 출력하지 않고 string을 이용해서 저장한 다음 한 번에 출력하면 12ms로 줄일 수 있다.
* 함수 호출때문에 시간이 더 걸리나 싶어서 전부 main으로 옮겨봤는데 걸리는 시간이 같았다. 재귀함수와 같이 함수 호출이 지수함수 형태로 증가하지 않는 이상 그건 별 상관이 없는 것 같다.
* 4ms짜리 코드도 있었는데 출력하는 수도 char배열에 저장해놓고 하나하나씩 계산하는 형태였다. 계산도 미리 2진수로 바꿔놓고 shift연산을 이용해 자릿수를 맞추는 방식이었고 edge case 처리도 그냥 출력하는 배열의 포인터를 조절해서 맞춰줬다. 저런게 커팅 장인인가 싶었다.

</div>
</details>
</div>