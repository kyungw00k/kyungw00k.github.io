# 링크를 브라우저에서 열기

```js
// `win` is an instance of BrowserWindow
win.webContents.on('new-window', function (e, url) {
    e.preventDefault()
    require('shell').openExternal(url)
  })
```