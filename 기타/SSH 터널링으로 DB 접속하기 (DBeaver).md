# SSH 터널링으로 DB 접속하기 (DBeaver)

다음과 같은 흐름으로 DB에 연결한다.
```
[로컬 PC - DBeaver]
       |
       | SSH 접속
       v
[Bastion 서버 (Public IP: 203.0.XX.XX:22)]
       |
       | SSH 터널링 내부 연결
       v
[DB 서버 (Private IP: 10.0.XX.XX:3306)]
```

## Main 탭
<img width="624" height="637" alt="Main" src="https://github.com/user-attachments/assets/cb0613c5-9c57-4a8e-9a38-a8dcc9d72673" />

## SSH 탭
<img width="624" height="637" alt="SSH" src="https://github.com/user-attachments/assets/524774b4-2f8b-4ba5-8c33-305e3033d68c" />
