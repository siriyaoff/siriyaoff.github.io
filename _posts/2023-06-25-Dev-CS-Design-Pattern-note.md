---
layout: single
title: "디자인 패턴과 프로그래밍 패러다임"
categories:
  - Dev
tags:
  - Java
  - JavaScript
  - Singleton
  - Factory
  - Strategy
  - Observer
  - extends
  - implements
  - Proxy
  - CORS
  - Iterator
  - 노출모듈
  - MVC
  - MVP
  - MVVM
  - Imperative programming
  - Procedural programming
  - Object-oriented programming
  - OOP
  - SOLID
  - Declarative programming
  - Functional programming
  - First-class object
  - Reactive programming
---

# 1.1 디자인 패턴

- 디자인 패턴 : 프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용하여 해결할 수 있도록 하나의 ‘규약’ 형태로 만들어 놓은 것

## 1.1.1 Singleton 패턴

- 하나의 클래스가 하나의 인스턴스를 가짐
- 장점
  - 인스턴스 생성 비용 감소
- 단점
  - 인스턴스에 대한 의존성이 높아짐
  - 테스트마다 독립적인 instance를 만들기가 어렵기 때문에 TDD가 불편함
    - DI로 해결 가능
- DB 연결 모듈에 자주 사용

  - JS
    ```jsx
    const URL = "URL";
    const createConnection = (url) => ({ url: url });
    class DB {
      constructor(url) {
        if (!DB.instance) {
          DB.instance = createConnection(url);
        }
        return DB.instance;
      }
      connect() {
        return this.instance;
      }
    }
    const a = new DB(URL);
    const b = new DB(URL);
    console.log(a === b); // true
    ```
  - Java

    ```java
    class Singleton {
        private static class singleInstanceHolder {
            private static final Singleton INSTANCE = new Singleton();
        }
        public static Singleton getInstance() {
            return singleInstanceHolder.INSTANCE;
        }
    }

    public class HelloWorld{
         public static void main(String []args){
            Singleton a = Singleton.getInstance();
            Singleton b = Singleton.getInstance();
            System.out.println(a.hashCode());
            System.out.println(b.hashCode());
            if (a == b){
             System.out.println(true);
            }
         }
    }
    ```

    - `singleInstanceHolder`가 `static`이고 `static` 필드에 `Singleton`의 인스턴스를 저장하기 때문에 `Singleton` 클래스의 instance는 앱 당 하나만 존재함(멀티 스레드 환경에서도 마찬가지)
    - `Singleton` 클래스가 메모리로 로드되지 않고, `getInstance()` 메소드가 호출될 때 `Singleton` instance가 리턴됨
    - `static`이기 때문에 다른 곳에서 `Singleton.getInstance()`가 호출되는 시점에 `Singleton` 객체가 생성되고 `INSTANCE` 필드가 초기화됨

  - mongoose

    ```jsx
    Mongoose.prototype.connect = function (uri, options, callback) {
      const _mongoose = this instanceof Mongoose ? this : mongoose;
      const conn = _mongoose.connection;
      return _mongoose._promiseOrCallback(callback, (cb) => {
        conn.openUri(uri, options, (err) => {
          if (err != null) {
            return cb(err);
          }
          return cb(null, _mongoose);
        });
      });
    };

    // 메인 모듈
    const mysql = require("mysql");
    const pool = mysql.createPool({
      connectionLimit: 10,
      host: "example.org",
      user: "kundol",
      password: "secret",
      database: "승철이디비",
    });
    pool.connect();
    // 모듈 A
    pool.query(query, function (error, results, fields) {
      if (error) throw error;
      console.log("The solution is: ", results[0].solution);
    });
    // 모듈 B
    pool.query(query, function (error, results, fields) {
      if (error) throw error;
      console.log("The solution is: ", results[0].solution);
    });
    ```

## 1.1.2 Factory 패턴

- 기본 구조
  - 사용할 객체 : 세부적인 설정이 들어가있는 객체
  - abstract 객체 : 사용할 객체가 상속받는 객체
  - 팩토리 객체 : 사용할 객체를 생성하는 메소드를 가지고 있음
    - 세부 설정을 파라미터로 받고 리턴 타입은 abstract 객체로 설정하는 경우가 많음
