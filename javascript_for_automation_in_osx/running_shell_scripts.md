# Running Shell Scripts

## Method 1. Use `app.doShellScript`
https://github.com/dtinth/JXA-Cookbook/wiki/Shell-and-CLI-Interactions#running-shell-scripts

### Example
```js
app.doShellScript('ls')
  //    => "README.md\rbutton.svg"
```
**Note**
* `stdout` or `stdout` cannot be captured by code.
* But easy to use.

## Method 2. Use `Cocoa`
http://stackoverflow.com/questions/27586694/pipe-to-subprocess-stdin-for-jxa

### Example
```js
ObjC.import('Cocoa')
var stdin = $.NSPipe.pipe
var stdout = $.NSPipe.pipe
var task = $.NSTask.alloc.init
task.launchPath = "/bin/cat"
task.standardInput = stdin
task.standardOutput = stdout

task.launch

var dataIn = $("foo$HOME'|\"").dataUsingEncoding($.NSUTF8StringEncoding)
stdin.fileHandleForWriting.writeData(dataIn)
stdin.fileHandleForWriting.closeFile

var dataOut = stdout.fileHandleForReading.readDataToEndOfFile
var stringOut = $.NSString.alloc.initWithDataEncoding(dataOut, $.NSUTF8StringEncoding).js
console.log(stringOut)
```

**Note**
* `stdout` and `stdout` can be captured by code.
* But easy to use.
