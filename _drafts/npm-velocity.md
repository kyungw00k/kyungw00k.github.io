---
layout: post
title: velocity
date: 2016-01-20 16:34:12.000000000 +09:00
type: post
categories:
- JavaScript
- NPM
---
> A node velocity template engine.
> (from [velocity](https://github.com/fool2fish/velocity))

템플릿 엔진을 떠올렸다면, 맞다.
그 `velocity`가 맞다. 하지만 Java를 사용하지 않아서 가볍게 쓰기에 좋다.

## Usage
```js
var Engine = require('velocity').Engine
var engine = new Engine( { template: './template.vm'} )
var result = engine.render( {
    name : 'world'
})
console.log(result) // Hello, world
```

```sh
# template.vm

Hello, $name
```
