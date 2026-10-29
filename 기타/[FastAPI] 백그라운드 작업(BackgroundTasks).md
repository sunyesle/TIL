# [FastAPI] 백그라운드 작업(BackgroundTasks)
**백그라운드 작업**은 HTTP 응답을 전송한 후 실행할 작업을 등록하는 기능이다.
요청 처리 과정에서 수행해야 하지만, 클라이언트가 응답을 받기 전에 작업이 완료될 때까지 기다릴 필요가 없는 경우에 유용하다.

- **이메일 전송**: 이메일 서버에 연결하고 이메일을 전송하는 것은 느린 경향이 있으므로, 응답은 즉시 반환하고 이메일 알림은 백그라운드에서 전송할 수 있다.
- **파일 처리**: 처리에 오랜 시간이 걸리는 프로세스를 거쳐야 하는 파일을 받았다면, Accepted(HTTP 202) 응답을 반환하고 백그라운드에서 파일을 처리할 수 있다.

> 무거운 연산이나 장시간 실행되는 작업을 수행해야 하고, 애플리케이션 프로세스와 작업을 분리하거나 재시도 및 작업 큐 등의 기능이 필요하다면 Celery와 같은 작업 큐 도구를 고려할 수 있다.

## 예시

```py
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


# 백그라운드 작업으로 실행될 함수를 정의한다.
def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)


# 경로 처리 함수의 매개변수에 BackgroundTasks 타입을 정의한다.
@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    # .add_task() 메서드로 백그라운드에서 실행될 작업 함수와 작업 함수에 전달해야되는 파라미터를 전달한다.
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```

---
**Reference**
- https://fastapi.tiangolo.com/ko/tutorial/background-tasks/
