---
layout: post
title: node-fixmyjs
date: 2011-11-13 23:20:41.000000000 +09:00
type: post
published: true
status: publish
categories:
- Node.js
---

재미난 모듈이 있어 소개해본다.

https://github.com/goatslacker/node-fixmyjs

Static Code Analysis Tool인 [JSHint](https://github.com/goatslacker/jshint)를 바탕으로 작성한 코드에 문제점을 수정해주는 툴이다.

사실 node-fixmyjs는 코드 수정은 [jshint-autofix](https://github.com/goatslacker/jshint-autofix) 모듈을 이용하고, 이를 Node에서 사용할 수 있게 Wrapping한 것이다.

현재 jshint-autofix에서 지원해주는 코드 수정은 다음과 같다.

- Missing semicolon.
- Missing spaces. white
- Multiple definitions of a variable in scope.
- Statements written better in dot notation vs square bracket notation.
- Mixed spaces/tabs
- Unnecessary semicolons
- Removes confusing trailing decimal points
- Obj & Array literals instead of new Array | new Object
- Adds 0 to leading decimals
- Adds parenthesis when invoking a constructor without them
- Removes undefined when assigning to variables
- Removes debugger statements
- Uses isNaN function rather than comparing to NaN
- Moves the invocation of a function within it's parenthesis

작성해둔 코드 한번씩 돌려보기 바란다.

단, 주의할 점이 있다면 모듈 실행시 파일을 수정해 덮어쓰니, 필요한 경우에는 따로 백업을 해놓고 사용하는 것이 좋겠다.
