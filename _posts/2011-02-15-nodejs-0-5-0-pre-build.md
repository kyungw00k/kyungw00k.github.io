---
layout: post
title: NodeJS 0.5.0-pre Build시..
date: 2011-02-15 18:08:58.000000000 +09:00
type: post
published: true
status: publish
categories:
- Node.JS
---

(별 중요한건 아니지만 혹시) git를 통해 node를 빌드 하시려는 분들 중, Mac에서 빌드 에러가 발생할 수 있습니다.

`src/node_buffer.cc` 에 strnlen이 없다는군요. 

실제로 컴파일러 마다 빠져있기도 하답니다. 하여...

(참고로 제 맥에서 쓰는 gcc 버전은 `i686-apple-darwin10-gcc-4.2.1 (GCC) 4.2.1 (Apple Inc. build 5646) (dot 1)` 입니다.)

그냥 코드에 추가했습니다.

적절한 위치에 아래의 코드를 추가합니다.(전 53행에 넣었음)

```c
static size_t strnlen (const char* string, size_t maxlen)
{
    size_t count = 0;
    for (; maxlen > 0 && *string != ''; maxlen--, string++)
        count++;

    return count;
}
```
