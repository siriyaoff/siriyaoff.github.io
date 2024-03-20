---
layout: single
title: "Spring Boot와 AWS로 혼자 구현하는 웹 서비스 3 - Mustache(Ch.04)"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - Spring Boot 2
  - Template Engine
  - Mustache
---

### Template Engine

- Template Engine : 지정된 template 양식과 코드를 합쳐서 HTML 문서를 출력하는 소프트웨어
  - Server Template Engine : Spring+JSP, Freemarker, …
  - Client Template Engine : React, Vue, …

```html
<script type="text/javascript">

$(document).ready(function(){
	if(a=="1"){
		<% System.out.println("test");%>
	}
});
```

- 서버 템플릿 엔진을 사용해서 실행할 경우 : 서버에서 Java 코드를 통해 화면을 생성하고 브라우저로 전달
  - 이때 JS 코드는 단순한 문자열로 취급됨
  - 위 코드의 경우 if 문에 관계없이 test가 콘솔에 출력됨
- 클라이언트 템플릿 엔진을 사용해서 실행할 경우 : 브라우저에서 JS 코드를 통해 화면을 생성
  - 서버에서는 json이나 xml로 데이터만 전달하고, 클라이언트에서 페이지 조립
  - 최근에는 react, vue 등의 JS framework에서 Server Side Rendering을 지원하기도 함
    - JS의 화면 생성 방식을 서버에서 실행함
    - V8 기반의 Nashorn, J2V8 등의 JS 엔진을 사용해서 JVM에서 JS를 돌릴 수 있음

### Mustache란

- 대부분의 언어를 지원하는 template engine
  - Java에서는 Server template engine, JS에서는 Client template engine으로 사용될 수 있음  
    → 하나의 문법으로 client/server template을 모두 사용 가능
  - 문법이 간단함
  - 로직 코드를 사용할 수 없어서 View와 서버의 역할이 명확하게 분리됨
  - IntelliJ 무료 버전에서 지원
- 다른 template engine들
  - JSP, Velocity : spring boot에서 권장되지 않음
  - Freemarker : 높은 자유도를 위해 구현된 기능이 많아서 코드에 비즈니스 로직이 추가될 수 있음
  - Thymeleaf : spring에서 권장하지만 문법이 어려움
- Settings - Plugins에서 Handlebars/Mustache plugin 설치

### 기본 페이지 만들기

- mustache template의 기본 위치는 `src/main/resources/templates`임
- `index.mustache`를 위 위치에 생성
  ```html
  <!DOCTYTPE HTML>
  <html>
    <head>
      <title>spring boot web service</title>
      <meta http-equiv="Content-Type" content="text/html;charset=UTF-8" />
    </head>
    <body>
      <h1>spring boot web services</h1>
    </body>
  </html>
  ```
- `web` 패키지 안에 `IndexController` 생성

  ```java
  @Controller
  public class IndexController {

      @GetMapping("/")
      public String index() {
          return "index";
      }
  }
  ```

  - controller에서 문자열을 반환하면 경로와 확장자는 mustache starter에 의해 자동을 지정됨
  - 위 코드에서는 `src/main/resources/templates/index.mustache`로 변환되어 View Resolver가 처리함

- `IndexController`의 테스트 생성

  ```java
  @ExtendWith(SpringExtension.class)
  @SpringBootTest(webEnvironment = RANDOM_PORT)
  class IndexControllerTest {

      @Autowired
      private TestRestTemplate restTemplate;

      @Test
      public void Main_Loading() {
          //when
          String body = this.restTemplate.getForObject("/", String.class);

          //then
          assertThat(body).contains("spring boot web services");
      }
  }
  ```

### 게시글 등록 화면 만들기

- Frontend library를 사용하는 방법
  - 외부 CDN 사용
    - CDN을 서비스하는 곳에 문제가 생기면 앱에도 문제가 생기기 때문에 잘 사용하지 않음
  - 라이브러리를 직접 받아서 사용(npm/yarn + gulp/webpack도 여기 속함)
- 이번 예제에서는 외부 CDN으로 Bootstrap, jQuery를 끌어와서 레이아웃 방식으로 추가함
  - 레이아웃 방식 : 공통 영역을 별도의 파일로 분리하여 만들고 필요할 때 가져다 쓰는 방식
