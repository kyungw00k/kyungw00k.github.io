# SPOOL로 Dump 할 때 Query 에러 코드를 Return Code에 전달하기

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
SELECT
    DT            || ',' ||
    ADUNIT_ID     || ',' ||
    DSP           || ',' ||
    PRODUCT_TYPE  || ',' ||
    ADUNIT_STATUS || ',' ||
    CLK_ALL       || ',' ||
    CLK_VALID     || ',' ||
    IMP_ALL       || ',' ||
    IMP_VALID     || ',' ||
    CASH_BIZ
FROM
    TEST.I_AM_ONLY_EXISTS_IN_THE_REAL_DB
;
SPOOL OFF

EXIT SQL.SQLCODE
```