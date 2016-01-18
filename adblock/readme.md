# AdBlock과 친해지기

AdBlock이 어떻게 동작하는지 궁금해서 정리해본다.

일단 큰 흐름은...

1. Rule Set을 내려받은 후, 내부에서 사용하는 Filter(정규 표현식)로 변환한다.
    1. 
2. Domain Filter
    1. 페이지를 방문할 때 발생하는 모든 Resource(`img src`, `script src` 등등) URL의 도메인을 살펴보고
    2. Filter에 Match되면 제거하거나 숨긴다.
3. CSS Filter
  1. 페이지 `onload` 이후에 모든 CSS RuleSet에 대해 query 한 후,
  2. 해당 Node를 (CSS로) 숨김 처리한다.
4. Success Callback 실행

일부 로직을 가져와 `광고 찾아주는 CLI`을 만들어볼까 해서 고민을 해봤는데...

**넘어야할 산** 

* [ ] RuleSet Fetch & Parsing
  * RuleSet은 주기적으로 업데이트 해서 LocalStorage에 저장하면 되겠고..

* [ ] `Domain Filter 적용`을 위해 어떤 DOM이 Resource를 요청하는지 가로채는 방법 해결하기
  * `PhantomJS` 에서는 HTTP Request를 요청한 Node 정보를 알 수가 없다.
  * Safari 계열에서는 `beforeload` 이벤트를 사용할 수 있다.
  * Chrome 계열에서는 `chrome.webRequest.onBeforeRequest` API가 구현되어 있으면 사용이 가능한데, Electron에서는 ~~가능할 것 같다~~ [가능하다](http://github-bj.daocloud.io/atom/electron/blob/d9d821cea5a2425a4dde23f7eb630250cd237a60/spec/api-web-request-spec.js).

* [ ] `CSS Filter 적용`
  * RuleSet만 Parsing되어 있다면 onLoad 이후 찾을 수 있다.
