---
layout: post
title: BeagleBone에서 Node.js Source Build 하기
date: 2012-02-13 01:59:55.000000000 +09:00
type: post
published: true
status: publish
categories:
- BeagleBone
tags:
- nodejs
---

BeagleBone을 받으면 기본적으로 node가 최신버전이 아닌 0.4.x다.

개인적으로 버전 욕심이 생겨 빌드해 봤더니 그냥 되진 않았다.

(맨날 이렇지 뭐.. =_=)

그냥 받아서 Build하면 다음과 같은 에러 메시지를 보게 된다.

>"For thumb inter-working we require an architecture which supports box" (어쩌고저쩌고)

이 경우에는 deps/v8/SConstruct 파일을 수정한다.

node 0.6.10 기준으로 Line 82 줄을 다음과 같이 수정한다.

>'CCFLAGS':      ['$DIALECTFLAGS', '$WARNINGFLAGS','-march=armv7-a'],


빌드가 잘되고, 실행 또한 잘 되는것을 확인할 수 있다.
그 밖에 OpenEmbedded Platform에서 node.js를 빌드가 한방에 되지 않는다면 다음의 문서를 참고하면 되겠다.

(Reference : [http://fastr.github.com/articles/Node.js-on-OpenEmbedded.html](http://fastr.github.com/articles/Node.js-on-OpenEmbedded.html))
