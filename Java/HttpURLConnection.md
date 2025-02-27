# HttpUrlConnection

`URLConnection`은 Java 애플리케이션과 URL 간의 연결(connection)을 나타내는 모든 클래스의 슈퍼클래스다.<br>
`URLConnection`의 하위클래스인 `HttpUrlConnection`은 HTTP 관련 기능을 추가적으로 지원한다.

## 예시
### POST 요청
```java
String targetUrl = "http://localhost:8080/api/v1/test";
String postData = "{\"name\":\"가나다\", \"age\":27}";
HttpURLConnection con = null;

try {
    // URL 객체 생성
    URL url = new URL(targetUrl);

    // URL에서 URLConnection 객체를 가져온다.
    con = (HttpURLConnection) url.openConnection();

    // 요청 설정
    con.setConnectTimeout(5000);
    con.setReadTimeout(5000);
    con.setRequestMethod("POST");
    con.setRequestProperty("Content-Type", "application/json; utf-8");
    con.setRequestProperty("Accept", "application/json");
    con.setDoOutput(true);

    // 요청 본문 전송
    try (OutputStream os = con.getOutputStream()) {
        byte[] input = postData.getBytes("utf-8");
        os.write(input, 0, input.length);
    }

    // 응답 코드 확인
    int responseCode = con.getResponseCode();
    System.out.println("Response Code: " + responseCode);

    // 응답 본문 읽기
    try (BufferedReader br = new BufferedReader(new InputStreamReader(con.getInputStream(), "utf-8"))) {
        StringBuilder response = new StringBuilder();
        String responseLine;
        while ((responseLine = br.readLine()) != null) {
            response.append(responseLine.trim());
        }
        System.out.println("Response: " + response.toString());
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (con != null) {
        con.disconnect(); // 연결 해제
    }
}
```

---
**Reference**<br>
- https://www.codejava.net/java-se/networking/how-to-use-java-urlconnection-and-httpurlconnection
- https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/net/URLConnection.html
