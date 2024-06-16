## @Transactional(readOnly = true)
트랜잭션을 읽기 전용 모드로 설정한다.

- JPA의 변경 감지(dirty checking)를 위한 작업이 생략되어 성능상의 이점이 있다.
- JDBC 드라이버에서 readOnly 속성을 지원하는 경우 해당 수준에서의 최적화가 수행되고, 쓰기 작업을 거부할 수도 있다. (H2는 readOnly 지원 안함)
- 읽기 작업만 수행한다는 것을 명시적으로 드러낼 수 있다.
