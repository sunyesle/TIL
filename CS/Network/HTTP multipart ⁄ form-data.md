# HTTP multipart/form-data

## 멀티파트(Multipart)
멀티파트는 클라이언트와 서버 간에 전송되는 HTTP 요청 또는 응답에서 텍스트, 파일 등 **여러 종류의 데이터를 동시에 전송하기 위해 사용되는 방식**이다.

주로 파일 업로드와 관련된 데이터를 전송하는 데 사용된다.

## 멀티파트 요청 raw 데이터
```
POST /upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="text"

Hello, world!
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="example.jpg"
Content-Type: image/jpeg

(binary data)
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

멀티파트 요청은 `Content-Type` 헤더에 `multipart/form-data` 값을 가지며, 여러 개의 파트(part)로 구성된다.

`Content-Type: multipart/form-data; boundary=--[임의 문자열]`

boundary에 지정되어 있는 문자열은 각 파트를 구분하는 역할을 한다. 각 파트의 시작은 `--[boundary]`로 표시되며, 요청 본문의 끝은 `--[boundary]--`로 표시된다.

---
**Reference**<br>
- https://varaprasadh.medium.com/what-the-heck-is-multipart-form-data-8df091d598b5
- https://lng1982.tistory.com/209
- https://sharonprogress.tistory.com/197
