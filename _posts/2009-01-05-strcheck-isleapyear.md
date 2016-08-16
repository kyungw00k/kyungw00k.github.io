---
layout: post
title: strcheck / isLeapYear
date: 2009-01-05 09:25:31.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
tags:
- Tmax 연수
- C
---

```c
int strcheck(char *s, char ch) {
    char *p;

    /* 문자가 널이거나 같을때 종료함 */
    for ( p = s ; *p && *p != ch ; p++ );

    /* 문자가 널이 아니면 위치를, 널이면 -1을 리턴함 */
    return (*p) ? (int)(p-s)+1 : -1;
}

/*
 * 1 : 윤년
 * 0 : 평년
 */
int isLeapYear(int y) {
        /* 4로 나누어 떨어지고, 100으로 나누어 떨어지지 않거나
         * 400으로 나누어 떨어지면 윤년이다. */
        return ( ( y % 4 == 0 &&  y % 100 ) || y % 400 == 0  );
}
```
