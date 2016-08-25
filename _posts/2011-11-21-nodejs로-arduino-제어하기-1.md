---
layout: post
title: 'Node.js로 Arduino 제어하기 #1'
date: 2011-11-21 01:21:35.000000000 +09:00
type: post
published: true
status: publish
categories:
- Node.js
- Arduino
- JavaScript
---

생각보다 많은 분들이 이 글을 보고 있어서 놀랐습니다.

아래 내용을 좀 더 쉽게(?) 써놓은 포스트가 있어서 링크를 걸어봅니다.

>[Arduino 101 using NodeJS](/2013/08/13/arduino-101-using-nodejs/)

<iframe src="//player.vimeo.com/video/17437907" width="400" height="300" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

이번 가이드에서는 [Rhio님이 만든 동영상](http://vimeo.com/17437907)에 있는것 처럼 nodeJS에서 LED를 ON/OFF 해보겠습니다.

이 가이드를 이해하기 위해서는 2가지 실습을 해보신분이어야 합니다.

* nodeJS로 웹 서버 띄우는 예제를 실행해봤다.
* Arduino로 LED를 한번이라도 켜봤다.

위 2가지를 해보셨고, nodeJS로 Arduino를 만지작 거리고 싶은 분들을 위해 정리해봅니다.

먼저 실습에 앞서 필요한 것이 있다면..

* Arduino 호환 보드
* USB포트 달린 컴퓨터

위의 하드웨어가 준비가 되었다면, [Arduino IDE](http://arduino.cc/en/Main/Software)와 [nodeJS](http://nodejs.org/#download)를 설치합니다.

설치에 대한 가이드는 패스하고, 바로 실습으로 들어가보도록 하겠습니다.

먼저 Arduino로 LED를 깜빡거리는 소스를 살펴볼텐데요,

Arduino IDE를 설치하신 후, Example > 1. Basis > Blink 를 선택하시면 아래와 같은 소스를 볼 수 있습니다.

```c
void setup() {
  Serial.begin(9600);

  // initialize the digital pin as an output.
  // Pin 13 has an LED connected on most Arduino boards:
  pinMode(13, OUTPUT);
}

void loop() {
  digitalWrite(13, HIGH); // set the LED on
  delay(1000); // wait for a second
  digitalWrite(13, LOW); // set the LED off
  delay(1000); // wait for a second
}
```

위 코드는 Arduino에서 디지털 13번 핀에 전류를 1초 간격으로 줬다가 안줬다가 하는 형태로 LED를 깜빡거리게 하고 있습니다.

이걸 nodeJS에서 하려면 어떻게 할까요?

Arduino와 nodeJS간에는 어떠한 연관성도 없습니다.

사실, 이 작업은 nodeJS가 아닌 ruby로도 할 수 있고, Flash의 ActionScript로도 할 수 있죠.

nodeJS와 Arduino 간에 연결 방법은 여러가지가 있습니다만, 단순히 Arduino 보드를 구매하신 분이라면.. USB 케이블 밖에 없겠죠?

다시 말해, nodeJS에서 USB Serial 통신으로 Arduino에 메시지를 보내고, Arduino에서는 보낸 메시지에 따라 LED를 켰다 껐다 하면 되겠죠.

이제 Arduino를 제어하는 주체가 nodeJS 쪽이 되야하기 때문에, 아래와 같이 JS 코드가 작성 될 것입니다.

```js
var isOn = false;

setInterval(function(){
    if ( !isOn ) { // OFF면
        setLED(1);
    } else { // ON이면
        setLED(0);
    }

    isOn = !isOn;
}, 1000);

function setLED(flag) {
    // TODO: 뭔가 데이터를 장치로 보내는 코드가 있을꺼임
}
```

이제 setLED만 완성하면 되겠네요!

가만, Arduino쪽에서도 기존의 Blink 코드를 바꿔줘야할 것인데...

일단, nodeJS에서 setLED에 1과 0을 줬기 때문에 Arduino쪽에서도 마찬가지로 '1'을 보내면 LED가 ON, '0'을 보내면 LED가 OFF라고 합시다.

여기서 '1' 또는 '0'이 되는건 이유는 없고, 단순히 데이터 교환을 위한 약속이라고 보시면 됩니다.

뭔가 프로토콜이 정해진것 같으니.. Arduino쪽 loop() 함수 코드를 다음과 같이 수정합니다.

```c
void setup() {
  Serial.begin(9600);

  // initialize the digital pin as an output.
  // Pin 13 has an LED connected on most Arduino boards:
  pinMode(13, OUTPUT);
}

void loop() {
  int incomingValue = 0;           // nodeJS에서 보낸값

  if ( Serial.available() > 0 ) {  // 뭔가 입력값이 있다면
    incomingValue = Serial.read();
  }

  if ( incomingValue == 49 ) {     // 값이 '1' 이면
    digitalWrite(13, HIGH);        // LED를 켠다.
  }

  if ( incomingValue == 48 ) {     // 값이 '0' 이면
    digitalWrite(13, LOW);         // LED를 끈다.
  }
}
```


자, 이젠 nodeJS쪽에서만 잘 보내면 되겠네요 :D

근데, 어떻게 보낼까요?

OS에서는 [Device File](http://en.wikipedia.org/wiki/Device_file)이라는게 있는데요,

간단히 말해 장치가 가상의 파일과 1:1로 연결되어 입/출력을 주거니 받거니 한다는 것입니다.

예를 들어, 마우스를 움직이면 화면에 커서가 움직이는것 처럼, 마우스와 OS간에 뭔가 서로 통신을 하는거죠.

Arduino도 장치이기 때문에 컴퓨터와 나름 입/출력을 할 수 있는 뭔가가 있을껍니다.

nodeJS쪽에서 Arduino에 연결하기 위해서는... 일단 Arduino가 연결된 장치를 찾아야겠죠?

앞에서 설치한 Arduino IDE를 실행해 보면 메뉴에 Tools > Serial Port 쪽에서 사용 가능한 Serial 장치를 확인할 수 있습니다. 이 중 하나를 nodeJS에서 사용하면 되겠습니다.

그럼... nodeJS에서 장치로 어떻게 연결할까요?

앞에서 장치가 '가상의 파일'로 연결되어 있다고 했으니,
[nodeJS File System API](http://nodejs.org/docs/v0.6.2/api/fs.html)를 사용하면 될 것 같네요.

여기서는 Arduino가 /dev/tty-usbserial1 에 연결되어 있다고 가정합시다.

일단 장치를 열어야겠죠?

```js
var fs = require('fs');

// Appending 모드('a')로 /dev/tty-usbserial1 장치를 Open
fs.open('/dev/tty-usbserial1', 'a', 666, function( e, fd ) {
    // 장치가 정상적으로 열리면 값을 보내야겠죠?
});
```

그렇다면 값은 어떻게 보낼까요? 파일을 읽고 쓰는것 처럼, 장치에 뭔가를 보내려면 파일에 기록을 하는 형태가 되면 될 것 같습니다.

여기서는 [fs.write](http://nodejs.org/docs/v0.6.2/api/fs.html#fs.write)를 사용해보도록 하겠습니다.

`fs.write()` API는 아래와 같습니다.

>fs.write(fd, buffer, offset, length, position, [callback])

일단 fd는 fs.open이 성공적으로 이뤄지면 받을 수 있겠죠?

buffer에 장치에 실어보낼 값을 넣어주면 되겠네요. 나머지는 마지막 callback을 제외하고 크게 문제가 안되니 null로 처리해 봅시다.

```
var fs = require('fs');
// Appending 모드('a')로 /dev/tty-usbserial1 장치를 Open

fs.open('/dev/tty-usbserial1', 'a', 666, function (e, fd) {
    fs.write( fd, '1', null, null, null, function(){
        // 장치에 '1'이 잘 보내졌을 경우
    }
});
```

일단 장치에 연결을 잘 했고, '1'을 보냈다면 LED가 켜지겠죠?


혹시 안켜진다면.. Arduino IDE가 해당 장치를 사용하고 있을 수 있습니다. 닫고, 다시 실행해보세요.


(그것도 아니라면.. 음.. `=_=;;`)

LED가 잘 켜졌으니, 장치에 더 이상 보낼 값이 없겠죠? 해당 연결을 닫아줘야겠습니다.


이때는 fs.close()를 사용해 현 시점에서 연결을 종료시켜줍니다.

```
var fs = require('fs');

// Appending 모드('a')로 /dev/tty-usbserial1 장치를 Open
fs.open('/dev/tty-usbserial1', 'a', 666, function( e, fd ) {
    fs.write( fd, '1', null, null, null, function(){
        // 장치에 '1'이 잘 보내졌을 경우, 연결을 닫는다.
        fs.close(fd, function() {
            // 연결 종료.
        });
    }
});
```

여기까지 잘 따라했다면, 위에서 만든 코드를 가지고 setLED 함수를 만들어봅시다.

setLED에 1을 주면 '1'이 전송되고, 0을 주면 '0'이 전송되게 하면 되겠죠?

```js
function setLED(flag) {
    var fs = require('fs');

    // Appending 모드로 /dev/tty-usbserial1 장치를 Open
    fs.open('/dev/tty-usbserial1', 'a', 666, function( e, fd ) {
        // flag가 0이 아니면 '1'을 보내고,
        //        0이면 '0'을 보낸다.
        fs.write( fd, flag ? '1' : '0', null, null, null, function(){
            fs.close(fd, function(){
                console.log('Sending ', flag);
            });
        });
    });
}
```

여기까지 완성한 후, 실행을 시키면 LED가 깜빡이는 것을 볼 수 있을겁니다.

```js
var isOn = false;

setInterval(function(){
    if ( !isOn ) { // OFF면
        setLED(1);
    } else {       // ON이면
        setLED(0);
    }

    isOn = !isOn;
}, 1000);

function setLED(flag) {
    var fs = require('fs');

    // Appending 모드로 /dev/tty-usbserial1 장치를 Open
    fs.open('/dev/tty-usbserial1', 'a', 666, function( e, fd ) {
        // flag가 0이 아니면 '1'을 보내고,
        //        0이면 '0'을 보낸다.
        fs.write( fd, flag ? '1' : '0', null, null, null, function(){
            fs.close(fd, function(){
                console.log('Sending ', flag);
            });
        });
    });
}
```

일단 연결은 했으니 웹에 붙이는건 정말 쉽겠죠?

그런데, 연결 과정이 다소 복잡해 보이죠.

> '좀 더 쉬운 방법은 없을까?'

물론 구글링 하면 node에서 arduino를 쉽게 제어할 수 있는 몇몇 Project를 Github에서 찾을 수 있습니다.

[다음 가이드](http://kyungw00k.wordpress.com/2011/11/22/nodejs%eb%a1%9c-arduino-%ec%a0%9c%ec%96%b4%ed%95%98%ea%b8%b0-2/)에서는  
[tobeytailor가 만든 node-arduino 소스](https://github.com/tobeytailor/node-arduino)를 시작으로 
[voodootikigod가 수정한 node-arduino](https://github.com/voodootikigod/node-arduino)와,
[제가 살짝 수정한 node-arduino](http://github.com/kyungw00k/node-arduino)를 예를 들어 살펴보도록 하겠습니다.
