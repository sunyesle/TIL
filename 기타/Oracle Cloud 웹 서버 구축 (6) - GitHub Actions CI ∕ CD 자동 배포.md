# Oracle Cloud 웹 서버 구축 (6) - GitHub Actions CI/CD 자동 배포
현재는 코드가 변경될 때마다 애플리케이션을 빌드하고 서버에 직접 재배포해야 한다.
이런 반복적인 작업을 자동화해 주는 것이 바로 **CI/CD**(**지속적 통합 및 배포**)이다.

GitHub Actions를 활용하여 CI/CD 파이프라인을 구축해 보자.

## GitHub Secrets 등록
GitHub Actions에서 빌드한 애플리케이션을 서버에 배포하기 위해서는 서버 접속 정보가 필요하다.
이런 정보를 코드에 직접 적는 것은 보안상 위험하므로 GitHub Secrets에 등록하여 사용한다.

**GitHub Secrets 등록 방법**
1. GitHub Repository 메인페이지 접속
2. Settings 탭 클릭
3. Secrets and variables 슬라이드 바에서 Actions 클릭
4. Secrets 탭에서 New repository secret 클릭
5. Name과 Secret을 입력하고 Add secret 클릭 

등록한 값들은 이후 작성할 워크플로우에서 `${{ secrets.<NAME> }}` 형태로 접근할 수 있다.

웹 서버 접속 정보와 기존에 `start.sh`파일에 하드코딩 해두었던 환경변수 정보를 Secrets로 등록한다.
- `SSH_HOST`: <웹 서버 public IP>
- `SSH_USERNAME`: ubuntu
- `SSH_KEY`: <ssh_key_web.key의 내용>
- `DB_URL`: jdbc:mysql://<DB 서버 private IP>:3306/mydb
- `DB_USERNAME`: testuser
- `DB_PASSWORD`: password1111

## 애플리케이션 실행 스크립트 수정
`start.sh` 파일에서 환경 변수 설정 부분을 삭제한다.

스크립트 파일에 민감 정보가 평문으로 저장되는 보안 위험을 제거하고, 파이프라인에서 환경 변수를 유연하게 관리할 수 있다.
```bash
# 다음 내용을 삭제한다.
export DB_URL="jdbc:mysql://<DB 서버 private IP>:3306/mydb"
export DB_USERNAME="testuser"
export DB_PASSWORD="password1111"
```

## 워크플로우 작성
1. GitHub Repository 메인페이지 접속
2. Actions 탭 클릭
3. Java with Gradle의 Configure 클릭

Java with Gradle의 내용을 기반으로 다음과 같이 워크플로우를 작성했다.
```yml
name: CI CD

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      # GitHub 저장소 코드를 체크아웃
      - uses: actions/checkout@v4

      # JDK 21 환경 설정
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      # Gradle 캐싱 및 환경 설정
      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@af1da67850ed9a4cedd57bfd976089dd991e2582 # v4.0.0

      # gradlew 실행 권한 부여
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # Gradle 빌드 실행
      - name: Build with Gradle Wrapper
        run: ./gradlew build

      # 빌드된 JAR 파일명을 app.jar로 변경
      - name: Change build file name
        run: mv ./build/libs/*SNAPSHOT.jar ./app.jar

      # SCP로 빌드된 JAR 파일을 서버에 업로드
      - name: SCP
        uses: appleboy/scp-action@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          source: app.jar
          target: /home/ubuntu/app/temp

      # SSH로 서버 접속 후 애플리케이션 교체 및 재시작
      - name: SSH
        uses: appleboy/ssh-action@v1
        env:
          DB_URL: ${{ secrets.DB_URL }}
          DB_USERNAME: ${{ secrets.DB_USERNAME }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          envs: DB_URL,DB_USERNAME,DB_PASSWORD
          script: |
            set -e
            /home/ubuntu/app/stop.sh
            mv /home/ubuntu/app/temp/app.jar /home/ubuntu/app/current/app.jar
            /home/ubuntu/app/start.sh
            rm -f /home/ubuntu/app/temp/app.jar
```
