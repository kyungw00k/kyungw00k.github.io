---
layout: post
title: "[Tmax 연수] Oracle scott 계정 복구"
date: 2009-01-17 09:54:36.000000000 +09:00
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
(DDL 문제 5번) scott 계정 복구

scott에 table을 삭제하여 다시 복구하고 싶을 경우 
scott 계정에 관한 스크립트 파일인 `scott.sql`을 찾아 실행

(예시)scott.sql 파일이

c:\oracle\ora92\rdbms\admin\ 위치에 있는 경우

```
SQL> @c:\oracle\ora92\rdbms\admin\scott
```
