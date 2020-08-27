---
layout: single
title: "C++ I/O 헷갈리는 내용 정리"
categories:
  - Miscellaneous
tags:
  - C++
---
<div markdown="1" style="font-size:18px;font-family:'Consolas', 맑은 고딕;">

## 헷갈리는 I/O 함수들
1. scanf의 포맷에서 delimeter를 직접 지정할 수 있다.  
ex) csv파일의 경우 `"%d,%d"`등으로 받으면 편함

2. `<string>`의 `getline()`과 `<istream>`의 `getline`
<div markdown="1" style="font-size:20px;font-family:'Consolas', 맑은 고딕;">

| Header | Declaration | Ex. |
| :--- | :--- | :--- |
| `<string>` | `istream& getline(istream& is, string& str, char delim)` | getline(cin, s, ' ') |
| `<istream>` | `istream& getline(char* s, streamsize n, char delim)` | cin.getline(s, 100, ' ') |

</div>

- Ex.의 `cin.getline(s, 100, ' ')`에서
  * `streamsize n`은 n-1개의 문자를 입력받고 맨마지막에 `'\0'`를 추가해서 길이가 n인 문자열을 s에 저장
  * 만약 입력받는 버퍼에 저장된 문자열이 n보다 작을경우 버퍼에 저장된 것 까지만 읽고 에러 없이 종료
- `<istream>`의 `getline()`과 `<cstdio>`의 `fgets(char * str, int num, FILE * stream)`은 모두 개행문자까지 입력으로 받고 맨뒤에 `'\0'` 추가

## File I/O in C++
1. 필요에 따라서 `<ifstream>`, `<ofstream>`, `<fstream>`을 골라 쓰면됨
2. `fscanf`, `fprintf`는 `FILE*`에 대해서만 쓸 수 있기 때문에 위의 ios를 상속받은 클래스들에 대해서는 사용할 수 없음  
-> `getline`, `<<`를 이용하자
3. `fstream`클래스들의 경우 생성자로 열 수도 있지만 `ifs.open("filename")`과 같이 open함수로도 열 수 있는데, 두번째 parameter로 open mode를 설정할 수 있다. (cplusplus의 [ifstream](http://www.cplusplus.com/reference/fstream/ifstream/open/) 항목 참고)

</div>