# Spring Boot에서 전략 패턴 사용하기

### Strategy 인터페이스
```java
public interface MessageSender {
    MessageType getMessageType();
    void send(Message message);
}
```
```java
public enum MessageType {
    SMS, EMAIL
}
```

### Strategy 구현체
**EmailSender**<br>
```java
@Service
public class EmailSender implements MessageSender {
    @Override
    public MessageType getMessageType() {
        return MessageType.EMAIL;
    }

    @Override
    public void send(Message message) {
        if (!(message instanceof EmailMessage emailMessage)) {
            throw new IllegalArgumentException("Invalid request type for EmailSender");
        }
        System.out.println("Sending Email to: " + emailMessage.getRecipientEmail());
    }
}
```
```java
@Getter
@RequiredArgsConstructor
public class EmailMessage extends Message {
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
public class SmsSender implements MessageSender {
    @Override
    public MessageType getMessageType() {
        return MessageType.SMS;
    }

    @Override
    public void send(Message message) {
        if (!(message instanceof SmsMessage smsMessage)) {
            throw new IllegalArgumentException("Invalid request type for SmsSender");
        }
        System.out.println("Sending SMS to: " + smsMessage.getRecipientPhone());
    }
}
```
```java
@Getter
@RequiredArgsConstructor
public class SmsMessage extends Message {
    private final String content;
    private final String recipientPhone;
    private final String senderPhone;
}
```

### 팩토리 메서드
```java
@Component
public class MessageSenderFactory {
    private final Map<MessageType, MessageSender> senderMap = new HashMap<>();

    public MessageSenderFactory(List<MessageSender> messageSenders){
        for(MessageSender messageSender : messageSenders) {
            senderMap.put(messageSender.getMessageType(), messageSender);
        }
    }

    public MessageSender getSender(MessageType messageType) {
        if(!senderMap.containsKey(messageType)){
            throw new IllegalArgumentException("Unsupported SendType: " + messageType);
        }
        return senderMap.get(messageType);
    }
}
```

### 적용 예시
```java
@Service
@RequiredArgsConstructor
public class MyService {
    private final MessageSenderFactory messageSenderFactory;

    public void sendVerificationCode(CodeRequest request) {
        // 인증 코드 생성 및 저장 로직 생략...
        String verificationCode = "testcode";

        Message sendRequest = createVerificationCodeSendRequest(request, verificationCode);
        MessageSender messageSender = messageSenderFactory.getSender(request.getMessageType());
        messageSender.send(sendRequest);
    }

    private Message createVerificationCodeSendRequest(CodeRequest request, String verificationCode) {
        switch (request.getMessageType()) {
            case EMAIL -> {
                String content = "인증 코드는 [" + verificationCode + "] 입니다.";
                String subject = "인증 코드 안내";
                return new EmailMessage(content, subject, request.getEmail(), "sendonly@test.com", "테스트");
            }
            case SMS -> {
                String content = "인증 코드: [" + verificationCode + "]";
                return new SmsMessage(content, request.getPhone(), "0000-0000");
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
        CodeRequest codeRequest = new CodeRequest(MessageType.EMAIL, "member@test.com", "010-0000-0000");
        myService.sendVerificationCode(codeRequest);
        // 출력: Sending Email to: member@test.com
    }

    @Test
    void SendSmsVerificationCode() {
        CodeRequest codeRequest = new CodeRequest(MessageType.SMS, "member@test.com", "010-0000-0000");
        myService.sendVerificationCode(codeRequest);
        // 출력: Sending SMS to: 010-0000-0000
    }
}
```

---
**Reference**<br>
- https://velog.io/@kyle/디자인-패턴-전략패턴이란
- https://medium.com/@dori_dori/strategy-pattern-with-spring-bc1b2cbca591
