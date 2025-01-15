# Spring Boot에서 전략 패턴 사용하기

### Stratege 인터페이스
```java
public interface Sender {
    SendType getSendType();
    void send(Object sendRequest);
}
```
```java
public enum SendType {
    SMS, EMAIL
}
```

### Stratege 구현체
**EmailSender**<br>
```java
@Service
public class EmailSender implements Sender {
    @Override
    public SendType getSendType() {
        return SendType.EMAIL;
    }

    @Override
    public void send(Object sendRequest) {
        if (!(sendRequest instanceof EmailSendRequest emailSendRequest)) {
            throw new IllegalArgumentException("Invalid request type for EmailSender");
        }
        System.out.println("Sending Email to: " + emailSendRequest.getRecipientEmail());
    }
}
```
```java
@Getter
@RequiredArgsConstructor
public class EmailSendRequest {
    private final String content;
    private final String subject;
    private final String recipientEmail;
    private final String senderEmail;
    private final String senderEmailName;
}
```

**SmsSender**<br>
```java
@Service
public class SmsSender implements Sender {
    @Override
    public SendType getSendType() {
        return SendType.SMS;
    }

    @Override
    public void send(Object sendRequest) {
        if (!(sendRequest instanceof SmsSendRequest smsSendRequest)) {
            throw new IllegalArgumentException("Invalid request type for SmsSender");
        }
        System.out.println("Sending SMS to: " + smsSendRequest.getRecipientPhone());
    }
}
```
```java
@Getter
@RequiredArgsConstructor
public class SmsSendRequest {
    private final String content;
    private final String recipientPhone;
    private final String senderPhone;
}
```

### 팩토리 메서드
```java
@Component
public class SenderFactory {
    private final Map<SendType, Sender> senderMap = new HashMap<>();

    public SenderFactory(List<Sender> senders){
        for(Sender sender : senders) {
            senderMap.put(sender.getSendType(), sender);
        }
    }

    public Sender getSender(SendType sendType) {
        if(!senderMap.containsKey(sendType)){
            throw new IllegalArgumentException("Unsupported SendType: " + sendType);
        }
        return senderMap.get(sendType);
    }
}
```

### 적용 예시
```java
@Service
@RequiredArgsConstructor
public class MyService {
    private final SenderFactory senderFactory;

    public void sendVerificationCode(CodeRequest request) {
        // 인증 코드 생성 및 저장 로직 생략...
        String verificationCode = "testcode";

        Object sendRequest = createVerificationCodeSendRequest(request, verificationCode);
        Sender sender = senderFactory.getSender(request.getSendType());
        sender.send(sendRequest);
    }

    private Object createVerificationCodeSendRequest(CodeRequest request, String verificationCode) {
        switch (request.getSendType()) {
            case EMAIL -> {
                String content = "인증 코드는 [" + verificationCode + "] 입니다.";
                String subject = "인증 코드 안내";
                return new EmailSendRequest(content, subject, request.getEmail(), "sendonly@test.com", "테스트");
            }
            case SMS -> {
                String content = "인증 코드: [" + verificationCode + "]";
                return new SmsSendRequest(content, request.getPhone(), "0000-0000");
            }
            default -> throw new IllegalArgumentException("지원되지 않는 sendType 입니다.");
        }
    }
}
```
```java
@Getter
@RequiredArgsConstructor
public class CodeRequest {
    private final SendType sendType;
    private final String email;
    private final String phone;
}
```

### 테스트
```java
@SpringBootTest
class MyServiceTest {

    @Autowired
    private MyService myService;

    @Test
    void SendEmailVerificationCode() {
        CodeRequest codeRequest = new CodeRequest(SendType.EMAIL, "member@test.com", "010-0000-0000");
        myService.sendVerificationCode(codeRequest);
        // 출력: Sending Email to: member@test.com
    }

    @Test
    void SendSmsVerificationCode() {
        CodeRequest codeRequest = new CodeRequest(SendType.SMS, "member@test.com", "010-0000-0000");
        myService.sendVerificationCode(codeRequest);
        // 출력: Sending SMS to: 010-0000-0000
    }
}
```

---
**Reference**<br>
- https://velog.io/@kyle/디자인-패턴-전략패턴이란
- https://medium.com/@dori_dori/strategy-pattern-with-spring-bc1b2cbca591
