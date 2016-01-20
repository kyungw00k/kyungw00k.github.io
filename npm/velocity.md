# velocity
> pure node용 velocity template renderer

템플릿 엔진을 떠올렸다면, 맞다.
그 `velocity`가 맞다. 가볍게 쓰기에 좋다. 


## Usage


```sh
# template.vm

$name
```

```js
// context.js
module.exports = {
    name : '나야'
}
```

```js
// templateRender.js

var Engine = require('velocity').Engine

function templateRender (context, template) {
  var instance = new Engine({
    template: template
  })

  return instance.render(context)
}

module.exports = templateRender
```

```js
const context = require('./context.js');
const template  = require('./template.vm');
templateRender(context, __dirname + '/../mjs/adam.B.vm')
```