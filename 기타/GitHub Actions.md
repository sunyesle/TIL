# GitHub Actions
## GitHub Actions란?
GitHub Actions는 빌드, 테스트 및 배포 파이프라인을 자동화할 수 있는 CI/CD 플랫폼이다.

### CI/CD
**CI/CD**는 지속적 통합(CI)과 지속적 배포(CD)를 결합한 소프트웨어 개발 방법론이다.

조기에 결함을 발견하고, 생산성을 높이며 더 빠른 배포 주기를 제공하는 것을 목표로 하며,
애플리케이션을 빌드, 테스트, 배포하는 모든 과정을 자동화한다.

- **CI**(**Continuous Integration, 지속적 통합**): 변경된 코드를 빌드하고 테스트하는 과정을 자동화한다.
- **CD**(**Continuous Deployment, 지속적 배포**): 통합된 코드를 실제 운영 환경에 배포하는 과정을 자동화한다.

## GitHub Actions 구성 요소

### 워크플로우 (Workflow)
자동화된 프로세스를 정의한 yaml 파일이다.
하나 이상의 Job으로 구성된다.
### 이벤트 (Event)
Workflow를 언제 실행할지 결정하는 트리거이다.
`push`, `pull_request`, `schedule`, `workflow_dispatch` 등이 있다.
### 러너 (Runner)
Workflow가 실행될 인스턴스이다.
GitHub에서 제공하는 `GitHub-hosted Runner`, 사용자가 직접 구축한 `Self-hosted Runner`가 있다.
### 작업 (Job)
독립된 Runner에서 실행되는 하나의 처리 단위이다.
기본적으로 모든 Job은 병렬로 실행된다.
### 스탭 (Step)
Job 내부에서 순차적으로 실행되는 작업 단계이다.
각 Step에서는 명령어 실행, action 호출 등을 할 수 있다. 
### 액션 (Action)
자주 사용되는 작업을 재사용할 수 있도록 패키징한 단위이다.
GitHub Marketplace에서 가져와서 사용하거나 직접 개발자가 작성하여 사용할 수 있다.

---
**Reference**<br>
- https://en.wikipedia.org/wiki/CI/CD
- https://blog.kakaocloud.com/138
