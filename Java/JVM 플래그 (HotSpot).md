# JVM 플래그 (HotSpot)
OpenJDK, Oracle JDK 등 HotSpot 계열 JVM에서 제공하는 플래그를 알아보자.

## 플래그 카테고리
| 접두사   | 예시                                          | 설명           |
|-------|---------------------------------------------|--------------|
| `-`   | `-version`, `-jar`                          | 표준 옵션        |
| `-X`  | `-Xms512m`, `-Xmx2g`                        | 비표준 확장 옵션    |
| `-XX` | `-XX:+PrintFlagsFinal`, `-XX:MaxHeapSize=2` | 진단/튜닝용 내부 옵션 |

## -XX 플래그
### 사용 방법
- **Boolean 플래그**: `-XX:+옵션`(활성화), `-XX:-옵션`(비활성화)
- **Value 플래그**: `-XX:옵션=값`

### -XX:+PrintFlagsFinal
현재 사용 중인 JVM의 모든 플래그와 그 값을 출력한다.
```bash
$ java -XX:+PrintFlagsFinal -version

[Global flags]
int ActiveProcessorCount                     = -1                                        {product} {default}
uintx AdaptiveSizeDecrementScaleFactor         = 4                                         {product} {default}

...생략...

java version "18.0.2.1" 2022-08-18
Java(TM) SE Runtime Environment (build 18.0.2.1+1-1)
Java HotSpot(TM) 64-Bit Server VM (build 18.0.2.1+1-1, mixed mode, sharing)
```

다음과 같이 특정 키워드로 검색할 수도 있다.
```bash
$ java -XX:+PrintFlagsFinal -version | grep 'GC'

    uintx AdaptiveSizeMajorGCDecayTimeScale        = 10                                        {product} {default}
     uint ConcGCThreads                            = 2                                         {product} {ergonomic}

...생략...

java version "18.0.2.1" 2022-08-18
Java(TM) SE Runtime Environment (build 18.0.2.1+1-1)
Java HotSpot(TM) 64-Bit Server VM (build 18.0.2.1+1-1, mixed mode, sharing)

```

### -XX:+PrintCommandLineFlags
JVM이 실행될 때 사용된 커맨드 라인 플래그를 출력한다. 사용자 지정 또는 JVM이 자동으로 조정한 값들을 확인할 수 있다. 
```bash
$ java -XX:+PrintCommandLineFlags -version
-XX:ConcGCThreads=2 -XX:G1ConcRefinementThreads=6 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=535773568 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=8572377088 -XX:MinHeapSize=6815736 -XX:+PrintCommandLineFlags -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation
java version "18.0.2.1" 2022-08-18
Java(TM) SE Runtime Environment (build 18.0.2.1+1-1)
Java HotSpot(TM) 64-Bit Server VM (build 18.0.2.1+1-1, mixed mode, sharing)
```
---
**Reference**
- https://mangchhe.github.io/java/2024/03/04/how-to-find-jvm-flag/?utm_source=chatgpt.com
- https://linux.systemv.pe.kr/programming/유용한-jvm-플래그-part-2-플래그-카테고리들과-jit-컴파일/
