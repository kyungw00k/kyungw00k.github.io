---
layout: post
title: blink-diff
date: 2016-01-20 16:23:26.000000000 +09:00
type: post
categories:
- JavaScript
- NPM
---

> A lightweight image comparison tool
> (from [blink-diff](http://yahoo.github.io/blink-diff/))

imagemagick 등의 dependencies가 없음

## Usage

```js
const BlinkDiff = require('blink-diff')

function noop () {
  return 0
}

function compareImages (options, callback) {
  callback = callback || noop

  var diff = new BlinkDiff({
    imageAPath: options.imageAPath,
    imageBPath: options.imageBPath,

    thresholdType: BlinkDiff.THRESHOLD_PERCENT,
    threshold: options.threshold || 0.01 // 1% threshold
  })

  diff.run(function (error, result) {
    if (error) {
      throw error
    }

    callback(diff.hasPassed(result.code), result.differences)
  })
}

module.exports = compareImages
```
