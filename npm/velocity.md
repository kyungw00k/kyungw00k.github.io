# velocity

템플릿 엔진. 그 `velocity`가 맞다. 가볍게 쓰기에 좋다. 

## Usage

```js
//
// templateRender module
//

var Engine = require('velocity').Engine

function templateRender (context, template) {
  var instance = new Engine({
    template: template
  })

  return instance.render(context)
}

module.exports = templateRender
```
