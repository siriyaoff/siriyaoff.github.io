---
layout: single
title: "HTTP note"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - HTTP
  - TCP/IP
  - IP
  - URL
  - Stateless
  - HTTP message
  - HTTP header
  - Content-Type
  - Content-Encoding
  - Content-Language
  - Content-Length
  - Content-Range
  - Transfer-Encoding
  - Accept
  - Accept-Encoding
  - Accept-Language
  - From
  - Referer
  - User-Agent
  - Server
  - Date
  - Host
  - Location
  - Allow
  - Retry-After
  - Authorization
  - WWW-Authenticate
  - Set-Cookie
  - Cache-Control
  - Pragma
  - Expires
  - Age
  - Last-Modified
  - ETag
  - If-Modified-Since
  - If-None-Match
  - Content negotiation
  - CDN
  - GET
  - POST
  - PUT
  - PATCH
  - DELETE
  - HEAD
  - OPTIONS
  - Idempotency
  - HTTP Status code
  - Redirection
  - Permanent Redirection
  - Temporary Redirection
  - PRG
---

# TCP/IP 스택

![HTTP-TCP-IP-Stack.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-TCP-IP-Stack.png)

- IP(Internet Procotol)
  - 지정한 IP 주소에 packet 단위로 데이터 전달
  - 한계
    - 비연결성 : 대상이 없어도 패킷 전송
    - 비신뢰성 : 패킷이 사라지거나 순서대로 오지 않을 수 있음
    - 프로그램 구분 : 같은 IP로 통신하는 앱이 둘 이상일 수 있음
- TCP/UDP
  - TCP : HTTP/1.1, 2
    - 현재는 HTTP/1.1 주로 사용
  - UDP : HTTP/3
    - IP와 거의 비슷하지만, 포트, 체크섬이 추가됨
    - 앱에서 추가 처리 필요
- 프로토콜 분석  
  ![HTTP-Protocol.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Protocol.png)
- TCP/IP 패킷  
  ![HTTP-TCP-IP-Packet.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-TCP-IP-Packet.png)
  - 연결지향(3-way handshake)
  - 데이터 전달 보증
  - 순서 보장
  - 3-way handshake : SYN → SYN+ACK → ACK
    - 마지막 ACK와 함께 데이터 전송 가능
  - 포트
    - 20, 21 : FTP
    - 22 : ssh
    - 23 : telnet
    - 80 : http, 443 : https

# URI(Uniform Resource Identifier)

![HTTP-URI.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-URI.png)

- URI는 Locator, Name 또는 둘 다 추가로 분류할 수 있음
  - URL을 주로 사용(이후 URI, URL을 같은 의미로 사용함)

![HTTP-URL-URN.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-URL-URN.png)

- URL : 리소스가 있는 위치 지정
  - `scheme://[userinfo@]host[:port][/path][?query][#fragment]`
    - scheme : 주로 프로토콜 사용
      - http, https를 사용하면 포트 생략 가능
    - userinfo : URL에 사용자정보를 포함해서 인증
    - host : 도메인 또는 IP
    - path : 리소스 경로(계층적 구조)
    - query : key=value 형태
      - ?로 시작, &로 추가 가능
    - fragment : html 내부 북마크 등에 사용
- URN : 리소스에 이름 부여
  - `urn:isbn:83728347293`
  - URN 만으로 리소스를 찾는 방법이 보편화 되지 않음

# HTTP 특징

- 클라이언트-서버 구조
  - Request-Response 구조
  - 클라이언트 : 서버에 요청 보내고 응답 대기
    - UI/UX 집중
  - 서버 : 요청에 대한 결과를 만들어서 응답
    - 서버 아키텍처, 대용량 트래픽 처리 등 집중
- 무상태(Stateless) : 서버가 client의 이전 상태를 보존하지 않음
  - 중간에 응답 받는 서버가 바뀌어도 됨 → scale out, 장애 대응에 유리
  - Stateful : 중간에 응답 받는 서버가 바뀌면 안됨
