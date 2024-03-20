---
layout: single
title: "Spring Boot와 AWS로 혼자 구현하는 웹 서비스 2 - JPA(Ch.03)"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - Spring Boot 2
  - JPA
  - Annotation
  - Request
  - form
  - Test
  - CRUD
  - Spring Web Layer
  - JPA Auditing
---

### 패러다임 불일치

- 관계형 DB : 어떻게 데이터를 저장할지에 초점이 맞춰짐
- OOP : 메시지를 기반으로 기능과 속성을 한 곳에서 관리하는데 초점이 맞춰짐
- 객체를 DB에 저장할 때 객체의 모델링을 표현할 방법이 부족했음

  ```java
  // Java
  User user = findUser();
  Group group = user.getGroup();

  // Java + DB
  User user = userDao.findUser();
  Group group = groupDao.findGroup(user.getGroupId());
  ```

  - `User`, `Group`은 부모-자식 관계이기 때문에 자바 코드에서는 `user`만으로 `group`까지 조회가 가능하지만, DB가 들어간 코드에서는 따로 조회해야 함

- JPA(Java Persistence API) : 관계형 DB에 맞게 SQL을 대신 생성해주기 위한 Interface  
  → SQL에 종속적인 개발을 하지 않아도 됨

### Spring Data JPA

- JPA의 구현체 : Hibernate, Eclipse Link, …
- Spring Data JPA : 구현체들을 더 쉽게 사용하기 위해 추상화시킨 모듈  
  JPA ← Hibernate ← Spring Data JPA
  - 구현체 교체의 용이성
  - 저장소 교체의 용이성
    - 관계형 DB → MongoDB 등 : Spring data JPA → Spring Data MongoDB로 의존성만 교체하면 됨
    - Spring Data의 프로젝트들은 CRUD의 interface가 같기 때문

### 요구사항 분석

- 게시판 구현, 배포 예정
- 게시판 기능 : CRUD
- 회원 기능
  - 구글/네이버 로그인
  - 로그인한 사용자 글 작성 권한
  - 본인 작성 글에 대한 권한 관리

### Spring Data Jpa 적용

- `build.gradle`에 아래 의존성 추가
  ```java
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'com.h2database:h2'
  ```
  - `application.properties`에 따로 적지 않아도 됨
- 최상위 경로에 `domain.posts` 패키지 추가 후 `Posts` 클래스 생성

  ```java
  @Getter
  @NoArgsConstructor
  @Entity
  public class Posts {

      @Id
      @GeneratedValue
      private Long id;

      @Column(length = 500, nullable = false)
      private String title;

      @Column(columnDefinition = "TEXT", nullable = false)
      private String content;

      private String author;

      @Builder
      public Posts(String title, String content, String author) {
          this.title = title;
          this.content = content;
          this.author = author;
      }
  }
  ```

  - `Posts` : 실제 DB의 테이블과 매칭될 Entity 클래스
  - JPA를 사용할 때 실제 쿼리를 날리는 대신 Entity 클래스의 수정을 통해 작업할 수 있음
  - `@Entity` : 테이블과 링크될 클래스임을 나타냄
    - `_classname`이 기본 테이블 이름  
      e.g. `SalesManager.java` → `sales_manager` table
  - `@Id` : 테이블의 PK 필드
  - `@GeneratedValue(strategy = GenerationType.IDENTITY)` : PK의 생성 규칙
    - strategy를 위와 같이 추가해야만 auto increment됨
  - `@Column` : 테이블의 column을 나타냄
    - 기본적으로 선언되며, 명시적으로 선언해서 column의 옵션 변경 가능
      - `length = 500` : 문자열의 크기를 기본값 `255`에서 `500`으로 늘림
      - `columnDefinition = "TEXT"` : 타입을 `TEXT`로 변경
  - `@NoArgsConstructor` : 기본 생성자(`public Posts() {}`) 자동 추가
  - `@Builder` : 클래스의 builder pattern class를 생성
    - 생성자에 선언하면 생성자에 포함된 필드만 빌더에 포함
  - 초기에는 테이블 설계가 자주 변경되는데, lombok annotation을 사용하면 코드 변경량을 최소화시켜줌

