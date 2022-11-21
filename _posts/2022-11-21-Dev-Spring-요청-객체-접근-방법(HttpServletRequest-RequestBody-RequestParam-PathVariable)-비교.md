---
layout: single
title: "Spring 응답 객체 접근 방법(`HttpServletRequest`, `@RequestBody`, `@RequestParam`, `@PathVariable`) 비교"
categories:
  - Dev
tags:
  - Spring
  - Request
  - HttpServletRequest
  - MultipartFile
  - MultipartHttpServletRequest
  - RequestBody
  - RequestParam
  - PathVariable
---

- `HttpServletRequest`
  - `req.getParameter("param")`으로 접근 가능
  - multipartfile을 받을 경우 `MultipartHttpServletRequest`로 받아야 하고, `req.getFile("filename")`으로 접근 가능
- `@RequestBody`
  - GET 요청은 하나의 문자열로 받을 수 있음
    분리해서 받을 수 있는지는 테스트해봐야 함
  - POST 요청은 객체를 지정하면 알아서 생성해줌(따로 파싱할 필요 없음)
    `public String inference(@RequesBody Dto_A dto_A)`
- `@RequestParam`
  - GET 요청은 변수 하나씩 분리해서 저장 가능
    `public String inference2(@RequestParam int age, @RequestParam String name)`
  - POST 요청은 파싱하지 못함
    - url 상에서 데이터를 찾기 때문에 `/create?age=19&name=asdf`와 같이 요청 uri에 추가해야 파싱 가능
- `@PathVariable`
  - `@RequestParam`과 마찬가지로 uri에서 데이터를 찾지만, 모든 요청 마다 하나씩만 사용 가능
  - 주로 post 요청에서 많이 사용함
    ```java
    @PostMapping("/create/{id}")
    public Dto_A create(@PathVariable("id") int id, ...)
    ```

### 정리

- POST로 JSON을 받을 때는 `@RequestBody`
- POST에서 uri에 전달되는 값을 받을 때는 `@PathVariable`
- GET으로 받을 때는 `@RequestParam`
- multipartfile을 받아야 할 경우 form으로 전송하고 `MultipartHttpServletRequest`로 받는게 가장 간단한 듯
