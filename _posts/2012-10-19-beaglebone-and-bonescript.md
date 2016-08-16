---
layout: post
title: BeagleBone과 BoneScript
date: 2012-10-19 13:06:37.000000000 +09:00
type: post
published: true
status: publish
categories:
- BeagleBone
tags:
- bonescript
---

아주 오래전에 BeagleBone에서 digitalRead()를 추가하는 [포스트](http://kyungw00k.wordpress.com/2012/02/13/beaglebone%EC%9D%98-bonescript%EC%97%90%EC%84%9C-digitalread-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/)를 쓴 적이 있었는데...

최근에 Raspberry Pi 때문에 잠시 뒷방에 물러난 Beaglebone을 만지작 거리면서 살펴봤더니,

[**Arduino-like API가 대부분 구현이 되어 있었다.**](https://github.com/jadonk/bonescript/blob/master/node_modules/bonescript/index.js)

```
-----------------
Arduino functions
-----------------
Digital I/O, Analog I/O, Advanced I/O and Time:**
analogRead(pin) -> value
analogWrite(pin, value, [freq])
attachInterrupt(pin, handler, mode)
detachInterrupt(pin)
delay(milliseconds)
digitalRead(pin) -> value
digitalWrite(pin, value)
getEeproms([callback]) -> eeproms
getPinMode(pin) -> pinMode
pinMode(pin, direction, [mux], [pullup], [slew])
shiftOut(dataPin, clockPin, bitOrder, val)

Bits/Bytes, Math, Trigonometry and Random Numbers:
lowByte(value)
highByte(value)
bitRead(value, bitnum)
bitWrite(value, bitnum, bitdata)
bitSet(value, bitnum)
bitClear(value, bitnum)
bit(bitnum)
min(x, y)
max(x, y)
abs(x)
constrain(x, a, b)
map(value, fromLow, fromHigh, toLow, toHigh)
pow(x, y)
sqrt(x)
sin(radians)
cos(radians)
tan(radians)
randomSeed(x)
random([min], max)
```

역시 누군가는 귀차니즘을 이겨내고 다 구현을 해주는구나!

(결국 누가 더 오래 참고 기다리는가에 문제? =_=;;)

p.s.

Raspberry Pi 오기 전까지는 http://www.ewanleith.com/blog/1066/high-performance-wordpress-on-arm 보면서

Beaglebone 만지작 거리고 있어야겠다 싶어 며칠 전부터 셋팅 중인데, Micro SDHC Class 4 카드로는 도저히 안되겠다 싶다.

아직 지르지 않은게 SD 카드니... SDHC 16GB Class 10로 3개 질러야겠다.

도메인도 지르고... 이 정도면 왠만한 가상 호스팅보다는 나아지겠지 ;;;

빨랑 개발 플랫폼 셋팅해서 하루프레스로 이사가자. =_=;