> Entity의 PK는 Long 타입의 Auto-increment가 좋음  
> unique key나 복합키를 PK로 잡을 경우)
>
> 1. FK(Foreign Key)를 맺을 때 다른 테이블이 복합키를 전부 갖거나, 중간 테이블을 하나 더 둬야하는 상황이 발생함
> 2. 유니크한 조건이 변경되면 PK 전체를 수정해야 함

- Entity 클래스는 instance의 필드가 변경되는 시점을 구분이 가능해야 유지보수할 때 편함  
  → Setter method를 만들지 않고, 필드의 변경이 필요할 경우 목적을 분명히 나타내는 메소드를 추가함

      ```java
      public class Order {
      	publlic void cancelOrder() {
      		this.status = false;
      	}
      }

      public void cancelation() {
      	order.cancelOrder();
      }
      ```

- Setter method가 없는데 어떻게 값을 채울까? → 생성자를 통해 삽입할 값을 채움

  - 이 코드에서는 생성자 대신 `@Builder`에 의해 제공되는 빌더 클래스 사용
  - 생성자는 필드를 지정할 수 없지만, 빌더 클래스는 채울 필드를 지정 가능

    ```java
    Bag bag = new Bag("name", 1000, "memo", "abc", "what");

    Bag bag = Bag.builder()
            	.money(1000)
    					.name("name")
            	.memo("memo")
            	.letter("This is the letter")
            	.box("This is the box")
            	.build();
    ```

- 값 변경이 필요한 경우 해당 이벤트에 맞는 public method 호출

- `posts` 패키지에 `PostsRepository` 인터페이스 생성
  ```java
  public interface PostsRepository extends JpaRepository<Posts, Long> {
  }
  ```
- JPA에서의 Repository : Dao라고 부르는 DB Layer 접근자 역할
  - interface로 생성 후 `JpaRepository<Entity 클래스, PK 타입>`를 상속하면 CRUD 메소드가 자동으로 생성됨
  - `@Repository`를 추가하지 않아도 됨
  - Entity 클래스와 Entity Repository는 같은 경로에 존재해야 함

### Spring Data JPA 테스트 구현

- `PostsRepository` 인터페이스 테스트 생성

  ```java
  @ExtendWith(SpringExtension.class)
  @SpringBootTest
  class PostsRepositoryTest {

      @Autowired
      PostsRepository postsRepository;

      @AfterEach
      public void cleanup() {
          postsRepository.deleteAll();
      }

      @Test
      public void saveAndload() {
          //given
          String title = "test post";
          String content = "test content";

          postsRepository.save(Posts.builder()
                  .title(title)
                  .content(content)
                  .author("hwy16016@gmail.com")
                  .build());

          //when
          List<Posts> postsList = postsRepository.findAll();

          //then
          Posts posts = postsList.get(0);
          assertThat(posts.getTitle()).isEqualTo(title);
          assertThat(posts.getContent()).isEqualTo(content);
      }
  }
  ```

  - `postsRepository.save()` : id가 있다면 update, 없다면 insert 쿼리가 실행됨
  - `@SpringBootTest`를 다른 설정 없이 사용하면 H2가 자동으로 실행됨

- `application.properties`에 `spring.jpa.show_sql=true` 추가하면 쿼리 출력됨
  - 아래 내용 추가해서 MySQL 문법으로 바꿀 수 있음
    ```java
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
    spring.jpa.properties.hibernate.dialect.storage_engine=innodb
    spring.datasource.hikari.jdbc-url=jdbc:h2:mem://localhost/~/testdb;MODE=MYSQL
    ```

### 등록/수정/조회 API 만들기

- API를 만들기 위해 3개의 클래스가 필요함
  - Request 데이터를 받을 Dto
  - API 요청을 받을 Controller
  - Transaction, Domain 기능 간의 순서를 보장하는 Service
    - Service는 business logic을 처리하지 않고 transaction, domain 간의 순서 보장의 역할만 함
