# Spring REST Docs

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

베이스 클래스를 상속받아 테스트 코드를 작성한다.
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

---
**Reference**<br>
- https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle
- https://backtony.tistory.com/37
- https://dukcode.github.io/spring/spring-rest-docs/