- 비연결성
  - HTTP는 기본적으로 연결을 유지하지 않음
    - 연결할 때마다 3-way handshake, 자원 새로 다운로드
    - HTTP 지속 연결(Persistent Connections)로 해결
      - n초 동안 유지 등의 설정 가능

## 웹 브라우저 요청 흐름

1. 웹 브라우저에 도메인 입력
2. 웹 브라우저가 DNS 조회, HTTP 요청 메시지 생성

   ![HTTP-Web-Browser-Request.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Web-Browser-Request.png)

3. OS에서 TCP/IP 패킷 생성
4. 네트워크 인터페이스를 통해 요청 패킷 전달
5. 응답 서버에서 HTTP 메시지 해석 후 응답 메시지 생성
6. 응답 패킷 도착하고 웹 브라우저가 HTML 렌더링

# HTTP 메시지

- 메시지 구조  
  ![HTTP-Message.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Message.png)
  - start-line : request-line 또는 status-line
  - header : `field-name ":" OWS field-value OWS`
    - `OWS` : 띄어쓰기 허용
    - `field-name` : 대소문자 구분하지 않음
    - `field-value` : 대소문자 구분
    - 표준 헤더는 매우 많음(message body 빼고 필요한 모든 정보가 다 있음)
    - 임의의 헤더 추가 가능
  - message body : 실제 전송할 데이터
    - byte로 표현할 수 있는 모든 데이터

## 요청 메시지

![HTTP-Request-Message.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Request-Message.png)

- start-line : request-line
  - `method SP request-target SP HTTP-version CRLF`
    - `SP` : space
    - `request-target` : 절대 경로, 전체 주소 등

## 응답 메시지

![HTTP-Response-Message.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Response-Message.png)

- start-line : status-line
  - `HTTP-version SP status-code SP reason-phrase CRLF`
    - 200 : 성공
    - 400 : 클라이언트 요청 오류
    - 500 : 서버 내부 오류

## HTTP 헤더

- HTTP 전송에 필요한 모든 부가정보가 들어감
  - 압축, 인증, 클라이언트, 서버 정보, 메시지 바디의 내용, 크기, …
- RFC2616 : 과거 헤더 분류
- RC7230 - 7235 : Entity → Representation으로 변화  
  ![HTTP-Header.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Header.png)
  - Representation : Representation Metadata + Representation Data
  - message body : payload라고도 함

### 자주 사용하는 헤더들

