---
layout: post
title: Docker로 SQL*Plus 및 SQL Loader 사용하기
date: 2016-04-21 11:32:42.000000000 +09:00
type: post
categories:
- Docker
---

## SQL\*Plus
```
$ docker run  \
  -e NLS_LANG=KOREAN_KOREA.AL32UTF8 \
  --interactive guywithnose/sqlplus \
  sqlldr <ID>/<PASS>@<HOST>:<PORT>/<DB>
```

`-e NLS_LANG=KOREAN_KOREA.AL32UTF8` 로 `sqlplus`에 `NLS_LANG` 환경변수를 넘겨줄 수 있었음


## SQL Loader
