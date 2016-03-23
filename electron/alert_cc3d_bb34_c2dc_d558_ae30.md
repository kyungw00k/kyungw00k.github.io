# Alert 창 무시하기

https://discuss.atom.io/t/how-to-disable-alert-dialogs-when-errors-occur/20037

```js
const dialog = require('electron').dialog;
dialog.showOpenDialog = function() {/* do nothing */};
dialog.showMessageBox = function() {/* do nothing */};
dialog.showErrorBox = function() {/* do nothing */};
dialog.showSaveDialog  = function() {/* do nothing */};
```