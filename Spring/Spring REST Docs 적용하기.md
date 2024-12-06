# Spring REST Docs 적용하기

Spring REST Docs는 테스트 코드 기반으로 RESTful 문서 생성을 도와주는 도구이다.<br>
테스트 코드를 기반으로 생성된 Asciidoc 스니펫을 조합하여 Acsciidoctor로 문서를 작성할 수 있다.<br>
Swagger와 달리 운영 코드에 침투적이지 않고, 테스트를 통과해야 문서가 만들어지기 때문에 신뢰성이 높다는 장점이 있다.

## build.gradle
```gradle
plugins {
	id "org.asciidoctor.jvm.convert" version "3.3.2" // Asciidoctor 플러그인 적용
}

configurations {
	asciidoctorExt // asciidoctorExt 설정 선언
}

dependencies {
	// Spring REST Docs에서 Asciidoctor와 통합하여 REST API 문서를 생성할 때 필요한 기능을 제공한다.
	asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
	
	// MockMvc에 기반해서 스니펫을 뽑아낼 수 있도록 하는 라이브러리
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
}

ext {
	snippetsDir = file('build/generated-snippets') // 스니펫 경로 지정
}

test {
	outputs.dir snippetsDir // snippetsDir 디렉토리를 test의 output으로
}

// restdocs 태그가 추가된 테스트만 실행하는 태스크 정의
tasks.register("restDocsTest", Test) {
	outputs.dir snippetsDir
	useJUnitPlatform {
		includeTags("restdocs")
	}
}

// asciidoctor 태스크 정의
asciidoctor {
	dependsOn restDocsTest // restDocsTest가 수행된 다음에 asciidoctor 수행

	inputs.dir snippetsDir // snippetsDir를 입력으로 사용
	configurations 'asciidoctorExt' // asciidoctor의 확장 플러그인을 설정
	sources {
		include("**/index.adoc") // 특정 파일만 html로 만든다. source가 없으면 .adoc파일을 전부 html로 만든다.
	}
	baseDirFollowsSourceFile() // 특정 .adoc에 다른 adoc 파일을 가져와서(include) 사용하고 싶을 경우 경로를 baseDir로 맞춰주는 설정
}

// asciidoctor task에서 생성된 HTML을 정적 리소스 폴더로 복사하는 태스크 정의
tasks.register('copyDocument', Copy) {
	dependsOn asciidoctor

	from file("${asciidoctor.outputDir}")
	into file("src/main/resources/static/docs")

	doFirst {
		delete file("src/main/resources/static/docs")
	}
}
```

## 테스트 코드 작성
REST Docs 테스트의 공통 설정을 제공하는 추상 클래스를 정의한다.
```java
import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.documentationConfiguration;
import static org.springframework.restdocs.operation.preprocess.Preprocessors.prettyPrint;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;

@Tag("restdocs") // restdocs 태그 지정
@ExtendWith(RestDocumentationExtension.class)
@AutoConfigureRestDocs
public abstract class BaseRestDocsTest {

    @Autowired
    protected MockMvc mockMvc;

    @Autowired
    protected ObjectMapper objectMapper;

    @BeforeEach
    void setUp(WebApplicationContext webApplicationContext,
               RestDocumentationContextProvider restDocumentation) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .apply(documentationConfiguration(restDocumentation)
                        .operationPreprocessors()
                        .withRequestDefaults(prettyPrint()) // request 포매팅
                        .withResponseDefaults(prettyPrint())) // response 포매팅
                .alwaysDo(print()) // 항상 로그를 남기도록 함
                .build();
    }
}
```

