---
layout: single
title: "Java stream optimization"
categories:
  - Dev
tags:
  - Java
  - Stream
  - Terminal operation
  - Intermediate operation
  - skip
  - filter
  - distinct
  - findFirst
  - findAny
  - Lazy invocation
  - Null-safe stream
  - Null Dereference
  - empty Collection
  - null Collection
  - Optional
  - Optional.ofNullable
  - map
  - orElseGet
---

## Stream optimization

- stream을 chaining으로 처리할 때 원소 하나씩에 대해 vertical하게 연산들이 수행됨
  - 1 intermediate → 1 terminal → 2 intermediate → 2 terminal → …
    - `IntStream.*of*(1, 2, 3).peek(System.*out*::print).forEach(System.*out*::print)`의 실행 결과 : `112233`
- intermediate operation 중 `skip()`, `filter()`, `distinct()` 등의 operation을 먼저 배치해서 offline query 최적화처럼 실행 시간을 줄일 수 있음
- terminal operation 중에서도 `findFirst()`, `findAny()`와 같은 operation은 한 원소만 찾아서 리턴하기 때문에 성능을 높일 수 있음
- terminal operation이 이루어진 stream은 재사용할 수 없음
  - terminal oepration을 수행한 stream을 재사용하는 경우 syntax error로 잡히지 않기 때문에 주의해야 함

### Lazy invocation

- Lazy invocation : stream의 intermediate operation은 terminal operation이 호출되는 시점에 terminal operation 보다 먼저 실행됨
- terminal operation이 호출되지 않으면 intermediate operation도 실행되지 않음

```java
IntStream.of(1, 2, 3)
        .filter(x->{
            System.out.println("filter invoked");
            return x>2;
        });
```

- 위 코드는 terminal operation이 실행되지 않았기 때문에 코드를 실행해도 `"filter invoked"`는 출력되지 않음
- `@PreDestroy` 어노테이션이 빈 제거 이전에 호출되는 것과 비슷함

## Null-safe stream 생성

### Prevent Null Dereferences

```java
Stream<String> collectionAsStream(Collection<String> collection) {
    return collection == null
      ? Stream.empty()
      : collection.stream();
}
```

- SE 8부터는 `null`을 값이 없는 데이터 구조를 나타내기 위한 표현으로 쓰는 것이 권장되지 않음
- empty collection과 null collection은 각각 다른 상황을 의미함
  - empty collection : query doesn’t have results or elements to show
  - null collection : error happened during the process

### `Optional` 사용

```java
public <T> Stream<T> collectionToStream(Collection<T> collection) {
    return Optional.ofNullable(collection)
      .map(Collection::stream)
      .orElseGet(Stream::empty);
}
```

- `Optional.ofNullable(collection)` : null collection일 경우 EMPTY `Optional` 객체가 생성됨
- `map(Collection::stream)` : 생성된 `Optional` 객체의 값들을 사용하여 `Stream` 생성
- `orElseGet(Stream::empty)` : 생성된 `Optional` 객체가 EMPTY 일 경우 empty stream 객체를 생성
- 생성할 때와 마찬가지로 intermediate operation에서도 함수 정의를 확인하고 연산이 `Optional`을 리턴하지 않는 경우 NPE를 잘 처리해야 함
