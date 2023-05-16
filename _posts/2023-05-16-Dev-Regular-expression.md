---
layout: single
title: "Regex 정리"
categories:
  - Dev
tags:
  - Regex
  - Flag
  - Quantifier
  - Greedy quantifier
  - Lazy quantifier
  - Group
  - Capturing
---

# 플래그

- `g` : 모든 결과 리턴
  - 없을 경우 매치하는 첫 결과만 리턴
- `i` : case-insensitive
- `m` : multi line
  - 줄이 바뀌어도 계속 검색
  - `^`, `$`가 각 줄마다 적용됨
- `s` : `.`이 `\n`도 포함하도록 설정
- `u` : 유니코드 전체 지원
- `y` : sticky 모드 활성화
  - 문자 내 특정 위치에서 검색

# 한 문자 매칭

- `-` : 문자 범위 지정
  - `a-zA-Z`(알파벳 하나), `ㄱ-ㅎ가-힣`(한글 하나), `0-9`(숫자 하나)
- `.` : 줄바꿈 제외 한 문자
  - `....` : 길이가 4인 문자열
- `\d` : 숫자
  - `\D` : 숫자가 아닌 문자
- `\w` : 언더바 포함 알파벳, 숫자
  - `a-zA-Z0-9_`와 동일
  - `\W` : not `\w`
- `\s` : space
  - `\S` : not `\s`
- `\*, \^, \&, \!, \?, ...`
- `\b` : 문자(알파벳, 숫자, `_`)와 비문자 사이의 경계
  - `\B` : 문자와 문자, 비문자와 비문자 사이의 경계
- `\x` : 16진수 문자
  - `\0` : 8진수 문자
  - `\u` : 유니코드 문자
- `\c` : 제어 문자
- `\f` : FF(U+000C)
  - `\n` : LF(U+000A)
  - `\r` : CR(U+000D)
  - `\t` : 탭(U+0009)

# 검색 패턴

- `|` : OR(`a|b`)
  - `/(Mon|Tues|Wednes)day/g`
- `[]` : 괄호 안의 문자들 중 하나
  - `/abc/` : `abc`를 포함
  - `/[abc]/` : `a` or `b` or `c`를 포함
- `[^c]` : 괄호 안의 문자를 제외
  - `[^lgEn]` : `l, g, E, n`를 제외
- `^str` : 시작 문자열
  - `/^www/` : `www`로 시작
- `str$` : 종료 문자열

# 반복 패턴

- `?` : 0≤n≤1
  - 다른 quantifier 뒤에 붙으면 최솟값으로 고정함
- `*` : 0≤n
- `+` : 1≤n
- `{k}` : n=k
  - `*?` : `{0}`과 동일
  - `+?` : `{1}`과 동일
- `{k,}` : k≤n
- `{m, M}` : m≤n≤M

# 그룹 패턴

- `()` : 그룹화 + 캡처
- `(?:패턴)` : 그룹화
- `{pattern}(?=)` : 앞쪽 일치(Lookahead)
  - `/\w{3}(?=g)/g` : 길이가 3인 `\w` 문자열을 선택하는데 `g` 앞에 위치한 문자열들만 매치
  - `(?=g)asdf`과 같이 수식해주는 내용이 없으면 안됨
- `(?!)` : 부정 앞쪽 일치(Negative Lookahead)
- `(?<=)` : 뒤쪽 일치(Lookbehind)
  - Lookahead와 반대로 뒤쪽을 수식하기 때문에 뒤에 패턴이 없으면 안됨
- `(?<!)` : 부정 뒤쪽 일치(Negative Lookbehind)

## 정규식 그룹화와 캡처

```jsx
"abcabcabc".match(/abc+/); // 'abc'
"abbbbbcabcccccabccc".match(/abc+/); // 'abccccc'
"abcabcabc".match(/(abc)+/); // 'abcabcabc', 'abc'
"abbbbbcabcccccabccc".match(/(abc)+/); // 'abc', 'abc'
"abcabcabcccabcabc".match(/(abc)+/); // 'abcabcabc', 'abc'
"iamasdfasitis".match(/iam(.*)asitis/); // 'iamasdfasitis', 'asdf'
"abcabcabc".match(/(?:abc)+/); // 'abcabcabc'
```

- `()`로 그룹화하여 문자 하나 대신 사용 가능
- 패턴에서 그룹화된 표현식을 캡처해놓은 후, 전체 패턴으로 한 번 검색하고 캡처된 그룹 표현식들을 사용해서 다시 검색함
- `(?:)`으로 캡처 기능 비활성화 가능
  - 플래그 `g`를 붙여도 캡처 기능이 비활성화됨

## Greedy/Lazy quantifier

- Greedy quantifier : 패턴을 만족하는 가장 큰 부분을 찾음(`*`, `+`, `{n,}`)
- Lazy quantifier : 패턴을 만족하는 가장 작은 부분을 찾음(`*?`, `+?`, `{n,}?`)

```jsx
str = "<div> <p>First</p> <p>Second</p></div>";
str.match(/<p>(.+)<\/p>/);
// '<p>First</p> <p>Second</p>', 'First</p> <p>Second'
str.match(/<p>(.+)<\/p>/g);
// '<p>First</p> <p>Second</p>'
str.match(/<p>(.+?)<\/p>/);
// '<p>First</p>', 'First'
str.match(/<p>(.+?)<\/p>/g);
// '<p>First</p>', '<p>Second</p>'
```

- 위의 예시처럼 고정된 개수의 괄호는 매치할 수 있지만, 임의의 괄호 매치는 불가능함
  - 정규식이 regular language이기 때문
  - 괄호 매치와 같은 문제를 풀기 위해선 context-free language이어야 함
