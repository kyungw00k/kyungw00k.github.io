# `Not an managed type`
개인 플잭을 하다가 아래와 같은 이슈가 발생했다.

## Problem
```
Caused by: java.lang.IllegalArgumentException: Not an managed type: class blah.blahblah
```

## Solution
`@SpringBootApplication`이 있는 곳에 `@EntityScan` Annotation을 추가로 넣어주면 잘 됨.

## Reference
* http://stackoverflow.com/questions/23366226/spring-boot-w-jpa-move-entity-to-different-package
