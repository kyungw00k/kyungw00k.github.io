---
layout: post
title: BeagleBone의 BoneScript에서 digitalRead() 사용하기
date: 2012-02-13 02:22:55.000000000 +09:00
type: post
published: true
status: publish
categories:
- BeagleBone
tags:
- digitalread
---
Bonescript로 간단히 Input/Output 테스트를 하려는데 잘 되지 않았다.

선을 잘못 연결했나 싶어 한참을 해메다 단순히 OUTPUT 선을 INPUT에 연결해도 되지 않는것이었다.

보드의 문제는 아닐테고.. Bonescript의 문제일까 해서 봤는데, Pinmode에서 Input/Output일 경우에 분기 처리가 없었다.

node-arduino와 마찬가지로 bonescript 에서도 Input이 문제긴 하나보다.

포럼에 [Hacking GPIO digital input into BoneScript](http://groups.google.com/group/beaglebone/browse_thread/thread/7106fdc126f898c3?hl=en) 라는 글을 보면 이미 누군가 해결책을 내놓았다.

하지만 PWM을 사용하는 AnalogInput/Output에 대한 해결 방안은 아직까지는 없는것 같다.

(구매한지 얼마 안되서 많이 들여다보지 않았지만 그렇게 어려울 것 같진 않는데... 음...)

일단, 위 글을 참고로 bonescript의 index.js를 간단히 수정해 digitalRead를 추가해보자.

수정 전 index.js 소스 Line 164는 다음과 같다.

```js
// ... 
    fs.writeSync(muxfile, "7", null); 
// ...
```

위 부분은 현재 OUTPUT만을 고려하고 있으니, INPUT일 경우에도 한 줄 추가가 되어야 할 것이다.

```js
// ... 
    if ( mode == OUTPUT ) {
        fs.writeSync(muxfile, "7", null);    // setting for mode 7 and no pull up/down
    } else {
        fs.writeSync(muxfile, "27", null);   // setting for mode 27 and AM3XX_PULL_UP
    }
// ...
```

그리고 digitalWrite 함수와 마찬가지로, 아래와 같이 digitalRead 함수를 추가해야 한다.

```js
// ... 
digitalRead = exports.digitalRead = function(pin)
{
    return fs.readFileSync(gpio[pin.gpio].path);
};
// ...
```

다음은 추가된 digitalRead를 테스트하기 위해 간단히 스위치로 LED를 ON/OFF를 보여주는 데모 영상이다.

소스는 영상에 살짝 나온게 전부라.. 따로 코드를 적진 않겠다.

<iframe src="//player.vimeo.com/video/36529601" width="400" height="225" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
