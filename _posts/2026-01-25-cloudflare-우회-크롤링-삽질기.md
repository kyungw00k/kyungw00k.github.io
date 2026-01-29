---
layout: post
title: "Cloudflare 우회 크롤링 삽질기"
date: 2026-01-25 22:14:00 +0900
type: post
published: true
status: publish
categories:
  - Development
---

# Cloudflare 우회 크롤링 삽질기

웹사이트를 크롤링하려고 했다. `requests`로 간단하게 될 줄 알았는데, Cloudflare가 막고 있었다. 삽질의 시작.

---

## 1단계: requests로 시작

```python
import requests

resp = requests.get("https://example.com/api/posts")
print(resp.status_code)  # 403
```

403 Forbidden. User-Agent를 추가해봤다.

```python
headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ..."
}
resp = requests.get(url, headers=headers)
# 여전히 403
```

안 된다. Cloudflare의 JS Challenge가 걸려있다.

---

## 2단계: cloudscraper

[cloudscraper](https://github.com/VeNoMouS/cloudscraper)는 Cloudflare의 JS Challenge를 자동으로 풀어준다.

```python
import cloudscraper

scraper = cloudscraper.create_scraper()
resp = scraper.get("https://example.com/api/posts")
print(resp.status_code)  # 200
```

된다! 단순한 JS Challenge는 이걸로 충분하다.

**동작 원리**:
1. Cloudflare가 보내는 JS 코드를 파싱
2. 수학 연산이나 DOM 조작을 Python으로 에뮬레이션
3. 올바른 답을 계산해서 쿠키 획득

WordPress REST API 같은 곳은 이걸로 해결됐다.

---

## 3단계: 더 강력한 보호

일부 페이지는 cloudscraper로도 안 됐다. Cloudflare Turnstile이나 더 복잡한 JS 검증이 있는 경우.

이럴 땐 실제 브라우저가 필요하다.

---

## 4단계: SeleniumBase UC Mode

[SeleniumBase](https://github.com/seleniumbase/SeleniumBase)의 UC(Undetected Chrome) 모드를 사용했다.

```python
from seleniumbase import SB

with SB(uc=True, incognito=True) as sb:
    sb.uc_open_with_reconnect("https://example.com", 4)
    sb.sleep(3)

    # 페이지 로드 완료
    html = sb.get_page_source()
```

**uc=True**가 핵심이다. 일반 Selenium은 Cloudflare가 탐지한다.

### UC 모드가 하는 것들

1. **webdriver 속성 숨김**: `navigator.webdriver = false`
2. **Chrome DevTools Protocol 숨김**
3. **User-Agent 정상화**
4. **Canvas/WebGL 핑거프린팅 우회**
5. **자동화 플래그 제거**

---

## 5단계: 팝업과 광고 처리

사이트에 광고가 덕지덕지 붙어있었다. 클릭하면 새 탭이 열리고, 전면 광고가 뜨고...

```python
def _close_popup(sb):
    """광고 팝업 제거"""
    sb.execute_script("""
        // Google Vignette 광고 제거
        document.querySelectorAll('iframe[src*="google"]').forEach(el => el.remove());
        document.querySelectorAll('[id*="google_vignette"]').forEach(el => el.remove());

        // z-index 높은 오버레이 제거
        document.querySelectorAll('div[style*="position: fixed"]').forEach(el => {
            if (parseInt(el.style.zIndex) > 1000) el.remove();
        });
    """)

def _handle_new_tabs(sb):
    """광고로 열린 새 탭 닫기"""
    windows = sb.driver.window_handles
    if len(windows) > 1:
        main_window = windows[0]
        for w in windows[1:]:
            sb.driver.switch_to.window(w)
            sb.driver.close()
        sb.driver.switch_to.window(main_window)
```

---

## 6단계: 카운트다운 타이머 대기

"Please wait 10 seconds" 같은 타이머가 있는 사이트. 기다려야 한다.

```python
def _wait_for_timer(sb):
    """카운트다운 타이머 대기"""
    for _ in range(60):  # 최대 60초
        try:
            # 타이머 텍스트 찾기
            timer_text = sb.get_text("#timer")
            seconds = int(timer_text)
            if seconds <= 0:
                break
        except:
            pass
        sb.sleep(1)
```

---

## 7단계: 프록시 로테이션

같은 IP로 계속 요청하면 차단당한다. 프록시를 로테이션해야 한다.

```python
from seleniumbase import SB

proxies = [
    "proxy1.example.com:8080",
    "proxy2.example.com:8080",
    "socks5://proxy3.example.com:1080",
]

def resolve_with_proxy(url: str, proxy: str):
    with SB(uc=True, proxy=proxy, headless=True) as sb:
        sb.uc_open_with_reconnect(url, 4)
        return sb.get_page_source()
```

### 병렬 처리 + 프록시 분배

```python
from concurrent.futures import ThreadPoolExecutor

def process_urls(urls: list, proxies: list):
    results = []

    with ThreadPoolExecutor(max_workers=len(proxies)) as executor:
        futures = []
        for i, url in enumerate(urls):
            proxy = proxies[i % len(proxies)]  # 라운드 로빈
            future = executor.submit(resolve_with_proxy, url, proxy)
            futures.append(future)

        for future in futures:
            results.append(future.result())

    return results
```

---

## 8단계: 헤드리스 모드 감지 우회

헤드리스 모드가 감지되는 경우가 있다.

```python
# 감지됨
with SB(uc=True, headless=True) as sb:
    ...

# 덜 감지됨 - headless2 사용
with SB(uc=True, headless2=True) as sb:
    ...
```

SeleniumBase의 `headless2`는 `--headless=new` 플래그를 사용한다. Chrome 109+에서 지원하는 새로운 헤드리스 모드로, 일반 브라우저와 더 비슷하게 동작한다.

그래도 안 되면 Xvfb(가상 디스플레이)를 사용:

```bash
# Linux에서
apt-get install xvfb
xvfb-run python crawler.py
```

---

## 9단계: UC Driver 미리 다운로드

병렬 실행 시 여러 워커가 동시에 ChromeDriver를 다운로드하면 충돌이 난다.

```python
from seleniumbase import SB

# 메인 스레드에서 미리 다운로드
with SB(uc=True) as sb:
    pass  # 초기화만 하고 종료

# 이후 병렬 실행
with ThreadPoolExecutor(max_workers=4) as executor:
    ...
```

---

## 최종 구조

```python
import cloudscraper
from seleniumbase import SB

# 1단계: cloudscraper로 시도 (API 등 단순한 경우)
def collect_api_data(url):
    scraper = cloudscraper.create_scraper()
    return scraper.get(url).json()

# 2단계: SeleniumBase UC로 시도 (복잡한 JS 검증)
def resolve_protected_url(url, proxy=None, headless=False):
    sb_kwargs = {"uc": True, "incognito": True, "headless": headless}
    if proxy:
        sb_kwargs["proxy"] = proxy

    with SB(**sb_kwargs) as sb:
        sb.uc_open_with_reconnect(url, 4)
        sb.sleep(3)

        # 팝업 제거
        _close_popup(sb)

        # 타이머 대기
        _wait_for_timer(sb)

        # 버튼 클릭 등
        if sb.is_element_visible("#continue-btn"):
            sb.click("#continue-btn")

        return sb.get_current_url()
```

---

## 삽질 정리

| 문제 | 해결 |
|------|------|
| 403 Forbidden | cloudscraper |
| JS Challenge 복잡함 | SeleniumBase UC mode |
| 봇 탐지됨 | `uc=True`, `incognito=True` |
| 광고 팝업 | JS로 DOM 제거 |
| 새 탭 광고 | `window_handles`로 닫기 |
| 카운트다운 | 폴링으로 대기 |
| IP 차단 | 프록시 로테이션 |
| 헤드리스 감지 | `headless2` 또는 Xvfb |
| 병렬 시 충돌 | 미리 드라이버 다운로드 |

---

## 결론

Cloudflare 우회는 단계적으로 접근해야 한다:

1. **cloudscraper**: 단순한 JS Challenge
2. **SeleniumBase UC**: 복잡한 검증
3. **프록시 로테이션**: IP 차단 우회
4. **Xvfb**: 헤드리스 감지 우회

그리고 예의 바르게 크롤링하자. rate limit 지키고, robots.txt 확인하고.

> 크롤링은 고양이와 쥐 게임이다. 막으면 뚫고, 뚫으면 막고.
