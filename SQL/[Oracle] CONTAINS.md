# [Oracle] CONTAINS
`CONTAINS`는 대용량 텍스트 검색을 빠르게 처리하기 위해 사용하는 연산자로, 사용하려면 컬럼에 Oracle Text 인덱스가 미리 생성되어 있어야 한다.
`CONTAINS`는 선택된 각 행에 대한 관련성 점수를 반환하며, 이 점수는 `SCORE` 연산자를 사용하여 얻을 수 있다.

## 예시
```sql
SELECT 
    SCORE(1) AS score, -- CONTAINS의 세 번째 인자(1)와 동일한 번호 지정
    board_title, 
    board_content
FROM board
WHERE CONTAINS(board_content, 'ORACLE%', 1) > 0
ORDER BY score DES
```

## Oracle Text CONTAINS 쿼리

### 연산자
자세한 내용은 [공식문서](https://docs.oracle.com/en/database/oracle/oracle-database/19/ccref/oracle-text-CONTAINS-query-operators.html)에서 확인할 수 있다.

- `blue & green & red`
- `cats | dogs`
- `animals ~ dogs`
- `scal%`
- `_ing`

> 와일드카드 문자(`%`)를 단독으로 사용하는 경우 스톱워드로 취급되며, 쿼리에서 제외된다.

### 이스케이프 문자
- **`{}`**
   - 중괄호 사이의 내용을 이스케이프한다. (여는 중괄호 문자 포함)
   - 이스케이프 처리된 문자에 닫는 중괄호 문자를 포함하려면 `}}`를 사용한다.
   - 중괄호를 사용하여 단어 내의 개별 문자를 이스케이프 처리하면 토큰이 분리된다. (ex: `a{-}b` → `a - b`)
- **`\`**
   - 백슬래시 뒤의 단일 문자를 이스케이프한다.

## 관련 오류
```
ORA-29902: ODCIIndexStart() 루틴을 수행시 오류가 생겼습니다
ORA-20000: Oracle Text error:
DRG-51030: wildcard query expansion resulted in too many terms
29902. 00000 -  "error in executing ODCIIndexStart() routine"
*Cause:    The execution of ODCIIndexStart routine caused an error.
*Action:   Examine the error messages produced by the indextype code and
           take appropriate action.
```

## 검색어 처리 예시
```java
private static final Pattern ORACLE_TEXT_SPECIAL = Pattern.compile("[=&|~;?%_$!<>*()\\[\\]{}\\\\-]");

private static String buildOracleTextQuery(String keyword) {
    StringTokenizer st = new StringTokenizer(keyword);
    StringBuilder sb = new StringBuilder();

    while (st.hasMoreTokens()) {
        String token = st.nextToken();

        // 각 토큰을 AND 조건으로 연결한다.
        if (sb.length() > 0) {
            sb.append(" & ");
        }

        // 예약문자 이스케이프 처리
        String escapedToken = escapeOracleTextToken(token);

        if (isNumber(token) || token.length() == 1) {
            // 너무 짧은 토큰이나 숫자만으로 이루어진 토큰은 prefix 검색을 적용하지 않는다. (DRG-51030 오류 발생 가능성이 높음)
            sb.append(escapedToken);
        } else {
            sb.append(escapedToken).append("%");
        }
    }

    return sb.toString();
}

private static String escapeOracleTextToken(String token) {
    return ORACLE_TEXT_SPECIAL.matcher(token)
            .replaceAll("\\\\$0");
}

private static boolean isNumber(String token) {
    return token.matches("\\d+");
}
```

---
**Reference**
- https://docs.oracle.com/en/database/oracle/oracle-database/19/ccref/special-characters-in-oracle-text-queries.html
