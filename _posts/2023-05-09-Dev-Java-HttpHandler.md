---
layout: single
title: "HttpHandler in Java"
categories:
  - Dev
tags:
  - Java
  - HttpHandler
  - Http header
  - OutputStream
---

```java
public class sumcontroller implements HttpHandler {
    @Override
    public void handle(HttpExchange t) throws IOException {
        String filePath="/Users/yanghyowon/Desktop/gjtest/data/input/user.json";
        ObjectMapper objectMapper=new ObjectMapper();
        int sum=0;
        User[] users = objectMapper.readValue(new File(filePath), User[].class);

        for (User user : users) {
            System.out.println(user.getPost_count());
            sum += user.getPost_count();
        }
        String response="{\n\t\"sum\": \""+sum+"\"\n}";
        t.getResponseHeaders().add("Content-Type", "application/json");
        t.sendResponseHeaders(200, response.getBytes().length);
        OutputStream os = t.getResponseBody();
        os.write(response.getBytes());
        os.close();
    }
}
```

- `HttpHandler`도 추상 클래스이기 때문에 controller를 구현체 클래스로 생성해서 넣어줘야 함
- `t.getResponseHeaders()` : HTTP response header가 저장되는 mutable Map 리턴
  - Map 내부의 key들은 case-insensitive
  - value가 여러 개일 경우 `List<String>`으로 들어가야 함
- `t.sendResponseHeaders(int rCode, long responseLength)` : 현재 response header, `rCode`를 사용해서 response를 보내기 시작함
  - `responseLength`>0 : `responseLength` byte 길이의 내용이 `t.getResponsebody()`에 쓰여야 함
  - `responseLength`=0 : chunked encoding 사용
  - `responseLength`<0 : response body가 없음
  - response header 중 content-length 헤더가 선언되지 않았다면, 이 메소드에서 `responseLength`를 사용해서 적절하게 설정해줌
- `t.getResponseBody()` : response body가 들어갈 `OutputStream` 리턴
  - 호출된 OutputStream을 종료함으로써 response body를 끝냄
  - `t.getResponseHeaders()`, `t.sendResponseHeaders()`가 먼저 호출되어야 함
  - `t.getRequestBody()`가 닫히지 않았을 때 response body stream을 종료하면 request body stream도 implicitly 닫힘
