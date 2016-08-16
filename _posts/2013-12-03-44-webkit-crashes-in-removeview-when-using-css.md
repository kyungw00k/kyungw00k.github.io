---
layout: post
title: 4.4 webkit crashes in removeView when using css transformations
date: 2013-12-03 07:22:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Android
tags:
- Webview
- CSS
---
4.4에서 테스트 하다가 아래와 같이 웹뷰가 생각보다 자주 죽어서 왜 이러나 싶었다. 검색에 검색을 거듭한 끝에 찾아냈다.

Android 4.4 KitKat에서 잘 되던 웹뷰가 쌩뚱맞게 죽는다면, 이 에러를 의심해봐야한다. 기기 파편화도 파편화지만 OS 분기 처리도 만만치 않네.

**요약 : 하드웨어 가속을 끄세요.**

```
Our banners work fine in listviews pre 4.4, but in 4.4 crashes quite horribly. I’ve tracked the issue down to css transformations + hardware acceleration. With setLayerType(View.LAYER_TYPE_SOFTWARE, null); the banner works fine.

See attached code to reproduce the issue.

The issue produces a segfault in libwebviewchromium.so.

2-03 16:02:08.243  30682-30682/net.daum.adam.publisher.activity I/chromium﹕ [INFO:async_pixel_transfer_manager_android.cc(56)] Async pixel transfers not supported
12-03 16:02:08.283  30682-30682/net.daum.adam.publisher.activity I/chromium﹕ [INFO:async_pixel_transfer_manager_android.cc(56)] Async pixel transfers not supported
12-03 16:02:08.323  30682-30682/net.daum.adam.publisher.activity A/libc﹕ Fatal signal 6 (SIGABRT) at 0x000077da (code=-6), thread 30682 (lisher.activity)
12-03 16:02:08.423  26662-26662/? I/DEBUG﹕ *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
12-03 16:02:08.423  26662-26662/? I/DEBUG﹕ Build fingerprint: 'google/razor/flo:4.4/KRT16S/920375:user/release-keys'
12-03 16:02:08.423  26662-26662/? I/DEBUG﹕ Revision: '0'
12-03 16:02:08.423  26662-26662/? I/DEBUG﹕ pid: 30682, tid: 30682, name: lisher.activity  >>> net.daum.adam.publisher.activity

[4.4 webkit crashes in removeView when using css transformations](https://code.google.com/p/android/issues/detail?id=62578)
```
