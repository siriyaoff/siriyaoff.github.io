---
layout: single
title: "ASP.NET MVC Tutorial"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - ASP.NET
  - C#
  - MVC
  - ER Core
  - Scaffolding
  - Migration
---

## MVC pattern

- Models : 유효성 검사, 비즈니스 로직 적용, 뷰에 결과 제공
- Views : 모델 데이터 표시, UI 구성 요소
- Controllers : 브라우저 요청 처리, 모델 데이터 검색(모델에 전달), 뷰 템플릿 호출

## MVC 앱 생성

```csharp
dotnet new mvc -o HelloMvc
```

- `HelloWorldController.cs` 추가 후 아래 코드 입력

  ```csharp
  using Microsoft.AspNetCore.Mvc;
  using System.Text.Encodings.Web;

  namespace MvcMovie.Controllers;

  public class HelloWorldController : Controller
  {
      //
      // GET: /HelloWorld/
      public string Index()
      {
          return "This is my default action...";
      }
      //
      // GET: /HelloWorld/Welcome/
      public string Welcome()
      {
          return "This is the Welcome action method...";
      }
  }
  ```

  - `localhost:port/HelloWorld`로 접속하면 Index()가 실행됨
  - https는 아래 명령어 실행해서 인증서 신뢰하도록 설정해야 함
    ```bash
    dotnet dev-certs https --trust
    ```

## 라우팅 포맷

- `/[Controller]/[ActionName]/[Parameters]` 서식을 따름
- `Program.cs` 파일에서 설정 가능

  ```csharp
  app.MapControllerRoute(
      name: "default",
      pattern: "{controller=Home}/{action=Index}/{id?}");
  ```

- `Welcome()`을 아래와 같이 변경:

  ```csharp
  // GET: /HelloWorld/Welcome/
  // Requires using System.Text.Encodings.Web;
  public string Welcome(string name, int numTimes = 1)
  {
      return HtmlEncoder.Default.Encode($"Hello {name}, NumTimes is: {numTimes}");
  }
  ```

  - `localhost:port/HelloWorld/Welcome?name=yang&numtimes=5`와 같이 파라미터를 입력 후 실행해서 결과 나오는지 확인

- `Welcome()`을 Path Variable을 사용하도록 변경:
  ```csharp
  public string Welcome(string name, int ID = 1)
  {
      return HtmlEncoder.Default.Encode($"Hello {name}, ID: {ID}");
  }
  ```
  - routing format에서 path variable은 대소문자 신경쓰지 않는 것 같음
  - `{id?}` : `?`는 `id`가 optional임을 나타냄

## View 추가

- `HelloWorldController`의 `Index()`도 view method를 호출하도록 변경

  ```csharp
  public IActionResult Index()
  {
      return View();
  }
  ```

- `Views/HelloWorld/Index.cshtml` 추가

  ```csharp
  @{
      ViewData["Title"] = "Indextitle";
  }

  <h2>Index</h2>

  <p>Hello from our View Template!</p>
  ```

- `Views/Shared/_Layout.cshtml` 변경

  ```csharp
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>@ViewData["Title"] - Movie App</title>
  ...
  <div class="container-fluid">
      <a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>
  ...
  <div class="container">
      &copy; 2023 - Movie App - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
  </div>
  ```

  - 현재는 `Movies` 컨트롤러가 없기 때문에 Moive App 링크가 작동하지 않음
  - title 끝부분이 Movie App으로 바뀐 것 확인
    - 레이아웃을 변경했기 때문에 모든 페이지가 동일하게 변경됨
      - `Views/_ViewStart.cshtml`에서 `_Layout.cshtml`을 불러오기 때문

- `Views/HelloWorld/Index.cshtml`을 아래처럼 변경

  ```csharp
  @{
      ViewData["Title"] = "Movie List";
  }

  <h2>My Movie List</h2>
  ...
  ```

  ![aspdnet mvc 1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/asdpnet-mvc1.png)

  - layout template을 사용해서 `Index.cshtml`과 `Views/Shared/_Layout.cshtml`을 합침

## Controller에서 View로 데이터 전달

- Controller는 View에서 렌더링할 데이터를 제공해야 함
- `HelloWorldController`에서 `ViewData`에 데이터를 추가하고, view에서 가져다 쓰도록 구현할 예정
- `HelloWorldController.cs`을 아래처럼 변경
  ```csharp
  public IActionResult Welcome(string name, int numTimes=1)
  {
      ViewData["Message"]="Hello "+name;
      ViewData["NumTimes"]=numTimes;
      return View();
  }
  ```
- `Views/HelloWorld/Welcome.cshtml` 생성

  ```csharp
  @{
      ViewData["Title"] = "Welcome";
  }

  <h2>Welcome</h2>

  <ul>
      @for (int i = 0; i < (int)ViewData["NumTimes"]!; i++)
      {
          <li>@ViewData["Message"]</li>
      }
  </ul>
  ```

  ![aspdnet mvc 2](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/asdpnet-mvc2.png)

  - `ViewData`를 사용해서 컨트롤러에서 뷰로 데이터를 전달함

