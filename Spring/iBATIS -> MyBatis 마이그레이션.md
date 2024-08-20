# iBATIS -> MyBatis 마이그레이션

## xml 변환툴
1. https://github.com/mybatis/ibatis2mybatis 에서 코드를 내려받는다.
2. source폴더에 iBATIS sqlMap xml 파일을 넣는다.
3. build.xml이 있는 폴더에서 ant 명령어를 실행한다. (ant 설치: https://ant.apache.org/)
4. destination에서 변환된 MyBatis mapper xml 파일을 확인할 수 있다.
5. 자동 변환에 실패한 부분을 수동 변환해 준다.

## xml 수동 변환 케이스

### typeAlias
typeAlias 태그를 XML 설정파일(mybatis-config.xml)로 옮긴다.
```xml
<!-- iBATIS -->
<typeAlias alias="Product" type="com.example.demo.model.Product" />
```
```xml
<!-- MyBatis -->
<configuration>
  <typeAliases>
    <typeAlias alias="Product" type="com.example.demo.model.Product" />
  </typeAliases>
</configuration>
```

### cache
[MyBaits 공식문서](https://mybatis.org/mybatis-3/ko/sqlmap-xml.html#cache)를 참고하여 변환하였다.

```xml
<!-- iBATIS -->
<cacheModel id="jongCodeListCache" type="LRU">
  <flushInterval minutes="5"/>
  <property name="size" value="1024" />
</cacheModel>
```

```xml
<!-- MyBatis -->
<cache
  eviction="LRU"
  flushInterval="300000"
  size="1024"
  readOnly="true"/>
```

---
**Reference**<br>
- https://expert0226.tistory.com/297
- http://webprogramer.kr/blog/P000000379/post.do
- https://papababo.tistory.com/entry/%EC%A0%84%EC%9E%90%EC%A0%95%EB%B6%80-37-ibatis-Mybatis-%EB%B3%80%ED%99%98%EC%9D%BC%EC%A7%80-1%ED%83%84
