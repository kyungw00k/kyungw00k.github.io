# Caching

> 특정 URI만 no cache 처리 해야하는 경우?

[angular-cache-buster](https://github.com/saintmac/angular-cache-buster) 사용

```js
angular
    .module('app')
    .config(function (/* ... , */httpRequestInterceptorCacheBusterProvider /* ,... */) {
        //Cache everything except rest api requests
        httpRequestInterceptorCacheBusterProvider.setMatchlist([/.*api.*/], true);
        //...
    }
```