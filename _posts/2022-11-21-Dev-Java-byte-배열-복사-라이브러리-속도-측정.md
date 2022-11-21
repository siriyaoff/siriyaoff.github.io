---
layout: single
title: "Java byte 배열 복사 관련 라이브러리 속도 측정"
categories:
  - Dev
tags:
  - Spring
  - Java
  - byte[]
  - Files
  - FileUtils
  - FileOutputStream
---

```java
MultipartFile psd = new CommonsMultipartFile(fileItem);

byte[] b=psd.getBytes();
String psdb = Arrays.toString(b);

File of = new File("C:\\Users\\yang\\Downloads\\returniris.png");
if(file.exists()){
    System.out.println("delete existing one");
    file.delete();
}
file.createNewFile();
FileWriter fw = new FileWriter(of);
BufferedWriter writer = new BufferedWriter(fw);
writer.write(psdb);
writer.close();

// 1
long st1= System.currentTimeMillis(), et1;
Path path1=Paths.get("C:\\Users\\yang\\Downloads\\r1.png");
File f1 = new File(path1.toString());
// if (f1.exists()) { // (a)
//     System.out.println("delete file1");
//     f1.delete();
// }
Files.write(path1, b);
et1 = System.currentTimeMillis();
System.out.println(et1 - st1);

// 2
long st2=System.currentTimeMillis(), et2;
try {
    FileOutputStream fos = new FileOutputStream("C:\\Users\\yang\\Downloads\\r2.png");
    fos.write(b);
} catch (Exception e) {
    System.out.println("error on 2");
}
et2 = System.currentTimeMillis();
System.out.println(et2 - st2);

// 3
long st3=System.currentTimeMillis(), et3;
FileUtils.writeByteArrayToFile(new File("C:\\Users\\yang\\Downloads\\r3.png"), b);
et3 = System.currentTimeMillis();
System.out.println(et3 - st3);
```

- `MultipartFile psd`를 `byte[] b`로 변환 후 복사하는 방법들의 속도를 측정했다.
  - 1 번째 방법 : `java.nio.file.Files`의 `Files.write()` 사용
  - 2 번째 방법 : `java.io`의 `FileOutputStream` 사용
  - 3 번째 방법 : `apache.commons.io`의 `FileUtils.writeByteArrayToFile()` 사용
- 각각의 방법을 10번씩 반복해서 실행 시간의 평균을 구했다.

### 실행 결과

```
// test case 1
9
4
7
-----------------
7
15
4
-----------------
7
4
5
-----------------
7
4
5
-----------------
7
15
5
-----------------
8
5
5
-----------------
7
5
4
-----------------
7
5
5
-----------------
6
7
10
-----------------
6
15
5

// test case 2
8
5
5
-----------------
9
7
5
-----------------
8
5
6
-----------------
14
4
5
-----------------
9
5
4
-----------------
11
6
6
-----------------
12
4
5
-----------------
8
5
4
-----------------
16
8
5
-----------------
12
4
5
-----------------
8
5
5
```

- test case 1 : 기존 파일을 삭제하고 반복
  - average
    - 1 번째 방법 : 7.1ms
    - 2 번째 방법 : 7.9ms
    - 3 번째 방법 : 5.5ms
- test case 2 : 기존 파일이 존재하는 채로 반복
  - average
    - 1 번째 방법 : 11.5ms
    - 2 번째 방법 : 5.8ms
    - 3 번째 방법 : 5.5ms
- 기본 제공 라이브러리보다는 `apache.commons.io`에서 제공하는 함수가 더 효율적인 것 같다.

### 복사가 제대로 되지 않는 경우

```java
File of = new File("C:\\Users\\yang\\Downloads\\returniris.png");
if(file.exists()){
    System.out.println("delete existing one");
    file.delete();
}
file.createNewFile();
FileWriter fw = new FileWriter(of);
BufferedWriter writer = new BufferedWriter(fw);
writer.write(psdb);
writer.close();
```

- `FileWriter`, `BufferedWriter`는 문자 기반 스트림이기 때문에 `byte[] b`를 바로 넣을 수 없음
- 실행 결과  
  ![byte copy](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/_posts/img/java-byte-copy.png)
  - `serveriris.png`가 원본 파일, `returniris.png`가 위 코드를 실행한 결과물
  - 복사가 제대로 되지 않고 크기가 4.64배 정도로 차이가 나는 이유
    1. `psdb`는 `Arrays.toString(b)`로 만들어진 문자열임
    2. 이때 각 byte 원소가 문자열로 변환되면서 1~3 자리수의 숫자가 문자열로 변환됨
    3. 문자 기반 스트림으로 입력했기 때문에 각 byte가 최대 3개의 char로 변환됨(1 B → 3 \* 2 B)
    4. `b`의 내용에 따라 파일의 크기가 1~8배로 차이나게 됨
- 입출력 스트림 공부
  [https://joont92.github.io/java/입출력/](https://joont92.github.io/java/%EC%9E%85%EC%B6%9C%EB%A0%A5/)
