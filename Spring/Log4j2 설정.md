# Log4j2 설정

## PatternLayout
| 패턴                            | 설명                                                | garbage-free|
|-------------------------------|---------------------------------------------------|-------------|
| `%d`                          | 로그 이벤트가 발생한 시각 (기본 형식: `yyyy-MM-dd HH:mm:ss.SSS`) | O |
| `%p`                          | 로그 레벨 (`INFO`, `DEBUG`, `ERROR` 등)                | O |
| `%c`                          | 로거의 이름 (패키지명 포함)                                  | O |
| `%t`                          | 현재 실행 중인 스레드 이름                                   | O |
| `%m`                          | 로그 메시지                                            | O |
| `%n`                          | 줄 바꿈 (`\n`)                                       | O |
| `%r`                          | 애플리케이션 시작 이후 경과한 시간 (밀리초)                         | O |
| `%x`                          | NDC (Nested Diagnostic Context)                   | O |
| `%X{key}`                     | MDC (Mapped Diagnostic Context)에서 특정 key 값 출력     | O |
| `%F`                          | 소스 파일명                                            | X |
| `%l`                          | 로그 발생 위치 (`클래스명.메서드명(파일명:줄번호)`)                   | X |
| `%C`                          | 클래스명                                              | X |
| `%M`                          | 메서드명                                              | X |
| `%L`                          | 로그가 발생한 코드의 줄 번호                                  | X |
| `%style{pattern}{ANSI style}` | ANSI를 사용해 스타일링                                    | O |
| `%highlight{pattern}{style}`  | 특정 로그 레벨에 따라 색상을 적용                               | O |

---
**Reference**
- https://logging.apache.org/log4j/2.x/manual/pattern-layout.html
