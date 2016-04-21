# Docker로 SQLPlus 사용하기

```
$ docker run  \
  -e NLS_LANG=KOREAN_KOREA.AL32UTF8 \
  --interactive guywithnose/sqlplus \
  sqlplus <ID>/<PASS>@<HOST>:<PORT>/<DB>
```

`-e NLS_LANG=KOREAN_KOREA.AL32UTF8` 로 `sqlplus`에 `NLS_LANG` 환경변수를 넘겨줄 수 있었음