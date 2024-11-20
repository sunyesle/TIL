# @Mock과 @MockBean

## @Mock
테스트 클래스에 @ExtendWith(MockitoExtension.class) 어노테이션을 추가한다.

@Mock을 사용하여 모의 객체를 생성한다.

@InjectMocks를 사용하여 모의 객체를 대상 객체에 주입한다.

모든 종속성을 모의해야 한다.

```java
@ExtendWith(MockitoExtension.class)
class MemberServiceMockTest {
    @Mock
    private MemberRepository memberRepository;

    @Mock
    private ObjectMapper objectMapper;

    @InjectMocks
    private MemberService memberService;

    @Test
    void test() throws JsonProcessingException {
        Member member = new Member(1L, "김이름");
        given(memberRepository.findById(1L)).willReturn(member);
        given(objectMapper.writeValueAsString(any())).willReturn(new ObjectMapper().writeValueAsString(member));

        String memberString = memberService.getMember(1L);

        assertThat(memberString).contains("김이름");
    }
}
```

## @MockBean
테스트 클래스에 @SpringBootTest 어노테이션을 추가하여 테스트에서 Spring 컨텍스트를 로드하도록 한다.

@MockBean 어노테이션을 추가하여 Spring 컨텍스트에 모의 객체를 빈으로 등록한다.

@Autowired로 빈을 주입받는다.

```java
@SpringBootTest
class MemberServiceMockBeanTest {
    @MockBean
    private MemberRepository memberRepository;

    @Autowired
    private MemberService memberService;

    @Test
    void test() throws JsonProcessingException {
        Member member = new Member(1L, "김이름");
        given(memberRepository.findById(1L)).willReturn(member);

        String memberString = memberService.getMember(1L);

        assertThat(memberString).contains("김이름");
    }
}
```

---
**Reference**<br>
- https://www.baeldung.com/spring-test-autowired-injectmocks
