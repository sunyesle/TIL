# Spring Security 테스트를 위한 @WithMockUser 커스텀하기

### @WithMockUser
어노테이션 속성값을 이용하여 `org.springframework.security.core.userdetails.User` 객체를 생성하고, SecurityContext에 인증 정보를 설정해 준다.
간편하게 사용 가능하지만 UserDetails를 커스텀하여 사용하는 경우 사용할 수 없다.

### @WithUserDetails
지정한 사용자 이름으로 UserDetails 객체를 조회하여 SecurityContext 인증 정보를 설정한다.

### @WithSecurityContext
직접 SecurityContext를 생성한다. WithSecurityContextFactory를 구현하여 커스텀 SecurityContext를 생성할 수 있다.

## 커스텀 @WithMockUser 만들기
> `@WithMockUser`, `WithMockUserSecurityContextFactory`와 유사하게 만들기 때문에 구현 이전에 코드를 참고해 보면 좋다.

우선 어노테이션을 만들어준다. 유저의 id와 권한을 지정할 수 있도록 속성을 추가했다.
```java
@Retention(RetentionPolicy.RUNTIME)
@WithSecurityContext(factory = WithCustomMockUserSecurityContextFactory.class)
public @interface WithCustomMockUser {
    long id() default 1L;

    MemberRole role() default MemberRole.USER;
}
```

위 어노테이션에서 팩토리로 지정한 `WithCustomMockUserSecurityContextFactory`를 만들어준다.<br>
`WithSecurityContextFactory` 인터페이스를 구현한다.
```java
public class WithCustomMockUserSecurityContextFactory implements WithSecurityContextFactory<WithCustomMockUser> {

    @Override
    public SecurityContext createSecurityContext(WithCustomMockUser annotation) {
        Member member = new Member(annotation.id(), "test@email.com", "password", "name", "010-0000-0000", annotation.role());

        UserDetails userDetails = new CustomUserDetails(member);
        Authentication authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());

        SecurityContext context = SecurityContextHolder.getContext();
        context.setAuthentication(authentication);
        return context;
    }
}
```

다음과 같이 테스트 코드에서 `@WithCustomMockUser` 어노테이션을 사용할 수 있다.
```java
@WithMockUser
@Test
void testSaveBoard() throws Exception {
    CreateResponse response = new CreateResponse(1L);
    given(boardService.saveBoard(any(), any())).willReturn(response);

    BoardRequest boardRequest = new BoardRequest("제목", "내용");
    mockMvc.perform(post("/api/v1/boards")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(boardRequest)))
            .andDo(print())
            .andExpect(status().isCreated());
}
```

---
**Reference**<br>
- https://smjeon.dev/etc/with-mock-user
- https://skyriv312079.tistory.com/179
- https://dwaejinho.tistory.com/entry/%EC%BB%A4%EC%8A%A4%ED%85%80-UserDetails-SecurityContext-Test-%EC%BD%94%EB%93%9C-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0
- https://velog.io/@onetuks/WithMockUser-WithUserDetails-WithSecurityContext
