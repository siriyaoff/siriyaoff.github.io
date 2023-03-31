---
layout: single
title: "ASP.NET 환경 구축"
categories:
  - Dev
tags:
  - ASP.NET
  - C#
---

## 설치 및 테스트

- M1 Pro, Ventura 13.1 기준

[.NET 6.0 다운로드(Linux, macOS 및 Windows)](https://dotnet.microsoft.com/ko-kr/download/dotnet/6.0)

- Arm64 설치관리자
- `dotnet --info`로 설치 확인

> 홈페이지에서 수동으로 설치하는게 homebrew로 설치하는 것보다 안정적임

## dotnet webapp 생성

```bash
dotnet new -l
dotnet new webapp -o HelloApp --no-https -f net6.0
cd HelloApp
dotnet watch
```

- `dotnet new -l`으로 사용 가능한 템플릿 확인
- `dotnet watch`하면 build, run이 포함되는 듯

## dotnet webapi 생성

```bash
dotnet new webapi -o HelloApi --no-https
```

### API 변경

- `WeatherForecastController.cs` 지우고 `HelloApiController.cs` 생성 후 아래 코드 입력

```bash
using Microsoft.AspNetCore.Mvc;

namespace HelloApi.Controllers;

[ApiController]
[Route("[controller]")]
public class HelloController : ControllerBase
{
    [HttpGet(Name="hello")]
    public string Get()
    {
        return "hello";
    }
}
```

- `dotnet watch`로 실행하면 swagger라는 템플릿이 자동으로 실행되는데, 이를 통해서 API 상태 확인 가능
- `/hello` 들어가서 `"hello"` 반환 확인

![aspdnet conf](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/aspdnet-conf.png)

## 설치된 버전 제거

[.NET 런타임 및 SDK 제거 - .NET](https://learn.microsoft.com/ko-kr/dotnet/core/install/remove-runtime-sdk-versions?pivots=os-macos)

- 3.0 이상일 경우 NuGet 대체 폴더 고려할 필요 없음
- M1 칩일 경우 경로 잘 보고 지우기
