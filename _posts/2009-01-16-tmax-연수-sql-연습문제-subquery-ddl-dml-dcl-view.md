---
layout: post
title: "[Tmax 연수] SQL 연습문제 - SUBQUERY(11) / DDL(8) / DML(5) / DCL(6) / VIEW(2)"
date: 2009-01-16 13:45:10.000000000 +09:00
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

```sql
-- SUBQUERY
-- 1.
select empno, ename, job, hiredate, sal from emp
where emp.job = ( select job from emp where empno = 7521 )
and  emp.sal > (select sal from emp where empno = 7934);

-- 2.
select empno, ename, job, sal, deptno from emp where sal < (select avg(sal) from emp);

-- 3.
select deptno, avg(sal) from emp group by deptno having avg(sal) <= (select min(avg(sal)) from emp group by deptno);

-- 4.
select sal, comm from emp e where e.deptno<>30 and (sal, comm) in ( select sal,comm from emp where deptno=30);

-- 5.
select ename, job, sal from emp e, dept d where e.deptno = d.deptno and d.loc = 'DALLAS';

-- 6.
select * from emp where (job,sal) in ( select job,sal from emp where ename='FORD');

-- 7.
-- 1)
select * from emp where empno not in ( select mgr from emp where mgr is not null
 );

-- 2)
select l.ename"부하직원",h.ename"상사",h.empno,h.ename,h.job,h.sal
from emp l
right outer join
emp h on  l.mgr = h.empno
where l.ename is null;

-- 3)
select * from emp e where
not exists
 ( select empno from emp where e.empno = mgr );

-- 8.
select ename, job, mgr from emp where mgr = (select mgr from emp where ename='BLAKE');

-- 9.
select level, ename from emp connect by prior empno=mgr start with ename='KING';

-- 10.
select ename, job, sal, deptno from emp where job in ( select job from emp e, dept d where d.deptno = e.deptno and d.loc='CHICAGO');

-- 11.
select * from (select * from emp order by hiredate asc) where rownum < 6;

-- DDL
-- 1.
alter table emp drop constraint FK_DEPTNO;
alter table dept drop constraint PK_DEPT;

-- 2.
alter table add bigo varchar2(10);

-- **3. 안배웠음(구글에서 찾아서 했습니다. ㅡㅡ;;)**
alter table emp set unused (bigo);

-- **4. 안배웠음(찍어서 맞았습니다. ㅡㅡ;;)**
alter table emp drop unused columns;

-- 5.
drop table dept;

-- 6.
select table_name, constraint_name, column_name from
**user_cons_columns**
 where table_name='EMP' or table_name='DEPT';

-- 7.
alter table dept add constraint UK_DNAME unique(dname);

-- 8.
alter table dept
disable
 constraint UK_DNAME;

```

확인은... 요걸로!


``` sql
select status, constraint_name, constraint_type, table_name from user_constraints where table_name='DEPT';

-- DML


-- 1. 급여 변경
update emp set sal=(select sal from emp where ename='BLAKE') where empno=7902;
rollback;

-- 2. 사원 삭제
delete emp where empno in (select empno from emp e where sal < (select avg(sal) from emp d where e.deptno = d.deptno));


rollback;

-- 3. 테이블 생성
create table t_emp as select * from emp where deptno=10;
rollback;

-- 4. 급여 인상
insert into t_emp (select * from emp where sal <= 2000 and deptno<>10);
update t_emp set sal=sal+1000;

-- **5. 병합(시험에 나오지 않음)**
merge into t_emp t using emp e on (t.empno = e.empno)
when matched then
update set t.sal = e.sal
when not matched then
insert values (e.empno, e.ename, e.job, e.mgr, e.hiredate, e.sal, e.comm, e.deptno);

-- DCL


-- 1.
create user kim identified by ksm default tablespace users;

-- 2.
create role r1;
grant connect, resource to r1;

-- 3.
grant r1 to kim;

-- 4.
grant select on scott.emp to r1;
grant update on scott.emp to r1;

-- 5.
revoke r1 from kim;

-- 6.
drop user kim;

-- VIEW

-- 1.
create view v_d20 as select * from emp where deptno=20 with check option constraint ck_d20;

-- 2.
select table_name, constraint_type, constraint_name from user_constraints;
```