- JS에서의 `new Object()`가 팩토리 패턴의 대표적인 예시임
- abstract 클래스에서는 instance의 생성 방식을 알 필요가 없음
- 생성 로직이 분리되어 있기 때문에 유지 보수가 쉬움
- JS

  ```jsx
  class CoffeeFactory {
    static createCoffee(type) {
      const factory = factoryList[type];
      return factory.createCoffee();
    }
  }
  class Latte {
    constructor() {
      this.name = "latte";
    }
  }
  class Espresso {
    constructor() {
      this.name = "Espresso";
    }
  }

  class LatteFactory extends CoffeeFactory {
    static createCoffee() {
      return new Latte();
    }
  }
  class EspressoFactory extends CoffeeFactory {
    static createCoffee() {
      return new Espresso();
    }
  }
  const factoryList = { LatteFactory, EspressoFactory };

  const main = () => {
    // 라떼 커피를 주문한다.
    const coffee = CoffeeFactory.createCoffee("LatteFactory");
    // 커피 이름을 부른다.
    console.log(coffee.name); // latte
  };
  main();
  ```

  - `CoffeeFactory.createCoffee("LatteFactory")`가 호출되면 `LatteFactory.createCoffee()`가 호출되어 `Latte` instance를 생성 후 DI가 수행됨
  - `createCoffee` 메소드는 `static`으로 선언되어 instance 없이 호출 가능

- Java

  ```jsx
  enum CoffeeType {
      LATTE,
      ESPRESSO
  }

  abstract class Coffee {
      protected String name;

      public String getName() {
          return name;
      }
  }

  class Latte extends Coffee {
      public Latte() {
          name = "latte";
      }
  }

  class Espresso extends Coffee {
      public Espresso() {
          name = "Espresso";
      }
  }

  class CoffeeFactory {
      public static Coffee createCoffee(CoffeeType type) {
          switch (type) {
              case LATTE:
                  return new Latte();
              case ESPRESSO:
                  return new Espresso();
              default:
                  throw new IllegalArgumentException("Invalid coffee type: " + type);
          }
      }
  }

  public class Main {
      public static void main(String[] args) {
          Coffee coffee = CoffeeFactory.createCoffee(CoffeeType.LATTE);
          System.out.println(coffee.getName()); // latte
      }
  }
  ```

## 1.1.3 Strategy 패턴

- policy pattern이라고도 함
- 결제 프로세스에서 결제 방법, JS의 `passport` 모듈이 대표적인 예시
  - `passport` : 인증 모듈을 구현할 때 사용하는 미들웨어
    - `LocalStrategy` : 서비스 내 아이디 정보로 인증
    - 이외에 `OAuth`도 사용 가능
- Java

  ```java
  import java.text.DecimalFormat;
  import java.util.ArrayList;
  import java.util.List;
  interface PaymentStrategy {
      public void pay(int amount);
  }

  class KAKAOCardStrategy implements PaymentStrategy {
      private String name;
      private String cardNumber;
      private String cvv;
      private String dateOfExpiry;

      public KAKAOCardStrategy(String nm, String ccNum, String cvv, String expiryDate){
          this.name=nm;
          this.cardNumber=ccNum;
          this.cvv=cvv;
          this.dateOfExpiry=expiryDate;
      }

      @Override
      public void pay(int amount) {
          System.out.println(amount +" paid using KAKAOCard.");
      }
  }

  class LUNACardStrategy implements PaymentStrategy {
      private String emailId;
      private String password;

      public LUNACardStrategy(String email, String pwd){
          this.emailId=email;
          this.password=pwd;
      }

      @Override
      public void pay(int amount) {
          System.out.println(amount + " paid using LUNACard.");
      }
  }

  class Item {
      private String name;
      private int price;
      public Item(String name, int cost){
          this.name=name;
          this.price=cost;
      }

      public String getName() {
          return name;
      }

      public int getPrice() {
          return price;
      }
  }

  class ShoppingCart {
      List<Item> items;

      public ShoppingCart(){
          this.items=new ArrayList<Item>();
      }

      public void addItem(Item item){
          this.items.add(item);
      }

      public void removeItem(Item item){
          this.items.remove(item);
      }

      public int calculateTotal(){
          int sum = 0;
          for(Item item : items){
              sum += item.getPrice();
          }
          return sum;
      }

      public void pay(PaymentStrategy paymentMethod){
          int amount = calculateTotal();
          paymentMethod.pay(amount);
      }
  }

  public class HelloWorld{
      public static void main(String []args){
          ShoppingCart cart = new ShoppingCart();

          Item A = new Item("kundolA",100);
          Item B = new Item("kundolB",300);

          cart.addItem(A);
          cart.addItem(B);

          // pay by LUNACard
          cart.pay(new LUNACardStrategy("kundol@example.com", "pukubababo"));
          // pay by KAKAOBank
          cart.pay(new KAKAOCardStrategy("Ju hongchul", "123456789", "123", "12/01"));
      }
  }
  /*
  400 paid using LUNACard.
  400 paid using KAKAOCard.
  */
  ```

