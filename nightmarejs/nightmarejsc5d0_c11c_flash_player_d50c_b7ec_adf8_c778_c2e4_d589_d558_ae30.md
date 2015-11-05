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

## Refer to
https://github.com/atom/electron/blob/master/docs/tutorial/using-pepper-flash-plugin.md