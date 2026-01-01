# UUID Version 7 생성하기

## UUID v7 구조
<img width="470" height="120" alt="uuid v7" src="https://github.com/user-attachments/assets/047a3b27-ed6d-4c08-b970-876cd598de1e" />

| 필드            | 비트 수 | 설명                                 |
|---------------|----|------------------------------------|
| **Unix Timestamp** | 48 | Unix Epoch 기준 밀리초(ms) 단위 타임스탬프     |
| **Version**       | 4  | UUID 버전 (v7의 경우 이진수 `0111`)        |
| **Variant**       | 2  | UUID 변형 유형 (RFC 9562 표준에 따라 `10`) |
| **Random**        | 74 | 고유성을 보장하기 위한 랜덤 비트                 |

## UUID v7 특징
1. **시간 기반**
   - Unix Epoch 기준의 타임스탬프를 48비트 크기로 포함한다.
   - 시간 순서를 기반으로 UUID를 정렬, 필터링할 수 있다.
2. **랜덤 데이터 결합**
   - 시간 기반이지만 랜덤 값을 포함하여 고유성을 보장한다.
3. **v1, v6의 개선**
   - v1는 시간 기반이지만 비트의 배치 순서로 인해 정렬이 불가능하다. 또한, 생성한 기기의 MAC 주소가 포함되어있어 개인정보 노출 문제가 있다.
   - v6는 정렬 가능하도록 시간 비트 배치를 변경했지만, 여전히 Gregorian 타임스탬프(1582년 10월 15일 00:000:00.00부터 100 나노초 간격의 카운트)를 사용한다.
   - v7은 현대 시스템에서 보편적인 Unix 타임스탬프(1970년 1월 1일 00:00:00부터 밀리초 간격의 카운트)를 사용한다.

## UUID v7 활용 예시
1. **데이터베이스 기본 키**
   - 시간순으로 삽입되므로 인덱싱 및 조회 성능을 최적화한다.
   - 여러 서버에서 동시에 ID를 생성해도 충돌 없이 순차적인 값을 얻을 수 있어 분산 DB 환경에서 동기화 없이 PK를 생성할 수 있다.
2. **로그 및 이벤트 추적**
   - 타임스탬프가 포함되어 있어 ID 값만으로 언제 발생한 이벤트인지 파악할 수 있고, 특정 시간대의 로그를 정렬 및 필터링 할 수 있다.

## 예시
현재 Java 표준 라이브러리인 `java.util.UUID`는 UUID Version 7 생성을 지원하지 않는다.
대신 `java-uuid-generator` 같은 외부 라이브러리를 사용하여 생성할 수 있다.

```gradle
dependencies {
    implementation 'com.fasterxml.uuid:java-uuid-generator:5.2.0'
}
```
```java
import com.fasterxml.uuid.Generators;
import java.util.UUID;

void main() {
    // Version 7
    UUID uuidV7 = Generators.timeBasedEpochGenerator().generate();

    // Version 7 with per-call random values
    UUID uuidV7_2 = Generators.timeBasedEpochRandomGenerator().generate();
}
```

---
**Reference**
- https://nozee.tistory.com/46
- https://github.com/cowtowncoder/java-uuid-generator
