---
layout: post
title: 'Node.js로 Arduino 제어하기 #2'
date: 2011-11-22 13:40:03.000000000 +09:00
type: post
published: true
status: publish
categories:
- Node.js
- Arduino
- JavaScript
---
[Node.js로 Arduino 제어하기 #1](http://kyungw00k.wordpress.com/2011/11/21/nodejs%eb%a1%9c-arduino-%ec%a0%9c%ec%96%b4%ed%95%98%ea%b8%b0-1/)에 이어서 arduino API를 Wrapper한 package를 사용해 nodeJS에서 Arduino를 제어하는 방법을 살펴보겠다.

먼저 [tobeytailor의 node-arduino](https://github.com/tobeytailor/node-arduino)를 살펴보자.

* 현재 npm에 등록된 Arduino 패키지의 부모 프로젝트로, Arduino API 중 pinMode와 Digital/Analog Write만 사용 가능하다.
* nodeJS와 Arduino의 통신은 node-serialport 모듈을 이용한다.

위 소스를 nodeJS에서 사용하기 전에, Arduino에 업로드해야할 소스(node.pde)를 먼저 살펴보도록 하자.

```c
/* 소스 위치 : https://raw.github.com/tobeytailor/node-arduino/master/src/node.pde */

/*
 *  node-arduino: Control your Arduino with Node
 *
 *  Copyright (c) 2010 Tobias Schneider
 *  node-arduino is freely distributable under the terms of the MIT license.
*/

#define SERIAL_BAUDRATE 9600

#define OPC_PIN_MODE         0x01 /* nodeJS에서 해당 상수값이 들어오면,  */
#define OPC_DIGITAL_READ     0x02 /* Arduino에서 지정된 함수를 실행한다. */
#define OPC_DIGITAL_WRITE    0x03
#define OPC_ANALOG_REFERENCE 0x04
#define OPC_ANALOG_READ      0x05
#define OPC_ANALOG_WRITE     0x06

void setup() {
  Serial.begin(SERIAL_BAUDRATE);
}

void loop() {
  while (Serial.available() > 0) {
    switch (Serial.read()) {
      case OPC_PIN_MODE: { // Ex. nodeJS에서는 0x01와 pin값을 차례로 전송한다.
        Serial.println("pinMode");
        pinMode(Serial.read(), Serial.read());
        break;
      }
      case OPC_DIGITAL_READ: { // Ex. nodeJS에서는 0x02 값을 전송한다.
        digitalRead(Serial.read());
        break;
      }
      case OPC_DIGITAL_WRITE: {
        Serial.println(&quot;digitalWrite&quot;);
        digitalWrite(Serial.read(), Serial.read());
        break;
      }
      case OPC_ANALOG_REFERENCE: {
        analogReference(Serial.read());
        break;
      }
      case OPC_ANALOG_READ: {
        analogRead(Serial.read());
        break;
      }
      case OPC_ANALOG_WRITE: {
        analogWrite(Serial.read(), Serial.read());
        break;
      }
    }
  }
}
```

위 Arduino 코드에서는 기존의 API에서 주로 사용하는 pinMode, digitalRead/Write, analogRead/Write를 임의의 상수로 정의하고, nodeJS에서 해당 상수값을 보내면 해당 함수를 실행하는 형태가 되겠다.

이와 같은 방법으로 Arduino를 nodeJS에서 컨트롤하게 되는 것이다.(간단하죠?)

nodeJS쪽 코드를 사용하기 전에 반드시 위 코드를 Arduino IDE를 사용해 보드에 Upload 하도록 하자.

그럼 [nodeJS쪽 소스(arduino.js)](https://raw.github.com/tobeytailor/node-arduino/master/lib/arduino.js)를 살펴보도록 하자.

```c
/*
 * 소스 위치 : https://raw.github.com/tobeytailor/node-arduino/master/lib/arduino.js
 */
/*
 *  node-arduino: Control your Arduino with Node
 *
 *  Copyright (c) 2010 Tobias Schneider
 *  node-arduino is freely distributable under the terms of the MIT license.
 */

var sys = require('sys')
  , SerialPort = require('../deps/node-serialport/serialport').SerialPort
  ;

const SERIAL_BAUDRATE = 9600;

const OPC_PIN_MODE         = 0x01; // Arduino쪽 코드와 값을 맞춘다.
const OPC_DIGITAL_READ     = 0x02;
const OPC_DIGITAL_WRITE    = 0x03;
const OPC_ANALOG_REFERENCE = 0x04;
const OPC_ANALOG_READ      = 0x05;
const OPC_ANALOG_WRITE     = 0x06;

exports.HIGH = 0x01;
exports.LOW  = 0x00;

exports.INPUT  = 0x00;
exports.OUTPUT = 0x01;

exports.true  = 0x01;
exports.false = 0x00;

exports.EXTERNAL = 0x00;
exports.DEFAULT  = 0x01;
exports.INTERNAL = 0x03;

Board = function (path) {
  this.sp = new SerialPort(path, SERIAL_BAUDRATE);
}

Board.prototype = {
  pinMode : function (pin, mode) {
    // 값을 Buffer에 실어서 SerialPort 객체의 write 메서드를 통해 전송한다.
    this.sp.write(new Buffer([OPC_PIN_MODE, pin, mode]), 3);
  }

, digitalRead : function (pin) {
    // TODO
  }

, digitalWrite : function (pin, val) {
    this.sp.write(new Buffer([OPC_DIGITAL_WRITE, pin, val]), 3);
  }

, analogReference : function (type) {
    this.sp.write(new Buffer([OPC_ANALOG_REFERENCE, type]), 2);
  }

, analogRead : function (pin) {
    // TODO
  }

, analogWrite : function (pin, val) {
    this.sp.write(new Buffer([OPC_ANALOG_WRITE, pin, val]), 3);
  }
};

exports.connect = function (path) {
  return new Board(path);
};
```

결국은 Arduino에 사용할 함수를 미리 상수로 정의하고, nodeJS쪽에서 해당 상수값을 보내서 Arduino API를 사용하면 되는것이다.

생각보다 간단하다.(아닌가? `=_=;`)

위 코드를 사용해서 Arduino로 LED 깜빡이는 예제를 살펴보자.


(아래 코드를 실행하기 전에 Arduino IDE에서 Serial Monitor를 띄워주도록 하자. 이 문제점에 대한 해결책은 아래에서 다시 설명하도록 한다.)

```js
/*
 *  node-arduino: Control your Arduino with Node
 *
 *  Copyright (c) 2010 Tobias Schneider
 *  node-arduino is freely distributable under the terms of the MIT license.
 */

var arduino = require('../lib/arduino')
  , board = arduino.connect('/dev/tty.usbserial-A700fkLn')
  , val = arduino.LOW
  ;

board.pinMode(13, arduino.OUTPUT); // 13번으로 출력함

setInterval(function () {
  val = val == arduino.LOW ? arduino.HIGH : arduino.LOW;
  board.digitalWrite(13, val);
}, 500);
```

실행하면 보드에 LED가 깜빡거릴것이다.

이전 포스트에서 `fs.open`, `fs.write` 뭐 이런거 전혀 사용 안하고 직관적인 코드로 LED를 제어할 수 있게 되었다!(와~:D)

하지만 [arduino.js 소스](https://raw.github.com/tobeytailor/node-arduino/master/lib/arduino.js)를 보면, 정작 중요한(?) read 함수는 TODO 상태다.

Serial 통신이라 보니 뭔가 주거니 받거니에 대한 read 처리를 JavaScript로 Sync 한 방법으로 처리가 애매해서 구현을 남겨두지 않았나 싶었다.

실제로 위 소스를 가지고 막상 실습을 해보면 뭔가 이상한점을 발견하게 되는데,
**Arduino IDE의 Serial Monitor를 띄우지 않고서는 실행이 제대로 안된다는 점**
이다.

당시에는 단순 버그려니 하고 넘겼지만, [node-serialport쪽 예제 코드](https://github.com/voodootikigod/node-serialport)를 살펴보고난 후에 arduino.js에 빠진 부분을 찾았다.

아래는 node-serialport 소개 페이지에 있는 셈플 코드다.

```js
var serialport = require('serialport');
var SerialPort = serialport.SerialPort; // localize object constructor

var sp = new SerialPort('/dev/tty-usbserial1', {  // 장치 /dev/tty-usbserial1에 연결한다.
  parser: serialport.parsers.raw
});

serialPort.on('data', function (data) { // 데이터가 들어오면 출력한다.
    sys.puts('here: '+data);
});
```

간단히 말하면, 위와 같이 arduino.js에 data 이벤트를 처리할 수 있는 함수가 정의되어 있어야 한다는 것이다.

arduino.js에 data 이벤트 처리 함수를 넣어주면 실질적으로 Port Monitering을 `node-serialport`에서 하기 때문에 Arduino IDE의 Serial Monitor를 실행시키지 않아도 된다.

```js
var sys = require('sys')
  , SerialPort = require('../deps/node-serialport/serialport').SerialPort
  ;

const SERIAL_BAUDRATE = 9600;

const OPC_PIN_MODE         = 0x01; // Arduino쪽 코드와 값을 맞춘다.
const OPC_DIGITAL_READ     = 0x02;
const OPC_DIGITAL_WRITE    = 0x03;
const OPC_ANALOG_REFERENCE = 0x04;
const OPC_ANALOG_READ      = 0x05;
const OPC_ANALOG_WRITE     = 0x06;

// ...

Board = function (path) {
  this.sp = new SerialPort(path, SERIAL_BAUDRATE);
  this.sp.on( &quot;data&quot;, function(data){ console.log(data); }); // 추가 코드 : data 이벤트 발생시 출력
}

// ...
```

이렇게 Digital/Analog Write만으로도 node에서 Arduino로 자유롭고 간편하게 출력을 할 수 있겠다. 하지만.. 단순 Write만 하기에는 재밌는 센서들이 너무나도 많다!

[제가 임의로 Read를 추가한 node-arduino](https://github.com/kyungw00k/node-arduino)는 [voodootikigod의 node-arduino](https://github.com/voodootikigod/node-arduino) 패키지를 Fork해서 만든 것으로, 기존에 Read 부분을 추가했다. voodootikigod가 pull request를 받아준다면 npm에 반영될꺼라 생각했지만... (깔끔하지 못하다고 느꼈는지 몰라도 일단 받아주지 않고 있다 =_=)

일단 추가한 Read 부분을 설명하기 앞서, 어떻게 하면 JavaScript 답게 Read를 하면 좋을까 생각을 했었다. 해서 내린 결론은...

* Arduino에서 pin마다 입력 값이 생기면, 이를 nodeJS에 값과 pin값을 묶어서 전송한다.
* nodeJS쪽에서는 pin마다 입력값이 들어오면 실행될 Callback을 둔다.

일단 Arduino와 nodeJS는 Serial 통신으로 데이터를 주고 받는다.

![](/images/2011/11/22/processingread.png)

그렇다면 Arduino쪽에서 뭔가 값을 읽는 즉시 출력을 해야 nodeJS쪽의 data 이벤트 헨들러의 Callback이 데이터를 받을 수 있을 것이다.

하나의 값의 출력으로 pin번호까지 같이 전달해야하고, 일관된 데이터 양식이면 좋겠다는 생각에 일단, 간단히 packing을 하기로 했다.

Arduino에서는 기본적으로 int가 2byte다.(long이 4byte) 보통 센서 값이 10000이 넘지 않는다고 해서 일단 하위 2byte에 Digital/Analog Read로 읽어드린 값을 넣고, 상위 2byte에 pin번호 값을 넣어서 4byte 숫자 String을 전송하는 형태로 데이터 양식을 정해봤다.

자, 그럼 첫번째로 Arduino쪽 수정된 코드를 살펴보자.

```c
/*
 *  node-arduino: Control your Arduino with Node
 *
 *  Copyright (c) 2010 Tobias Schneider
 *  node-arduino is freely distributable under the terms of the MIT license.
 */

#define SERIAL_BAUDRATE 115200

#define OPC_PIN_MODE         0x01
#define OPC_DIGITAL_READ     0x02
#define OPC_DIGITAL_WRITE    0x03
#define OPC_ANALOG_REFERENCE 0x04
#define OPC_ANALOG_READ      0x05
#define OPC_ANALOG_WRITE     0x06

long pinVal = 0;
int  inpVal = 0;
long outVal = 0;

void setup() {
  Serial.begin(SERIAL_BAUDRATE);
}

void loop() {
  pinVal = 0, inpVal = 0, outVal = 0;
  while (Serial.available() > 0) {
    switch (Serial.read()) {
      case OPC_PIN_MODE: {
        pinMode(Serial.read(), Serial.read());
        break;
      }
      case OPC_DIGITAL_READ: {
        delay(50);
        pinVal = Serial.read();         // pin값을 pinVal에 저장하고,
        inpVal = digitalRead(pinVal);   // 해당 핀에 연결된 값을 읽어서 inpVal에 저장한다.
        outVal = pinVal << 16 | inpVal; // pin값을 상위 2byte로 shift한 후에 inpVal을 하위 2byte에 넣는다.
        Serial.println(outVal);         // 읽어들인 값을 출력하는 형태로 전송한다.
        break;
      }
      case OPC_DIGITAL_WRITE: {
        digitalWrite(Serial.read(), Serial.read());
        break;
      }
      case OPC_ANALOG_REFERENCE: {
        analogReference(Serial.read());
        break;
      }
      case OPC_ANALOG_READ: {
        delay(50);
        pinVal = Serial.read();
        inpVal = analogRead(pinVal);
        outVal = pinVal << 16 | inpVal;
        Serial.println(outVal);
        break;
      }
      case OPC_ANALOG_WRITE: {
        analogWrite(Serial.read(), Serial.read());
        break;
      }
    }
  }
}
```

앞에서 "nodeJS쪽에서는 pin마다 입력값이 들어오면 실행될 Callback을 둔다." 라고 했다.

예를 들어, 다음과 같이 7번과 8번 핀에서 입력값을 받아들여 화면에 찍는다고 가정해보자.

```js
var arduino = require('../lib/arduino')
  , board = arduino.connect('/dev/tty.usbserial-A700fkLn')
  ;

board.pinMode(7, arduino.INPUT); // 7번 PIN은 입력단자
board.pinMode(8, arduino.INPUT); // 8번 PIN은 입력단자

setInterval(function () {
  board.digitalRead(7, function(val){
      console.log(val);
  });

board.digitalRead(8, function(val){
      console.log(val);
  });
}, 500);
```

Arduino에서 값이 들어오면, 위에서 정의한 Callback 함수를 실행해야한다.

그렇다면 두 번째로, Arduino에서 출력한 값을 nodeJS에서 읽어서 처리하는 코드를 살펴보자.

(이때, 하나의 PIN에는 하나의 Callback만이 실행한다고 가정한다.)

```js

/*
 *  node-arduino: Control your Arduino with Node
 *
 *  Copyright (c) 2010 Tobias Schneider
 *  node-arduino is freely distributable under the terms of the MIT license.
 */

var sys = require('sys'),
	SerialPort = require('serialport').SerialPort,
	parsers = require('serialport').parsers;

const SERIAL_BAUDRATE     = 115200;

const OPC_PIN_MODE         = 0x01;
const OPC_DIGITAL_READ     = 0x02;
const OPC_DIGITAL_WRITE    = 0x03;
const OPC_ANALOG_REFERENCE = 0x04;
const OPC_ANALOG_READ      = 0x05;
const OPC_ANALOG_WRITE     = 0x06;

// ...

Board = function (path) {
	// pin별로 callback을 저장하기 위한 객체
	var callback = {};

  // Arduino에서 출력되는 값을 개행 단위로 읽어들임
	this.sp = new SerialPort(path, {
    baudrate: SERIAL_BAUDRATE,
    parser: parsers.readline('\n')
  });

  var onData = (function(callback) {
			return function( data ) { // 뭔가 값이 있을때
				var pin, value;
				data = +data; // 숫자 값으로 바꾼다.(실제로 숫자 스트링이기 때문에 가능)
				if ( data && data > 1 ) {
					pin = data >> 16; // 상위 2byte에서 pin값을 꺼낸다.
					value = data &amp; 0xFFFF; // 하위 2byte에서 읽어드린 값을 꺼낸다.

          if ( !callback['pin'+pin] ) {
						sys.puts('no callback for pin '+pin);
					} else {  // 해당 핀에 지정된 Callback이 있는 경우, 값을 넘겨 호출한다.
						callback['pin'+pin](value);
					}
				}
			};
		})(callback);

  this.sp.on( &quot;data&quot;, onData);
};

Board.prototype = {
	pinMode : function (pin, mode) {
		this.sp.write(new Buffer([OPC_PIN_MODE, pin, mode]), 3);
	},
	// ...
	digitalRead : function (pin, callback) {
		// Arduino쪽에 값을 읽을 수 있게 Pin번호와 OPC_DIGITAL_READ를 전송한다.
		this.sp.write(new Buffer([OPC_DIGITAL_READ, pin]), 2);

    // callback이 함수일 경우, pinX 형태로 callback Reference를 저장한다.
		if ( typeof callback == 'function' ) {
			this.callback['pin'+pin] = callback;
		}
	},
	// ...
	analogRead : function (pin, callback) {
		this.sp.write(new Buffer([OPC_ANALOG_READ, pin]), 2);

    if ( typeof callback == 'function' ) {
			this.callback['pin'+pin] = callback;
		}
	},
	// ...
};
// ...
```

[node-arduino Read Demo](http://www.twitvid.com/HAGDJ) 영상은 위 코드를 바탕으로 테스트한 화면이다.

버튼이 눌리면 1을 찍고, 눌리지 않으면 0을 찍고 있다.

아직 node-arduino에 빠진 API가 많지만, Read, Write만 가지고도 왠만한건 다 할 수 있을것이라 생각한다.

여기까지 node-arduino를 사용해 arduino를 제어하는 법을 간단히 살펴봤다.

사실 여기까지 실습이 제대로 되었다면, 왠만한건 다 nodeJS로 엮을 수 있을 것이다. :D
