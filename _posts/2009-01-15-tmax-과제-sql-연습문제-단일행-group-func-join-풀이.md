---
layout: post
title: "[Tmax 연수] SQL 연습문제 - 단일행(9) / GROUP FUNC(5) / JOIN (7) 풀이"
date: 2009-01-15 20:48:29.000000000 +09:00
type: post
published: true
status: publish
categories:
- Database
tags:
- Tmax 연수
- Oracle 9i
- SQL
---

### 단일행 함수

```sql
-- 1. 현재까지 근무일수
select ename, hiredate,
      floor(abs(sysdate-hiredate)/7)||'주 '||
      floor(mod(abs(sysdate-hiredate),7))||'일' "근무일수"
from emp;

-- 2. 현재까지 근무월수
select ename, trunc(months_between(sysdate,hiredate))
from emp;

-- 3. 입사일자로부터 돌아오는 금요일
select ename, next_day(hiredate,'금')
from emp;

-- 4.
select ename, hiredate, last_day(hiredate)-hiredate
from emp;

-- 5.
select ename, hiredate, 
       to_char(hiredate,'dd Mon yyyy'),
        to_char(hiredate,'yyyy"년" mm"월" dd"일"')
from emp;

-- 6.
select ename, hiredate, floor(sal/12/to_number(to_char(last_day(hiredate),'dd'))*(last_day(hiredate)-hiredate))
from emp;

-- 7. (잘모르겠습니다!!!)
select
    empno
    , ename
    , sal
    , (case when mgr is null then 'NO MGR' else to_char(mgr) end) MGR
from
    emp
where
    empno=&empno
    and ename='&ename'
    and sal = &sal;

-- 8.
select
    empno
    , ename
    , job
    , sal
    , nvl(comm,0)
    , (case job when 'ANALYST' then sal*0.1
                when 'CLERK' then sal*0.15
                when 'MANAGER' then sal*0.20
                else comm
        end) BONUS
from
    emp;

-- 9.
select
    empno
    , ename
    , job
    , sal
    , nvl(comm,0)
    , (case when sal >= 3000 then 1500
            when sal >= 2000  then 1000
            else 800
            end) BONUS
from
    emp;

-- GROUP FUNC 문

-- 1.
select avg(sal), max(sal), min(sal), sum(sal) from emp where job='SALESMAN';

-- 2.
select count(*) 인원수, count(comm) "널 아닌 보너스인원", avg(nvl(comm,0)) "보너스평균", count(job) "부서의 수" from emp;

-- 3.
select job, sum(sal) from emp where job <> 'SALESMAN' group by job having sum(sal) > 5000 order by 2 desc;

-- 4.
select deptno, job, count(*), mgr from emp group by grouping sets ( (deptno,job), (deptno,mgr) );

-- 5. 부서별 업무별 급여의 평균을 출력하는 SELECT 문장을 작성하라(세자리구분기호,정수에서반올림)
select job,
 to_char(round(avg(decode(deptno,10,sal))),'9,990') "DEPTNO 10",
 to_char(round(avg(decode(deptno,20,sal))),'9,990') "DEPTNO 20",
 to_char(round(avg(decode(deptno,30,sal))),'9,990') "DEPTNO 30",
 to_char(round(avg(sal)),'9,990') "TOTAL"
from emp group by job;

-- JOIN 문

-- 1.
select e.empno, e.ename, e.job, d.deptno, d.dname, d.loc
from emp e, dept d
where d.deptno = e.deptno(+);

-- 2.
select e.empno, e.ename, e.job, e.sal, s.grade, s.losal, s.hisal
from emp e, salgrade s
where e.sal <= s.hisal and e.sal >= s.losal;

-- 3.
select e1.ename, e1.empno, e1.mgr, e2.ename
from emp e1, emp e2
where e1.mgr = e2.empno and e1.mgr = '7698';

-- 4.
select e.empno, e.ename, e.sal, e.job, d.dname, s.grade
from emp e, dept d, salgrade s
where d.deptno = e.deptno and (job='MANAGER' or job='CLERK') and (e.sal >= s.losal and e.sal <= s.hisal);

-- 5.
select e1.empno, e1.ename, e1.sal, e1.mgr, e2.ename, e2.job
from emp e1, emp e2
where e1.mgr = e2.empno(+);

-- 6.
select e1.ename, e1.hiredate, e2.ename, e2.hiredate
from emp e1, emp e2
where e1.mgr = e2.empno and e1.hiredate < e2.hiredate;

-- 7.
select e1.empno, e1.ename, e1.deptno, d.dname, e1.mgr,e2.sal, s.grade
from emp e1, emp e2, dept d, salgrade s
where e1.mgr = e2.empno(+) and (e2.sal >= s.losal(+) and e2.sal <= s.hisal(+)) and e1.deptno = d.deptno;
```
