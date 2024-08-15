# HttpClient

HttpClient는 Apache에서 제공하는 라이브러리로 HTTP 프로토콜을 손쉽게 사용할 수 있도록 도와준다.

예시 코드에서는 `HttpClient 5.3`을 사용했다.

## GET
```java
@Test
void getTest() throws IOException {
    try (CloseableHttpClient httpclient = HttpClients.createDefault()) {
        final ClassicHttpRequest httpGet = ClassicRequestBuilder
                .get("http://localhost:8080/api/v1/boards")
                .setHeader("Content-Type", "application/json")
                .addParameter("orderBy", "LATEST")
                .addParameter("pageNumber", "0")
                .addParameter("pageSize", "20")
                .build();

        final Result result = httpclient.execute(httpGet, response -> {
            System.out.println(httpGet + " -> " + new StatusLine(response));
            return new Result(response.getCode(), EntityUtils.toString(response.getEntity()));
        });

        System.out.println(result);
    }
}
```

## POST
```java
@Test
void postTest() throws IOException {
    try (CloseableHttpClient httpclient = HttpClients.createDefault()) {
        final String jsonContent = "{\"title\": \"제목\", \"content\": \"내용\"}";
        final StringEntity entity = new StringEntity(jsonContent, StandardCharsets.UTF_8);

        final ClassicHttpRequest httpPost = ClassicRequestBuilder
                .post("http://localhost:8080/api/v1/boards")
                .setHeader("Content-Type", "application/json")
                .addHeader("Authorization", "Bearer " + TOKEN)
                .setEntity(entity)
                .build();

        final Result result = httpclient.execute(httpPost, response -> {
            System.out.println(httpPost + " -> " + new StatusLine(response));
            return new Result(response.getCode(), EntityUtils.toString(response.getEntity(), StandardCharsets.UTF_8));
        });

        System.out.println(result);
    }
}
```

> Spring Framework 6 에서 Apache HttpClient 4에 대한 지원이 제거되었으며 Apache HttpClient 5로 대체되었다.

---
**Reference**<br>
- https://hc.apache.org
- https://github.com/apache/httpcomponents-client/tree/5.3.x/httpclient5/src/test/java/org/apache/hc/client5/http/examples
- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.1-Release-Notes
