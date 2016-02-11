# [meow](https://github.com/sindresorhus/meow)
> CLI app helper

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