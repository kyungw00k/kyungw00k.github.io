# Nightmare.js에서 Flash Player 플러그인 실행하기

페이지 Content에 Flash가 있는 경우라면?

## Flash Player를 설치
https://get.adobe.com/kr/flashplayer/

## Nightmare.js를 설치
```
npm install --save nightmare
```

## `<project>/node_modules/nightmare/lib/runner.js`를 수정
```
...

// Specify flash path.
// On Windows, it might be /path/to/pepflashplayer.dll
// On OS X, /path/to/PepperFlashPlayer.plugin
// On Linux, /path/to/libpepflashplayer.so
app.commandLine.appendSwitch('ppapi-flash-path', '/path/to/libpepflashplayer.so');

// Specify flash version, for example, v19.0.0.226
app.commandLine.appendSwitch('ppapi-flash-version', '19.0.0.226');

app.on('ready', function() {
...
```

## Nightmare 사용 예제
```
var Nightmare = require('nightmare');

var options = {
    'width': 800,
    'height': 600,
    'plugin':true,
    'web-preferences': {
        'plugins': true
    }
};

var wwwDaumNet = new Nightmare(options)
    .viewport(1000, 1000)
    .useragent("mozilla/5.0 (macintosh; intel mac os x 10_11_1) applewebkit/601.2.7 (khtml, like gecko) version/9.0.1 safari/601.2.7")
Chrome/38.0.2125.111 Safari/537.36")
    .goto('http://www.daum.net')
    .wait(5000)
    .screenshot('wwwDaumNet.png')
    .run(function (err, nightmare) {
      if (err) return console.log(err);
      console.log('Done!');
    });
```

## Refer to
https://github.com/atom/electron/blob/master/docs/tutorial/using-pepper-flash-plugin.md