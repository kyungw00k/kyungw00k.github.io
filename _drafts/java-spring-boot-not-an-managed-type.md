---
layout: post
title: Not an managed type
date: 2015-10-13 10:40:56.000000000 +09:00
type: post
categories:
- Java
- Spring
---
## Problem
```
Caused by: java.lang.IllegalArgumentException: Not an managed type: class blah.blahblah
```

## Solution
`@SpringBootApplication`이 있는 곳에 `@EntityScan` Annotation을 추가로 넣어주면 잘 됨.

## Reference
* http://stackoverflow.com/questions/23366226/spring-boot-w-jpa-move-entity-to-different-package
