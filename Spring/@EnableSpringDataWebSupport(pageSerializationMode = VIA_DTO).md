# @EnableSpringDataWebSupport(pageSerializationMode = VIA_DTO)

페이징 기능 작업 중 다음과 같은 경고 로그를 마주했다.
```log
WARN 24616 --- [o-auto-1-exec-3] PageModule$PlainPageSerializationWarning : Serializing PageImpl instances as-is is not supported, meaning that there is no guarantee about the stability of the resulting JSON structure!
	For a stable JSON structure, please use Spring Data's PagedModel (globally via @EnableSpringDataWebSupport(pageSerializationMode = VIA_DTO))
	or Spring HATEOAS and Spring Data's PagedResourcesAssembler as documented in https://docs.spring.io/spring-data/commons/reference/repositories/core-extensions.html#core.web.pageables.
```

## 글로벌하게 간소화된 Page렌더링을 활성화하기
전역적으로 컨트롤러가 `Page` 인스턴스를 반환할 때, 이 인스턴스를 자동으로 `PagedModel`로 변환하는 설정을 제공한다. [공식문서](https://docs.spring.io/spring-data/jpa/reference/repositories/core-extensions.html#core.web.page.config)

설정 방법은 다음과 같다.
```java
@Configuration
@EnableSpringDataWebSupport(pageSerializationMode = VIA_DTO) // PageImpl 인스턴스가 PagedModel로 래핑되도록 한다.
public class MyConfig {
}
```

컨트롤러 메서드의 반환 값은 `Page`를 사용하면서, 렌더링하는 데이터는 `PagedModel`으로 변환된다.
```java
@RestController
@RequestMapping("/api/v1/boards")
@RequiredArgsConstructor
public class BoardController {
    private final BoardService boardService;

    @GetMapping
    public ResponseEntity<Page<BoardResponse>> getBoards(Pageable pageable) {
        Page<BoardResponse> boards = boardService.getBoards(pageable);
        return ResponseEntity.ok(boards);
    }
}
```

적용 이후 응답 JSON
```json
{
    "content": [
        {
            "id": 6,
            "title": "테스트 게시글1",
            "content": "테스트 내용1",
            "createdAt": "2024-08-13T23:10:53.536789",
            "writer": {
                "id": 3,
                "name": "테스트"
            }
        },
        {
            "id": 5,
            "title": "테스트 게시글0",
            "content": "테스트 내용0",
            "createdAt": "2024-08-13T23:10:53.502789",
            "writer": {
                "id": 3,
                "name": "테스트"
            }
        }
    ],
    "page": {
        "size": 20,
        "number": 0,
        "totalElements": 2,
        "totalPages": 1
    }
}
```

---
**Reference**<br>
- [관련 Github 이슈](https://github.com/spring-projects/spring-data-commons/issues/3024)
- [관련 커밋](https://github.com/spring-projects/spring-data-commons/commit/5dd7b322b65652f90717c4f8cb930c5e471ad483)
