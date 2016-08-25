---
layout: post
title: Electron에서 Alert 창 무시하기
date: 2016-03-23 14:37:08.000000000 +09:00
type: post
categories:
- Electron
---

```js
const dialog = require('electron').dialog;
dialog.showOpenDialog = function() {/* do nothing */};
dialog.showMessageBox = function() {/* do nothing */};
dialog.showErrorBox = function() {/* do nothing */};
dialog.showSaveDialog  = function() {/* do nothing */};
```
