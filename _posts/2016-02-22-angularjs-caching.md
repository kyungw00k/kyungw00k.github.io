---
layout: post
title: AngularJS에서 Caching 처리 하기
date: 2016-02-22 12:55:54.000000000 +09:00
type: post
categories:
- AngularJS
---
Feb 22 11:40:38 2016
> 특정 URI만 no cache 처리 해야하는 경우?

[angular-cache-buster](https://github.com/saintmac/angular-cache-buster) 사용

## Usage

```js
angular
    .module('app')
    .config(function (/* ... , */httpRequestInterceptorCacheBusterProvider /* ,... */) {
        //Cache everything except rest api requests
        httpRequestInterceptorCacheBusterProvider.setMatchlist([/.*api.*/], true);
        //...
    }
```
