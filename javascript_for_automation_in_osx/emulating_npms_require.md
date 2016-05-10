# Emulating npm's require()

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