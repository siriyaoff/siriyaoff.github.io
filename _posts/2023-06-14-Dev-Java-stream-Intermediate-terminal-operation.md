---
layout: single
title: "Java stream Intermediate, terminal operation"
categories:
  - Dev
tags:
  - Java
  - Stream
  - Intermediate operation
  - sorted
  - peek
  - Comparator
  - map
  - flatMap
  - flattening
  - mapping
  - Functional interface
  - Consumer
  - Supplier
  - Function
  - Accumulator
  - Predicate
  - filter
  - Terminal operation
  - count
  - min
  - max
  - sum
  - average
  - ifPresent
  - Optional
  - reduce
  - forEach
  - collect
  - Collectors
  - anyMatch
  - allMatch
  - noneMatch
  - findFirst
  - findAny
  - parallelStream
  - forEachOrdered
---

## Intermediate operation

```java
String[] strAr = {"abc", "cde", "awef", "ccd"};
Stream<String> stringStream = Arrays.stream(strAr)
        .sorted(Comparator.comparingInt(String::length).reversed())
        .peek(System.out::println)
        .map(String::toUpperCase)
        .map(x->{
            System.out.println(x);
            return x;
        })
        .filter(x -> x.contains("C"));
List<String> ls = stringStream.collect(Collectors.toList());
System.out.println("ls = " + ls);
```

- 실행 결과
  ```
  awef
  AWEF
  abc
  ABC
  cde
  CDE
  ccd
  CCD
  ls = [ABC, CDE, CCD]
  ```
- `Stream<T> sorted([Comparator<? super T> comparator])` : 정렬
  - `Comparator.reverseOrder()` : 내림차순
  - `Comparator.comparingInt(String::length))` : 글자수 기준 오름차순
    - 내림차순으로 하려면 `reversed()`를 체이닝하면 됨
    - 위 결과에서는 단순히 글자수로 정렬했기 때문에 `reversed()`를 사용해도 이전의 순서가 유지되는 것처럼 보임
    - 정렬 기준이 복잡해지면 stable하게 바뀌는지는 모름
  - 람다로도 가능 : `(s1, s2) -> s2.length() - s1.length()`
    - 연산 결과가 `true`인 경우 두 원소의 순서 변경
- `Stream<T> peek(Consumer<? super T> action)` : stream 원소 별로 `action` 연산 수행
  - consumer가 아닌 연산이 들어와서 연산 결과가 리턴되어도 원소에 영향을 미치지 않음
- `<R> Stream<R> map(Function<? super T, ? extends R> mapper)` : stream에 `mapper` 매핑
  - 원소 값이 리턴되는 연산 결과가 원소에 대입됨
  - `<R> Stream<R>` 타입이 뭔지는 모르겠음
  - `flatMap()` : 리턴 값에서 중첩 구조를 한 단계 제거하고 붙여줌
    - 객체를 flattening 할 때도 사용 가능
      ```java
      // obj1 : String s1, s2, s3을 필드로 가짐
      obj1[] objs = new obj1[3];
      objs[0] = new obj1("a1", "b1", "c1");
      objs[1] = new obj1("a2", "b2", "c2");
      objs[2] = new obj1("a3", "b3", "c3");
      List<String> ls = Arrays.stream(objs)
              .map(o -> Stream.of(o.getS1(),
                      o.getS2(),
                      o.getS3())
                      .reduce("", (a, b) -> a + b))
              .collect(Collectors.toList());
      String as1 = Arrays.stream(objs)
              .map(o -> Stream.of(o.getS1(),
                      o.getS2(),
                      o.getS3())
                      .reduce("", (a, b) -> a + b))
              .reduce("", (a, b) -> a + b);
      String as2 = Arrays.stream(objs)
              .flatMap(o -> Stream.of(
                      o.getS1(),
                      o.getS2(),
                      o.getS3()))
              .reduce("", (a, b) -> a + b);
      ```
      - 실행 결과
        ```java
        ls = [a1b1c1, a2b2c2, a3b3c3]
        as1 = a1b1c1a2b2c2a3b3c3
        as2 = a1b1c1a2b2c2a3b3c3
        ```
      - 위 코드에서는 필드가 `String`이라 동일한 결과를 만들어낼 수 있지만, 객체 별로 필드 개수가 다른 상황에서 전체 필드들의 평균을 구하는 등의 상황에서는 `.average()`를 `.reduce()` 대신 사용하면 모든 필드들의 평균이 제대로 구해지지 않기 때문에 `flatMapToInt()`를 사용해서 flattened stream으로 만든 다음 `.average()`를 사용해야 함
- `Stream<T> filter(Predicate<? super T> predicate)` : `predicate` 연산의 결과가 `true`인 원소만 보존

## Terminal operation

- `count()`, `min()`, `max()`, `sum()`, `average()`
  - stream이 비어 있는 경우를 대비하여 `min()`, `max()`, `average()`는 `OptionalInt` 타입으로 리턴함
    - `ifPresent(System.out::println)` 등의 메소드로 chaining 가능
- `reduce()`

  - 3가지 오버로딩이 있음

  ```java
  Optional<T> reduce(BinaryOperator<T> accumulator);

  T reduce(T identity, BinaryOperator<T> accumulator);

  <U> U reduce(U identity,
  	BiFunction<U, ? suer T, U> accumulator,
  	BinaryOperator<U> combiner);
  ```

  - `accumulator` : 각 원소들을 누적하는 연산
  - `identity` : 초기값
  - `combiner` : 병렬 처리할 때 합치는 연산

- `forEach()`
  - intermediate operation 중 `peek()`와 유사함
- `<R, A> R collect(Collector<? super T, A, R> collector)`
  - `Collectors.toList()` : 리스트로 반환
  - `Collectors.joining()` : 결과를 하나의 `String`으로 반환(delimiter, prefix, suffix 파라미터로 넘길 수 있음)
  - `Collectors.averageInt()`, `Collectors.summingInt()`, `Collectors.summarizingInt()`
    - `summarizingInt()` : `IntSummaryStatistics` 타입 반환(count, sum, min, average, max를 담은 객체)
  - `Collectors.groupingBy()` : functional interface Function을 파라미터로 받아 `Map` 타입으로 반환
    - key : Function의 결과값
    - value : 피연산 객체들 리스트
  - `Collectors.partitioningBy()` : functional interface Predicate를 파라미터로 받아 `Map` 타입으로 반환(`groupingBy()`와 동일)
  - `public static<T,A,R,RR> Collector<T,A,RR> collectingAndThen(Collector<T,A,R> downstream, Function<R,RR> finisher)` : `downstream`을 수행한 다음 `finisher`를 수행
  - 직접 만들 수도 있음
    ```java
    public static<T, A, R> Collector<T, A, R> of(Supplier<A> supplier,
                  BiConsumer<A, T> accumulator,
                  BinaryOperator<A> combiner,
                  Function<A, R> finisher,
                  Characteristics... characteristics)
    ```

### short-circuiting terminal operation

```java
boolean anyMatch(Predicate<? super T> predicate);

boolean allMatch(Predicate<? super T> predicate);

boolean noneMatch(Predicate<? super T> predicate);

Optional<T> findFirst();

Optional<T> findAny();
```

- `boolean anyMatch(Predicate<? super T> predicate)` : 조건을 충족하는 원소가 하나 이상일 경우 `true`
- `noneMatch()` : 조건을 충족하는 원소가 0개일 경우 `true`
- `allMatch()` : 모든 원소가 조건을 충족하는 경우 `true`

## `parallelStream` 관련

- `parallelStream`에서는 원소가 등록된 순서대로 intermediate operation이 진행된다는 보장이 없음(대부분 랜덤하게 진행됨)
  - `sorted()`, `limit()`, `reduce()` 등의 다른 원소들의 영향을 받는 연산을 `parallelStream`에서 실행하면 parallelism의 이점이 줄어듬
- `parallelStream`에서 `reduce()`를 실행하면 쓰레드의 개수만큼 `reduce()`가 실행되기 때문에 `identity` 파라미터를 사용할 때 주의해야 함(누적 합일 경우 identity가 쓰레드의 개수만큼 더해짐)
- terminal operation으로 `forEach()`를 사용하면 병렬 처리되는 순서대로 실행되기 때문에, encounter order로 실행하기 위해선 `forEachOrdered()`를 사용해야 함
- `collector()` 또한 `reduce()`와 같이 combiner가 있는 오버로딩이 존재함
  ```java
  <R> R collect(Supplier<R> supplier,
            BiConsumer<R, ? super T> accumulator,
            BiConsumer<R, R> combiner);
  ```
- `collect(Collectors.toList())`는 encounter order로 실행됨