- JS

  ```jsx
  var passport = require("passport"),
    LocalStrategy = require("passport-local").Strategy;

  passport.use(
    new LocalStrategy(function (username, password, done) {
      User.findOne({ username: username }, function (err, user) {
        if (err) {
          return done(err);
        }
        if (!user) {
          return done(null, false, { message: "Incorrect username." });
        }
        if (!user.validPassword(password)) {
          return done(null, false, { message: "Incorrect password." });
        }
        return done(null, user);
      });
    })
  );
  ```

## Observer 패턴

- 객체의 상태 변화가 있을 때마다 옵저버들에게 변화를 알려줌
  - 객체 관찰, 옵저버들에게 알림을 담당하는 주체를 따로 두기도 함
- 이벤트 기반 시스템에서 주로 사용함
  - 트위터가 대표적인 예시임
- MVC에도 사용됨
  - Model : `@Observable`, `notifyAll()` 등의 메소드를 통해 `@Observer`들에게 전파함
  - Controller : `@EventListener`
  - View : `@Observer`, `update()` 메소드를 통해 변경 사항을 받음
- Java

  ```java
  import java.util.ArrayList;
  import java.util.List;

  interface Subject {
      public void register(Observer obj);
      public void unregister(Observer obj);
      public void notifyObservers();
      public Object getUpdate(Observer obj);
  }

  interface Observer {
      public void update();
  }

  class Topic implements Subject {
      private List<Observer> observers;
      private String message;

      public Topic() {
          this.observers = new ArrayList<>();
          this.message = "";
      }

      @Override
      public void register(Observer obj) {
          if (!observers.contains(obj)) observers.add(obj);
      }

      @Override
      public void unregister(Observer obj) {
          observers.remove(obj);
      }

      @Override
      public void notifyObservers() {
          this.observers.forEach(Observer::update);
      }

      @Override
      public Object getUpdate(Observer obj) {
          return this.message;
      }

      public void postMessage(String msg) {
          System.out.println("Message sended to Topic: " + msg);
          this.message = msg;
          notifyObservers();
      }
  }

  class TopicSubscriber implements Observer {
      private String name;
      private Subject topic;

      public TopicSubscriber(String name, Subject topic) {
          this.name = name;
          this.topic = topic;
      }

      @Override
      public void update() {
          String msg = (String) topic.getUpdate(this);
          System.out.println(name + ":: got message >> " + msg);
      }
  }

  public class HelloWorld {
      public static void main(String[] args) {
          Topic topic = new Topic();
          Observer a = new TopicSubscriber("a", topic);
          Observer b = new TopicSubscriber("b", topic);
          Observer c = new TopicSubscriber("c", topic);
          topic.register(a);
          topic.register(b);
          topic.register(c);

          topic.postMessage("amumu is op champion!!");
      }
  }
  /*
  Message sended to Topic: amumu is op champion!!
  a:: got message >> amumu is op champion!!
  b:: got message >> amumu is op champion!!
  c:: got message >> amumu is op champion!!
  */
  ```

  - Interface `Observer`, `Subject`를 선언해놓고 각각의 구현체로 옵저버 패턴을 구현함
  - 이벤트는 psvm에서 수동으로 넣어줌

- JS
  ```jsx
  function createReactiveObject(target, callback) {
    const proxy = new Proxy(target, {
      set(obj, prop, value) {
        if (value !== obj[prop]) {
          const prev = obj[prop];
          obj[prop] = value;
          callback(`${prop}가 [${prev}] >> [${value}] 로 변경되었습니다`);
        }
        return true;
      },
    });
    return proxy;
  }
  const a = {
    형규: "솔로",
  };
  const b = createReactiveObject(a, console.log);
  b.형규 = "솔로";
  b.형규 = "커플";
  // 형규가 [솔로] >> [커플] 로 변경되었습니다
  ```
  - `proxy` 객체를 선언하면 레퍼런스처럼 기존 객체의 메소드, 필드를 사용할 수 있음
  - `Proxy` 객체의 파라미터
    - `target` : 프록시 객체를 만들 대상
    - `handler` : 객체에서 이벤트를 가로채서 실행할 함수들
      - `get`, `set`, `has`(`in` 연산자)
  - Vue에서도 DOM에 있는 요소를 변경할 때 proxy 패턴을 사용함

