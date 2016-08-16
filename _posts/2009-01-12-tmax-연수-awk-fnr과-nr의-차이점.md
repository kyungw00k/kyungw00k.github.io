---
layout: post
title: "[Tmax 연수] awk FNR과 NR의 차이점!"
date: 2009-01-12 15:17:42.000000000 +09:00
type: post
published: true
status: publish
categories:
- Command
tags:
- Awk
- Unix
- Tmax 연수
---

```
NR - 현재까지 레코드 개수
FNR - 현재 파일의 레코드 개수
```

2개 파일 이상을 인자로 넣어주면 확연한 차이를 알 수 있습니다.

아래와 같이 두개의 파일이 있다고 합시다.

f1 파일 내용
```
a
b
c
d
```

f2 파일 내용
```
e
f
g
```
위의 파일을 만들고 아래의 명령을 수행해봅시다.

```
awk '{printf("file->[%s] NR->[%d] FNR->[%d] str->[%s]\n", FILENAME, NR, FNR, $0)}' f1 f2
```

결과는 아래와 같습니다.

```
file->[f1] NR->[1] FNR->[1] str->[a]
file->[f1] NR->[2] FNR->[2] str->[b]
file->[f1] NR->[3] FNR->[3] str->[c]
file->[f1] NR->[4] FNR->[4] str->[d]
file->[f2] NR->[5] FNR->[1] str->[e]
file->[f2] NR->[6] FNR->[2] str->[f]
file->[f2] NR->[7] FNR->[3] str->[g]
```

결국 NR은 출력되는 내용의 전체 레코드 개수를 카운트 하는거고
FNR은 파일에서 가리키는 레코드 순서를 출력하고 있는걸 알 수 있습니다.
