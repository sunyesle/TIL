# [Git] 특정 폴더만 clone 하기

## 명령어
```bash
# git 초기화
git init

# remote 연결
git remote add origin [git repository 주소]

# sparsecheckout 기능 활성화
git config core.sparsecheckout true

# 받을 경로 설정
echo '[경로]/*' >> .git/info/sparse-checkout

# 데이터 가져오기
git pull origin [브랜치명]
```

## 예시
`examples` 저장소 내의 `servlet-example` 폴더만 내려받는다.
```bash
git init
git remote add origin https://github.com/sunyesle/examples.git
git config core.sparsecheckout true
echo 'servlet-example/*' >> .git/info/sparse-checkout
git pull origin main
```
