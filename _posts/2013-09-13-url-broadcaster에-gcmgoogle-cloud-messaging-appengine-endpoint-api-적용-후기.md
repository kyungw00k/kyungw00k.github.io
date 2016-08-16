---
layout: post
title: "URL Broadcaster에 GCM(Google Cloud Messaging) + AppEngine EndPoint API 적용 후기"
date: 2013-09-13 01:00:10.000000000 +09:00
type: post
published: true
status: publish
categories:
- Project
tags:
- android
- 'google app engine'
- 'google cloud messaging'
---
[URL Broadcaster][]는 단순 URL을 받아서 모바일 폰으로 Broadcasting 해주는 프로젝트다.<br>
수 많은 안드로이드 기기로 테스트 하는 분들에게 일일이 타이핑 하는걸 줄여주고 싶다는 승 팀장님의 아이디어를 쉽고 간단한 컨셉으로 만들어보겠다고 위젯으로 만들어본거다.

하지만 지나고 보니 유지보수가 생각보다 많이 들었다.

#### 뭐가 문제인건가?
* NodeJS + socket.io로 만든 서버를 (공짜로) 서비스할 곳이 마땅치 않았다.
    * Nodester가 나왔을때 반짝 만든건데.. 없어져서 좀... ㅠㅠ
    * 공짜로 운영하기엔 socket.io가 차지하는 메모리량이 예상외로 컸다.
* Login 기반 서비스가 필요했다.
    * 사실, 귀찮아서 붙이지 않았었다. 굳이 필요 없을 것 같아서...

어떻게 하면 좋을까 며칠을 고민하다, 우연히 GCM Google IO 세션을 듣다가 GCM이 공짜로 풀린걸 알게되었다.<br>
물론, 메시지 히스토리 보관에는 돈이 든다고는 하지만 이 프로젝트에는 GCM 히스토리를 서버쪽에 100건 이상 보관할 필요가 없어보였다.<br>
`완전 딱이야!` 싶었지만, 많이 고쳐야할 것 같아서 귀찮았다.

하지만 한번 해보기로 했다. 해서..

#### 개편은 이렇게 해보자
1. Google Cloud Endpoints를 Backend로 사용하자.
2. Socket.io 대신 Google Cloud Messaging을 사용하자.
3. Google 인증을 붙이자.

...그래서 하루에 최소 2시간 이상 투자해 개편해봤다.

#### Google Cloud Endpoint?
![](https://developers.google.com/appengine/docs/images/endpoints.png)

API 구조로 AppEngine Backend에 비지니스 로직을 구현하고 이를 허용된 Client에서 사용할 수 있게 한다는 건데 자세한건 [Overview of Google Cloud Endpoints][] 를 보면 될 것 같다.

#### Google Cloud Endpoint & Google Cloud Messaging 적용
일단, [Adding a Backend to Your App In Android Studio][] 를 따라하는 것으로 간단한 안드로이드 앱을 만든 후, AppEngine Endpoint를 Generate해 봤다.

따라해 보니 생성된 코드로만으로 URL Broadcaster를 사용할 수 있었다.<br>
(하지만 내부가 어케 돌아가는지 알 수 없어서 Concept을 이해하는데 시간이 좀 걸렸다.)

게다가, GCM코드 까지 생성되었다. 우왕굳.

그래서, 셈플의 RegisterActivity.java에 구현된 기능을 위젯으로 옮기는 것으로 개발을 시작했다.

자세한 사항은 [위젯 코드](https://github.com/kyungw00k/URLBroadcasterProject/blob/master/URLBroadcaster/src/main/java/kyungw00k/URLBroadcaster/SwitchWidgetProvider.java)를 참고 하면 될 것 같다. (주석은 없다.)

#### 결과물
여전히 [앱을 내려 받고](https://play.google.com/store/apps/details?id=kyungw00k.URLBroadcaster), [사이트를 방문](http://url-broadcaster.appspot.com/) 하는 형태는 동일하다.(아래 스샷을 봐도 기능은 예전과 다를바 없다.)

![](https://lh5.ggpht.com/rzx--PhGapgmevvm95x6gUOcgVpJX3XGrtDAQXwEzXBSmzWuHc_XTF7pIV4kqP3Owp53=h900-rw)

![](https://lh6.ggpht.com/m8M8Q3FT1a0JWo_f4SsLe3sei7yLxGUCiPg8wsWiAIQcxyjFM8kAHUkMF4zOqHd-EA=h900-rw)

#### 뭐가 개선되었을까?
* GCM을 사용하면서 WIFI/3G 전환시에 연결이 끊어지던 불편함이 없어졌다.
* Socket.io를 사용할 때보다 베터리 소모량이 많이 줄었다.
* 더 이상 서버를 옮겨다니지 않아도 된다.

#### 그 다음은?
사용자의 의견을 수렴해 기능을 하나 하나 추가해 나가야겠다는 생각이 들었다. 간단한 앱이지만 그래도 어딘가 발전 가능성이 있지 않을까 싶어서...

#### 다음에 할일을 적어보면...
* [Chrome Extension을 만들어 바로 Mobile에 Push 해야겠다.](https://github.com/kyungw00k/URLBroadcasterProject/issues/1)
* [필요하면 특정 폰에 ScreenShot을 받아올 수 있도록 해야겠다.](https://github.com/kyungw00k/URLBroadcasterProject/issues/2)

일단... 나 부터 즐겁게 사용해야겠다. 뿅~

[URL Broadcaster]: https://github.com/kyungw00k/URLBroadcaster
[Overview of Google Cloud Endpoints]: https://developers.google.com/appengine/docs/java/endpoints/
[Adding a Backend to Your App In Android Studio]: http://android-developers.blogspot.kr/2013/06/adding-backend-to-your-app-in-android.html
