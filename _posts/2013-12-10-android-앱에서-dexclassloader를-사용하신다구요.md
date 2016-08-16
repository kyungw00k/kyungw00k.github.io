---
layout: post
title: Android 앱에서 DexClassLoader를 사용하신다구요?
date: 2013-12-10 07:45:00.000000000 +09:00
type: post
published: true
status: publish
categories:
- Android
---

Android 쪽 앱에 들어가는 Library를 개발 중에 배포 이슈를 피해보고자 Android Dynamic Dex Loader를 사용해 Library에 Self Update 기능을 넣기로 하고 개발을 진행했었습니다.

그런데 Google Play 정책 중 아래 문구가 있더군요.

> Google Play에서 다운로드한 앱은 Google Play의 업데이트 메커니즘 이외의 방법을 사용하여 앱의 APK 바이너리 코드를 수정, 변경 또는 업데이트할 수 없습니다.

[Facebook도 자사 앱에 Self Update 기능을 넣었다가 뺀 히스토리](http://arstechnica.com/information-technology/2013/04/google-bans-self-updating-android-apps-possibly-including-facebooks/)가 있습니다.

(결국 제 프로젝트는 Drop 되었죠)

요약 하면,

**원격지에 있는 파일을 내려받아 DexClassLoader를 하신다면 정책 위반 가능성이 높으니 되도록 피해서 사용하시는게 좋습니다.**

그래도 사용하고 싶으시다면,

* [Custom Class Loading in Dalvik](http://android-developers.blogspot.kr/2011/07/custom-class-loading-in-dalvik.html) [
[번역링크](http://blog.naver.com/PostView.nhn?blogId=huewu&logNo=110120966664&parentCategoryNo=18&viewDate=&currentPage=1&listtype=0)]
* [DexClassLoader](http://developer.android.com/reference/dalvik/system/DexClassLoader.html)
