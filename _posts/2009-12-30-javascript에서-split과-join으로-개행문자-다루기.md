---
layout: post
title: JavaScript에서 split과 join으로 개행문자 다루기
date: 2009-12-30 14:21:45.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
- JavaScript
---

## Introduction
종종 문자열을 개행 문자로 잘라서 장바구니처럼 리스트로 나열 하고 싶을때가 있을것이다.
새 항목을 추가하고 삭제도 하고 싶고, 중복 검사도 하고 싶다면 문자열을 비교해야 할텐데,
여기서 종종 문자열 비교가 제대로 되지 않는 경우가 있(을지도 모른)다.

## Problem

```html
<textarea id="box">
나는
천재일지도
모른다는
생각을
해본다.
</textarea>
```

```js
var textbox = document.getElementById('box');
var array = textbox.value.split('\n');  // ["나는", "천재일지도", "모른다는", "생각을", "해본다."]
var findStr = "나는"; //
for ( var idx = array.length-1 ; idx > -1 ; idx-- ) {
   if ( array[idx] === findStr ) {
        alert("중복!");
        break;
   }
}
```

만약 위의 경우 처럼 textarea 같은데에다 문자열을 개행문자로 잘라서 넣고, 그 중에 중복된 부분을 검사하고 싶은데 검사 루틴이 제대로 안먹는 경우가 있는가?

`"대체 뭐가 꼬인거야? 화면에는 잘 보이잖아!"` 라고 생각된다면, String에 대해 아주 사소한 부분을 잊고 있는게 분명하다.

## Solution

`CR` `LF` 라고 들어보셨는지 모르겠다. 이때 보자마자 직감하고 코드를 수정하고 있다면 당신은 센스쟁이!

C언어때 \n을 찍으면 OS에 따라 \r\n이 나오기도 하고 \n이 나오기도 한다. 왜 그럴까?

* Wikipedia의 Newline에 대한 설명을 보니 다음과 같은 설명이 있다.

```
Software applications and operating systems usually represent
a newline with one or two control characters:

Systems based on ASCII or a compatible character set use either
LF (Line feed, 0x0A, 10 in decimal) or CR (Carriage return, 0x0D,
13 in decimal) individually, or CR followed by LF (CR+LF, 0x0D 0x0A);
see below for the historical reason for the CR+LF convention.

These characters are based on printer commands:
The line feed indicated that one line of paper should feed out of the printer,
and a carriage return indicated that the printer carriage should return to the
beginning of the current line.

LF:    Multics, Unix and Unix-like systems (GNU/Linux, AIX, Xenix, Mac OS X, FreeBSD, etc.)
       , BeOS, Amiga, RISC OS, and others

CR+LF: DEC RT-11 and most other early non-Unix, non-IBM OSes, CP/M, MP/M, DOS, OS/2, Microsoft Windows, Symbian OS

CR:    Commodore 8-bit machines, Apple II family, Mac OS up to version 9 and OS-9
```

* Cross Browsing 검사할때 만약 위와 같은 코드가 있다면, 아무리 성능 좋은 윈도우 PC의 최신 브라우저인 Internet Explorer 8에서도 동작하지 않는다.


하여 아래와 같이 고치면 잘 쪼개지고, 잘 붙을것이다. 

```js
// 쪼갤때는 \r, \r\n, \n으로 쪼개고
var array = textbox.value.split(/\r\n|\r|\n/)

// 붙일때는 CRLF 형태로 붙인다.
textbox.value = array.join("\r\n");
```
