# Docker로 SQL*Plus 및 SQL Loader 사용하기

## SQL*Plus
```
$ docker run  \
  -e NLS_LANG=KOREAN_KOREA.AL32UTF8 \
  --interactive guywithnose/sqlplus \
  sqlldr <ID>/<PASS>@<HOST>:<PORT>/<DB>
```

`-e NLS_LANG=KOREAN_KOREA.AL32UTF8` 로 `sqlplus`에 `NLS_LANG` 환경변수를 넘겨줄 수 있었음


## SQL Loader