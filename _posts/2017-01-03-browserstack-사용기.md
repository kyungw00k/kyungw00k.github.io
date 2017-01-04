---
layout: post
title: BrowserStack 사용기
date: 2017-01-03 16:53:38.000000000 +09:00
type: post
published: true
status: publish
---

> 이 글은 제품 사용기에 가깝습니다. 자동화 테스트는 [BrowserStack으로 Cross Browser Test 하기](/2017/01/04/browserstack으로-cross-browser-test-하기/)에서 상세히 다룰께요

## 시작하기 전에

작년 하반기에는 개인/회사 Front 쪽 코드에 SauceLabs를 메인으로 썼었다.

하지만, 개인 계정으로 회사에서 업무를 보는 건 `아무래도 좀 아니다` 싶었고, 무엇보다 회사에서 보다 맘 편하게 쓰고 싶은 마음에 구매 요청을 했었다.

그래서 구매 허락을 받고 나서 SauceLabs를 구매하려다,

> BrowserStack은 어떨까? 괜찮나?

궁금해서 한 달 정도 고민해볼 생각으로 BrowserStack으로 구매를 신청했다.

후기를 적으려고 기능 이곳저곳을 돌아다니다 보니 자연스럽게 SauceLabs와 비교를 하게 되었다.

> 속도는 빠른가요?

어짜피 국내에 있는 회사가 아니라 속도는 애초에 기대도 하지 않았던 터라, 어느정도 포기는 했었다.

> 가격은?

## 0. Pricing

![browserstack-pricing](/images/2017/01/03/browserstack-pricing.png)

요금표는 https://www.browserstack.com/pricing 에서 확인할 수 있다.

가격은 SauceLabs도 얼추 비슷한 것 같고, Parallel Test(SauceLabs에서는 Concurrent VM) 수가 높아질수록 (당연히) 비싸진다.

> 그래서 어떤 게 더 좋아? 네가 쓰는 거 그냥 쓸게

몇 가지 항목으로 나눠서 살펴보면,

- 자동화 테스트 설정 난이도(ex. `karma`, `nightwatchjs` 등등): 비슷
- 가격: 비슷
- 문서화: 비슷
- UI: (개취에 가깝지만 일단) BrowserStack

개인적으로는 SauceLabs 보다 BrowserStack이 더 맘에 들었다.

> 단순 UI 이슈일지도 모르지만 몇 가지 포인트는 있습니다. 흠. 흠.

일단 리뷰는 아래 네 가지 기능을 살펴보는 것으로 한다.

## 1. Features
- Live
- Automate
- Screenshots
- Responsive

### 1.1. Live

![browserstack-live-select-browser](/images/2017/01/03/browserstack-live-select-browser.png)

일단 Live는 BrowserStack의 기본 기능이고, OS 별로 사용할 수 있는 브라우저를 클릭하면 Instance가 생성된다.

![browserstack-browser-view](/images/2017/01/03/browserstack-browser-view.png)

사이트를 방문 한 모습이고, 우측 하단에 Toolbar가 있는데,

![browserstack-live-browser-switch](/images/2017/01/03/browserstack-live-browser-switch.png)

`SWITCH`를 누르면 위와 같이 OS/브라우저를 선택할 수 있는 화면이 뜨고, 선택하면 해당 OS/브라우저로 동일한 URL을 열게 된다.

![browserstack-live-features](/images/2017/01/03/browserstack-live-features.png)

`FEATURES`를 누르면 위와 같이 사용할 수 있는 기능에 대한 설명을 볼 수 있다.

![browserstack-live-general-settings](/images/2017/01/03/browserstack-live-general-settings.png)

`톱니 모양의 아이콘`을 누르면 위와 같이 일반적인 설정(`Session timeout`이나 Local 설정 등등)을 할 수 있고,

![browserstack-live-general-settings-keyboard-layout-no-korean](/images/2017/01/03/browserstack-live-general-settings-keyboard-layout-no-korean.png)

역시나 `한국어` 자판은 없다.

![browserstack-live-generate-screenshot](/images/2017/01/03/browserstack-live-generate-screenshot.png)

`카메라 아이콘`을 누르면 보고 있는 화면을 캡춰해주는데, 간단히 추가로 내용을 적을 수 있는 UI를 제공한다.

