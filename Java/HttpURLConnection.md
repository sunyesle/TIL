# HttpUrlConnection

`URLConnection`은 Java 애플리케이션과 URL 간의 연결(connection)을 나타내는 모든 클래스의 슈퍼클래스다.<br>
`URLConnection`의 하위클래스인 `HttpUrlConnection`은 HTTP 관련 기능을 추가적으로 지원한다.

## 연결 설정
**URLConnection**는 연결을 설정을 위해 다음과 같은 메서드를 제공한다.
- `setConnectTimeout(int timeout)`: 연결되기 전까지의 기다리는 시간을 밀리초 단위로 설정한다. 시간 초과 시 java.net.SocketTimeoutException 예외를 던진다. 0이면 경우 무한히 기다린다. (기본값 0)
- `setReadTimeout(int timeout)`: 연결되고 응답을 받기까지 기다리는 시간을 밀리초 단위로 설정한다. 시간 초과 시 java.net.SocketTimeoutException 예외를 던진다. 0이면 경우 무한히 기다린다. (기본값 0)
- `setDefaultUseCaches(boolean default)`: URLConnection이 기본적으로 캐시를 사용할지 여부를 설정한다. (기본값 true)
- `setUseCaches(boolean useCaches)`: 이 연결에서 기본적으로 캐시를 사용할지 여부를 설정한다. (기본값 true)
- `setDoInput(boolean doInput)`: URLConnection을 사용하여 서버에서 콘텐츠를 읽을 수 있는지 여부를 설정한다. (기본값 true)
- `setDoOutput(boolean doOutput)`: URLConnection을 사용하여 서버로 데이터를 전송할 수 있는지 여부를 설정한다. (기본값 false)
- `setRequestProperty(String key, String value)`: 요청 헤더를 설정한다. 이미 key가 존재하는 경우 덮어써진다.

**HttpURLConnection**은 HTTP 연결을 설정하기 위해 다음과 같은 메서드를 제공한다.
- `setRequestMethod(String method)`: HTTP 요청 메서드를 설정한다. (기본값 GET)
- `setChunkedStreamingMode(int chunkLength)`: 요청 데이터의 크기를 미리 알 수 없는 경우 청크 분할 전송 인코딩 모드를 활성화한다.
- `setFixedLengthStreamingMode(long contentLength)`: 요청 데이터의 크기를 미리 알 수 있는 경우, 고정 길이 스트리밍 모드를 활성화한다.
- `setFollowRedirects(boolean follow)`: HttpURLConnection 클래스가 HTTP 리다이렉션을 자동으로 따라야 하는지 여부를 설정한다. (기본값 true)
- `setInstanceFollowRedirects(boolean follow)`: 이 연결에서 HTTP 리다이렉션을 자동으로 따라야 하는지 여부를 설정한다. (기본값 true)

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
