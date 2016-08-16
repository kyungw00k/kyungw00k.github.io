---
layout: post
title:  Linux에서 getch() 사용법
date: 2009-01-06 09:32:45.000000000 +09:00
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
#include <stdio.h>

int main() {
    int ch;
    char aaa;

    initscr();

    /*for문으로 돌면서 배열에다가 넣어두 되죠.. ^^*/
    ch = getch();
    aaa = c;

    printf("[%c][%c]\n", ch, aaa);

    endwin();

    return 0;
}
```

위의 코드를 아래와 같이 컴파일 합니다.

```sh
$ gcc -o test test.c -lcurses
```

단, 시스템에 curses 라이브러리가 있어야 합니다.
