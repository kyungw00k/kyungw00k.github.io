---
layout: post
title: Zeppelin 설치하기
date: 2015-10-21 14:54:01.000000000 +09:00
type: post
categories:
- Scala
---
## 이건 뭔가요
> Apache Zeppelin은 국내에서 주도하고 있는 오픈소스 프로젝트로써, Spark를 훨씬 더 편하고 강력하게 사용할 수 있게 해주는 도구입니다.

-라고 합니다.

## 빌드하기
`mvn clean package -Pspark-1.5 -Dhadoop.version=2.0.0-cdh4.3.0 -Phadoop-2.0 -DskipTests`

## 참고문서
* http://zeppelin-project.org
* http://engineering.vcnc.co.kr/2015/05/data-analysis-with-spark/
