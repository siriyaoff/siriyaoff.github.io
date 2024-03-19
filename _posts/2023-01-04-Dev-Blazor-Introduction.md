---
layout: single
title: "Blazor 소개"
categories:
  - Dev
tags:
  - Blazor
  - Razor
  - C#
---

## Blazor

- JS 대신 C#을 사용해서 interactive web UI를 구축할 수 있는 SPA framework

![blazor architecture](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/blazor-archi.png)

- WebAssembly : bytecode instruction을 실행함
  - DOM을 직접적으로 건드리진 못함 → JS를 사용해서 접근

## Razor

- Blazor가 사용하는 template engine임
  (blazor : browser + razor)
- view를 Razor markup, HTML, C#으로 구성함
  - razor 구성 요소는 .razor으로 생성하고 재사용 가능
  - 일반적인 view 구성 요소, 페이지들은 .cshtml으로 정의됨

```csharp
<ul>
    @foreach(int num in Fibonacci)
    {
        <li>@num * 2 = @(num*2)</li>
    }
</ul>
@code {
    public int[] Fibonacci = new int[] { 1, 1, 2, 3, 5, 8 };
}
```

## ServerSideBlazor, ClientSideBlazor

- ServerSideBlazor
  - 코드가 client의 WebAssembly가 아닌 서버에서 완전히 실행됨
  - 렌더링을 거친 결과만 클라이언트로 전송됨
- ClientSideBlazor : client가 preview를 종료할 때 serverside를 대체하는 데 사용됨

## Wrapped JavaScript Control

- Blazor control : blazor로 작성된 구성 요소
- Wrapped JS control : JS를 기반으로 하는 구성 요소
  - JS와의 호환성을 위해서 지원됨

### 주의점

- Blazor에서도 rendering tree를 생성하는데, 클라이언트, 서버 양쪽 모두 UI 변화를 이 트리에 참조된 구성 요소를 변경하는 방식으로 처리함
- Wrapped JS control은 참조하는 rendering tree가 없기 때문에 DOM을 거쳐야할 수도 있음
- 유효성 검사에 바인딩되지 않기 때문에 사용자 지정 검사 또는 브릿지를 설계해야 함

## 튜토리얼

[Blazor와 WebAssembly 소개 > 시티즌 인사이트](https://dev.grapecity.co.kr/bbs/board.php?bo_table=Insight&wr_id=16&page=2)
