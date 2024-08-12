# LocalDateTime.now Mocking하기

LocalDateTime.now()는 Clock을 파라미터로 받는 메서드도 존재한다.
Clock을 사용하여 현재 시각을 커스터마이징할 수 있다.

우선 Clock을 빈으로 등록한다.
```java
@Configuration
public class TimeConfig {
    @Bean
    public Clock clock() {
        return Clock.systemDefaultZone();
    }
}
```

Clock을 주입받도록 서비스 코드를 수정한다.
```java
@Service
@RequiredArgsConstructor
public class MemberService {
    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final Clock clock;

    @Transactional
    public void updatePassword(Long id, String password) {
        // ...
        
        member.setPasswordModifiedAt(LocalDateTime.now(clock));
    }
}
```

테스트 시에 Clock을 Mocking하여 원하는 시각에 대한 테스트를 진행할 수 있다.
```java
@ExtendWith(MockitoExtension.class)
class MemberServiceTest {
    @Mock
    private MemberRepository memberRepository;

    @Mock
    private Clock clock;

    @InjectMocks
    private MemberService memberService;

    @Test
    void updatePasswordTest() {
        LocalDateTime now = LocalDateTime.parse("2024-01-11T00:00:01");
        given(clock.instant())
                .willReturn(now.atZone(ZoneId.systemDefault()).toInstant());
        given(clock.getZone())
                .willReturn(ZoneId.systemDefault());

        // ...
    }
}
```

---
**Reference**<br>
- https://www.baeldung.com/automated-visual-regression-testing
- https://medium.com/@jojiapp/junit-localdatetime-now-mocking-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0-fb24119976f1
- https://targetcoders.com/localdatetime-now-%ED%85%8C%EC%8A%A4%ED%8A%B8/
