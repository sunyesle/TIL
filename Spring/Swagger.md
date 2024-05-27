## Swagger JWT 인증 설정

Swagger를 통한 REST 요청에 JWT 기반 인증을 설정하는 방법을 알아보자.

```java
@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        final String securitySchemeName = "bearerAuth";

        return new OpenAPI()
                .components(new Components()
                        .addSecuritySchemes(securitySchemeName, new SecurityScheme()
                                .type(SecurityScheme.Type.HTTP)
                                .scheme("bearer")
                                .bearerFormat("JWT")
                                .name(securitySchemeName))
                )
                .addSecurityItem(new SecurityRequirement()
                        .addList(securitySchemeName))
                .info(new Info()
                        .title("멤버십 적립 서비스 API")
                );
    }
}
```

어노테이션으로도 설정 가능하다. 내용은 위와 동일하다.
```java
@Configuration
@SecurityScheme(
        name = "bearerAuth",
        type = SecuritySchemeType.HTTP,
        scheme = "bearer",
        bearerFormat = "JWT"
)
@OpenAPIDefinition(
        info = @Info(title = "멤버십 적립 서비스 API"),
        security = @SecurityRequirement(name = "bearerAuth")
)
public class SwaggerConfig {

}
```

설정이 적용되면, 상단에 authorize 버튼이 생기며 전역 설정이 가능해진다. 또한 각 EndPoint에 인증이 필요하다는 자물쇠 마크가 생긴다.

---

### Reference

https://www.baeldung.com/openapi-jwt-authentication

https://www.baeldung.com/spring-openapi-global-securityscheme

https://swagger.io/docs/specification/authentication/bearer-authentication/

https://stackoverflow.com/questions/59898874/enable-authorize-button-in-springdoc-openapi-ui-for-bearer-token-authentication

https://velog.io/@zinna_1109/Toy-Project-Swagger-Bearer-Token%EC%84%A4%EC%A0%95

https://taegyunwoo.github.io/tech/Tech_Swagger_Cookie_Auth