- Spring Web Layer
  ![springawsnote1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/springawsnote1.png)
  - Web Layer : View Template 영역
    - `@Controller`, JSP/Freemarker, `@Filter`, `@ControllerAdvice`, 인터셉터 등의 **외부 요청과 응답**에 대한 영역
  - Service Layer : 서비스 영역
    - `@Service`, `@Transactional` 등이 사용됨
    - Controller와 Dao의 중간 영역에서 사용됨
  - Repository Layer : DB에 접근하는 영역(Dao 영역)
  - DTOs : view template engine에서 사용될 객체나 Repository Layer에서 결과로 넘겨준 객체 등
  - Domain Model : domain(개발 단위)을 단순화시킨 것  
    e.g. 탭시 앱의 경우 배차, 탑승, 요금 등이 모두 도메인이 됨 - `@Entity`가 사용된 영역도 domain model의 한 종류임 - 무조건 테이블과 관련있어야 하는 것은 아님(e.g. VO)
- Service 객체에서 비즈니스 로직을 처리할 경우:

  ```java
  @Transactional
  public Order cancelOrder(int orderId) {
      OrdersDto order = ordersDao.selectOrders(orderId);
      BillingDto billing = billingDao.selectBilling(orderId);
      DeliveryDto delivery = deliveryDao.selectDelivery(orderId);

      String deliveryStatus = delivery.getStatus();

      if("IN_PROGRESS".equals(delivveryStatus)) {
          delivery.setStatus("CANCEL");
          deliveryDao.update(delivery);
      }

      order.setStatus("CANCEL");
      deliveryDao.update(billing);

      return order;
  }
  ```

  - 모든 로직이 Service 내부에서 처리됨 → 서비스 계층이 무의미함

- 도메인 모델에서 처리할 경우:

  ```java
  @Transactional
  public Order CancelOrder(int orderId) {
      Orders order = ordersRepository.findById(orderId);
      Billing billing = billingRepository.findByOrderId(orderId);
      Delivery delivery = deliveryRepository.findByOrderId(orderId);

      delivery.cancel();

      order.cancel();
      billing.cancel();

      return order;
  }
  ```

  - order, billing, delivery가 각자 도메인의 취소 이벤트를 처리함
  - 서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장해줌

- `web` 패키지에 `PostsApiController` 클래스 생성

  ```java
  @RequiredArgsConstructor
  @RestController
  public class PostsApiController {

      private final PostsService postsService;

      @PostMapping("/api/v1/posts")
      public Long save(@RequestBody PostsSaveRequestDto requestDto) {
          return postsService.save(requestDto);
      }

  }
  ```

- `service.posts` 패키지에 `PostsService` 클래스 생성

  ```java
  @RequiredArgsConstructor
  @Service
  public class PostsService {
      private final PostsRepository postsRepository;

      @Transactional
      public Long save(PostsSaveRequestDto requestDto) {
          return postsRepository.save(requestDto.toEntity()).getId();
      }
  }
  ```

  - `final` 필드에 `@Autowired`가 없는 이유
    - Bean을 주입 받는 방식 3가지(`@Autowired`, setter, 생성자) 중 생성자 주입을 이용하기 때문
    - `@RequiredArgsConstructor`가 `final` 필드가 모두 포함된 생성자를 자동으로 생성해주기 때문에 생성자 코드는 생략함

- `web.dto` 패키지에 `PostsSaveRequestDto` 클래스 생성

  ```java
  @Getter
  @NoArgsConstructor
  public class PostsSaveRequestDto {
      private String title;
      private String content;
      private String author;

      @Builder
      public PostsSaveRequestDto(String title, String content, String author) {
          this.title = title;
          this.content = content;
          this.author = author;
      }

      public Posts toEntity() {
          return Posts.builder()
                  .title(title)
                  .content(content)
                  .author(author)
                  .build();
      }
  }
  ```

  - Entity 클래스와 유사한 형태이지만 Dto 대신 Entity를 Request/Response 클래스로 사용하면 안됨
    - Entity 클래스를 기준으로 테이블이 생성되고 변경됨
    - Service 클래스나 비즈니스 로직(도메인)들도 Entity 클래스를 기준으로 동작함
    - Request와 Response용 Dto는 View를 위한 클래스이기 때문에 자주 변경됨
    - Controller에서 여러 테이블을 조인해서 줘야 할 경우가 많은데, 이런 경우 Entity 클래스만으로는 표현하기 어려움
    - view를 위해 Entity를 변경하는 것은 비용 소모가 심하기 때문에 Entity 클래스와 Controller에서 사용할 Dto는 분리해서 사용해야 함