위에서 작성한 `BaseRestDocsTest` 클래스를 상속받아 테스트 코드를 작성한다.
```java
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.springframework.restdocs.mockmvc.RestDocumentationRequestBuilders.*;
import static org.springframework.restdocs.payload.PayloadDocumentation.*;
import static org.springframework.restdocs.request.RequestDocumentation.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(BoardController.class)
class BoardDocsTest extends BaseRestDocsTest {

    @MockBean
    private BoardService boardService;

    @Test
    void getBoardTest() throws Exception {
        BoardDetailResponse response = new BoardDetailResponse(1L, "제목", "내용", LocalDateTime.of(2024, 11, 10, 10, 0), LocalDateTime.of(2024, 11, 20, 10, 0), 1L, "작성자 이름");
        given(boardService.getBoard(any()))
                .willReturn(response);

        mockMvc.perform(get("/api/v1/boards/{id}", 1)
                        .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andDo(MockMvcRestDocumentation.document("get-board",
                        pathParameters(
                                parameterWithName("id").description("게시글 id")
                        ),
                        responseFields(
                                fieldWithPath("id").description("게시글 id"),
                                fieldWithPath("title").description("게시글 제목"),
                                fieldWithPath("content").description("게시글 내용"),
                                fieldWithPath("createdAt").description("게시글 생성일시"),
                                fieldWithPath("modificationDeadline").description("게시글 수정가능일시"),
                                fieldWithPath("writer").description("작성자 정보"),
                                fieldWithPath("writer.id").description("작성자 id"),
                                fieldWithPath("writer.name").description("작성자 이름"))));
    }
}
```

테스트 코드를 작성한 뒤 **restDocsTest** 태스크를 실행하면 `build/generated-snippets` 경로에 스니펫이 생성된 것을 확인할 수 있다.

![Spring-REST-Docs_1](https://github.com/user-attachments/assets/8659521f-b01a-4780-b390-eee728c72e0b)


## Snippet 커스텀
기본적으로 템플릿을 제공하지만, 필요하다면 출력되는 템플릿을 커스텀할 수 있다.<br>
`src/test/resources/org/springframework/restdocs/templates` 경로에 커스텀할 스니펫의 템플릿 파일을 정의하면 된다.

![Spring-REST-Docs_6](https://github.com/user-attachments/assets/5a7e0b11-3810-485a-b017-d82e1f2360b1)

> response-fields.snippet
```snippet
.Response Fields
|===
|필드명|타입|필수값|설명

{{#fields}}
|{{path}}
|{{type}}
|{{^optional}}true{{/optional}}
a|{{description}}
{{/fields}}
|===
```

- [공식 문서](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/#documenting-your-api-customizing-snippets)
- [기본 템플릿 참고](https://github.com/spring-projects/spring-restdocs/tree/main/spring-restdocs-core/src/main/resources/org/springframework/restdocs/templates/asciidoctor)

## 문서 작성
문서 작성에 앞서 adoc 파일 작성의 편의를 위해 Intellij AsciiDoc 플러그인을 설치한다.

![Spring-REST-Docs_2](https://github.com/user-attachments/assets/99dcae03-c1c6-4c4f-916d-4be9c2ed4b7a)

생성된 스니펫 조각들을 이용해 문서를 작성해 보자.

`src/docs/asciidoc` 경로에 index.adoc 파일을 생성한다.<br>
API마다 adoc 파일을 분리하고, index.adoc에서 해당 파일들을 include 하였다.

> index.adoc
```asciidoc
= API Documentation
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 2
:sectlinks:
:sectnums:
:docinfo: shared-head

== Board API
include::api/board.adoc[]
```

> board.adoc
```asciidoc
=== 게시글 상세 조회

.request
include::{snippets}/get-board/http-request.adoc[]
include::{snippets}/get-board/path-parameters.adoc[]

.response
include::{snippets}/get-board/http-response.adoc[]
include::{snippets}/get-board/response-fields.adoc[]
```
**asciidoctor** 태스크를 실행하면 `build/docs` 경로에 html 파일이 생성된다.

![Spring-REST-Docs_3](https://github.com/user-attachments/assets/e354d828-0767-4870-80e0-0aa4e0d7e326)

## 문서를 정적 리소스로 제공
생성된 문서를 정적 콘텐츠로 제공하려면 **copyDocument** 태스크를 실행하여 html 파일을 정적 리소스 폴더에 복사한다.

![Spring-REST-Docs_4](https://github.com/user-attachments/assets/8fa60ccd-609c-4a3f-82bb-d24c5084fe64)

---
**Reference**<br>
- https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle
- https://backtony.tistory.com/37
- https://dukcode.github.io/spring/spring-rest-docs/