- `templates`에 `layout` 디렉토리 추가하고 안에 `header.mustache`, `footer.mustache` 생성

  ```html
  <!DOCTYPE html>
  <html>
    <head>
      <title>spring boot web service</title>
      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

      <link
        rel="stylesheet"
        href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
      />
    </head>
    <body></body>
  </html>
  ```

  ```html
  <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
  <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

  </body>
  </html>
  ```

  - css는 header, js는 footer에 배치해서 페이지 로딩속도를 높일 수 있음
    - js의 용량이 클 경우 head에 두면 로딩이 완료될 때까지 화면에 아무것도 보이지 않음
  - bootstrap.js는 jQuery에 의존하기 때문에 jquery 뒤에 로딩함

- 레이아웃을 사용해서 `index.mustache` 수정, 글 등록 버튼 추가
  ```
  {% raw %}{{>layout/header}}{% endraw %}
  <h1>spring boot web services</h1>
  {% raw %}{{>layout/footer}}{% endraw %}
  ```
- `IndexController`에 `/posts/save`에 해당하는 매핑 추가
  ```java
  @GetMapping("/posts/save")
  public String postsSave() {
      return "posts-save";
  }
  ```
- `templates`에 `posts-save.mustache` 생성

  ```html
  {% raw %}{{>layout/header}}{% endraw %}

  <h1>Upload post</h1>

  <div class="col-md-12">
    <div class="col-md-4">
      <form>
        <div class="form-group">
          <label for="title">Title</label>
          <input
            type="text"
            class="form-control"
            id="title"
            placeholder="Enter title"
          />
        </div>
        <div class="form-group">
          <label for="author"> Author </label>
          <input
            type="text"
            class="form-control"
            id="author"
            placeholder="Enter author"
          />
        </div>
        <div class="form-group">
          <label for="content"> Content </label>
          <textarea
            class="form-control"
            id="content"
            placeholder="Enter content"
          ></textarea>
        </div>
      </form>
      <a href="/" role="button" class="btn btn-secondary">Cancel</a>
      <button type="button" class="btn btn-primary" id="btn-save">
        Upload
      </button>
    </div>
  </div>

  {% raw %}{{>layout/footer}}{% endraw %}
  ```

- `resources`에 `static/js/app`디렉토리 추가하고 안에 `index.js` 생성

  ```jsx
  var main = {
    init: function () {
      var _this = this;
      $("#btn-save").on("click", function () {
        _this.save();
      });
    },
    save: function () {
      var data = {
        title: $("#title").val(),
        author: $("#author").val(),
        content: $("#content").val(),
      };

      $.ajax({
        type: "POST",
        url: "/api/v1/posts",
        dataType: "json",
        contentType: "application/json; charset=utf-8",
        data: JSON.stringify(data),
      })
        .done(function () {
          alert("Post uploaded.");
          window.location.href = "/";
        })
        .fail(function (error) {
          alert(JSON.stringify(error));
        });
    },
  };

  main.init();
  ```

  - `window.location.href='/'`를 통해 루트로 이동
  - `main` 객체를 만들어서 그 안에 함수를 넣는 이유 : 브라우저의 scope는 공용 공간이기 때문에 중복된 함수가 들어올 경우 나중에 로드된 함수가 먼저 로드된 동일한 이름의 함수를 덮어씀, 중복을 피하기 위해 `main` 객체를 만들어서 `index.js`만의 scope를 정의함
    - 최신 JS와 framework에서는 scope 관리가 지원됨

- `footer.mustache`에 `index.js` 추가
  ```html
  <!--index.js 추가-->
  <script src="/js/app/index.js"></script>
  ```
  - static resource들의 경로에서 `src/main/resources/static` 생략 가능

### 전체 조회 화면 만들기

- `index.mustache` 수정

  ```html
  {% raw %}{{>layout/header}}{% endraw %}

  <h1>spring boot web services Ver.2</h1>
  <div class="col-md-12">
    <div class="row">
      <div class="col-md-6">
        <a href="/posts/save" role="button" class="btn btn-primary"
          >Upload post</a
        >
      </div>
    </div>
    <br />
    <!--목록 출력 영역-->
    <table class="table table-horizontal table-bordered">
      <thead class="thead-strong">
        <tr>
          <th>No.</th>
          <th>Title</th>
          <th>Author</th>
          <th>Last Modified</th>
        </tr>
      </thead>
      <tbody id="tbody">
        {% raw %}{{#posts}}{% endraw %}
        <tr>
          <td>{% raw %}{{id}}{% endraw %}</td>
          <td>{% raw %}{{title}}{% endraw %}</td>
          <td>{% raw %}{{author}}{% endraw %}</td>
          <td>{% raw %}{{modifiedDate}}{% endraw %}</td>
        </tr>
        {% raw %}{{/posts}}{% endraw %}
      </tbody>
    </table>
  </div>
  {% raw %}{{>layout/footer}}{% endraw %}
  ```

  - `{% raw %}{{#posts}}{% endraw %}` : `posts`라는 List를 순회함(for문과 동일)
    - 내부에서 `{% raw %}{{id}}{% endraw %}` 등으로 필드를 뽑아낼 수 있음

