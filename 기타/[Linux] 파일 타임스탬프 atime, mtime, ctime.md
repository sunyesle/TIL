# [Linux] 파일 타임스탬프 atime, mtime, ctime

## atime(Access Time, 접근 시간)
파일의 내용이 마지막으로 읽힌 시각이다.

- **갱신 시점**: 파일 내용 읽기 (`cat`, `less`, `grep`, `cp`의 소스 파일 등)
- **갱신되지 않는 경우**: 파일 속성만 조회 (ls -l, stat)
- **주의**: 현대 리눅스는 성능 이슈로 `relatime` 옵션을 기본 적용하여, mtime이나 ctime보다 atime이 오래됐을 때만 갱신된다.

## mtime(Modify Time, 수정 시간)
파일의 내용이 마지막으로 변경된 시각이다. `ls -l`로 기본 출력되는 날짜이다.

- **갱신 시점**: 파일 내용 쓰기 (`vi` 저장, `echo` 리다이렉션, 프로그램이 파일에 데이터 기록)
- **갱신되지 않는 경우**: 파일 권한 변경(`chmod`), 소유자 변경(`chown`), 파일명 변경(`mv`)
- mtime이 변경되면 ctime도 반드시 함께 변경된다.

## ctime(Change Time, 변경 시간)
파일의 inode 정보가 마지막으로 변경된 시각이다. 파일 내용 뿐아니라 권한, 소유자, 링크 수 등 메타데이터 변경까지 모두 추적한다.

- **갱신 시점**: 파일 내용 변경(mtime 갱신 시 포함), chmod, chown, ln(하드링크 추가), mv(같은 파일 시스템 내)
- **갱신되지 않는 경우**: 파일 내용 읽기만 하는 경우.
- ctime은 시스템 내부에서 관리되므로 사용자가 임의로 변경할 수 없다. (보안 감사에서 중요한 이유)

## 확인 방법
```bash
stat filename.txt

  File: `filename.txt'
  Size: 1024
Access: 2026-10-25 12:03:59.759297994 +0900
Modify: 2026-10-25 12:03:59.759297994 +0900
Change: 2026-10-25 12:03:59.759297994 +0900
```

## 활용 예시

```bash
find /var/log -mtime -1 # mtime - 최근 1일 이내 내용이 수정된 파일
find /etc -ctime -7     # ctime - 최근 7일 이내 속성이 변경된 파일
find /home -atime +30   # atime - 30일 이상 접근하지 않은 파일


# 증분 백업 - mtime을 기준으로 변경된 파일만 선별
find /data -mtime -1 -type f | xargs tar -czf /backup/incremental_$(date +%F).tar.gz

# 보안 감사 - 최근 24시간 이내 ctime이 변경된 시스템 파일 탐지
find /etc /bin /usr/bin -ctime -1 -type f

# 스토리지 정리 - 180일 이상 접근하지 않은 파일 목록 출력
find /home -atime +180 -type f -ls
```

---
**Reference**
- https://frog11.tistory.com/127