- `postsApiController` 클래스의 테스트 생성

  ```java
  @ExtendWith(SpringExtension.class)
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  class PostsApiControllerTest {

      @LocalServerPort
      private int port;

      @Autowired
      private TestRestTemplate restTemplate;

      @Autowired
      private PostsRepository postsRepository;

      @AfterEach
      public void tearDown() throws Exception {
          postsRepository.deleteAll();
      }

      @Test
      public void Create_Posts() throws Exception {
          //given
          String title = "title";
          String content = "content";
          PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                  .title(title)
                  .content(content)
                  .author("author")
                  .build();

          String url = "http://localhost:" + port + "/api/v1/posts";

          //when
          ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

          //then
          assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
          assertThat(responseEntity.getBody()).isGreaterThan(0L);

          List<Posts> all = postsRepository.findAll();
          assertThat(all.get(0).getTitle()).isEqualTo(title);
          assertThat(all.get(0).getContent()).isEqualTo(content);
      }
  }
  ```

  - `@WebMvcTest`에서는 JPA가 작동하지 않고 Controller, controllerAdvice 등의 외부 연동과 관련된 부분만 활성화됨  
    → JPA 기능까지 한 번에 테스트할 경우 `@SpringBootTest`, `TestRestTemplate`을 사용해야 함

- `PostsApiController`에 수정, 조회 기능 업데이트

  ```java
  @PutMapping("/api/v1/posts/{id}")
  public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto) {
      return postsService.update(id, requestDto);
  }

  @GetMapping("/api/v1/posts/{id}")
  public PostsResponseDto findById(@PathVariable Long id) {
      return postsService.findById(id);
  }
  ```

- `web.dto`에 `PostsResponseDto` 클래스 생성

  ```java
  @Getter
  public class PostsResponseDto {

      private Long id;
      private String title;
      private String content;
      private String author;

      public PostsResponseDto(Posts entity) {
          this.id = entity.getId();
          this.title = entity.getTitle();
          this.content = entity.getContent();
          this.author = entity.getAuthor();
      }
  }
  ```

  - Entity의 필드 중 일부만 사용하기 때문에 생성자에서 entity를 받아서 처리함

- `web.dto`에 `PostsUpdateRequestDto` 생성

  ```java
  @Getter
  @NoArgsConstructor
  public class PostsUpdateRequestDto {
      private String title;
      private String content;

      @Builder
      public PostsUpdateRequestDto(String title, String content) {
          this.title = title;
          this.content = content;
      }
  }
  ```

- `domain.posts.Posts` 클래스에 `update` 메소드 추가
  ```java
  public void update(String title, String content) {
      this.title = title;
      this.content = content;
  }
  ```
- `service.posts.PostsService` 클래스에 `update`, `findById` 메소드 추가

  ```java
  @Transactional
  public Long update(Long id, PostsUpdateRequestDto requestDto) {
      Posts posts = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("No such post. id=" + id));

      posts.update(requestDto.getTitle(), requestDto.getContent());

      return id;
  }

  public PostsResponseDto findById(Long id) {
      Posts entity = postsRepository.findById(id).orElseThrow(() -> new IllegalArgumentException("No such post. id=" + id));

      return new PostsResponseDto(entity);
  }
  ```

  - DB에 직접 쿼리를 날리지 않음(JPA의 persistence context 때문)

- JPA의 persistence context : entity를 영구적으로 저장해주는 환경
  - Spring Data Jpa를 사용하면 JPA의 EntityManager가 활성화됨
  - entity를 사용해서 테이블이 udpate되는 과정
    1. EntityManager가 활성화된 상태에서 transaction이 시작됨
    2. transaction 도중 DB의 데이터를 가져와서 그 데이터가 persist됨
    3. persist된 상태에서 데이터를 변경됨
    4. transaction이 끝나는 시점에서 dirty checking을 통해 데이터의 변경 여부를 확인하고 변경 내용을 반영함
  - Entity 객체의 값만 변경하고 별도의 Update 쿼리를 날릴 필요가 없음
