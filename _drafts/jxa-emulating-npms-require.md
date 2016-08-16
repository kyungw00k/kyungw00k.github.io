---
layout: post
title: Emulating npm's require()
date: 2016-05-11 07:28:22.000000000 +09:00
type: post
categories:
- JXA
- JavaScript
---

```js
ObjC.import('Foundation');
var fm = $.NSFileManager.defaultManager;
var require = function (path) {
  var contents = fm.contentsAtPath(path.toString()); // NSData
  contents = $.NSString.alloc.initWithDataEncoding(contents, $.NSUTF8StringEncoding);

  var module = {exports: {}};
  var exports = module.exports;
  eval(ObjC.unwrap(contents));

  return module.exports;
};
```
([from JXA-Cookbook/wiki/Importing-Scripts#emulating-npms-require](https://github.com/dtinth/JXA-Cookbook/wiki/Importing-Scripts#emulating-npms-require))