## 모델 추가

- 보통 모델은 EF Core와 함께 DB 접근에 사용됨
  - EF Core(Entity Framework Core) : JPA Entity 같은 ORM framework
- 모델 클래스 자체는 POCO(Plain Old CLR Objects)로 EF Core 종속성이 없음
  - DB에 저장되는 데이터의 속성만 정의함
- 예제에서는 모델 클래스, EF Core, DB 순으로 만들 예정

## 데이터 모델 클래스 추가

- `Models/Movie.cs` 생성

  ```csharp
  using System.ComponentModel.DataAnnotations;

  namespace MvcMovie.Models;

  public class Movie
  {
      public int Id { get; set; }
      public string? Title { get; set; }
      [DataType(DataType.Date)]
      public DateTime ReleaseDate { get; set; }
      public string? Genre { get; set; }
      public decimal Price { get; set; }
  }
  ```

  - `string?` : `nullable`을 의미함
  - `DataType.Date` : 시간을 제외한 날짜

- NuGet 패키지 추가
  ```csharp
  dotnet tool uninstall --global dotnet-aspnet-codegenerator
  dotnet tool install --global dotnet-aspnet-codegenerator
  dotnet tool uninstall --global dotnet-ef
  dotnet tool install --global dotnet-ef
  dotnet add package Microsoft.EntityFrameworkCore.Design
  dotnet add package Microsoft.EntityFrameworkCore.SQLite
  dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
  dotnet add package Microsoft.EntityFrameworkCore.SqlServer
  ```
  - .NET 6.0일 경우
    - `dotnet-aspnet-codegenerator`, `Microsoft.VisualStudio.Web.CodeGeneration.Design`는 버전 맞춰줘야 함
      ```bash
      dotnet tool install --global dotnet-aspnet-codegenerator --version 6.0.0
      dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 6.0.10
      ```

## 영화 페이지 스캐폴드

```bash
export PATH=$HOME/.dotnet/tools:$PATH
dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovie.Data.MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries -sqlite
```

- 스캐폴딩 실행 후에는 아래와 같은 기능들이 추가됨
  - `Controllers/MoviesController.cs`
  - `Vies/Movies/*.cshtml` : CRUD, Index 페이지에 대한 Razor 뷰 파일
  - `Data/MvcMovieContext.cs` : DB context 클래스

## 초기 마이그레이션

- migration : Data model과 일치하는 DB를 만들고 수정할 수 있는 도구 모음

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

- 앱 실행해서 Movie App 클릭 후 정상 작동 확인
  ![aspdnet mvc 3](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/asdpnet-mvc3.png)

## DB 초기화

- `SeedData/SeedData.cs` 생성

  ```csharp
  using Microsoft.EntityFrameworkCore;
  using Microsoft.Extensions.DependencyInjection;
  using MvcMovie.Data;
  using System;
  using System.Linq;

  namespace MvcMovie.Models;

  public static class SeedData
  {
      public static void Initialize(IServiceProvider serviceProvider)
      {
          using (var context = new MvcMovieContext(
              serviceProvider.GetRequiredService<
                  DbContextOptions<MvcMovieContext>>()))
          {
              // Look for any movies.
              if (context.Movie.Any())
              {
                  return;   // DB has been seeded
              }
              context.Movie.AddRange(
                  new Movie
                  {
                      Title = "When Harry Met Sally",
                      ReleaseDate = DateTime.Parse("1989-2-12"),
                      Genre = "Romantic Comedy",
                      Price = 7.99M
                  },
                  new Movie
                  {
                      Title = "Ghostbusters ",
                      ReleaseDate = DateTime.Parse("1984-3-13"),
                      Genre = "Comedy",
                      Price = 8.99M
                  },
                  new Movie
                  {
                      Title = "Ghostbusters 2",
                      ReleaseDate = DateTime.Parse("1986-2-23"),
                      Genre = "Comedy",
                      Price = 9.99M
                  },
                  new Movie
                  {
                      Title = "Rio Bravo",
                      ReleaseDate = DateTime.Parse("1959-4-15"),
                      Genre = "Western",
                      Price = 3.99M
                  }
              );
              context.SaveChanges();
          }
      }
  }
  ```

- `Program.cs` 수정

  ```csharp
  ...
  using MvcMovie.Models;

  ...

  using (var scope = app.Services.CreateScope())
  {
      var services = scope.ServiceProvider;

      SeedData.Initialize(services);
  }
  ```

  - 다시 실행해서 시드 데이터 들어간 것 확인

- `Models/Movie.cs` 수정

```csharp
...
using System.ComponentModel.DataAnnotations.Schema;

...

		[Display(Name ="Release Data")]
		[DataType(DataType.Date)]
		public DateTime ReleaseDate { get; set; }
		public string? Genre { get; set; }
		[Column(TypeName ="decimal(18, 2)")]
		public decimal Price { get; set; }
}
```