![browserstack-live-quicklaunch](/images/2017/01/03/browserstack-live-quicklaunch.png)

`Quick Launch`는 자주 쓰는 브라우저 즐겨찾기 하는 용도로 쓰면 될 것 같다.

![browserstack-live-resolution-switch](/images/2017/01/03/browserstack-live-resolution-switch.png)

마지막으로 `1242x877`과 같은 텍스트를 누르면 해상도를 변경할 수 있는데, 기본은 `Scale to fit`이다.

### 1.2. Automate
사실, Live만 하자고 구매하기엔 좀 아깝긴 하다. ~~Live는 어찌 보면 노동력으로 해결할 수 있...~~

실제 프로젝트 테스트 때는 `Automate`를 주로 쓰고, 테스트 결과가 남아서 로그를 잘 묶어서 남기면 보기도 편하다.

![browserstack-automate-dashboard](/images/2017/01/03/browserstack-automate-dashboard.png)

위 화면이 Dashboard고, Automate 메뉴의 첫 페이지다. 현재 테스트 중인 항목이 있다면, 새로고침 없이 실시간으로 진행상황을 볼 수 있다.

![browserstack-automate-dashboard-search-panel](/images/2017/01/03/browserstack-automate-dashboard-search-panel.png)

좀 더 Filtering 해서 보고 싶다면, 좌측 검색 Form을 잘 활용하면 될 것 같다.

![browserstack-automate-instance-text-log](/images/2017/01/03/browserstack-automate-instance-text-log.png)

각 Test 별로 진행 상황(Text, Video)이 보이고 하단에 `Text Log`와 `Visual Log`가 남게 된다(위 화면은 `Text Log`)

![browserstack-automate-instance-visual-log](/images/2017/01/03/browserstack-automate-instance-visual-log.png)

`Visual Log`는 Test Action 수행할 때마다 Capture를 한다는 점 정도 차이가 있다.

> 그래서 코드랑 연동은 어떻게 하는데?

전에 글을 썼든 [SauceLabs로 Cross Browser Test 하기](https://kyungw00k.github.io/2016/09/05/SauceLabs%EB%A1%9C-cross-browser-test-%ED%95%98%EA%B8%B0/) 내용과 마찬가지로 방법은 비슷하다.

> 보다 더 자세한 이야기는 [BrowserStack으로 Cross Browser Test 하기](/2017/01/04/browserstack으로-cross-browser-test-하기/)에서 다룰께요

### 1.3. Screenshots
![browserstack-screenshots](/images/2017/01/03/browserstack-screenshots.png)

위 브라우저들 중 15개를 선택하고 URL을 입력하고 Generate 하면 ScreenShot을 찍어준다.

![browserstack-screenshots-local-testing-options](/images/2017/01/03/browserstack-screenshots-local-testing-options.png)

URL이 Local이거나 Private Network에 있는 경우에는 `Local Testing`을 위해 `Local Server` 기능을 사용할 수 있고

![browserstack-screenshots-local-folder](/images/2017/01/03/browserstack-screenshots-local-folder.png)

그 밖에 `Local Folder` 옵션을 사용하면 Local 파일을 Remote에서 임의의 URL로 접근할 수 있다.

![browserstack-screenshots-results-share](/images/2017/01/03/browserstack-screenshots-results-share.png)

스크린샷 결과는 위와 같이 생성되고, URL로 다른 이들과 공유할 수 있다.

### 1.4. Responsive
![browserstack-responsive](/images/2017/01/03/browserstack-responsive.png)

테스트 하는 페이지가 반응형 페이지라면 이 기능이 유용할 수 있겠으나 문제는... 한글이 깨집니다.

> (╯°□°）╯︵ ┻━┻

![browserstack-responsive-local-testing-options](/images/2017/01/03/browserstack-responsive-local-testing-options.png)

마찬가지로, `Local Server`, `Local Folder` 옵션을 제공한다.

## 2. 특이 사항
- 위 네 가지 기능 모두 `Local Testing` 접근 링크가 있는 점

## 3. 이런 분들에게 BrowserStack을 추천드려요
- **이미 작성한 Unit Test가 있는 분**
- Unit Test 및 UI 자동화 테스트 뿐 아니라 실제 Live Test도 자주 사용 하려는 분
