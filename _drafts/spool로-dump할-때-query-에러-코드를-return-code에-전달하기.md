# SPOOL로 Dump 할 때 Query 에러 코드를 Return Code에 전달하기

## Problem
```sql
SET HEADING OFF
SET PAGESIZE 0
SET LINESIZE  1000
SET ECHO OFF
SET TERM OFF
SET FEEDBACK OFF
SET TRIMOUT ON
SET TRIMSPOOL ON
SET ARRAYSIZE 5000

SPOOL result/data.dump
-- 나는 실 서버에서만 동작해요
SELECT
    A            || ',' ||
    B            || ',' ||
    C            || ',' ||
    D
FROM
    TEST.I_AM_ONLY_EXISTS_IN_THE_REAL_DB
;
SPOOL OFF

EXIT
```
`sqlplus` 로 쿼리 `test.sql`를 `sqlplus @test.sql`와 같이 실행하면 에러가 발생하는데 `echo $?`에서는 당당히(?) `0`을 내밷고 있다.

어찌하면 `SQL 에러를 return code에 전달할 수 있을까?` 싶어서 찾아봤다.

## Solution
답은 간단했다.

`EXIT SQL.SQLCODE` 로 종료하면 쿼리 에러 코드가 return code로 전달된다.

알면 쉽고 모르면 낭패인 그런 문제 아닐지...

## Reference
* https://docs.oracle.com/database/121/SQPUG/ch_twelve023.htm#SQPUG044
* https://docs.oracle.com/database/121/SQPUG/ch_twelve052.htm#SQPUG135