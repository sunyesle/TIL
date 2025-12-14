# Bash 스크립트 문법
## Shebang (#! interpreter)
쉘 스크립트 첫 줄에 `#!`로 시작하는 코드는 파일을 어떤 프로그램으로 실행시킬 지를 정의한다.

```bash
# 절대 경로로 지정
#!/bin/bash

# env파일의 PATH에서 지정
#!/usr/bin/env bash
```

## 변수
```bash
# 변수 선언
name="John"
number=10
command_res=`cat test.sh`

# ${...}, $... : 변수 참조
var="hello"
echo "${var}"
echo "$var"

# unset : 변수 삭제
var2="hello"
unset var2
echo "${var2}"

# readonly : 수정/삭제 불가 변수 선언
readonly var1="hello"
# var1="world" # Error 발생
```

### 특수한 변수들
| 변수      | 설명               |
| --------- | ---------------- |
| `$0`      | 실행되는 파일의 이름     |
| `$1`~`$n` | n번째 파라미터 값      |
| `$#`      | 파라미터 개수      |
| `$*`      | 모든 파라미터 (문자열)  |
| `$@`      | 모든 파라미터 (개별 유지. 반복문에서 주로 사용) |
| `$?`      | 마지막 명령 결과 (0:성공, 1:실패) |
| `$$`      | 현재 쉘 스크립트의 PID         |
| `$!`      | 마지막으로 실행한 백그라운드 PID |


## 배열
### 기본 배열
```bash
# 배열 선언
declare -a my_array

# 배열 초기화
my_array=("apple" "banana" "cherry")

# @ : 배열 전체 출력
# 배열의 값 개별로 인식한다. 출력 시에는 띄어쓰기로 구분해 출력한다. ("apple" "banana" "cherry")
echo "${my_array[@]}"

# * : 배열 전체 출력
# 배열의 모든 값을 하나로 묶어서 취급한다. ("apple banana cherry")
echo "${my_array[*]}"

# 인덱스 접근
echo "${my_array[0]} ${my_array[1]} ${my_array[2]}"

# 배열 갯수 출력
echo "${#my_array[@]}"

# 배열 값 변경
my_array[0]="pineapple"

# 배열 값 추가
my_array[4]="orange"

# 배열 특정 값 삭제
unset my_array[3]

# 배열 복사 후 추가
new_array=(${my_array[@]} "grape")
```

### 연관 배열
```bash
declare -A my_array

my_array=([name]="apple" [price]=2000)

my_array[category]=fruit

echo "${my_array[name]} / ${my_array[price]} / ${my_array[category]}" 
```

## 조건문
### if 문
```bash
a="foo"
b="bar"

# 기본 문법. 대괄호와 값 사이에는 반드시 공백이 있어야 한다.
if [ "$a" == "$b" ]
then
    echo "a equal b"
elif [ "$a" == "foo" ]
then
    echo "a equal foo"
else
    echo "a not equal b, a not equal foo"
fi

# 축약형
if [ "$a" == "$b" ]cherry then
    echo "a equal b"
elif [ "$a" == "foo" ]cherry then
    echo "a equal foo"
else
    echo "a not equal b, a not equal foo"
fi

# !를 앞에 써주면 반대 조건을 적용한다.
if ! [ "$a" == "foo" ]cherry then
    echo "a not equal foo"
fi
```

### 비교
#### 문자열 비교
```bash
# 같음: '=='
if [ "$a" == "$b" ]cherry then
    echo "a equal b"
fi

# 같지 않음: '!=' 
if [ "$a" != "$b" ]cherry then
    echo "a not equal b"
fi

# 비어있으면 true: '-z'
if [ -z "$a" ]cherry then
    echo "length of a = 0"
fi

# 내용이 있으면 true: '-n'
if [ -n "$a" ]cherry then
    echo "length of a > 0"
fi
```
**문자열 비교 연산자**
| 연산자  | 설명          |
| ---- | --------------|
| `=`  | 같음          |
| `!=` | 다름          |
| `>`  | 더 큰 문자열  |
| `<`  | 더 작은 문자열 |
| `-n` | 문자열 길이 > 0  |
| `-z` | 문자열 길이 == 0 |


