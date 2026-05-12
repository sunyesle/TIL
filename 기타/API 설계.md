# API 설계

## URL에서 이름과 식별자 중 선택하기
### 안정성을 위한 URL
고유 식별자를 사용하여 항상 동일한 객체를 반환한다는 것을 보장한다. 퍼머링크(permalinks)라고도 한다.
- **예시**: `https://libraries.gov/books/85ab9db7-abfd-46c4-823b-ad6997b96509`

### 사용자 친화적인 URL
이름이나 경로를 기반으로 하여 가독성이 높다. 일종의 쿼리(검색) 결과에 가까우며, 여러번 실행했을때 같은 리소스를 반환한다는 보장이 없다.
- **예시**: `https://libraries.gov/library/sunnyvale/shelf/american-classics/book/moby-dick`

### 결론
두 URL은 '**특정 객체**'와 '**검색 결과**'라는 다른 목적을 갖는다. 따라서 한가지 방식을 선택하기보다 두 가지 방식을 모두 지원하는 것이 합리적이다.

클라이언트는 단순 조회나 공유 등 사용자 친화적인 URL이 필요한 경우 **쿼리 URL**을 특정 리소스에 대한 영구적인 참조가 필요한 경우 **퍼머링크 URL**을 사용해야 한다.

---
**Reference**
- https://cloud.google.com/blog/products/api-management/false-dichotomy-stability-vs-human-centric-url-design-web-apis?hl=en
- https://cloud.google.com/blog/products/api-management/api-design-choosing-between-names-and-identifiers-in-urls?hl=en
