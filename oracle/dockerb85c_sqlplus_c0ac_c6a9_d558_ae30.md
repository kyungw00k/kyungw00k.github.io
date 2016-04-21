# Docker로 SQLPlus 사용하기

```
$ docker run  \
  -e NLS_LANG=KOREAN_KOREA.AL32UTF8 \
  --interactive guywithnose/sqlplus \
  sqlplus <ID>/<PASS>@<HOST>:<PORT>/<DB>
```