#### 숫자 비교
`[]`와 숫자 비교 연산자를 사용하거나, `(())`와 부등호를 사용하여 값을 비교할 수 있다. `(())` 내에서는 변수의 `$`를 생략할 수 있다.
```bash
# [] 사용
if [ "$a" -lt "$b" ]cherry then
    echo "a less than b"
fi

# (()) 사용
if (( a < b )); then
    echo "a less than b"
fi
```

**숫자 비교 연산자**
| 연산자 | 설명     |
|-------|--------|
| `-eq` | 같음     |
| `-ne` | 다름     |
| `-lt` | 작음     |
| `-gt` | 큼      |
| `-le` | 작거나 같음 |
| `-ge` | 크거나 같음 |

#### 파일 비교
```bash
# 파일 존재 유무 판단
if [ -f log_2021.txt ]; then
    echo "log file exists"
fi

# 디렉토리 존재 유무 판단
if [ -d log ]; then
    echo "log directory existed"
fi

# 파일이 실행 가능함
if ! [ -x prepare-commit-msg ]; then
    chmod 755 prepare-commit-msg
fi 

# 디렉토리 존재 유무 판단
if [ file1.txt -nt file2.txt ]; then
    echo "file1 newer"
fi
```

**파일 테스트 연산자**
| 연산자 | 설명     |
|-----|--------|
| `-e` | 파일 또는 디렉토리 존재     |
| `-f` | 파일 존재    |
| `-d` | 디렉토리 존재   |
| `-L` | 심볼릭 링크 여부    |
| `-s` | 파일 크기가 0보다 큼 |
| `-r` | 읽기 가능 |
| `-w` | 쓰기 가능 |
| `-x` | 실행가능 |
| `-nt`  | 더 최신 파일이면 true |
| `-ot` | 더 오래된 파일이면 true |
| `-ef` | 같은 파일이면 true |

## 반복문
### for 문
```bash
# 기본형
for item in "apple" "banana" "cherry"
do
    echo ${item}
done

# 축약형
for item in "apple" "banana" "cherry"; do
    echo ${item}
done

# 배열 사용
FRUITS=("apple" "banana" "cherry")
for item in ${FRUITS[@]}; do
    echo ${item}
done

# break
for item in ${FRUITS[@]}; do
    if [ ${item} == "banana" ]; then
        echo "stop"
        break
    else
        echo "${item}"
    fi
done

# continue
for item in ${FRUITS[@]}; do
    if [ ${item} == "banana" ]; then
        continue
    else
        echo "${item}"
    fi
done

# C, Java like
for ((i=0; i<10; i++)); do
    echo "$i"
done

# 파일 접근
for file in /home/jin/*; do
    echo "$file"
done
```

### while 문
```bash
cnt=0
while (( ${cnt} < 10 )); do
    echo ${cnt}
    cnt=$(( ${cnt}+1 ))
done
```

## 함수
```bash
# 선언
function test_func() {
    echo "hello"
}

# function은 생략해도 된다.
test_func2() {
    echo "world"
}

# 호출
test_func
test_func2
```

### 지역 변수
기본적으로 전역변수로 선언되며, 함수 내에서 `local`을 붙여서 지역변수를 선언할 수 있다.
```bash
# 전역변수 선언
name="a"
echo ${name} # 결과: a

var_test() {
    # 같은 이름으로 지역변수 선언. local이 없으면 전역 변수를 수정한다.
    local name="b"
    echo ${name}
}

var_test # 결과: b

# 지역변수를 사용했으므로 전역변수에는 영향이 없다.
echo ${name} # 결과: a
```

## eval
스크립트 중간에 명령어나 다른 파일을 실행시킬 수 있다.
```bash
eval "ls -al"
eval "./test.sh"
```
