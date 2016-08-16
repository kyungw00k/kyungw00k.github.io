---
layout: post
title: Finding length of the sequence
date: 2016-02-10 00:18:55.000000000 +09:00
type: post
categories:
- CodeWars
---
## My Solution
```js
var lengthOfSequence = function (arr, n) {
  var last = arr.lastIndexOf(n),
      first = arr.indexOf(n);

  if ( last === -1 || first === -1 || arr.length === 1 ||  arr.slice(first+1).indexOf(n) + first + 1 !== last) {
    return 0;
  }

  return last - first + 1;
};
```

하지만...

`filter`를 이용하면 좀 더 쉬울 수 있겠다.

```js
var lengthOfSequence = function (arr, n) {
  return arr.filter(function(v){ return v == n }).length == 2 ? arr.lastIndexOf(n) - arr.indexOf(n) + 1 : 0
};
```
