---
layout: post
title: AdBlock과 친해지기
date: 2016-02-26 18:37:11.000000000 +09:00
type: post
published: false
status: publish
---
# AdBlock과 친해지기

AdBlock이 어떻게 동작하는지 궁금해서 생각나는데로 `필요한 부분`만 정리해본다.

## Why

일부 로직을 가져와 광고 찾아주는 CLI aka `ads-finder`을 만들어볼까 해서 고민을 해봤는데...

사실, 사내에서 이미 작업은 진행중이고, PhantomJS로 되어있다.

RuleSet을 별도의 파일로 관리하고 있는걸 Develop 시켜볼까 하고 AdBlock을 들여다보게 되었는데...

뭐 여튼 그러하다.

## Overview

큰 흐름은...

1. Rule Set을 내려받은 후, 내부에서 사용하는 Filter(정규 표현식)로 변환한다.
2. Domain Filter
    1. 페이지를 방문할 때 발생하는 모든 Resource(`img src`, `script src` 등등) URL의 도메인을 살펴보고
    2. Filter에 Match되면 제거하거나 숨긴다.
3. CSS Filter
  1. 페이지 `onload` 이후에 모든 CSS RuleSet에 대해 query 한 후,
  2. 해당 Node를 (CSS로) 숨김 처리한다.
4. Success Callback 실행


Extension은 대략 다음과 같은 형태로 동작한다.

```
----------------------------------------------------
[Background]
- 외부에서 RuleSet을 가져와서 FilterSet으로 변환한 후
  LocalStorage에 보관한다.
- 주기적으로 RuleSet을 업데이트한다.
----------------------------------------------------
                        |
                        |
                        | 준비가 완료되면 Foreground에
                        | 이벤트로 알려준다.
                        |
                        |
----------------------------------------------------
[Foreground]
- Resource 요청시에 URL이 Domain 차단 Rule에 포함하면
  해당 노드를 제거한다.
- 페이지 로딩 이후 숨겨야 하는 광고가 있다면 숨긴다.
----------------------------------------------------
```
