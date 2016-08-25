---
layout: post
title: requirejs와 browserify가 충돌나는 경우
date: 2016-02-22 15:45:24.000000000 +09:00
type: post
categories:
- JavaScript
- Browserify
- RequireJS
---
## Problem

`browserify`로 만든 library js가 기존에 `requirejs`를 사용하고 있는 사이트에서 작동하지 않는 현상

## Solution

**1. Module Export 하는 부분을 실제로 사용하는 코드만 남기고 제거한다.**

``` js
// AS-IS
if (typeof define === 'function' && define.amd) { // window.define이 존재해서 export가 안되었음
    define(function () {
        return viewBinder;
    });
} else if (typeof module === 'object' && module.exports) {
    module.exports = viewBinder;
} else {
    exports.viewBinder = viewBinder;
}
```

``` js
// TO-BE
// 실제 프로젝트 내에서 모듈을 사용하는 기준은 아래 코드를 사용한다.
module.exports = viewBinder;
```

**2. `window.requirejs`를 바라보지 않도록 `browserify-derequire` 플러그인을 사용한다.**
