# 기존 플잭에 ES6 적용하기

## 환경 구축
```
npm install -g babel
babel-node
```

## 테스트 실행
예를 들어 `mocha`라면

```
mocha --compilers js:babel/register
```

## 적용 순서
1. 테스트 코드가 통과할 때 까지 모듈 수정
1. Entry Point용 `index.js` 추가
1. `package.json` main 수정
1. `package.json` test 실행 부분

## 적용 예시
* https://github.com/kyungw00k/node-ping-proxy/commit/593825531b963784c152c99416025660254fe888

## 참고 자료
* https://github.com/lukehoban/es6features
* https://gist.github.com/rauchg/93d8b831e286bcb30d84
* http://derpturkey.com/testing-asyncawait-with-babel-and-mocha/
* https://github.com/gotwarlost/istanbul/issues/329