| 분류           | 헤더                | 기능                                                                         | 예시                                                                              | 부가 설명                                                                    |
| -------------- | ------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Representation | `Content-Type`      | 표현 데이터의 type                                                           | `text/html;charset=UTF-8`                                                         | Representation 헤더들은 요청, 응답 모두에서 사용 가능                        |
|                | `Content-Encoding`  | 표현 데이터의 압축 방식                                                      | `gzip`                                                                            |                                                                              |
|                | `Content-Language`  | 표현 데이터의 자연 언어                                                      | `de-DE, en-CA`                                                                    |                                                                              |
|                | `Content-Length`    | 표현 데이터의 길이(Byte 단위)                                                | `123`                                                                             |                                                                              |
|                | `Content-Range`     | 전송하는 표현 데이터의 부분 정보                                             | `bytes 1001-2000 / 2000`                                                          |                                                                              |
|                | `Transfer-Encoding` |                                                                              | `chunked`                                                                         |                                                                              |
| 협상           | `Accept`            | client가 선호하는 media type                                                 | `text/*, text/plain, text/plain;format=flowed, */*`                               |                                                                              |
|                | `Accept-Charset`    | client가 선호하는 charset                                                    | 사용 x                                                                            | 현재 모든 브라우저에서 지원 x                                                |
|                | `Accept-Encoding`   | client가 선호하는 encoding                                                   | `gzip`                                                                            |                                                                              |
|                | `Accept-Language`   | client가 선호하는 자연 언어                                                  | `ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7`                                             | 협상 헤더들은 요청시에 사용                                                  |
| 일반 정보      | `From`              | user agent의 이메일 정보                                                     | `hwy16016@gmail.com`                                                              | 요청시에 사용(잘 사용하지 않음)                                              |
|                | `Referer`           | 이전 웹 페이지의 주소                                                        | `www.google.com`                                                                  | 요청시에 사용<br>유입 경로 분석할 때 사용                                    |
|                | `User-Agent`        | user agent의 앱(웹 브라우저 등) 정보                                         | `Mozilla/5.0 (platform; rv:geckoversion) Gecko/geckotrail Firefox/firefoxversion` | 요청시에 사용<br>어떤 종류의 브라우저에서 장애가 발생하는지 분석             |
|                | `Server`            | 요청을 처리하는 ORIGIN 서버의 SW 정보                                        | `Apache/2.2.22 (Debian)`                                                          | 응답시에 사용                                                                |
|                | `Date`              | 메시지가 발생한 날짜, 시간                                                   | `Mon, 1 Jan 2017 00:00:00 GMT`                                                    | 응답시에 사용                                                                |
| 요청 관련 정보 | `Host`              | 요청한 host 정보                                                             | `developer.mozilla.org`                                                           | 요청에서 사용(필수)<br>하나의 서버, IP에 여러 도메인이 적용되어 있을 때 사용 |
|                | `Location`          | 페이지 리다이렉션 주소                                                       | `/index.html`                                                                     | 201에서 Location 헤더의 값은 요청에 의해 생성된 리소스의 URI임               |
|                | `Allow`             | 허용 가능한 HTTP 메서드                                                      | `GET, HEAD, PUT`                                                                  | 405 (Method Not Allowed) 응답에 포함해야 함                                  |
|                | `Retry-After`       | user agent가 다음 요청을 하기까지 기다려야 하는 시간(초 단위)                | `Mon, 1 Jan 2017 00:00:00 GMT`<br>`120`                                           | 날짜는 주로 503 (Service Unavailable) 응답에 포함됨                          |
| 인증           | `Authorization`     | client 인증 정보를 전달                                                      | `Basic YWxhZGRpbjpvcGVuc2VzYW1l`                                                  |                                                                              |
|                | `WWW-Authenticate`  | 리소스 접근시 필요한 인증 방법 정의                                          | `Newauth realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"`   | 401 (Unauthorized) 응답에 포함해야 함                                        |
| 쿠키           | `Set-Cookie`        | 아래 참고                                                                    | 아래 참고                                                                         | 응답시에 사용                                                                |
| 캐시           | `Cache-Control`     | 아래 참고                                                                    | 아래 참고                                                                         |                                                                              |
|                | `Pragma`            | HTTP 1.0 하위 호환의 캐시 무효화를 위해 사용                                 | `no-cache`                                                                        |                                                                              |
|                | `Expires`           | 캐시 만료 시간                                                               | `Mon, 01 Jan 1900 00:00:00 GMT`                                                   | HTTP 1.0부터 사용되지만, Cache-Control: max-age와 함께 사용되면 무시됨       |
| 프록시 캐시    | `Age`               | orgin 서버에서 응답 후 프록시 캐시에 머문 시간                               | `Age: 60`                                                                         |                                                                              |
| 검증           | `Last-Modified`     | 리소스의 마지막 수정 시간 정보                                               | `Mon, 01 Jan 1900 00:00:00 GMT`                                                   | 1초 미만 단위로 캐시 조정 불가능                                             |
|                | `ETag`              | Entity Tag(캐시 유효성 검증용 해시)                                          | `"33ad42a148795d9f25f89d4"`                                                       |                                                                              |
| 조건부 요청    | `If-Modified-Since` | 값으로 들어간 날짜보다 서버의 Last-Modified가 최신이면 다시 요청(아니면 304) | `If-Unmodified-Since`                                                             | 요청시에 사용                                                                |
|                | `If-None-Match`     | 값으로 들어간 해시가 서버의 ETag와 다르면 다시 요청(아니면 304)              | `If-Match`                                                                        | 요청시에 사용                                                                |

### 협상(Content negotiation)