### Java `extends`, `implements` 차이

- `extends` : 자식 클래스에서 부모 클래스를 상속하여 메소드의 추가, 확장 가능
  - 코드 중복을 줄여줌
  - 일반 클래스, `abstract` 클래스를 상속받을 수 있음
    - `interface` → `interface`도 `extends` 사용
  - 다중 상속 불가
- `implements` : 부모 interface를 구현함
  - 반드시 부모의 메소드를 모두 구현해야 함
  - `interface`를 구현함
  - 여러 개의 인터페이스를 구현 가능

## 1.1.5 Proxy 패턴과 proxy 서버

- 객체에 대해 어떤 동작이 수행되기 전 흐름을 가로채 특정 작업(보안, 검증, 캐싱, 로깅 등)을 수행함
- proxy 서버 : client가 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해주는 프로그램(nginx 등)
  - nginx를 Node.js에 적용시켜 실제 포트를 숨기고 정적 자원 압축, 로깅 등의 기능을 수행할 수 있음
  - CloudFlare도 프록시 서버로써 CDN, HTTPS 구축, DDOS 방어 등을 제공함

### CORS와 FE의 proxy 서버

- CORS(Cross-Origin Resource Sharing) : 리소스를 로드할 때 다른 origin을 통해 로드하지 못하게 하는 HTTP 헤더 기반 메커니즘
  - origin : protocol, host name, port를 합친 것(`http://localhost:8080`)
- FE에 프록시 서버를 두면 FE에서 나가는 요청을 프록시 서버를 통해 BE의 오리진으로 바꿔서 내보낼 수 있음

## 1.1.6 Iterator 패턴

- iterator를 사용하여 collection의 요소에 접근
  - 자료형의 구조와 상관없이 iterator interface로 순회 가능

## 1.1.7 노출모듈(Revealing module pattern) 패턴

```jsx
const pukuba = (() => {
  const a = 1;
  const b = () => 2;
  const public = {
    c: 2,
    d: () => 3,
  };
  return public;
})();
console.log(pukuba);
console.log(pukuba.a);
// { c: 2, d: [Function: d] }
// undefined
```

- JS에서는 모든 스코프가 global이기 때문에 위와 같이 public, private과 같은 접근 제어자를 구현하는데 함수를 사용하기도 함

## 1.1.8 MVC 패턴

![router-MVC-DB.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/router-MVC-DB.png)

- Model : 앱이 포함해야 할 데이터를 정의
- View : 데이터를 보여주는 방식을 정의
- Controller : 이벤트를 통해 모델, 뷰를 업데이트하는 로직을 포함
- 제어 흐름
  1. View에서 user event로부터 받은 input을 Controller로 보냄
  2. Controller에서 받은 input을 바탕으로 Model 갱신
  3. Model은 갱신된 내용을 Controller에게 알림
  4. Controller는 갱신된 내용을 바탕으로 View 갱신
- 유동적으로 Model에서 View로 바로 update하기도 하고 Controller에서 View로 바로 업데이트하기도 함
- React가 대표적인 예시임
  - 가상 DOm을 사용해서 DOM 조작을 추상화

## 1.1.9 MVP 패턴

- MVC의 Controller가 Presenter로 교체된 패턴
- View와 Presenter는 1:1 관계로, MVC보다 강하게 결합됨

## 1.1.10 MVVM 패턴

- MVC의 Controller가 View Model로 교체된 패턴
- VM : View를 더 추상화함
  - VM → View로의 알림은 그대로 있지만 View, VM 사이에 양방향 data binding, 커맨드가 지원됨
    - 커맨드 : 사용자의 요청을 나중에 이용할 수 있도록 객체로 캡슐화하여 처리함
      - command, receiver, invoker, client로 이루어짐
    - Data binding : VM을 변경하면 View에서도 변경됨(양방향)

# 1.2 프로그래밍 패러다임

- Programming paradigm : 프로그래밍의 관점을 갖게 해주는 개발 방법론(객체 지향 프로그래밍, 함수형 프로그래밍 등)
  - Imperative(명령형) : Procedural(절차지향형), Object-oriented(객체 지향형)
  - Declarative(선언형) : Functional(함수형), Reactive(반응형, 데이터 흐름, 변화의 전파로 결과가 선언되는 것)

