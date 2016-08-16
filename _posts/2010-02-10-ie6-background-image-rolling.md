---
layout: post
title: IE6 Background Image Rolling
date: 2010-02-10 14:44:59.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
- JavaScript
tags:
- BOM
---

### Background
Server측에서 Image Cache가 지원는데도 불구하고, IE6에서 Image Flicker 현상 때문에 Background Position 변경을 해 이미지 Rolling할 경우 Flicker 현상 발생

### Requirement
Image Server에 Cache가 지원되어야 한다.

### Solution
```js
if (!!window.ActiveXObject && !window.XMLHttpRequest) { // IE6일 경우
    // BackgroundImageCache Command를 지원할 경우
    if (document.execCommand("BackgroundImageCache", false, true)) {
        // Background Position 변경하면서 Rolling;
    } else {
        // 이미지 절대좌표 변경하면서 Rolling;
    }
}
```

### Sample Code
*[http://keyword.biz.daum.net](http://keyword.biz.daum.net/) Top 화면 하단 Partners 부분(oIntro.partnerLinkScroll())
