---
layout: post
title: meow
date: 2016-02-12 00:12:11.000000000 +09:00
type: post
categories:
- JavaScript
- NPM
---
> CLI app helper
> (from [meow](https://github.com/sindresorhus/meow))

## Usage
```js
#!/usr/bin/env node
'use strict';
const meow = require('meow');
const foo = require('./');

const cli = meow(`
    Usage
      $ foo <input>

    Options
      -r, --rainbow  Include a rainbow

    Examples
      $ foo unicorns --rainbow
      🌈 unicorns 🌈
`, {
    alias: {
        r: 'rainbow'
    }
});
/*
{
    input: ['unicorns'],
    flags: {rainbow: true},
    ...
}
*/

foo(cli.input[0], cli.flags);
```