## 1.2.1 선언형과 함수형 프로그래밍

- 선언형 프로그래밍 : 원하는 결과의 성질을 선언함
- 함수형 프로그래밍 : 원하는 결과가 연속된 함수의 값으로 선언됨
  - 순수 함수의 연속을 통해 로직을 구현
  - 고차 함수를 통해 재사용성을 높임
- 순수 함수 : 출력이 입력에만 의존하는 함수
- 고차 함수 : 순수 함수를 파라미터로 받거나 결과로 반환할 수 있는 순수 함수
  - 고차 함수를 지원하기 위해선 언어가 일급 객체(First-class object)를 가져야 함
    - 일급 객체는 파라미터로 넘기기, 변수에 할당, 다른 함수에서 객체 반환 등을 지원함
- 자연수 배열에서 최댓값 탐색
  ```jsx
  const list = [1, 2, 3, 4, 5, 11, 12];
  const ret = list.reduce((max, num) => (num > max ? num : max), 0);
  console.log(ret); // 12
  ```
  - `reduce()` : 배열을 받아서 결과를 반환하는 순수 함수

## 1.2.2 객체지향 프로그래밍

- OOP : 데이터가 객체 내에 캡슐화되고 구성 요소가 아닌 객체 자체가 운용되는 프로그래밍 방식
  - 설계에 많은 시간이 소요됨
  - 처리 속도가 다른 패러다임에 비해 느림
- 자연수 배열에서 최댓값 탐색
  ```jsx
  const ret = [1, 2, 3, 4, 5, 11, 12];
  class List {
    constructor(list) {
      this.list = list;
      this.mx = list.reduce((max, num) => (num > max ? num : max), 0);
    }
    getMax() {
      return this.mx;
    }
  }
  const a = new List(ret);
  console.log(a.getMax()); // 12
  ```

### OOP의 특징

- 캡슐화 : 자료형의 자료 표현과 자료형의 연산을 내부에 숨기는 것
  - 캡슐화한 것으로 접근 제어를 할 수 있고, 자료형의 정보를 은닉할 수 있음
- 추상화 : 복잡한 시스템을 단순화된 방식으로 표현하는 것
  - 추상 자료형 : 추상화를 통해 정의된 자료형
  - 인터페이스 등을 사용해서 구현 방법을 지정하지 않고 메소드를 정의하여 추상화할 수 있음
- 상속 : 새로운 클래스가 기존 클래스의 자료, 연산을 이용할 수 있게 하는 기능
  - 상속을 받은 하위 클래스를 프로그램의 요구에 맞춰 수정할 수 있음
  - 코드의 재사용, 계층 관계 생성 효과로 유지 보수성이 증가함
- 다형성 : 하나의 요소가 여러 방법으로 동작하는 것
  - 오버로딩 : 파라미터에 따라서 같은 이름의 메소드의 동작 방식을 다르게 구현할 수 있음
  - 오버라이딩 : 하위 클래스에서 같은 이름의 상위 클래스의 메소드를 재정의할 수 있음

### OO 설계 원칙(SOLID)

- SRP(Single Responsibility Principle) : 하나의 모듈(클래스)는 하나의 일만 책임질 수 있도록 분리되어야 함
- OCP(Open Closed Principle) : 변경에는 닫혀있고 확장에는 열려 있어야 함(변경할 일은 잘 없어야 하고 확장은 쉽게 가능해야 함)
- LSP(Liskov Substitution Principle) : 하위 클래스는 상위 클래스로 치환될 수 있어야 함
- DIP(Dependency Inversion Principle) : 상위 클래스(모듈)은 하위 클래스에 의존해선 안되고 모든 것들을 추상에 의존해야 함
- ISP(Interface Segregation Principle) : 하나의 일반적인 interface 보단 구체적인 여러 개의 interface를 만들어야 함

## 1.2.3 절차형 프로그래밍

- 로직이 수행해야 할 연속적인 instruction으로 구성됨
- 코드의 가독성이 좋고 실행 속도가 빠름
- 모듈화가 어렵고 유지 보수성이 떨어짐
- 자연수 배열에서 최댓값 탐색
  ```jsx
  const ret = [1, 2, 3, 4, 5, 11, 12];
  let a = 0;
  for (let i = 0; i < ret.length; i++) {
    a = Math.max(ret[i], a);
  }
  console.log(a); // 12
  ```
