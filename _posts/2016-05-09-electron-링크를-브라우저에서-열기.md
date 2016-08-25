---
layout: post
title: Electron에서 링크를 브라우저에서 열기
date: 2016-05-09 23:01:08.000000000 +09:00
type: post
categories:
- Electron
---
```js
// `win` is an instance of BrowserWindow
win.webContents.on('new-window', function (e, url) {
    e.preventDefault()
    require('shell').openExternal(url)
  })
```