- Dirty checking은 기본적으로 모든 필드를 업데이트함
  - 생성되는 쿼리가 같아 부트 실행시점에 미리 만들어서 쿼리 재사용 가능
  - DB 입장에서도 동일한 쿼리를 받으면 이전에 파싱된 쿼리를 재사용 가능
  - 필드 개수가 많아서 전체 필드 update 쿼리가 부담스러울 경우 `@DynamicUpdate` annotation으로 변경 필드만 반영되도록 설정 가능
    - 사실 필드가 많은 것 자체가 정규화가 잘못된 것일 확률이 높음
- `PostsApiControllerTest`에 수정 기능의 테스트 구현

  ```java
  @Test
  public void Update_Posts() throws Exception {
      //given
      Posts savedPosts = postsRepository.save(Posts.builder()
              .title("title")
              .content("content")
              .author("author")
              .build());

      Long updateId = savedPosts.getId();
      String expectedTitle = "title2";
      String expectedContent = "content2";

      PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
              .title(expectedTitle)
              .content(expectedContent)
              .build();

      String url = "http://localhost:" + port + "/api/v1/posts/" + updateId;

      HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);

      //when
      ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PUT, requestEntity, Long.class);

      //then
      assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
      assertThat(responseEntity.getBody()).isGreaterThan(0L);

      List<Posts> all = postsRepository.findAll();
      assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
      assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
  }
  ```

### JPA Auditing으로 생성시간/수정시간 자동화하기

- JPA Auditing : Spring Data JPA에서 Entity에 대한 `@CreatedDate`, `@LastModifiedDate`를 자동으로 넣어주는 기능
- Java8에서, `Date`를 보완한 `LocalDate`, `LocalDateTime`이 나옴
  - `Date`, `Calender` 클래스는 불변 객체가 아니어서 멀티스레드 환경에서 문제 발생 가능
  - `Calender.OCTOBER`가 `9`임
  - SpringBoot 1.x버전일 경우 Hibernate 5.2.10 버전 이상을 사용하기 위해 별도로 설정해줘야 함
- `domain` 패키지에 `BaseTimeEntity` 클래스 생성

  ```java
  @Getter
  @MappedSuperclass
  @EntityListeners(AuditingEntityListener.class)
  public abstract class BaseTimeEntity {

      @CreatedDate
      private LocalDateTime createdDate;

      @LastModifiedDate
      private LocalDateTime modifiedDate;
  }
  ```

  - 모든 entity의 상위 클래스가 되어 `createdDate`, `modifiedDate`를 자동으로 관리함
  - `@MappedSuperclass` : JPA Entity 클래스들이 `BaseTimeEntity`를 상속할 경우 `BaseTimeEntity`의 필드들을 column으로 인식하도록 설정
  - `@EntityListeners(AuditingEntityListener.class)` : annotation이 붙은 클래스에 Auditing 기능을 포함시킴
  - `@CreatedDate` : Entity가 생성, 저장된 시간이 필드에 저장됨
  - `@LastModifiedDate` : 조회한 Entity의 값을 변경할 때의 시간이 저장됨

- `domain.posts.Posts` 클래스가 `BaseTimeEntity`를 상속받도록 변경
  ```java
  public class Posts extends BaseTimeEntity {
  ...
  ```
- `SpringawsApplication` 클래스에 `@EnableJpaAuditing` annotation 추가
- `PostsRepositoryTest`에 테스트 추가

  ```java
  @Test
  public void BaseTimeEntity_Create() {
      //given
      LocalDateTime now = LocalDateTime.of(2022, 9, 23, 0, 0, 0);
      postsRepository.save(Posts.builder()
              .title("title")
              .content("content")
              .author("author")
              .build());

      //when
      List<Posts> postsList = postsRepository.findAll();

      //then
      Posts posts = postsList.get(0);

      System.out.println(">>>>>>> createDate=" + posts.getCreatedDate() + ", modifiedDate=" + posts.getModifiedDate());

      assertThat(posts.getCreatedDate()).isAfter(now);
      assertThat(posts.getModifiedDate()).isAfter(now);
  }
  ```
