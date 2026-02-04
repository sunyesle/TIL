# JdbcTemplate KeyHolder로 자동 생성된 키 값 가져오기
## KeyHolder
`KeyHolder`는 Spring JDBC에서 자동 생성된 키 값을 가져올 때 사용하는 인터페이스이다.
일반적으로 키는 `List<Map>` 형태로 반환되며, JDBC 드라이버 구현체에 따라 해당 기능을 지원하지 않을 수도 있다.

## 사용 예시
### insert
```java
KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update("INSERT INTO member (name) VALUES (:name)", param, keyHolder);
Number key = keyHolder.getKey();
long memberId = key.longValue();
```

### batch insert
```java
KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.batchUpdate("INSERT INTO member (name) VALUES (:name)", params, keyHolder);
List<Map<String, Object>> keyList = keyHolder.getKeyList();
for (int i = 0; i < teamRequests.size(); i++) {
    Number key = (Number) keyList.get(i).values().iterator().next();
    long memberId = key.longValue();
}
```

## 동작 과정
KeyHolder로 키 값을 가져오는 내부 동작 과정을 확인해 보자.

우선 [`org.springframework.jdbc.core.JdbcTemplate`](https://github.com/spring-projects/spring-framework/blob/main/spring-jdbc/src/main/java/org/springframework/jdbc/core/JdbcTemplate.java) 클래스의 `update()` 메서드를 확인해보면,
`executeUpdate()`로 DB에서 쿼리를 실행한 뒤 `storeGeneratedKeys()` 메서드를 호출하고 있다.
```java
@Override
public int update(PreparedStatementCreator psc, KeyHolder generatedKeyHolder)
        throws DataAccessException {

    Assert.notNull(generatedKeyHolder, "KeyHolder must not be null");
    logger.debug("Executing SQL update and returning generated keys");

    return updateCount(execute(psc, ps -> {
        int rows = ps.executeUpdate(); // DB에서 쿼리 실행
        generatedKeyHolder.getKeyList().clear();
        storeGeneratedKeys(generatedKeyHolder, ps, 1); // storeGeneratedKeys 메서드 호출
        if (logger.isTraceEnabled()) {
            logger.trace("SQL update affected " + rows + " rows and returned " + generatedKeyHolder.getKeyList().size() + " keys");
        }
        return rows;
    }, true));
}
```

`storeGeneratedKeys()` 메서드 내부에서는 `getGeneratedKeys()`를 호출하여 생성된 키 값을 가져오고, 이를 `List<Map<String, Object>>` 형태로 변환하여 반환한다.
```java
private void storeGeneratedKeys(KeyHolder generatedKeyHolder, PreparedStatement ps, int rowsExpected)
        throws SQLException {

    List<Map<String, Object>> generatedKeys = generatedKeyHolder.getKeyList();
    ResultSet keys = ps.getGeneratedKeys();
    if (keys != null) {
        try {
            RowMapperResultSetExtractor<Map<String, Object>> rse =
                    new RowMapperResultSetExtractor<>(getColumnMapRowMapper(), rowsExpected);
            generatedKeys.addAll(result(rse.extractData(keys)));
        }
        finally {
            JdbcUtils.closeResultSet(keys);
        }
    }
}
```

여기까지가 Spring JDBC의 코드이다.

`PreparedStatement`는 JDBC 인터페이스로 구현은 JDBC 드라이버에게 맡긴다.<br>
현재 사용 중인 JDBC 드라이버 구현체인 `mysql-connector-j` 기준으로 `getGeneratedKeys()` 메서드를 확인해 보았다.

[`com.mysql.cj.jdbc.StatementImpl`](https://github.com/mysql/mysql-connector-j/blob/release/9.x/src/main/user-impl/java/com/mysql/cj/jdbc/StatementImpl.java) 클래스의
`getGeneratedKeys() → getGeneratedKeysInternal() → getGeneratedKeysInternal(long numKeys)` 순으로 타고 들어간다.

수정된 행 개수와 마지막 삽입 ID를 이용해 각 행의 ID 값을 계산하여 반환한다.
```java
protected ResultSetInternalMethods getGeneratedKeysInternal() throws SQLException {
    long numKeys = getLargeUpdateCount(); // ResultSet으로부터 수정된 행 개수를 가져온다.
    return getGeneratedKeysInternal(numKeys);
}

protected ResultSetInternalMethods getGeneratedKeysInternal(long numKeys) throws SQLException {
    Lock connectionLock = checkClosed().getConnectionLock();
    connectionLock.lock();
    try {
        String encoding = this.session.getServerSession().getCharsetSettings().getMetadataEncoding();
        int collationIndex = this.session.getServerSession().getCharsetSettings().getMetadataCollationIndex();
        Field[] fields = new Field[1];
        fields[0] = new Field("", "GENERATED_KEY", collationIndex, encoding, MysqlType.BIGINT_UNSIGNED, 20);

        ArrayList<Row> rowSet = new ArrayList<>();

        long beginAt = getLastInsertID(); // ResultSet으로부터 마지막 삽입 ID를 가져온다.

        if (this.results != null) {
            ...생략...

            if (beginAt != 0 && numKeys > 0) {
                for (int i = 0; i < numKeys; i++) {
                    byte[][] row = new byte[1][];

                    // ID 값을 byte 배열로 변환
                    if (beginAt > 0) {
                        // 양수인 경우
                        row[0] = StringUtils.getBytes(Long.toString(beginAt));
                    } else {
                        // 음수인 경우
                        byte[] asBytes = new byte[8];
                        asBytes[7] = (byte) (beginAt & 0xff);
                        asBytes[6] = (byte) (beginAt >>> 8);
                        asBytes[5] = (byte) (beginAt >>> 16);
                        asBytes[4] = (byte) (beginAt >>> 24);
                        asBytes[3] = (byte) (beginAt >>> 32);
                        asBytes[2] = (byte) (beginAt >>> 40);
                        asBytes[1] = (byte) (beginAt >>> 48);
                        asBytes[0] = (byte) (beginAt >>> 56);

                        BigInteger val = new BigInteger(1, asBytes);

                        row[0] = val.toString().getBytes();
                    }

                    // ID 추가
                    rowSet.add(new ByteArrayRow(row, getExceptionInterceptor()));
                    
                    // 다음 ID 계산
                    // 여러 건의 데이터를 저장하는 경우 LastInsertID는 첫 번째 행의 ID를 반환한다.
                    // beginAt에 MySQL에 설정된 auto_increment_increment 값만큼 더해서 다음 ID 값을 추측한다.
                    beginAt += this.connection.getAutoIncrementIncrement();
                }
            }
        }

        ResultSetImpl gkRs = this.resultSetFactory.createFromResultsetRows(ResultSet.CONCUR_READ_ONLY, ResultSet.TYPE_SCROLL_INSENSITIVE,
                new ResultsetRowsStatic(rowSet, new DefaultColumnDefinition(fields)));

        return gkRs;
    } finally {
        connectionLock.unlock();
    }
}
```

이때, 수정된 행 개수와 마지막 삽입 ID는 [`com.mysql.cj.jdbc.result.ResultSetImpl`](https://github.com/mysql/mysql-connector-j/blob/release/9.x/src/main/user-impl/java/com/mysql/cj/jdbc/result/ResultSetImpl.java)로부터 가져오고 있다.
`ResultSetImpl`의 생성자를 확인해보면 상위 클래스인 `NativeResultset`의 생성자를 호출하며, 전달받은 `OkPacket`으로 필드를 초기화하는 것을 확인할 수 있다.
```java
/**
 * Create a result set for an executeUpdate statement.
 */
public NativeResultset(OkPacket ok) {
    this.updateCount = ok.getUpdateCount();
    this.updateId = ok.getUpdateID();
    this.serverInfo = ok.getInfo();
    this.columnDefinition = new DefaultColumnDefinition(new Field[0]);
}
```

[**OkPacket**](https://dev.mysql.com/doc/dev/mysql-server/8.0.45/page_protocol_basic_ok_packet.html)은 명령이 성공적으로 완료되었다는 것을 알리기 위해 MySQL 서버에서 클라이언트로 보내는 패킷이다.

<img width="506" height="551" alt="OkPacket" src="https://github.com/user-attachments/assets/31ae1acd-608c-4724-92ca-478101984d1b" />

드라이버는 해당 패킷의 정보를 추출하여 수정된 행 개수(**affected_rows**)와 마지막 삽입 ID(**last_insert_id**)를 알아낼 수 있다.

---
**Reference**
- https://jonghoonpark.com/2025/01/12/keyholder-jdbc-template

