---
layout: post
title: Getting the Adafruit serial cable to work on OS X
date: 2015-12-10 13:28:15.000000000 +09:00
type: post
categories:
- Raspberry Pi
---
## 0. 케이블 구매
![](https://www.adafruit.com/images/970x728/954-02.jpg)
https://www.adafruit.com/product/954

## 1. 드라이버 설치
http://www.prolific.com.tw/us/showproduct.aspx?p_id=229&pcid=41

## 2. 케이블 연결
![](https://learn.adafruit.com/system/assets/assets/000/003/130/medium800/learn_raspberry_pi_gpio_closeup.jpg?1396791839)

## 3. 접속
```
$ screen /dev/cu.usbserial 115200
```
