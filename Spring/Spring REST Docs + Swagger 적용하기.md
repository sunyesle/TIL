# Spring REST Docs + Swagger 적용하기

> [Spring REST Docs 적용하기](https://github.com/sunyesle/TIL/blob/main/Spring/Spring%20REST%20Docs%20%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0.md)에서 이어지는 글입니다.

Spring 기반의 프로젝트에서 API 문서화 도구로 Swagger와 Spring REST Docs가 많이 사용된다.

각 문서화 도구의 장단점은 다음과 같다.

| 구분        | 장점                                                     | 단점                                               |
|-----------|--------------------------------------------------------|--------------------------------------------------|
| Swagger   | • 아름다운 문서<br>• 문서에서 API Test 가능                        | • Swagger 어노테이션이 비즈니스 코드와 섞임<br>• 테스트 코드가 강제되지 않음 |
| REST Docs | • 테스트 강제로 인한 높은 신뢰도<br>• 테스트 기반으로 문서화되므로 비즈니스 코드에 비침투적 | • adoc 작성 필요<br>• 문서에서 API Test 불가능              |

Spring REST Docs는 테스트 코드를 기반으로 문서가 생성되어 테스트 코드 작성을 강제할 수 있고 비즈니스 로직에 비침투적이라는 장점이 있지만,
문서가 추가될 때마다 일일이 asciidoc 파일을 편집해야 하고 문서에서 API Test가 불가능하다는 단점이 있다.

이번 글에서는 Swagger와 Spring REST Docs의 장점을 합치는 방법을 소개해 보려고 한다. 

## Swagger 정적 파일 세팅

[Swagger UI 설치 사이트](https://swagger.io/docs/open-source-tools/swagger-ui/usage/installation/#plain-old-htmlcssjs-standalone)
하단에 있는 Static files without HTTP or HTML 부분에서 latest release를 다운 받는다.

다운받은 파일의 `/dist` 폴더의 내용을 복사하여 `/resource/static/swagger` 폴더에 붙여 넣는다.

불필요한 파일을 삭제한다.
- oauth2-redirect.html
- swagger-ui.js
- swagger-ui-es-bundle-core.js
- swagger-ui-es-bundle.js

`index.html`의 이름을 `swagger-ui.html`로 변경한다.

`swagger-initializer.js`의 내용을 수정한다.
```javascript
window.onload = function() {
  //<editor-fold desc="Changeable Configuration Block">

  // the following lines will be replaced by docker/configurator, when it runs in a docker-container
  window.ui = SwaggerUIBundle({
    url: "openapi3.yaml", // swagger 페이지가 참조할 OAS 파일의 url을 변경한다.
    dom_id: '#swagger-ui',
    deepLinking: true,
    presets: [
      SwaggerUIBundle.presets.apis,
      SwaggerUIStandalonePreset
    ],
    plugins: [
      SwaggerUIBundle.plugins.DownloadUrl
    ],
    layout: "StandaloneLayout"
  });

  //</editor-fold>
};
```

작업을 마치면 다음과 같은 형태가 된다.

![Spring-REST-Docs-Swagger_1](https://github.com/user-attachments/assets/db9d8215-7887-4019-add5-5fa62c7d1c89)

## build.gradle
securitySchemesContent 부분은 프로젝트 인증 방식에 따라 변경하면 된다. 작성 방법은 [OAS 3.0 가이드](https://swagger.io/docs/specification/v3_0/authentication/) 참고
```gradle
plugins {
	id 'com.epages.restdocs-api-spec' version '0.18.2'
}

// OAS 파일을 생성하는 태스크
openapi3 {
	servers = [
			{
				url = "http://localhost:8080"
				description = "local server"
			}
	]
	title = "Test service API docs"
	description = "OpenApi Specification 3.0 기반의 API 문서입니다"
	version = "0.0.1"
	format = "yaml"
}

// 생성된 OAS 파일을 swagger 폴더로 복사하는 태스크 
tasks.register('copyOasToSwagger', Copy) {
	dependsOn "openapi3"

	from "${buildDir}/api-spec/openapi3.yaml"
	into "src/main/resources/static/swagger/"

	doFirst {
		delete "src/main/resources/static/swagger/openapi3.yaml"

		// 빌드 디렉토리 yaml 파일에 보안 스키마 추가
		def openapi3File = file("$buildDir/api-spec/openapi3.yaml")

		// JWT 인증을 위한 securitySchemes와 security 요소를 정의한다.
		def securitySchemesContent =
				"  securitySchemes:\n" +
				"    BearerAuth:\n" +
				"      type: http\n" +
				"      scheme: bearer\n" +
				"      bearerFormat: JWT\n" +
				"security:\n" +
				"  - BearerAuth: []  # Apply the security scheme here"

		// openapi3.yaml 파일에 내용을 추가한다.
		if (openapi3File.exists()) {
			openapi3File.append(securitySchemesContent)
		} else {
			println "Warning: openapi3.yaml not found at ${openapi3File.path}"
		}
	}
}
```

## 테스트 코드 작성
REST Docs 문서 생성에는 `org.springframework.restdocs.mockmvc.MockMvcRestDocumentation`을 사용하고,
Swagger 문서생 성에는 `com.epages.restdocs.apispec.MockMvcRestDocumentationWrapper`를 사용한다.
```java
import static com.epages.restdocs.apispec.ResourceDocumentation.resource;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.springframework.restdocs.mockmvc.RestDocumentationRequestBuilders.get;
import static org.springframework.restdocs.payload.PayloadDocumentation.fieldWithPath;
import static org.springframework.restdocs.payload.PayloadDocumentation.responseFields;
import static org.springframework.restdocs.request.RequestDocumentation.parameterWithName;
import static org.springframework.restdocs.request.RequestDocumentation.pathParameters;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(BoardController.class)
class BoardDocsTest extends BaseRestDocsTest {
    private static final String TAG = "Board API";

    @MockBean
    private BoardService boardService;

    @Test
    void getBoardTest() throws Exception {
        BoardDetailResponse response = new BoardDetailResponse(1L, "제목", "내용", LocalDateTime.of(2024, 11, 10, 10, 0), LocalDateTime.of(2024, 11, 20, 10, 0), 1L, "작성자 이름");
        given(boardService.getBoard(any()))
                .willReturn(response);

        ParameterDescriptor[] pathParameters = {
                parameterWithName("id").description("게시글 id")
        };
        FieldDescriptor[] responseFields = {
                fieldWithPath("id").description("게시글 id"),
                fieldWithPath("title").description("게시글 제목"),
                fieldWithPath("content").description("게시글 내용"),
                fieldWithPath("createdAt").description("게시글 생성일시"),
                fieldWithPath("modificationDeadline").description("게시글 수정가능일시"),
                fieldWithPath("writer").description("작성자 정보"),
                fieldWithPath("writer.id").description("작성자 id"),
                fieldWithPath("writer.name").description("작성자 이름")
        };

        mockMvc.perform(get("/api/v1/boards/{id}", 1)
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                // REST Docs
                .andDo(MockMvcRestDocumentation.document("get-board",
                        pathParameters(pathParameters),
                        responseFields(responseFields)))
                // OAS 3.0 - Swagger
                .andDo(MockMvcRestDocumentationWrapper.document("get-board",
                        resource(ResourceSnippetParameters.builder()
                                .tag(TAG)
                                .description("게시글 상세 조회")
                                .pathParameters(pathParameters)
                                .responseFields(responseFields)
                                .build())));
    }
}
```

테스트 코드를 작성한 뒤 **copyOasToSwagger** 태스크를 실행하면 `src/main/resources/static/swagger` 경로에 `openapi3.yaml` 파일이 생성된다.

애플리케이션을 구동시키고 `http://localhost:8080/swagger/swagger-ui.html` 에 접속해 보면 Swagger 문서가 생성된 것을 확인할 수 있다.

![Spring-REST-Docs-Swagger_2](https://github.com/user-attachments/assets/a5728fe7-7868-44b6-8b3e-14c49b8010ab)

---
**Reference**<br>
- https://helloworld.kurly.com/blog/spring-rest-docs-guide/
- https://tech.kakaopay.com/post/openapi-documentation/
- https://velog.io/@nefertiri/Spring-REST-Docs-%EC%99%80-Swagger-%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-API-%EB%AA%85%EC%84%B8%EC%84%9C-%EC%9E%91%EC%84%B1-%EC%9E%90%EB%8F%99%ED%99%94%ED%95%98%EA%B8%B0
- https://github.com/thefarmersfront/spring-rest-docs-guide
