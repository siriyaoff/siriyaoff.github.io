---
layout: single
title: "Spring Boot와 AWS로 혼자 구현하는 웹 서비스 1 - 초기 설정(Ch.01 - 02)"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - Spring Boot 2
  - TDD
  - JUnit
  - Unit test
  - Lombok
  - Dao
  - Dto
  - Annotation
---

## spring initializr

- project : gradle project
- group : com.sa
- artifact : springaws
- Java : 11
- dependencies : Spring Web, Mustache

## 깃헙 연동

- `.gradle`, `.idea`는 커밋하지 않음

## 테스트코드

### TDD와 Unit Test

- TDD(Test-driven Development) : 레드 그린 사이클을 개발 주기로 사용
  - Red : 항상 실패하는 테스트 작성
  - Green : 테스트가 통과하는 production 코드 작성
  - Refactor : 테스트가 통과하면 production 코드를 리팩토링함
- Unit test : 기능 단위의 테스트
  - 기능에 대한 불확실성 감소
  - 단위 테스트를 위키로 사용 가능

> JUnit4는 단일 jar이지만, JUnit5는 Junit Platform, Jupiter, Vintage, 3가지 모듈로 구성됨  
> Platform : JVM에서 동작하는 테스트 프레임워크(TestEngine의 인터페이스 정의)  
> Jupiter : TestEngine의 구현체  
> Vintage : JUnit 3, 4 기반 테스트를 실행하기 위한 기능 제공

### Hello Controller 테스트코드 작성

- 일반적으로 패키지명은 웹사이트 주소의 역순으로 함
- `Application` 클래스 생성
  ```java
  @SpringBootApplication
  public class Application {
      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
      }
  }
  ```
  - spring initializr를 사용하면 자동 생성해줌(패키지명 : group.artifact)
  - `@SpringBootApplication` : 스프링빈 읽기, 생성 등의 스프링 관련 설정이 이루어짐
    - `@SpringBootApplication`이 있는 위치부터 설정을 읽기 때문에 이 클래스는 항상 프로젝트의 최상단에 위치해야 함
  - `SpringApplication.run` : 내장 웹서버를 실행
    - 내장 웹서버를 이용 : 서버에 톰캣을 하지 않고 spring boot로 만들어진 Jar 파일로 실행
      - 언제 어디서나 같은 환경에서 스프링 부트를 배포할 수 있게 됨
      - 외장 WAS를 쓸 경우 모든 서버가 같은 WAS 환경을 구축해야 함
      - 톰캣도 Servlet으로 이루어진 자바 앱이기 때문에 내장/외장 성능 차이는 무시해도 됨
- `web` 패키지 생성 후 `HelloController` 클래스 생성

  ```java
  @RestController
  public class HelloController {

      @GetMapping("/hello")
      public String hello() {
          return "hello";
      }
  }
  ```

  - `@RestController` : JSON을 반환하는 컨트롤러로 변환시켜줌
    - `@ResponseBody`를 각 메소드마다 선언해야했던 것을 대체함
  - `@GetMapping` : Get 요청을 받을 수 있는 API를 만들어줌

- 테스트 경로에 동일하게 패키지 생성 후 `HelloControllerTest` 클래스 생성

  ```java
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.extension.ExtendWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
  import org.springframework.test.context.junit.jupiter.SpringExtension;
  import org.springframework.test.web.servlet.MockMvc;
  import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
  import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

  @ExtendWith(SpringExtension.class)
  @WebMvcTest(controllers = HelloController.class)
  public class HelloControllerTest {

      @Autowired
      private MockMvc mvc;

      @Test
      public void hello가_리턴된다() throws Exception {
          String hello = "hello";

          mvc.perform(get("/hello"))
                  .andExpect(status().isOk())
                  .andExpect(content().string(hello));
      }
  }
  ```

  - static import는 직접 해줘야함
  - `@ExtendWith` : 테스트 클래스(메소드)의 기능을 확장함
    - 위 코드는 `SpringExtension`이라는 기능을 JUnit Jupiter와 통합함
  - `@WebMvcTest` : `@Controller`, `@RestController`가 사용된 클래스들을 메모리에 생성
    - controller만을 메모리에 올리기 때문에 Web만 테스트할 때 사용
      - 다른 component들도 테스트할 때는 `@AutoConfigureMockMvc`를 사용해야 함
    - `@WebMvcTest`, `@SpringBootTest`는 둘 다 `MockMvc`를 모킹하기 때문에 함께 사용할 경우 충돌이 발생함
    - 파라미터로 인스턴스화(테스트)할 controller 지정 가능
  - `@Autowired` : 타입에 맞는 spring bean을 주입함
  - `MockMvc` : servlet container를 모킹한 객체
    - `hello가_리턴된다()`는 MockMvc 객체를 사용하는 컨트롤러에 대해 작성된 코드임
  - `@MockBean`을 통해 테스트 클래스에서의 DI 가능
  - `mvc.perform(get("/hello"))` : 모킹된 컨테이너에 요청을 보냄