- quality value q가 클수록 우선순위가 높음
  - 생략하면 1
  - `Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7` : `,`로 구분
    - `ko-KR,` : `ko-KR;q=1,`
- 구체적일수록 우선순위가 높음
  - `Accept: text/*, text/plain, text/plain;format=flowed, */*`
    - 매칭되는 media type 중 우선순위가 가장 높은 것이 선택됨

### 전송 방식

- 단순 전송 : `Content-Length` 지정
  - Content length를 알 수 있을 때 사용
- 압축 전송 : `Content-Encoding`, `Content-Length` 지정
- 분할 전송 : `Transfer-Encoding` 지정
  - `chunked` : chunk 단위로 전송
    - `Content-Length`를 넣으면 안됨
    - 마지막 청크는 아래와 같음
      ```jsx
      0
      \r\n
      ```
- 범위 전송 : `Content-Range` 지정

### `Set-Cookie`

- 쿠키 정보는 요청보낼 때 자동으로 포함됨
  ```jsx
  GET /welcome HTTP/1.1
  Cookie: user=홍길동
  ```
  - 최소한의 정보만 사용해야 함
  - 서버에 전송하지 않고, 웹 브라우저 내부에 남기려면 웹 스토리지(로컬, 세션 스토리지) 사용
- `Set-Cookie: sessionId=ab123; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure`
  - `expires` : 만료일이 되면 쿠키 삭제(영속 쿠키)
    - 생략하면 브라우저 종료시까지만 유지됨(세션 쿠키)
  - `max-age` : 초 단위
  - `domain` : 명시한 도메인 + 서브도메인을 포함한 경로에서 쿠키 접근 가능
    - 생략하면 현재 문서 기준 도메인에만 적용
      - `ex.org`에서 생성하고 `domain` 생략 → `dev.ex.org`에서는 쿠키 미접근
  - `path` : 경로를 포함한 하위 경로 페이지에서만 쿠키 접근 가능
    - 일반적으로 `/`로 지정
  - `Secure` : 적용시 https인 경우에만 전송
  - `HttpOnly` : HTTP 전송에서만 사용 가능
    - JS에서 접근 불가
  - `SameSite` : 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키 전송 가능

### `Cache-Control`

- 캐시 기본 동작
  - 웹 브라우저 캐시 저장소의 캐시가 만료되지 않았을 경우 그대로 사용
  - 캐시가 만료되었어도 데이터가 변경되지 않은 경우가 있음
    - 검증 헤더 `Last-Modified`, 조건부 요청 헤더 `If-Modified-Since`를 통해 데이터 재사용성을 높일 수 있음
      - 검증 결과 재사용 가능한 데이터일 경우 `304` (Not Modified) 응답을 보냄(http body가 없음)
- 프록시 캐시(CDN)  
  ![HTTP-CDN.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-CDN.png)
- field-value
  - `max-age` : 캐시 유효 시간, 초 단위
  - `no-cache` : 캐시를 해도 되지만, 항상 origin 서버(프록시(CDN) 서버 안됨)에 검증함
  - `no-store` : 캐시로 저장하지 않음
  - `public` : 응답이 public 캐시에 저장될 수 있음
  - `private` : 응답이 private 캐시에 저장됨(기본값)
  - `s-maxage` : 프록시 캐시에만 적용되는 max-age
  - `must-revalidate` : origin 서버에 접근할 수 없는 경우 항상 오류(504 Gateway Timeout 등)를 반환함
    - `no-cache`는 origin 서버에 접근할 수 없는 경우 origin 서버의 처리에 따라서 캐시 데이터를 반환할 수도 있음
- 확실한 캐시 무효화 응답
  ```jsx
  Cache-Control: no-cache, no-store, must-revalidate
  Pragma: no-cache
  ```

# HTTP API 설계

