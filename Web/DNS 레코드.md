# DNS 레코드
**DNS 레코드**는 도메인 네임 시스템(DNS) 서버에 저장되는 텍스트 기반의 지침 파일로, 도메인 이름과 IP 주소 또는 기타 서비스 정보를 매핑하는 설정 정보이다.

## 종류
| 레코드 | 역할 |
|---|---|
| **A** | 도메인을 IPv4 주소에 연결 |
| **AAAA** | 도메인을 IPv6 주소에 연결 |
| **CNAME** | 다른 도메인 이름을 가리키는 별칭 |
| **MX** | 도메인의 메일 서버 지정 |
| **TXT** | 임의의 텍스트 정보 저장 |
| **NS** | 도메인의 DNS 관리 서버 지정 |
| **SOA** | DNS 영역의 기본 관리 정보 |
| **SRV** | 특정 서비스의 서버와 포트 지정 |
| **PTR** | IP 주소를 도메인 이름으로 역조회 |

## 상세 설명

### A (Address)
도메인 이름을 IPv4 주소에 연결한다.

```
example.com.    A    192.0.2.10
```

- `example.com`을 조회하면 `192.0.2.10`이라는 IPv4 주소를 반환한다.

### AAAA (IPv6 Address)
도메인 이름을 IPv6 주소에 연결한다.

```
example.com.    AAAA    2001:db8::10
```

- `example.com`을 조회하면 `192.0.2.10`이라는 IPv6 주소를 반환한다.

### CNAME (Canonical Name)
하나의 도메인 이름을 다른 도메인 이름으로 연결한다. (별칭)

```
www.example.com.    CNAME    example.com.
```

- `www.example.com`을 조회하면 최종적으로 `example.com`의 DNS 레코드를 조회하여 IP 주소를 찾는다.
- 서브 도메인 관리가 편하다. 설정 변경 시 메인 도메인의 레코드만 변경하면 된다.
- 실제 IP주소(A 레코드)를 찾기위해 DNS 조회를 최소 한 번 더 거쳐야 한다.

### MX (Mail Exchange)
도메인으로 수신되는 이메일을 처리할 메일 서버를 지정한다.

```
example.com.    MX    10 mail1.example.com.
example.com.    MX    20 mail2.example.com.
```

- `example.com`으로 이메일을 보내면 `mail1.example.com`또는 `mail2.example.com`으로 전달한다.
- 여러 메일 서버를 지정할 수 있으며, 숫자가 낮을수록 우선한다.

### TXT (Text)
도메인에 임의의 텍스트 데이터를 저장한다.

```
example.com.    TXT    "hello world"
```

주로 도메인 소유권 검증이나 이메일 보안 설정에 사용한다.

- SPF: 이메일 발송 서버 인증
- DKIM: 이메일 서명 검증
- DMARC: 이메일 인증 정책 설정
- 외부 서비스의 도메인 소유권 확인

### NS (Name Server)
해당 도메인의 DNS 정보를 관리하는 네임 서버를 지정한다.

```
example.com.    NS    ns1.example.com.
example.com.    NS    ns2.example.com.
```

- 해당 도메인의 DNS 정보를 어느 네임 서버에서 관리하는지 나타낸다.
- 일반적으로 여러 NS 레코드를 설정하여 가용성을 확보한다.
- 도메인 등록기관에서 네임 서버를 지정하는 과정과 연결된다.

### SOA (Start of Authority)
DNS 영역(Zone)의 기본 관리 정보를 나타낸다.

```
example.com.    SOA    ns1.example.com. admin.example.com. (
                         2026111001
                         3600
                         900
                         1209600
                         300
                     )
```

| 필드 | 값 | 의미 |
| --- | --- | --- |
| **MNAME** | `ns1.example.com.` | 해당 영역의 기본 네임 서버 |
| **RNAME** | `admin.example.com.` | 관리자 이메일 주소. `@`대신 `.`로 표기하며 `admin@example.com` 형태로 해석된다. |
| **Serial** | `2026111001` | 영역 데이터의 버전 번호. 보조 DNS 서버가 영역 변경을 감지하는 데 사용된다. |
| **Refresh** | `3600` | 보조 DNS 서버가 변경 사항을 확인하는 주기 |
| **Retry** | `900` | 갱신 실패 시 재시도 주기 |
| **Expire** | `1209600` | 갱신 실패가 지속될 경우 기존 데이터 만료 시간 |
| **Minimum** | `300` | 주로 부정 응답 캐싱에 사용하는 TTL |

### SRV (Service)
특정 서비스가 제공되는 서버와 포트를 지정한다.

```
_sip._tcp.example.com.    SRV    10 5 5060 sip.example.com.
```

| 필드 | 값 | 설명 |
| --- | --- | --- |
| **레코드 이름** | `_sip._tcp.example.com.` | 일반적으로 `_서비스명._프로토콜명` 형태로 이름을 구성한다. |
| **Priority** | `10` | Priority가 낮은 서버가 우선된다. |
| **Weight** | `5` | Priority가 동일할 때 트래픽을 나누는 비율이다. |
| **Port** | `5060` | |
| **Target 서버** | `sip.example.com` | |

### PTR (Pointer)
IP 주소를 도메인 이름으로 변환하는 데 사용한다. (역방향 조회)

- **A 레코드**: 도메인 → IP 주소
- **PTR 레코드**: IP 주소 → 도메인

```
10.2.0.192.in-addr.arpa.    PTR    example.com.
```
- IPv4 `192.0.2.10`을 역조회했을 때 `example.com`을 반환한다.
- 주로 메일 서버의 역방향 DNS 검증이나 네트워크 진단에 사용한다.

---
**Reference**
- https://www.cloudflare.com/learning/dns/dns-records/
