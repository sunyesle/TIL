# Spring Security
Spring Security는 인증(Authentication), 인가(Authorization) 및 보호 기능을 제공하는 프레임워크이다.

## 동작 과정
![blogpost-spring-security-architecture](https://github.com/sunyesle/TIL/assets/45172865/f5980ae6-813a-4155-a5fa-d231e6921b70)
1. Http Request 수신
2. 사용자 자격 증명을 기반으로 AuthenticationToken을 생성한다.
3. 생성된 AuthenticationToken을 AuthenticationManager에게 전달한다.
4. AuthenticationToken을 실제 인증을 할 AuthenticationProvider에게 전달한다.
5. UserDetailsService에게 사용자 이름을 전달한다.
6. UserDetailsService 사용자 이름을 기반으로 세부정보를 검색한다.
7. AuthenticationProvider에게 UserDetails객체를 전달한다.
8. 인증 처리 후 인증된 Authentication 객체를 반환한다. 실패 시 AuthenticationException 발생시킨다.
9. AuthenticationManager은 Authentication 객체를 AuthenticationFilter로 반환한다.
10. SecurityContext에 Authentication 객체를 저장한다.

## 주요 모듈

### SecurityContextHolder
보안 주체의 세부정보, 응용프로그램의 현재 보안 컨텍스트에 대한 세부정보가 저장된다. 동일 스레드 내에서 전역적으로 접근 가능하도록 설계(SecurityContextHolder Strategy에 따라 다르다)
### SecurityContext
- Authentication을 보관하는 역할을 하며, Security Context로부터 Authentication 객체를 가져올 수 있다.
### Authentication
현재 접근하는 주체의 정보와 권한을 담는 인터페이스이다. 
### UsernamePasswordAuthenticationToken
Authentication을 implements한 AbstractAuthenticationToken의 하위클래스이다. 인증 전 상황과 인증 후 상황에 따라 사용되는 목적이 달라진다.

- **인증 전** : 인증을 요구하는 주체가 인증에 필요한 정보(로그인 아이디, 패스워드)를 제공하기 위해 사용
  - ``principal`` : 로그인 시도 아이디(``String``)
  - ``credentials`` : 로그인 시도 비밀번호(``String``)
- **인증 후** : 인증이 완료된 사용자의 정보를 저장하는데 사용
  - ``principal`` : 인증이 완료된 사용자 객체(``UserDetails``의 구현체)
  - ``credentials`` : 인증 완료후 유출 가능성을 줄이기 위해 삭제
  - ``authorities`` : 인증된 사용자가 가지는 권한 목록
``` java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
    // 주로 사용자의 ID에 해당함
    private final Object principal;
    // 주로 사용자의 PW에 해당함
    private Object credentials;
    
    // 인증 완료 전의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
        super(null);
        this.principal = principal;
        this.credentials = credentials;
        setAuthenticated(false);
    }
    
    // 인증 완료 후의 객체 생성
    public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
            Collection<? extends GrantedAuthority> authorities) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        super.setAuthenticated(true); // must use super, as we override
    }
}

public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer {
}
```
### AuthenticationManager
Spring Security Filter의 인증 수행 방식을 정의하는 인터페이스이다.
``` java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```
### ProviderManager
AuthenticationManager의 구현체이다. AuthenticationProvider 리스트를 가지고 있으며, 실제 인증 작업을 AuthenticationProvider에게 위임한다. 각 AuthenticationProvider마다 처리하는 인증 유형이 다르다.
### AuthenticationProvider
실질적으로 인증 작업을 수행하는 주체
``` java
public interface AuthenticationProvider {
    // 인증 전의 Authenticaion 객체를 받아서 인증된 Authentication 객체를 반환
    Authentication authenticate(Authentication authentication) throws AuthenticationException;

    boolean supports(Class<?> authentication);
}

```
### UserDetails
사용자 정보를 캡슐화한다. Authentication 객체로 캡슐화되는 사용자 정보를 저장합니다. (인증 후 Authentication의 principal) 이를 통해 보안과 관련되지 않은 사용자 정보(이메일, 전화번호 등)을 편리한 위치에 저장할 수 있다.
``` java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```
### UserDetailsService
사용자 정보를 가져오는 인터페이스이다. 일반적으로 이를 구현한 클래스의 내부에 UserRepository를 주입받아 DB와 연결하여 처리한다.
``` java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```
### GrantedAuthority
현재 사용자(principal)가 가지고 있는 권한을 의미한다. ROLE_ADMIN나 ROLE_USER와 같이 ROLE_*의 형태로 사용한다. Authentication.getAuthorities() 메소드를 통해 획득 가능하다.

---
### Reference
https://chathurangat.wordpress.com/2017/08/23/spring-security-authentication-architecture/
https://mangkyu.tistory.com/76
https://mysterlee.tistory.com/98
https://docs.spring.io/spring-security/reference/servlet/architecture.html
https://velog.io/@masondev108/Spring-Security-%EA%B8%B0%EB%B3%B8-API%EC%99%80-Filter
https://velog.io/@blessing333/Spring-Security1