- URI 설계
  - 리소스 식별, 계층 구조를 고려
    - Document : 단일 개념(파일 하나, 객체 인스턴스, DB row)
    - Collection : 서버가 관리하는 리소스 디렉토리(e.g. 회원 정보 API의 members)
    - Store : client가 관리하는 리소스 저장소(e.g. 파일 관리 API의 files)
      - `PUT`을 사용할 때는 client가 리소스 URI를 알고 있어야 함(PUT `/files/{filename}`)
- 기본적으로는 리소스에 대한 URI, HTTP 메서드만으로 HTTP API를 설계해야 함
  - 필요할 때는 URI를 사용하는 행위를 컨트롤 URI로 정의할 수 있음(e.g. `POST /orders/{orderId}/start-delivery`)

## HTTP 메서드 종류

- `GET` : 리소스 조회
  - query string을 통해 데이터 전달
  - 캐싱에 유리
- `POST` : 요청 데이터 처리(주로 등록)
  - message body를 통해 데이터 전달
  - 서버에서 요청 데이터를 처리
- `PUT` : 리소스 대체(없으면 생성)

  - 클라이언트가 리소스를 식별하고 URI를 지정함

    ```jsx
    PUT /members/100 HTTP/1.1
    Content-Type: application/json

    {
      "username": "hello",
      "age": 20
    }
    ```

    - `POST`에서는 `/members`로 요청을 보냄

  - 기존 리소스를 덮어쓰고 명시되지 않은 필드는 `null`이 들어감

- `PATCH` : 리소스 부분 변경
  - 명시되지 않은 필드는 그대로 둠
  - `PATCH`가 지원이 되지 않는 경우 `POST`를 사용하면 됨
- `DELETE` : 리소스 삭제
- `HEAD` : `GET`과 동일하지만 status, header만 반환
- `OPTIONS` : 리소스에 대한 통신 가능 옵션 설명(주로 CORS에서 사용)

## HTTP 메서드 속성

![HTTP-Methods-Properties.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-Methods-Properties.png)

- 안전(Safe Methods) : 호출해도 리소스를 변경하지 않음
- 멱등(Idempotent Methods) : 반복해서 호출해도 결과가 같음
  - 자동 복구 메커니즘에 사용 : 사용자가 처음 호출했을 때 timeout나서 재요청할 때 멱등성을 봄
  - 외부 요인으로 인한 변경은 고려하지 않음(`GET` → `PUT` → `GET`)
- 캐시가능(Cacheable Methods) : 응답 결과를 캐시해서 사용 가능함
  - 실무에선 `GET`, `HEAD` 정도가 캐시 가능
    - 스펙 상 `GET`, `HEAD`, `POST`, `PATCH`를 캐시할 수 있지만, `POST`, `PATCH`는 body까지 캐시 키로 고려해야 하기 때문에 구현되지 않은 경우가 많음

## 클라이언트에서 서버로 데이터 전송

데이터 전달 방식::

- 쿼리 파라미터 : `GET`
- Http message body : `POST`, `PUT`, `PATCH`

  - HTML Form 데이터 전송

    ```jsx
    <form action="/save" method="post">
      <input type="text" name="username" />
      <input type="text" name="age" />
    </form>
    ```

    ```jsx
    POST /save HTTP/1.1
    HOST: localhost:8080
    Content-Type: application/x-www-form-urlencoded

    username=kim&age=20
    ```

    - `method`를 `get`으로 바꾸면 자동으로 쿼리스트링으로 바꿔줌
    - 데이터는 자동으로 url encoding 처리함(한글 등)

  - HTML Form 데이터 전송(multipart/form-data)

    ```jsx
    <form action="/save" method="post" enctype="multipart/form-data">
      <input type="text" name="username" />
      <input type="text" name="age" />
      <input type="file" name="file1" />
      <button type="submit">전송</button>
    </form>
    ```

    ```jsx
    POST /save HTTP/1.1
    HOST: localhost:8080
    Content-Type: multipart/form-data; boundary=----XXX
    Content-Length: 10457

    ------XXX
    Content-Disposition: form-data; name="username"

    kim
    ...
    ------XXX
    Content-Disposition: form-data; name="file1"; filename="intro.png"
    Content-Type: image/png

    lkasdjfiwejfoijawef...
    ------XXX--
    ```

  - HTTP API 데이터 전송 : Form을 사용하지 않는 거의 모든 통신(서버-서버, 앱/웹 클라이언트, …)
    - `Content-Type: application/json`을 주로 사용

