---
layout: post
title: Flash Player Debug 로그 보기
date: 2015-10-16 16:57:55.000000000 +09:00
type: post
categories:
- Flash
---
> 디버거를 설치 했는데도 로그가 남지 않더라. 알고 봤더니 설정 이슈. 일단 Mac 기준으로 적어본다. 그 밖의 플랫폼은 `참고`란의 URL을 살펴볼 것

## 해결 방안
1. `https://www.adobe.com/support/flashplayer/debug_downloads.html`에 들어가서 Flash Player Content Debugger를 설치한다.
1. `/Library/Application Support/Macromedia` 경로에 `mm.cfg`를 만든다.
 ```
ErrorReportingEnable=1
TraceOutputFileEnable=1
```
1. 브라우저를 재시작하고 문제가 있는 사이트에 접속한다.
1. `/Users/username/Library/Preferences/Macromedia/Flash Player/Logs/flashlog.txt` 에 로그가 남는지 확인한다.

## 참고
https://helpx.adobe.com/flash-player/kb/configure-debugger-version-flash-player.html
