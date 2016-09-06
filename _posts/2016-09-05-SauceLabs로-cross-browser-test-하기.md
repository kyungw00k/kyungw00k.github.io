---
layout: post
title: SauceLabs로 Cross Browser Test 하기
date: 2016-09-05 15:40:38.000000000 +09:00
type: post
published: true
status: publish
---

## 시작하기 전에

> Javascript Code! 브라우저 별로 어떻게 테스트 하시나요?

개인 프로젝트든 회사 업무든 JavaScript로 코드 작성 시,

- [Mocha](https://mochajs.org/)로 테스트 코드 작성하고
- [Testem](https://github.com/testem/testem)을 Local에 설치 된 브라우저에서 테스트를 하고 있다.

> 요즘 웹 개발 하기 많이 좋아지지 않았나?

(세상이 좋아졌다고 하지만 그래도) `Internet Explorer`도 버전별로(특히 7/8/9) 테스트 하고 싶고,

(요즘 대세인) 모바일 브라우저도 버젼별로 테스트 하고 싶은데...

손 쉽게(이왕이면 공짜로?) 한 방(?)에 할 수 없을까 ~~아님 내가 쓸 바퀴 한 번 만들어볼까~~ 고민하던 차에...

SauceLabs가 [OpenSource의 경우에는 무료로 Instance를 제공해준다](https://saucelabs.com/opensauce/)는 글을 봤다.

![open sauce plan](/images/2016/09/05/open-sauce-plan.png)

유료로 사용할 경우에는 EXPERT와 PROFESSIONAL의 중간 정도 이면서, Automated Testing과 Manual Testing 시간이 무려 UNLIMITED이다!

![pricing plan](/images/2016/09/05/pricing-plan.png)

> 쓸만한가?

쓸만한지 정말 궁금해서, 개발에서 사용하는 일부 모듈 중,

- 비지니스 로직없는 모듈을 Github에 등록하고,
- 이 모듈로 OpenSource Instance를 받아서

테스트를 해 봤다.

![testing page](/images/2016/09/05/testing-page.png)

> 그래서, 쓸만한가?

![browser support](/images/2016/09/05/browser-support.png)

~~이 결과를 얻기 까지 소스 및 테스트 코드를 수정하는 아픔이 있긴 했지만~~

일단, `그렇다` 고 할 수 있다.

그렇다면, OpenSauce 계정 등록부터, 테스트 코드를 Travis CI로 태우기 까지의 과정을 살펴보자.

## 0. 테스트 할 Project
일단 테스트 하려면 테스트 할 프로젝트가 있어야 하겠다.

이번 글에서는 [kyungw00k/viewport-position](https://github.com/kyungw00k/viewport-position)를 중심으로 적어본다.

처음에 이 모듈은 다음과 같이 테스트를 했었다.

- [substack/tape](https://github.com/substack/tape) 기반으로 테스트 코드를 작성했고,
- [browserify](http://browserify.org/)로 코드를 빌드(란 표현이 맞을지 모르지만 여튼) 한 다음
- [testling](https://ci.testling.com/)으로 [Local Test](https://ci.testling.com/guide/local_tests)를 한 다음
- Travis CI에서 동일한 조건으로 수행하기 위해 [Chrome을 설치해서 Local Test를 돌렸](https://github.com/kyungw00k/viewport-position/commit/dea7074d0eb5fa733deeb055da6c7fa6dcaa47c5)다.

## 1. OpenSauce 계정 등록하기
https://saucelabs.com/beta/signup/OSS/None 링크를 통해 계정을 생성한다.

이때, `username`을 본인 ID 보다는 Test 하려는 모듈 또는 Product 기준으로 하는게 좋다.

그 이유는...

![build-status-banners](/images/2016/09/05/build-status-banners.png)

테스트 결과를 Browser Badge를 받을 수 있는데, 이 기준이 `username`이었다.

> :scream: :cry: :sob: 아놔

등록을 하고 나면 다음과 같은 Dashboard를 볼 수 있다.

![dashboard](/images/2016/09/05/dashboard.png)

## 2. 개발 환경 설정 하기
만약 테스트용 프로젝트가 Mocha로 되어 있으면 일단 시작은 순조로울 듯 하다.

Tape도 된다고는 했지만 Tape 쪽 코드든 내 코드든 IE6/7에서 어딘가 모르게 스크립트 에러가 났고,

에러가 나도 테스트가 멈추지 않아 Timeout에 걸려서 자꾸만 실패를 했다.

그래서 [Getting Started with JavaScript Unit Testing Example](https://wiki.saucelabs.com/display/DOCS/Getting+Started+with+JavaScript+Unit+Testing+Example) 문서 부터 살펴보기 시작했다.

그래서 내린 결론은

> 여기서 지원하는 Framework으로 변경하지 뭐. 그 정도 시간은 투자할 수 있어.

하지만 Grunt를 쓰고 싶지 않았다.

> 아, 물론 Grunt를 싫어하는건 아니에요.<br />
> 그냥 작은 프로젝트라 그 정도(?) 까지 쓰고 싶지 않았던 것일 뿐...<br />
> (스스로 정량화 할 수 없는 그 느낌 아시죠?)

### 2.1. zuul
> [@netflix의 zuul](https://github.com/netflix/zuul)이 아니라 [@defunctzombie의 zuul](https://github.com/defunctzombie/zuul)입니다.

네, 제가 알고 있는 그 `zuul` 인지 알았는데 아니었...

`zuul`은

- 처음 부터 알고 있었던건 아니었고
- 아마 `tape`, `saucelabs` 등등의 키워드로 구글신님에게 물어보다 얻어 걸리지 않았을까

싶은데, 여튼 이걸 쓰기로 결정한 건

- Grunt가 아니었고
- CLI 었고
- SauceLabs에서도 잘 되었기 때문이었고,

> '개취'입니다 존중해주시죠

기존에 테스트 코드가 있다면 손쉽게 이용할 수 있어서 좋았다.

이를 테면...

```sh
zuul --local 8080 -- test.js
```

위 명령으로 local 8080 포트로 (물론 브라우저에서 방문하는 형태로) 테스트 코드 실행이 가능했다.

사용하기 전에 몇 가지 설정이 필요한데, 다음 두 가지는 필수 값이라 꼭 적어줘야 한다.

- UI(Test Framework)
- Browsers(브라우저 지원 범위)

프로젝트 루트에 `.zuul.yml`에 위 항목을 적어주면 된다.

보다 더 자세한 항목은 [@defunctzombie/zuul/wiki/zuul.yml](https://github.com/defunctzombie/zuul/wiki/zuul.yml)에서 살펴보기 바란다.

여튼 `zuul`과 `SauceLabs`의 조합을 사용하기 위해서는 [SauceLabs에서 받은 키를 zuul에서 사용할 수 있도록 `~/.zuulrc` 파일에 해당 내용을 적어줘야](https://github.com/defunctzombie/zuul/wiki/Cloud-testing#2-educate-zuul) 한다.

```
sauce_username: my_awesome_username
sauce_key: 550e8400-e29b-41d4-a716-446655440000
```

그 밖에 내용은 [@defunctzombie/zuul/wiki/Cloud-testing](https://github.com/defunctzombie/zuul/wiki/Cloud-testing) 문서를 참고하기 바란다.

여튼 위 설정을 거친 후에, 로컬에서 SauceLabs 조합의 테스트를 해 봤다.

![cli-testing](/images/2016/09/05/cli-testing.png)

그런데 (로컬이든 Travis CI든)테스트 과정에서 자꾸 connection refused 이슈가 발생했다.

![zuul-connection-refused](/images/2016/09/05/zuul-connection-refused.png)

좀 찾아보니 이미 [알려진 알려진 이슈](https://github.com/defunctzombie/zuul/issues/251)고, 해결 방안은

- `ngrok`를 사용해
- `concurrency: 2` 로 하면 잘 될꺼야
- `zuul-ngrok` 쓰면 될꺼야

라는 내용이 보여서 바로 테스트 해봤다.

### 2.2. [zuul-ngrok](https://github.com/rase-/zuul-ngrok)
`zuul`에서 tunneling으로 `ngrok`을 사용하게 하는 프로젝트로, `zuul-ngrok`을 사용하려면 먼저 `ngrok`을 사용할 수 있게 가입을 하든 환경을 구축하든 해야하는데,

> ngrok? 먹는거야?

![google-ngrok](/images/2016/09/05/google-ngrok.png)


[@outsider/ngrok으로 로컬 네트워크의 터널 열기](https://blog.outsider.ne.kr/1159) 글을 보면

- 일단 먹는건 아니고
- 어디에 쓰는 물건인지

잘 설명해주고 있다.

`ngrok`은 로컬에서 사용할 수도 있지만 가입해서 사용할 수도 있다. Travis CI에서 사용할 수 있도록 가입해서 쓰자.

- [https://ngrok.com/](https://ngrok.com/) 에 방문해 가입하고
- [get-started](https://dashboard.ngrok.com/get-started) 링크에서 미리 발급된 `authtoken`을 확인한다.

현재 [kyungw00k/viewport-position](https://github.com/kyungw00k/viewport-position)의 `.zuul.yml`에는 다음과 같이 적용되어 있다.

```
tunnel:
  type: ngrok
  authtoken: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  proto: tcp
```

> **Updated**

> Q. Outsider: 근데 중간에 API 키 지워야 하지 않아요? 해보진 않았지만 ㅎ

> A. `authtoken`은 별도로 `NGROK_AUTH_TOKEN` 환경 변수로 설정이 가능해서 `.zuul.yml` 에 포함시킬 필요가 없다고 합니다.

적용 후에 Concurrency를 SauceLabs VM에 맞춰서 5개로 했다가

- 다른 모듈에서도 쓰려고 개수를 1개로 줄이기로 했으나,
- 1개로 했다가 iOS 쪽 테스트 시에 Simulator 부팅이 오래 걸려서

다시 2개로 변경한 상태다.

이제 마지막으로 Travis CI 연동을 해보자.

### 3. Travis CI
관련해서 [Travis CI 측의 sauce-connect 문서](https://docs.travis-ci.com/user/sauce-connect/)에도 잘 설명되어 있지만

요약하면, 아래와 같이 실행하고

```sh
$ gem install travis
$ travis encrypt SAUCE_USERNAME=<sauce username> -r <travis-username>/<repo> --add
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
$ travis encrypt SAUCE_ACCESS_KEY=<sauce access key> -r <travis-username>/<repo> --add
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

위 출력 결과를 `.travis.yml`에 추가해주면 되겠다.

```
sudo: required
dist: trusty
language: node_js
node_js:
- '5.0'
env:
  global:
  - secure: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  - secure: yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

## 끝으로...

원래 이 글의 시작은 사내에서 `Javascript Code! 브라우저 별로 어떻게 테스트 하시나요?` 라는 주제에서

- 사내에 `나만 모르는 어썸`한게 있다면 거기에 무임 승차 해보고 싶...기도 했지만
- 실제로 다른 분들은 어떻게 작업하고 있는지 노하우를 (건방지게) 공유 받고 싶어서

시작했던 거였는데, 제품 소개 삘로 글을 써서 그런가 공유 받기가 쉽지 않았다.

여튼,

- Open Source 프로젝트에서 무료로 사용할 수 있게 해준 SauceLabs가 무척 고맙고
- 간단한 거지만 까먹지 않기 위해서

라도 단순 사용기를 남겨본다.

이 글을 보는 당신!

`Javascript Code! 브라우저 별로 어떻게 테스트 하시나요?` :dizzy:
