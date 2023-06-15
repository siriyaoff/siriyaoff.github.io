---
layout: single
title: "Java Creating a stream"
categories:
  - Dev
tags:
  - Java
  - Stream
  - IntStream
  - Stream.of
  - Arrays.stream
  - Array
  - chars
  - boxed
  - Collection
  - IntStream
  - Builder
  - limit
  - generate
  - iterate
  - Regex
  - Pattern.compile
  - splitAsStream
  - HttpResponse
  - Iterator
  - Spliterator
---

- `Array`, `Collection`을 함수형으로 처리하는 기능을 제공
- 병렬 처리 가능
- Creating - Intermediate operation - Terminal operation 순서로 데이터를 처리함

## Creating a Stream

### Empty Stream 생성

```java
Stream<String> empty = Stream.empty();
List<String> strings = empty.collect(Collectors.toList());
// strings = []
```

### varargs, `Array`로부터 생성

```java
Stream<Integer> intStream = Stream.of(1, 2, 3);
List<Integer> ints = intStream.collect(Collectors.toList());
// ints = [1, 2, 3]
String[] strAr = {"a", "b", "c"};
Stream<String> stringStream = Arrays.stream(strAr, 0, 2);
```

- `int[]` → `Stream<Integer>` 변환하려면 뒤에 `.boxed()` 붙여야 함
- `Collection`에서는 `.stream()` 메소드 사용하면 됨
- `Stream.concat(stream1, stream2)`와 같이 스트림을 연결할 수도 있음

### `String`으로부터 생성

```java
String sentence = "Hello Duke";
List<String> letters =
        sentence.chars()
                .mapToObj(Character::toString)
                .collect(Collectors.toList());
// letters = [H, e, l, l, o,  , D, u, k, e]
```

- `String.chars()`는 반환형이 `IntStream`이지만, ASCII로 변환해주기 때문에 가능함
- SE 10에서는 `.mapToObj(c -> (char)c).map(Object::toString)`으로 써야 함

### `IntStream` 활용

```java
/**
 * Creating a Stream from a Range of Numbers
 */
String[] letters = {"A", "B", "C", "D"};
List<String> listLetters =
        IntStream.range(0, 10)
                .mapToObj(index -> letters[index % letters.length])
                .collect(Collectors.toList());
// listLetters = [A, B, C, D, A, B, C, D, A, B]

/**
 * Creating a Stream of Random Numbers
 */
Random random = new Random(314L);
List<Integer> randomInts =
        random.ints(10, 1, 5)
                .boxed()
                .collect(Collectors.toList());
// randomInts = [4, 4, 3, 1, 1, 1, 2, 2, 4, 2]
```

- `IntStream.rangeClosed(0, 10)`도 존재함
- `random.ints()`, `random.longs()`, `random.doubles()`은 각각 `IntStream`, `LongStream`, `DoubleStream`을 반환함

### `Builder` 활용

```java
Stream.Builder<String> builder = Stream.<String>builder();
builder.add("one")
        .add("two")
        .add("three")
        .add("four");
Stream<String> stream = builder.build();
```

- 한 줄로 chaining 가능

### `generate`, `iterate` 활용

```java
/**
 * Creating a Stream from a Supplier
 */
Stream<String> generated = Stream.generate(() -> "+");
List<String> strings =
        generated
                .limit(10L)
                .collect(Collectors.toList());
// strings = [+, +, +, +, +, +, +, +, +, +]

/**
 * Creating a Stream from a UnaryOperator and a Seed
 */
Stream<String> iterated = Stream.iterate("+", s -> s + "+");
iterated.limit(5L).forEach(System.out::println);
Stream<String> iterated2 = Stream.iterate("+", s -> s.length() <= 5, s -> s + "+");
iterated.forEach(System.out::println);
// +
// ++
// +++
// ++++
// +++++
```

- `generate()` : 각 원소별로 새로 람다를 실행함
- `iterate()` : 초기값을 가지고 람다를 매번 실행하고 값이 누적됨

### regex 활용

```java
String sentence = "For there is good news yet to hear and fine things to be seen";

Pattern pattern = Pattern.compile(" ");
Stream<String> stream = pattern.splitAsStream(sentence);
List<String> words = stream.collect(Collectors.toList());
// words = [For, there, is, good, news, yet, to, hear, and, fine, things, to, be, seen]
```

### 파일에서부터 생성

```java
Path log = Path.of("/tmp/debug.log"); // adjust to fit your installation
try (Stream<String> lines = Files.lines(log)) {

    long warnings =
            lines.filter(line -> line.contains("WARNING"))
                    .count();
    System.out.println("Number of warnings = " + warnings);

} catch (IOException e) {
    // do something with the exception
}
```

### `HttpResponse` 활용

```java
/**
 * Creating a Stream on an HTTP Source
 */
// The URI of the file
URI uri = URI.create("https://www.gutenberg.org/files/98/98-0.txt");

// The code to open create an HTTP request
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder(uri).build();

// The sending of the request
HttpResponse<Stream<String>> response = client.send(request, HttpResponse.BodyHandlers.ofLines());
List<String> lines;
try (Stream<String> stream = response.body()) {
    lines = stream
            .dropWhile(line -> !line.equals("A TALE OF TWO CITIES"))
            .takeWhile(line -> !line.equals("*** END OF THE PROJECT GUTENBERG EBOOK A TALE OF TWO CITIES ***"))
            .collect(Collectors.toList());
}
System.out.println("# lines = " + lines.size());
// # lines = 15904
```

### `Collection`, `Iterator` 활용
