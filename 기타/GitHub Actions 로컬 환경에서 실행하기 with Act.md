# GitHub Actions 로컬 환경에서 실행하기 with Act
## Act
`act`는 Docker 컨테이너를 사용하여 로컬에서 GitHub Actions 워크플로우를 실행하도록 설계된 CLI 도구이다.

커밋/푸시할 필요 없이 로컬에서 빠르게 워크플로우를 테스트할 수 있다.
하지만 `act`가 실제 GitHub Actions 환경과 완전히 동일하지는 않기 때문에, GitHub에서 최종적으로 검증을 수행해야 한다.

## 설치
[공식 문서 설치 가이드](https://nektosact.com/installation/index.html)

## 초기 설정
```shell
? Please choose the default image you want to use with act:
- Large size image: ca. 17GB download + 53.1GB storage, you will need 75GB of free disk space, snapshots of GitHub Hosted Runners without snap and pulled docker images
- Medium size image: ~500MB, includes only necessary tools to bootstrap actions and aims to be compatible with most actions
- Micro size image: <200MB, contains only NodeJS required to bootstrap actions, doesn't work with all actions
```
`act`를 처음 실행하면 기본 runner 이미지 크기를 선택하라는 메시지가 표시된다.

각 이미지에 대한 자세한 내용은 [공식 문서](https://nektosact.com/usage/runners.html)에서 확인할 수 있다.

## 사용법
터미널에서 루트 디렉터리(`.github` 폴더가 포함된 디렉터리)로 이동한다.

### 워크플로우 조회
```bash
# 워크플로우 조회
act -l

# 특정 이벤트에 대한 워크플로우 조회
act -l push
```

### 워크플로우 실행
```bash
# 특정 이벤트 트리거. 지정하지 않을 경우 push로 실행된다.
act push
act pull_request
act schedule

# 특정 job 실행하기
act -j job_name
```

---
**Reference**
- https://github.com/nektos/act
- https://apidog.com/kr/blog/how-to-run-your-github-actions-locally-a-comprehensive-guide-kr/
- https://naya-an-tech.github.io/gitops-local-setting-with-act