## lombok 설치 후 리팩토링

- `build.gradle`의 의존성에 아래 내용 추가 후 새로고침
  ```java
  dependencies {
  	compileOnly 'org.projectlombok:lombok'
  	annotationProcessor 'org.projectlombok:lombok'
  }
  configurations {
      compileOnly {
          extendsFrom annotationProcessor
      }
  }
  ```
  - 이후 Settings - BED - Compiler - Annotation Processors에서 Enable annotation processing 활성화
- `java`의 `web` 패키지 안에 `dto` 패키지 생성 후 `HelloResponseDto` 클래스 생성

  ```java
  import lombok.Getter;
  import lombok.RequiredArgsConstructor;

  @Getter
  @RequiredArgsConstructor
  public class HelloResponseDto {

      private final String name;
      private final int amount;
  }
  ```

  - `@Getter` : 선언된 모든 필드의 get 메소드를 생성해줌
  - `@RequiredArgsConstructor` : 선언된 모든 final 필드가 포함된 생성자를 생성해줌

- `test`에도 동일하게 패키지 생성 후 `HelloResponseDtoTest` 클래스 생성

  ```java
  public class HelloResponseDtoTest {

      @Test
      public void lombok_test() {
          //given
          String name = "test";
          int amount = 1000;

          //when
          HelloResponseDto dto = new HelloResponseDto(name, amount);

          //then
          assertThat(dto.getName()).isEqualTo(name);
          assertThat(dto.getAmount()).isEqualTo(amount);
      }
  }
  ```

  - `assertThat`을 사용하기 위해 `assertj`의 `assertThat`을 static import함
    - JUnit의 `assertThat`은 `is()` 등을 사용하기 위해 `CoreMatcher`와 같은 라이브러리가 필요하고, 자동완성 지원이 약함
  - 테스트 실행하면 성공적으로 수행됨
    - `@Data`는 Setter까지 무분별하게 생성해주는데, 객체에 대한 접근 제한이 없어지기 때문에 객체의 안정성이 보장되지 않음

- ResponseDto 매핑을 위해 `HelloController`에 아래 메소드 추가
  ```java
  @GetMapping("/hello/dto")
  public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
      return new HelloResponseDto(name, amount);
  }
  ```
  - `@RequestParam` : 외부에서 API로 넘긴 파라미터를 가져옴
    - `"name"`이라는 이름으로 넘긴 파라미터를 `String name`에 저장함
- `HelloControllerTest`에 아래 테스트 추가

  ```java
  @Test
  public void return_helloDto() throws Exception {
      String name = "hello";
      int amount = 1000;

      mvc.perform(
                      get("/hello/dto")
                              .param("name", name)
                              .param("amount", String.valueOf(amount)))
              .andExpect(status().isOk())
              .andExpect(jsonPath("$.name", is(name)))
              .andExpect(jsonPath("$.amount", is(amount)));
  }
  ```

  - `import static org.hamcrest.Matchers.is;`로 `is()`를 import해야 함
  - `.param()`을 통해 request parameter 설정 가능
    - `String`만 허용되기 때문에 숫자일 경우 위와 같이 `String.valueOf()`을 이용해 문자열로 바꿔야 함
  - `jsonPath()`를 통해 JSON response를 필드별로 검증 가능
    - `$`를 기준으로 필드명 선택

### Dao, Dto

- Dao(Data Access Object) : DB에 접근하기 위한 객체
  - DB 접근 로직, 비즈니스 로직을 분리하기 위해 사용됨
- Dto(Data Transfer Object) : 계층 간 데이터 교환을 위해 사용되는 객체
  - 로직을 가지지 않는 순수한 데이터 객체임(getter, setter만 가짐)
- VO(Value Object) : read-only → setter를 가지지 않음
- 유저가 입력한 데이터는 DTO를 통해 서버로 전송됨  
  → DTO를 받은 서버가 DAO를 이용해서 DB로 데이터를 집어넣음
