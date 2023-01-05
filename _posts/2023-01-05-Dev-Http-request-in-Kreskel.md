---
layout: single
title: "Kreskel 서버 간 Http 요청"
categories:
  - Dev
tags:
  - Blazor
  - Razor
  - C#
---

## MVC template을 사용한 서버 생성

1. HelloMvc
   - 이전에 사용한 HelloMvc를 그대로 사용함
2. ReqMvc
   - HttpClient를 사용해서 요청을 보낼 서버

### HelloMvc 설정

- `Controllers/HelloWorldController.cs`에 간단한 get 라우팅 추가
  ```csharp
  public string Hi()
  {
      return "Hi from HelloMvc";
  }
  ```

### ReqMvc 설정

- mvc template으로 생성
  - `--no-https` 옵션 추가하지 않음
- `Program.cs`에 아래 내용 추가
  ```csharp
  ...
  builder.Services.AddControllersWithViews();
  builder.Services.AddHttpClient();
  builder.Services.AddCors(options =>
  {
      options.AddPolicy("mypolicy",
      policy =>
      {
          policy.WithOrigins("https://*")
          .AllowAnyHeader();
      });
  });
  ...
  ```
  - CORS 처리
- `Controllers/SayController.cs` 생성

  ```csharp
  using System.Diagnostics;
  using Microsoft.AspNetCore.Mvc;
  using ReqMvc.Models;
  using Microsoft.Net.Http.Headers;

  namespace ReqMvc.Controllers;

  public class SayController : Controller
  {
      private readonly IHttpClientFactory _httpClientFactory;

      public SayController(IHttpClientFactory httpClientFactory) => _httpClientFactory=httpClientFactory;

      public string sayHi()
      {
          var requestMessage=new HttpRequestMessage(
              HttpMethod.Get,
              "https://localhost:7006/HelloWorld/Hi")
          {
              Headers=
              {
                  {HeaderNames.Accept, "text/plain"},
                  {HeaderNames.UserAgent, "HttpRequestsSample"}
              }
          };

          var httpClient=_httpClientFactory.CreateClient();
          var res=httpClient.Send(requestMessage);

          byte[] msg=new byte[100];
          //return res.ToString();
          res.Content.ReadAsStream().Read(msg);
          string strmsg=System.Text.Encoding.UTF8.GetString(msg).TrimEnd('\0');
          Console.WriteLine(strmsg);
          return strmsg;
      }

      public string ret(){
          string str="just string";
          return str;
      }
  }
  ```

  ![kreskel-http-req1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/kreskel-http-req1.png)

  - 뒤에 `'\0'`을 제거하지 않으면 파일으로 다운로드됨

  ![kreskel-http-req2](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/kreskel-http-req2.png)

## PNG GET

- `System.Drawing.Common`은 non-Windows 플랫폼에서 지원되지 않음
