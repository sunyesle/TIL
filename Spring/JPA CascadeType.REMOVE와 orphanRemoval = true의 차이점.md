# CascadeType.REMOVE와 orphanRemoval = true의 차이점

## CascadeType.REMOVE
Cascade 옵션은 부모 엔티티에서 자식 엔티티로 상태 전환(영속성 이벤트)을 전파시킬지 선택하는 옵션이다.

CascadeType.REMOVE를 사용할 경우 부모 엔티티가 삭제되었을 때 자식 엔티티도 함께 삭제된다.

## orphanRemoval = true
orphanRemoval 옵션은 **고아 객체**를 DB에서 자동으로 삭제할지 선택하는 옵션이다.

orphanRemoval = true를 사용할 경우 부모 엔티티가 삭제되었을 때 자식 엔티티도 함께 삭제된다. 
또한, 부모 엔티티와 자식 엔티티 사이의 연관관계를 제거하면 자식 엔티티가 고아 객체로 취급되어 삭제된다.

### 고아 객체
고아 객체란 부모 엔티티와의 연관관계가 끊어진 자식 엔티티이다.

다음과 같은 작업으로 인해 고아 객체가 될 수 있다.
- 부모 엔티티를 삭제하는 경우
- 부모 엔티티와 자식 엔티티 사이의 연관관계를 제거하는 경우

## 비교
### 부모 엔티티 삭제
- `CascadeType.REMOVE` : 자식 엔티티도 함께 삭제된다.
- `orphanRemoval = true` : 자식 엔티티도 함께 삭제된다.

### 부모 엔티티와 자식 엔티티 사이의 연관관계 제거
- `CascadeType.REMOVE` : 자식 엔티티의 외래키 값만 변경된다.
- `orphanRemoval = true` : 자식 엔티티가 고아 객체로 취급되어 삭제된다.

---
**Reference**<br>
- https://tecoble.techcourse.co.kr/post/2021-08-15-jpa-cascadetype-remove-vs-orphanremoval-true/
- https://velog.io/@yuseogi0218/JPA-CascadeType.REMOVE-vs-orphanRemoval-true#orphanremoval--true