# 상태코드

- ~~1xx (Informational) : 요청이 수신되어 처리중(거의 사용 x)~~
- 2xx (Succesful) : 요청 정상 처리
  - `200` : OK
  - `201` : Created
  - `202` : Accepted
    - 요청이 접수되었으나 처리는 완료되지 않음(배치 처리 등)
  - `204` : No Content
    - 요청은 수행했지만 보낼 데이터가 없음(저장 버튼 등)
- 3xx (Redirection) : 요청을 완료하려면 추가 행동 필요
  - 웹 브라우저는 응답 결과가 3xx이고 Location 헤더가 있으면 Location으로 자동 이동(리다이렉트)함
  - `~~300` : Multiple Choices : 거의 사용 x~~
  - `~~301` : Moved Permanently : 거의 사용 x~~
    - ~~리다이렉트시 요청 메서드가 `GET`으로 변하고, 본문이 제거될 수 있음~~
  - `302` : Found
    - 리다이렉트시 요청 메서드가 `GET`으로 변하고, 본문이 제거될 수 있음
    - 자동 리다이렉션시 `GET`으로 변해도 되면 `302`를 사용해도 큰 문제가 없음
  - `~~303` : See Other : 명확하게 메서드가 GET으로 변경되지만 거의 사용 x~~
    - ~~리다이렉트시 요청 메서드가 `GET`으로 변경~~
  - `304` : Not Modified
    - 캐시를 목적으로 사용
    - 로컬 캐시를 사용해야 하기 때문에 응답에 message body를 포함하면 안됨
    - 조건부 GET, HEAD 요청시 사용
  - `~~307` : Temporary Redirect : 명확하게 요청이 유지되지만 거의 사용 x~~
    - ~~리다이렉트시 요청 메서드와 본문 유지~~
  - `~~308` : Permanent Redirect : 거의 사용 x~~
    - ~~리다이렉트시 요청 메서드와 본문 유지~~
- 4xx (Client Error) : 클라이언트 오류
  - `400` : Bad Request
  - `401` : Unauthorized
    - Authentication이 되지 않음
    - `WWW-Authenticate` 헤더와 함께 인증 방법을 설명
  - `403` : Forbidden
    - 서버가 승인을 거부함(인증 자격은 있지만 권한이 없는 경우 등)
  - `404` : Not Found
- 5xx (Server Error) : 서버 오류
  - 5xx는 서버에 문제가 있을 경우에만 사용해야 함(비즈니스 로직의 예외는 5xx 대신 2xx, 4xx를 사용)
  - `500` : Internal Server Error
    - 애매하면 500
  - `503` : Service Unavailable
    - 서버의 일시적인 과부하, 예정된 작업으로 잠시 요청 처리 불가
    - `Retry-After` 헤더에 복구 정보를 알려줄 수 있음
- 클라이언트가 보낸 요청의 상태를 응답에서 알려줌
  - 모르는 코드가 나오면 상위 상태코드로 해석해서 처리하면 됨

> Authentication : 인증, 본인이 누구인지 확인(로그인)
> Authorization : 인가, 권한 부여

## 리다이렉션

- Permanent Redirection : 특정 리소스의 URI가 영구적으로 이동(e.g. `/members` → `/users`)
  - `301`, `308`
  - 검색 엔진 등에서도 변경을 인지함
- Temporary Redirection : 일시적인 변경(e.g. 주문 완료 후 주문 내역 화면으로 이동)
  - `302`, `307`, `303`
  - PRG : Post/Redirect/Get  
    ![HTTP-PRG.png](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/HTTP-PRG.png)
    - `POST`로 주문 후에 주문 결과 화면을 `GET`으로 리다이렉트
    - 새로고침해도 결과 화면 `GET` 요청이 다시 전송됨
