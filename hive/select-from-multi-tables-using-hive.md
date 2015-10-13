# Hive에서 Multi Table Select 하기

## 이런걸 하고 싶었다
그리고 (당연히) 될 줄 알았다.

```
SELECT
      t1.value
    , t2.value
FROM
    t1
  , t2
WHERE
  (t1.value = t2.value);
```


## JOIN 해야해요
아래 처럼 JOIN 해야 한다.

```
SELECT
    t1.value
  , t2.value
  , t3.value
FROM
  t1
JOIN
  t2
ON
  t1.value=t2.value
JOIN
  t3
ON
  t1.value=t3.value

;
```

## 난 되는데?
내가 쓰는 Hive 버전이 낮아서 그런거임.
(알고 보니 Hive 0.10을 쓰고 있었...)

```
Version 0.13.0+: Implicit join notation

Implicit join notation is supported starting with Hive 0.13.0 (see HIVE-5558). This allows the FROM clause to join a comma-separated list of tables, omitting the JOIN keyword. For example:

SELECT *
FROM table1 t1, table2 t2, table3 t3
WHERE t1.id = t2.id AND t2.id = t3.id AND t1.zipcode = '02535';

```


## Reference
* http://stackoverflow.com/questions/20059340/hive-grab-from-two-tables-without-join
* https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Joins
