# OSIV(Open Session In View)
OSIV(Open Session In View)란 영속성 컨텍스트를 뷰까지 열어두는 기능이다.
OSIV의 핵심은 뷰에서도 지연 로딩이 가능하도록 하는 것이다.

`spring.jpa.open-in-view` 옵션을 통해 JPA의 OSIV 기능 사용여부를 설정할 수 있다. 기본값은 `true`이다.

![OSIV_0](https://github.com/user-attachments/assets/75b16b94-888a-4482-8fc0-aed023553de1)
> spring.jpa.open-in-view=true

- 영속성 컨텍스트를 프레젠테이션 계층까지 유지한다.
- 프레젠테이션 계층에는 트랜잭션이 없으므로 엔티티를 수정할 수 없다.
- 프레젠테이션 계층에는 트랜잭션이 없지만 ***트랜잭션 없이 읽기***를 사용해서 지연 로딩을 할 수 있다.

### 트랜잭션 없이 읽기
단순 조회만 할 때 사용하는 방법으로 트랜잭션 없이 엔티티를 조회하는 것이다.
트랜잭션이 없어도 영속성 컨텍스트만 있다면 지연 로딩을 사용할 수 있게 된다.

## OSIV 사용시 주의점
spring.jpa.open-in-view의 값을 기본값(true)으로 어플리케이션을 구동하면, 어플리케이션 시작 시점에 다음과 같은 warn 로그를 남기게 된다.
```
2024-07-25T22:19:28.541+09:00  WARN 8304 --- [           main] JpaBaseConfiguration$JpaWebConfiguration : spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering. Explicitly configure spring.jpa.open-in-view to disable this warning
```

### OSIV의 단점
- OSIV를 적용하면 같은 영속성 컨텍스트를 여러 트랜잭션이 공유할 수 있다는 점을 주의해야 한다.
  - 프레젠테이션 계층에서 엔티티를 수정 하고 나서 비즈니스 로직을 수행하면 엔티티가 수정될 수 있다.
- 프레젠테이션 계층에서 지연 로딩에 의한 SQL이 실행된다. 따라서 성능 튜닝 시에 확인해야 할 부분이 넓다.
- 데이터베이스 연결은 UI 렌더링 단계 전반에 걸쳐 유지되므로 연결 임대 시간이 늘어나고 커넥션 풀에 대한 부하가 생길 수 있다.

# OSIV OFF
![OSIV_1](https://github.com/user-attachments/assets/15243773-c765-47d9-b586-ccfd46a2c661)
> spring.jpa.open-in-view=false

프리젠테이션 계층에서 엔티티가 준영속 상태가 되므로 지연 로딩을 사용할 수 없다.

## 준영속 상태 지연 로딩 해결 전략
- 글로벌 페치 전략 수정
  - 글로벌 페치 전략을 지연 로딩에서 즉시 로딩으로 변경하면 된다. -> N+1 문제가 발생한다.
- 강제로 초기화
  - 프리젠테이션 계층이 서비스 계층을 침범하게된다.
- **JPQL 페치 조인(fetch join)**

OSIV를 비활성화할 경우 화면에 맞춘 조회 메서드가 증가할 수 있다. 이때 복잡성을 관리하기 위해 `CQRS`(Command Query Responsibility Segregation) 패턴을 사용할 수 있다.
- OrderService
  - OrderService: 핵심 비즈니스 로직
  - OrderQueryService: 화면이나 API에 맞춘 서비스 (주로 읽기 전용 트랜잭션 사용)

---
**Reference**
- https://incheol-jung.gitbook.io/docs/study/jpa/13
- https://github.com/spring-projects/spring-boot/issues/7107
- https://ykh6242.tistory.com/entry/JPA-OSIVOpen-Session-In-View와-성능-최적화
- https://stackoverflow.com/questions/30549489/what-is-this-spring-jpa-open-in-view-true-property-in-spring-boot

