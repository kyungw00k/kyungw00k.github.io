# AdBlock

AdBlock이 어떻게 동작하는지 궁금해서 정리해본다.

일단 큰 흐름은...

1. Rule Set을 내려받은 후, 내부에서 사용하는 Filter(정규 표현식)로 변환한다.
2. Domain Filter
    1. 페이지를 방문할 때 발생하는 모든 Resource(`img src`, `script src` 등등) URL의 도메인을 살펴보고
    2. Filter에 Match되면 제거하거나 숨긴다.
3. CSS Filter
  1. 페이지 `onload` 이후에 모든 CSS RuleSet에 대해 query 한 후,
  2. 해당 Node를 (CSS로) 숨김 처리한다.
4. Success Callback 실행
