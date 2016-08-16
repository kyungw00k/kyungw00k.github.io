---
layout: post
title: "[Maven] Request to merge when 'filtering' is not identical 해결하기"
date: 2011-08-02 11:38:34.000000000 +09:00
type: post
published: true
status: publish
categories:
- Maven
tags:
- Eclipse
---

Eclipse에서 m2Eclipse 로 Maven 사용시,

```sh
$ mvn eclipse:eclipse
$ mvn eclipse:eclipse -Dwtpversion=2.0
```

아래와 같이 자꾸 eclipse:eclipse가 에러나는 경우를 볼 수 있다.

```
org.apache.maven.lifecycle.LifecycleExecutionException: Request to merge when 'filtering' is not identical. Original=source src/main/java: output=null, include=[**/*.java], exclude=[], test=false, filtering=false, merging with=resource src/main/java: output=target/classes, include=[**], exclude=[**/*.java|config.properties|**/*.java], test=false, filtering=true
```

**"아놔 왜 안되는거야"** 를 연달아 외치다가 구글링좀 해봤더니..

구글링하면 갖가지 해결책이 나오는데,

가장 확실했던 Goal은...

```sh
$ mvn org.apache.maven.plugins:maven-eclipse-plugin:2.6:eclipse
```

근데 이건 과연 버그인건가 아닌건가...

출처 : 
[http://www.inweb.de/chetan/_blog/2010/03/20/mvn-eclipse-request-to-merge-when-filtering-is-not-identical.html](http://www.inweb.de/chetan/_blog/2010/03/20/mvn-eclipse-request-to-merge-when-filtering-is-not-identical.html)
