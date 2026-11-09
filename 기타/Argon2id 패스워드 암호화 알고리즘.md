# Argon2id 패스워드 암호화 알고리즘

**Argon2id**는 메모리 집약적인(memory-hard) 해싱 알고리즘이다.

bcrypt는 주로 CPU 연산에 의존하며 약 4KB의 메모리를 사용한다. 메모리를 거의 사용하지 않는 연산은 GPU의 수천 개 코어에서 대량으로 병렬 처리하기 좋기 때문에 GPU를 이용한 병렬 공격에 취약하다.

Argon2id는 연산에 대량의 메모리 사용을 요구하여, GPU를 이용한 대량의 병렬 연산을 어렵게 만들고 공격 비용을 크게 높인다.

## 주요 설정 파라미터
메모리 사용량, 반복 횟수, 병렬 처리 수준 등을 조절할 수 있다. 자세한 내용은 [RFC 9106](https://www.rfc-editor.org/rfc/rfc9106.html) 참고

| 파라미터 | 설명 |
|---|---|
| `memory_cost` | 연산에 사용할 메모리 크기 |
| `time_cost` | 해시 연산을 반복하는 횟수 |
| `parallelism` | 병렬 스레드 수 |
| `salt_len` | Salt 길이 |
| `hash_len` | 해시 길이 |

파라미터는 보안성과 성능 사이의 균형을 고려하여 결정한다. 메모리 사용량을 우선 결정한 후 반복 횟수를 조정하고, 실제 운영 환경에서 해싱 및 검증 시간을 측정하여 적절한 값을 선택한다.

## 예시
파이썬 `argon2-cffi` 라이브러리를 사용했다.
```py
from argon2 import PasswordHasher

ph = PasswordHasher(
    salt_len=16, # Salt 길이
    hash_len=32, # 해시 길이
    time_cost=3, # 반복 횟수
    memory_cost=64*1024, # 메모리 사용량(64 MiB)
    parallelism=4, # 병렬 스레드 수
)

password = "password123"

# 비밀번호 해싱
hashed_password = ph.hash ( password) 
print( f"해시된 비밀번호: {hashed_password} " )
```

`argon2-cffi`에서 제공하는 CLI를 사용하면, 현재 환경에서 Argon2의 암호 검증 성능을 벤치마킹할 수 있다.

```bash
$ python -m argon2
Running Argon2id 100 times with:
hash_len: 32 bytes
memory_cost: 65536 KiB
parallelism: 4 threads
time_cost: 3 iterations

Measuring...

45.7ms per password verification
```

다음 명령행 인자로 해싱 파라미터를 설정할 수 있다.
- `-t`: 반복 횟수
- `-m`: 메모리 사용량
- `-p`: 병렬 스레드 수
- `-l`: 해시 길이

---
**Reference**
- https://schimizu.com/understanding-salt-bcrypt-argon2id-ncrypt-and-pepper-essential-concepts-for-secure-password-df160ab062bf
- https://argon2-cffi.readthedocs.io/en/stable/parameters.html
