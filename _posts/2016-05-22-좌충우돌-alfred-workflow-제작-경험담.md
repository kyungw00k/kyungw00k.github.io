---
layout: post
title: 좌충우돌 Alfred Workflow 제작 경험담
date: 2016-05-22 23:27:27.000000000 +09:00
type: post
published: true
status: publish
categories:
- Project
tags:
- Alfred
---
파워팩 사놓고 기본 사용자 정도로만 사용하다가, 최근에 간단한 Alfred Workflow 2개를 만들어 봤다.

* [Postify API를 사용한 우편번호 검색](https://github.com/kyungw00k/alfred-kozip-workflow)
* [Daum 책 검색 API를 사용한 책 검색](https://github.com/kyungw00k/alfred-daumbook-workflow)

뭐 경험 후기라고 해봐야 사실 별건 없지만...

* API 호출해서 JSON Output 가져다가
* Alfred에서 사용하는 Item List로 바꿔준 정도

라고 할 수 있다.

JavaScript를 좋아하는 관계로 JXA(Javascript For Automation)를 사용하게 된 점을 제외하고는..

흔한 경험담 정도일 듯 하다.

### API 호출해서 JSON Output 가져오기
CURL로 손쉽게 가져오면 되겠거니 싶었는데.. `app.doShellScript()` 를 사용해서 만들때 당황했던 점은...

Shell Command의 Output을 Capture 할 수 없다는 점이다. `-_-;;`

(뭐 사실, Output을 파일로 남기고 파일을 읽어와도 되긴 하지만...)

하지만 [Pipe to subprocess stdin for JXA](http://stackoverflow.com/questions/27586694/pipe-to-subprocess-stdin-for-jxa) 에 달린 Comment 중 하나를 사용해 손 쉽게 Output을 얻을 수 있었다.

여튼, Output을 얻었으니 `JSON.parse()`로 손쉽게 사용할 수 있게 되었다.

### Alfred에서 사용하는 Item List로 바꿔주기

이건 Alfred에서 제공하는 [가이드 문서](https://www.alfredapp.com/help/workflows/inputs/script-filter/xml/)를 살펴보면 된다.

앞에서 받은 Data를 아래와 같이 XML로 만들어 주면 되겠다.

```xml
<items>
    <item valid="YES" autocomplete="Desktop">
        <title>항목1</title>
        <subtitle>설명1</subtitle>
        <icon>아이콘1Path</icon>
    </item>
    <item valid="YES" autocomplete="Desktop">
        <title>항목2</title>
        <subtitle>설명2</subtitle>
        <icon>아이콘2Path</icon>
    </item>
</items>
```

아, Alfred 3이 나오면서 기존에 사용하던 XML 방식이 Legacy가 되버렸다.
[이제는 JSON으로!](https://www.alfredapp.com/help/workflows/inputs/script-filter/json/)

### 어쩌지! 한글이 분해가 되버렸어!
한글을 입력하면 query 값에 한글이 분해되어 들어가 있었다.

```
(╯°□°）╯︵ ┻━┻
```

어쩌지 하고 있었는데 이미 누군가 원인을 파악 했더라!

>Mac에서 alfred workflow를 만들다보면 한글 argument 비교 처리가 안 되는 현상이 있습니다. Mac에서 unicode를 NFD(Normalization Form Decomposition)으로 처리를 해서 생기는 문제입니다. Windows나 python 등에서는 NFC로 처리가 됩니다. (from http://jmjeong.com/unicode-in-alfred-workflow/)


JavaScript용을 찾다보니 [unorm](https://github.com/walling/unorm) 라이브러리를 알게 되었고, 이걸 사용해 다시 변환해서 해결했다.

```js
// Alfred에서 넘겨주는 문자열 q를 NFC로 처리한 다음 Encoding 함
encodeURIComponent(q.normalize('NFC'))
```

### 그 밖에...
* Workflow 만들다가 Step 간에 데이터 넘겨주는 부분은 어떻게 할까 싶었는데  args를 이용하면 되더라(가이드 문서에 있음)
* fn, ctrl, alt 키를 지정했는데 subtitle이 바뀌지 않아서 왜 그러나 싶었는데, 실제로 해당 키로 지정된 Action이 있는 경우에만 바뀌더라(역시 가이드 문서에 있음)

단순한 API 호출 결과를 Workflow에 보여주는 건 이제 단순한(?) 작업이 되어 버렸다.(아니면 내가 단순한건가...)

p.s. [@fallroot](https://twitter.com/fallroot?lang=ko)의 JXA 사용 경험담이 무척 큰 도움이 되었다(사실 그 전에는 JXA 존재 자체를 몰랐음). 감사요~