- 전체 조회를 위한 Controller, Service, Repository 코드 추가
- `PostsRepository` 인터페이스에 쿼리 추가
  ```java
  public interface PostsRepository extends JpaRepository<Posts, Long> {
      @Query("SELECT p FROM Posts p ORDER BY p.id DESC")
      List<Posts> findAllDesc();
  }
  ```
  - SpringDataJpa에서 제공하지 않는 메소드는 쿼리로 작성해도 됨  
    (위 메소드는 SpringDataJpa의 기본 메소드만으로 구현이 가능하지만, `@Query`를 사용하는게 가독성이 좋음)
  - 규모가 큰 프로젝트에서의 데이터 조회는 Entity 클래스 만으로 처리하기 복잡하기 때문에 조회용 프레임워크를 따로 씀
    - querydsl(레퍼런스가 가장 많음), jooq, MyBatis 등이 있음
- `PostsService` 클래스에 `findAllDesc()` 메소드 추가
  ```java
  @Transactional(readOnly = true)
  public List<PostsListResponseDto> findAllDesc() {
      return postsRepository.findAllDesc().stream()
              .map(PostsListResponseDto::new)
              .collect(Collectors.toList());
  }
  ```
  - `@Transactional`에 `readOnly = true` 옵션을 추가하면 transaction의 범위는 유지하지만 조회 기능만 남겨서 조회 속도가 개선됨
  - 람다식 `.map(PostsListResponseDto::new)`는 `.map(posts -> new PostsListResponseDto(posts))`와 같음
    - `~.stream()`에서 반환된 `Stream`을 `PostsListResponseDto`로 변환
  - `.collect(Collectors.toList())`를 통해 `List`로 다시 변환함
- `web.dto` 패키지에 `PostsListResponseDto` 클래스 생성

  ```java
  @Getter
  public class PostsListResponseDto {
      private Long id;
      private String title;
      private String author;
      private LocalDateTime modifiedDate;

      public PostsListResponseDto(Posts entity) {
          this.id = entity.getId();
          this.title = entity.getTitle();
          this.author = entity.getAuthor();
          this.modifiedDate = entity.getModifiedDate();
      }
  }
  ```

- `IndexController` 변경

  ```java
  @RequiredArgsConstructor
  @Controller
  public class IndexController {

      private final PostsService postsService;

      @GetMapping("/")
      public String index(Model model) {
          model.addAttribute("posts", postsService.findAllDesc());
          return "index";
      }

      @GetMapping("/posts/save")
      public String postsSave() {
          return "posts-save";
      }
  }
  ```

  - `Model` : 서버 템플릿 엔진에서 사용할 객체를 저장할 수 있음
    - 위 코드에서는 `postsService.findAllDesc()`의 결과를 `"posts"`라는 이름으로 추가해서 `index.mustache`에 넘김

### 게시글 수정, 삭제 화면 만들기

- 수정 API는 `PostsApiController`에 `public Long update()`로 이미 구현해놓음
- `templates`에 `posts-update.mustache` 생성

  ```html
  {% raw %}{{>layout/header}}{% endraw %}

  <h1>Update post</h1>

  <div class="col-md-12">
    <div class="col-md-4">
      <form>
        <div class="form-group">
          <label for="title">No.</label>
          <input
            type="text"
            class="form-control"
            id="id"
            value="{% raw %}{{post.id}}{% endraw %}"
            readonly
          />
        </div>
        <div class="form-group">
          <label for="title">Title</label>
          <input
            type="text"
            class="form-control"
            id="title"
            value="{% raw %}{{post.title}}{% endraw %}"
          />
        </div>
        <div class="form-group">
          <label for="author"> Author </label>
          <input
            type="text"
            class="form-control"
            id="author"
            value="{% raw %}{{post.author}}{% endraw %}"
            readonly
          />
        </div>
        <div class="form-group">
          <label for="content"> Content </label>
          <textarea class="form-control" id="content">
  {% raw %}{{post.content}}{% endraw %}</textarea
          >
        </div>
      </form>
      <a href="/" role="button" class="btn btn-secondary">Cancel</a>
      <button type="button" class="btn btn-primary" id="btn-update">
        Update
      </button>
    </div>
  </div>

  {% raw %}{{>layout/footer}}{% endraw %}
  ```

  - scope를 지정해주지 않고 `{% raw %}{{posts.id}}{% endraw %}`와 같이 객체의 필드에 접근할 수 있음
  - `readonly` : readonly 속성 추가

