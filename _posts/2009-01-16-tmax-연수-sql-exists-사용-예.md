---
layout: post
title: "[Tmax 연수] SQL exists 사용 예"
date: 2009-01-16 09:06:08.000000000 +09:00
type: post
published: true
status: publish
categories:
- Database
tags:
- Oracle 9i
- SQL
- Tmax 연수
---

과제를 했지만 상대 서버에서 메일을 거부당해서 다른 숙제로 대체...
간단한거지만 혹시나 모르는 분이 있다면 참고하세요! ^^


Oracle 9i에서 테스트했습니다.

```
-- select 칼럼명1, 칼럼명2,... from [테이블] where exists ( select 문 )
--
-- exists 오른편에 있는 select문을 통해 나오는 결과값이 [테이블] 자체에 존재하면 출력 결과가 나온다.
-- 단, IN은 해당하는 값이 있으면 그 값이 있는 행을 출력하지만,
-- exists는 참이면 where절 왼편까지의 select query의 결과가 나오고,
-- 없으면 결과가 나오지 않는다.
--
-- exists ( select dname from dept where dname='청소부')에서 청소부가 없으니 "거짓"이다.
--
-- Sample Query
--

select * from dept where exists ( select dname from dept where dname='영업부');

DEPTNO|DNAME     |LOC
----------|----------|----------
        10|총무부    |서울
        20|영업부    |대전
        30|전산부    |부산
        40|관리부    |광주

SQL> select * from dept where exists ( select dname from dept where dname='청소부');

선택된 레코드가 없습니다.

-- select 칼럼명1, 칼럼명2,... from 테이블 where not exists ( select 문 )
--
-- not exists 오른편에 있는 select문을 통해 나오는 결과값이 [테이블] 자체에 존재하지 않으면 출력 결과가 나온다.
-- 단, NOT IN은 해당하는 값이 없으면 행을 출력하지만,
-- not exists는 참이면 where절 왼편까지의 select query의 결과가 나오고,
-- 거짓이면 결과가 나오지 않는다.
--
-- not exists ( select dname from dept where dname='청소부')에서 청소부가 없으니 "참"이다.

--
-- Sample Query
--

SQL> select * from dept where not exists ( select dname from dept where dname='청소부');

DEPTNO|DNAME     |LOC
----------|----------|----------
        10|총무부    |서울
        20|영업부    |대전
        30|전산부    |부산
        40|관리부    |광주

SQL> select * from dept where exists ( select dname from dept where dname='관리부');

선택된 레코드가 없습니다.
```
