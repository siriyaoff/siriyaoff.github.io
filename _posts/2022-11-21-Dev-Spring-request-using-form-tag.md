---
layout: single
title: "Spring에서 form 태그를 사용한 요청 송/수신"
categories:
  - Dev
tags:
  - Spring
  - Request
  - form
  - MultipartHttpServletRequest
---

## form 태그를 사용한 요청

- `<form>` 태그를 사용하는데 `formData`를 사용해서 새로 ajax로 요청을 보내는 경우는 잘 없음
  - `formData.append(name, value)`으로 요소 추가하면 무조건 문자열로 들어감

```html
<form action="/reqdto" id="infform" method="post" enctype="multipart/form-data">
  <input type="file" name="psd" />
  <input type="number" name="height" />
  <input type="submit" value="POST" />
</form>
```

- `<input type="submit" value="gogo">`로 요청 바로 보내는게 간편함
  - `<input type="file"...>`이 있을 경우 form에 `enctype="multipart/form-data"` attribute 추가해야 함

### 매핑

```java
@PostMapping("/reqdto")
    public String reqdto(MultipartHttpServletRequest request) throws IOException {
        try {
            String height = request.getParameter("height");
            MultipartFile psd=request.getFile("psd");
            byte[] b = psd.getBytes();

            FileUtils.writeByteArrayToFile(new File("C:\\Users\\yang\\Downloads\\realtrue.png"), b);

            return height+"\nheight printed";
        } catch (Exception e) {
            System.out.println(e.getStackTrace()[0]);
        }
        return "error";
    }
```

- 다른 POST 요청과 달리 `@RequestBody` annotation을 붙이면 안됨
- `req.getParameter()`로 일반 변수들
  `req.getFile()`로 `MultipartFile` 받음
- 참고 자료
  [https://mchch.tistory.com/217](https://mchch.tistory.com/217)