- `static/js/app/index.js`의 `init` 함수 수정, `update` 함수 추가

  ```jsx
  init : function () {
      var _this = this;
      $('#btn-save').on('click', function () {
          _this.save();
      });

      $('#btn-update').on('click', function () {
          _this.update();
      });
  },
  ...
  update : function () {
      var data = {
          title: $('#title').val(),
          content: $('#content').val()
      };

      var id = $('#id').val();

      $.ajax({
          type: 'PUT',
          url: '/api/v1/posts/'+id,
          dataType: 'json',
          contentType:'application/json; charset=utf-8',
          data: JSON.stringify(data)
      }).done(function() {
          alert('Post updated.');
          window.location.href = '/';
      }).fail(function (error) {
          alert(JSON.stringify(error));
      });
  }
  ```

  - `$('#btn-update').on('click', func(){})` : 이벤트 등록
  - `PostsApiController`에서 REST 규약에 맞춰서 update를 `@PutMapping`으로 선언했기 때문에 맞춰서 `type: 'PUT'`으로 설정

> REST API 예시  
> Create : POST  
> Read : GET  
> Update : PUT  
> Delete : DELETE  
> 나중에 자세히 공부하기

- `index.mustache`에 수정 페이지로의 이동 기능 추가(`<tbody>`의 `{% raw %}{{title}}{% endraw %}`을 아래와 같이 변경)
  ```html
  <td>
    <a href="/posts/update/{% raw %}{{id}}{% endraw %}"
      >{% raw %}{{title}}{% endraw %}</a
    >
  </td>
  ```
- `IndexController`에 게시글 주소에 대한 매핑 추가

  ```java
  @GetMapping("/posts/update/{id}")
  public String postsUpdate(@PathVariable Long id, Model model) {
      PostsResponseDto dto = postsService.findById();
      model.addAttribute("post", dto);

      return "posts-update";
  }
  ```

### 게시글 삭제

- `posts-update.mustache`에 삭제 버튼 추가
  ```html
  ...
  <button type="button" class="btn btn-primary" id="btn-update">Update</button>
  <button type="button" class="btn btn-danger" id="btn-delete">Delete</button>
  ...
  ```
- `index.js`에 삭제 이벤트 등록

  ```jsx
  init : function () {
      var _this = this;
      $('#btn-save').on('click', function () {
          _this.save();
      });
      $('#btn-update').on('click', function () {
          _this.update();
      });
      $('#btn-delete').on('click', function () {
          _this.delete();
      });
  },
  ...
  delete : function () {
      var id = $('#id').val();

      $.ajax({
          type: 'DELETE',
          url: '/api/v1/posts/'+id,
          dataType: 'json',
          contentType:'application/json; charset=utf-8'
      }).done(function() {
          alert('Post deleted.');
          window.location.href = '/';
      }).fail(function (error) {
          alert(JSON.stringify(error));
      });
  }
  ```

- `PostsService`에 삭제 API 추가

  ```java
  @Transactional
  public void delete(Long id) {
      Posts posts = postsRepository.findById(id)
              .orElseThrow(() -> new IllegalArgumentException("No such post. id=" + id));

      postsRepository.delete(posts);
  }
  ```

  - 존재하는 Posts인지 확인 후 삭제
  - `JpaRepository`에서 `delete` 메소드를 지원함
  - `deleteById` 메소드를 사용하면 `id`를 사용해서 삭제 가능

- `IndexController`에 `delete` 메소드를 사용하는 매핑 추가
  ```java
  @DeleteMapping("/api/v1/posts/{id}")
  public Long delete(@PathVariable Long id) {
      postsService.delete(id);
      return id;
  }
  ```
