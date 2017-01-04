---
layout: post
title: BrowserStack 으로 Cross Browser Test 하기
date: 2017-01-04 16:51:38.000000000 +09:00
type: post
published: true
status: publish
---

## 0. 시작하기 전에

이 글은 [SauceLabs로 Cross Browser Test 하기](https://kyungw00k.github.io/2016/09/05/SauceLabs%EB%A1%9C-cross-browser-test-%ED%95%98%EA%B8%B0/)와 큰 틀에서 동일한 내용을 다루고 있고,

### 0.1. 주의할 점이 있다면

> 최적의 환경이 아닐 수 있습니다.

(네, 뭐... 이건 당연하죠. 그렇습니다.)

### 0.2. 다른 점이 있다면

- `SauceLabs`에서 `BrowserStack`으로 바뀌었고
- 기존에 `zuul`을 사용했던걸 [karma](https://karma-runner.github.io/1.0/index.html)로 대체하고
- 렌더링 테스트로 [Nightwatch](http://nightwatchjs.org/)를 사용한

정도라고 할 수 있겠다.

여튼,

- Unit Test는 [karma](https://karma-runner.github.io/1.0/index.html)를
- Rendering Test는 [Nightwatch](http://nightwatchjs.org/)를

사용하기로 하고, Local에 없는 브라우저는 BrowserStack을 사용해 `Cross Browser Test 하기`로 한다.

### 어려워요. 그림으로 그려줘요.
이 글에서 다루고자 하는 부분을 그림으로 그려보면 다음과 같다.

![test-scenario](/images/2017/01/04/test-scenario.png)

새로운 기능이 추가되면 Unit Test도 하고 Rendering Test도 해야 한다. 이를 다음 두 환경에서 해보는 것을 목표로 한다.

- Local 에서도 하고 -> 내 PC에 설치된 브라우저 먼저 테스트 하고
- Remote 에서도 하고 -> 내 PC에 없는 브라우저는 (돈을 주고) 외주를 줘서 테스트 하고

그러면, 시작해보자.

### 1. BrowserStack 계정 생성
- 계정 생성: https://www.browserstack.com/users/sign_up 를 통해서 진행하면 되겠다.
- 사용 방법: [BrowserStack 사용기](https://kyungw00k.github.io/2017/01/03/browserstack-사용기/)를 참고하면 되겠다.

> 오픈소스는 뭐 없나요?

![browserstack-free-for-open-source](/images/2017/01/04/browserstack-free-for-open-source.png)

[일단 연락해보세요!](https://www.browserstack.com/contact?ref=open_source#sales)


### 2. Karma로 Unit Test 하기

예제는 [github/kyungw00k/karma-browserstack-example](https://github.com/kyungw00k/karma-browserstack-example) 에 올려두었다.

#### 2.0. 프로젝트 구조
일단 프로젝트 구조는 다음과 같다.

```
karma-browserstack-example
├── src/
├── test/
├── README.md                       # This file
├── base.karma.conf.js              # base karma configuration
├── local.karma.conf.js             # to run tests on local machine
├── browserstack.karma.conf.js      # to run tests on BrowserStack
├── browserstack.launchers.conf.js  # Browser lists to test
├── webpack.config.js
└── package.json
```

여기에 `src/helper/call-function-by-name.js` 라는 코드가 있고

```js
/**
 * Call by name with context
 *
 * @param functionName
 * @param context
 * @returns {*}
 * @private
 */
function callFunctionByName (functionName, context /*, args1, args2 ... */) {
  var args = Array.prototype.slice.call(arguments, 2)
  var namespaces = functionName.split('.')
  var func = namespaces.pop()
  for (var i = 0; i < namespaces.length; i++) {
    context = context[namespaces[i]]
  }

  var retVal = null
  try {
    if (context[func]) {
      retVal = context[func].apply(window, args)
    } else if (typeof functionName === 'function') {
      retVal = functionName.apply(null, args)
    } else {
      throw new EvalError(`Not found callback function "${functionName}"`)
    }
  } catch (e) {
    throw e
  }
  return retVal
}

if (typeof module === 'object' && module.exports) {
  module.exports = callFunctionByName
} else {
  exports.callFunctionByName = callFunctionByName
}
```

`test/spec/call-function-by-name.spec.js`라는 테스트 코드가 있다.

```js
/* global describe, it, require */

// test/spec/call-function-by-name.spec.js

var assert = require('assert')
var callFunctionByName = require('../../src/helper/call-function-by-name')

describe('callFunctionByName', function () {
  it('should be called', function () {
    this.hello = function () {
      return 'world'
    }

    assert.equal(callFunctionByName('hello', this), 'world')
  })

  it('should be thrown exception', function () {
    try {
      callFunctionByName('hello', window)
    } catch (e) {
      assert.notEqual(e, null)
    }
  })
})

```

위 코드들은 Karma가 실행되는 시점에 Webpack을 거쳐서 브라우저에서 실행할 수 있도록 변환이 된다.

보통 `karma.conf.js` 하나로 퉁 치는데, 여기서는 아래와 같이 3개로 나눠봤다.

```
base.karma.conf.js          # 공통 설정. 테스트할 브라우저 목록을 제외한 정도로 보면 됨
local.karma.conf.js         # Local용 설정이 추가
browserstack.karma.conf.js  # BrowserStack 설정이 추가
```

그리고 BrowserStack에서 실행할 브라우저가 많아지면 설정 파일이 커져서, 이를 따로 관리 하기 위해 `browserstack.launchers.conf.js` 파일을 만들어 두었다.

`package.json`의 `scripts` 항목에 아래와 같이 넣고

```json
  "scripts": {
    "test:local:unit": "./node_modules/.bin/karma start local.karma.conf.js --singleRun true --debug",
    "test:ci:unit": "./node_modules/.bin/karma start browserstack.karma.conf.js --singleRun true --debug",
  }
```

다음과 같은 형태로 실행하면 되겠다.

```bash
npm run test:local:unit
```

일단 위 파일 3개를 대략 살펴보도록 하자.

#### 2.1. `base.karma.conf.js`
- `test/` 경로에 test 파일들이 잔득(?) 있음
- webpack을 쓰고 있(지만 그건 중요하지 않)음
- 테스트 코드들이 `mocha` 기반으로 작성 되어 있음

딱히, 특별한게 없다.

```js
/* global require, module */

// base.karma.conf.js

const webpackOptions = require('./webpack.config.js')

webpackOptions.resolve = {
  modulesDirectories: [
    "",
    "src",
    "node_modules"
  ]
}

module.exports = {
  files: [
    'test/**/*.spec.js'
  ],

  // frameworks to use
  frameworks: ['mocha'],

  preprocessors: {
    // only specify one entry point
    // and require all tests in there
    'src/**/*.js': ['webpack'],
    'test/**/*.spec.js': ['webpack']
  },

  reporters: ['mocha'],

  webpack: webpackOptions,

  webpackMiddleware: {
    // webpack-dev-middleware configuration
    noInfo: true
  },

  proxies: {
    // ...
  },

  singleRun: true
}
```

#### 2.2. `local.karma.conf.js`
`local` 용이고, 브라우저를 `PhantomJS`, `Chrome`, `Safari`를 쓰겠다는 정도의 설정이 있다.

```js
/* global require, module */

// local.karma.conf.js

const base = require('./base.karma.conf')

module.exports = function (config) {
  base.browsers = ['PhantomJS', 'Chrome', 'Safari']
  config.set(base);
};
```

#### 2.3. `browserstack.karma.conf.js`
- `base.browserStack` 부분이 `Automate` 화면에서 보이게 된다.
- UI 테스트에서도 동일한 Browser Set을 써야해서 별도로 `browserstack.launchers.conf.js`로 분리했다.

```js
/* global require, module, process */

// browserstack.karma.conf.js

const package = require('./package.json')
const base = require('./base.karma.conf')
const customLaunchers = require('./browserstack.launchers.conf.js')
const testDate = (new Date()).toISOString()

Object.keys(customLaunchers).forEach(function (key) {
  customLaunchers[key].base = 'BrowserStack'
})

module.exports = function (config) {
  if (!process.env.BROWSER_STACK_USERNAME || !process.env.BROWSER_STACK_ACCESS_KEY) {
    console.log('Make sure the BROWSER_STACK_USERNAME and BROWSER_STACK_ACCESS_KEY environment variables are set.')
    process.exit(1)
  }

  base.reporters = ['mocha', 'BrowserStack']

  base.browsers = Object.keys(customLaunchers)

  base.customLaunchers = customLaunchers

  base.concurrency = 2
  //base.browserDisconnectTolerance = 10
  //base.browserDisconnectTimeout = 360000
  //base.browserNoActivityTimeout = 360000

  base.browserStack = {}
  base.browserStack.build = `unit-test-${testDate}` // BrowserStack Dashboard에서 Grouping 해주기 위해 설정
  base.browserStack.project = package.name          // BrowserStack Dashboard에서 Project로 Filtering 하기 위해 설정
  base.browserStack.timeout = 1800
  base.browserStack.captureTimeout = 1800
  base.browserStack.startTunnel = true

  base.singleRun = true

  config.set(base);
};
```

#### 2.4. `browserstack.launchers.conf.js`
- 서비스에서 대응하는 브라우저를 적어주면 된다.
- [링크의 'Setting your operating system, browser, and screen resolution'](https://www.browserstack.com/automate/node) 항목을 참고해 원하는 브라우저 목록을 만들면 된다.
- 이 파일은 뒤에 Rendering Test 시에도 사용할 거라 별도의 파일로 분리했다.

```js
/* global require, module */

// browserstack.launchers.conf.js

module.exports = {
  /*
   * Modern Browser
   */

  sl_chrome: {
    "os": "OS X",
    "os_version": "Lion",
    "browser": "chrome",
    "browser_version": "49.0"
  },
  sl_firefox: {
    "os": "OS X",
    "os_version": "Lion",
    "browser": "firefox",
    "browser_version": "43.0"
  },
  sl_safari: {
    "os": "OS X",
    'os_version': 'Yosemite',
    'browser': 'Safari',
    'browser_version': '8.0'
  },

  /*
   * Internet Explorer
   */

  sl_ie_11: {
    "os": "Windows",
    "os_version": "7",
    "browser": "ie",
    "browser_version": "11.0"
  },
  // ...

  /*
   * iOS
   */

  sl_ios_10: {
    device: 'iPhone 7',
    os: 'ios',
    os_version: '10.0'
  },
  // ...

  /*
   * Android Browser
   */

  sl_android_5_0: {
    "os": "android",
    "os_version": "5.0",
    "browser": "android",
    'platform': 'ANDROID',
    "device": "Google Nexus 5"
  },

  // ...
};
```

![karma-browserstack-example](/images/2017/01/04/karma-browserstack-example.png)

잘 실행되는 것을 볼 수 있다.

### 3. Nightwatch로 Rendering Test
Nightwatch를 사용하기 전에는 [BackstopJS](https://garris.github.io/BackstopJS/)를 사용했다.

BackstopJS는 CasperJS/PhantomJS 기반으로 렌더링 테스트를 할 수 있는 좋은 툴이다. 무엇보다 테스트를 작성하기 쉬워서 처음에는 이걸 사용해 로컬에서 테스트를 했었다.

로컬에서 IE와 모바일(Android/iOS)브라우저 테스트 까지 같이 하고 싶었지만, 결국 로컬에 브라우저를 전부 설치하기 전에는 불가능 하다는 결론에 이르렀다.

> 결국 기승전 Selenium인걸로...

예제는 [github/kyungw00k/nightwatch-browserstack-example](https://github.com/kyungw00k/nightwatch-browserstack-example) 에 올려두었다.

#### 3.0. 프로젝트 구조
일단 프로젝트 구조는 다음과 같다.

```
nightwatch-browserstack-example
├── src/
├── test/visual                      # Nightwatch 에서 사용하는 영역
│        ├── html/                   # Rendering에 사용할 Test View
│        ├── nightwatch/             # Nightwatchjs 기능 확장을 위해 사용할 Directory
│        ├── reports/                # Nightwatchjs가 생성하는 Test Report
│        ├── screenshots/            # 렌더링 결과를 검증하기 위해 사용할 Resource
│        │   ├── baseline            # ├── Expect
│        │   ├── diffs               # ├── Diff
│        │   └── results             # └── Actual
│        └── spec/                   # Test Case
│
├── README.md                        # This file
├── base.nightwatch.conf.js          # base nightwatch configuration
├── local.nightwatch.conf.js         # to run tests on local machine
├── browserstack.nightwatch.conf.js  # to run tests on BrowserStack
├── browserstack.launchers.conf.js   # Browser lists to test
├── webpack.config.js
└── package.json
```

Karma Unit Test와 마찬가지로, Nightwatch 쪽 설정도 Local/BrowserStack 구분하기 위해 설정 파일을 다음과 같이 Local/BrowserStack 용으로 나누었고,


```
base.nightwatch.conf.js          # 공통 설정. 테스트할 브라우저 목록을 제외한 정도로 보면 됨
local.nightwatch.conf.js         # Local용 설정이 추가
browserstack.nightwatch.conf.js  # BrowserStack 설정이 추가
```

Unit Test때와 동일하게 BrowserStack에서 실행할 브라우저를 따로 관리 하기 위해 `browserstack.launchers.conf.js` 파일을 만들어 두었다.

그리고 Nightwatch에서 테스트 시에 사용하는 영역을 `test/visual`로 정했다.

여기서 다루는 Rendering 테스트 다음과 같은 순서로 진행한다.

1. `localhost` port 50000~59999 범위 내에 렌덤으로 하나 택해 static web server를 띄운 후
1. `test/visual/spec/render.spec.js`가 `test/visual/html`에 있는 각 HTML 파일들에 대해 Test Case를 동적으로 생성하고, 각각의 테스트 케이스에서는
  1. 해당 HTML 파일을 URL(ex. `http://localhost:50000/test/visual/html/test_page_1.html`)로 하나 씩 불러서
  1. ScreenShot을 찍고
  1. 기존에 가지고 있는 이미지와 비교해서
  1. Mismatch 비율이 10% 미만이면 통과시킨다.

> Q. HTML에 해당하는 Test Case를 일일이 동적으로 생성한 이유는 뭔가요?

이건... 사실 BackStopJS에서 사용했던 Test Case를 버리기가 아까워서 재활용 했던 측면이 컸다.

어짜피 단순 렌더링 결과만 확인하면 되는 수준이라 모양이 이렇게 되었...

테스트 작성은 http://nightwatchjs.org/guide#writing-tests 링크를 참고하면 되겠다.

이제 설정 파일을 하나 하나 살펴보자.

#### 3.1. `base.nightwatch.conf.js`
Nightwatch 공통 설정은 이 곳에 쓰면 되겠다.

```js
/* global module */

module.exports = {
  "src_folders": ["./test/visual/spec"],
  "output_folder": "./test/visual/reports",
  "custom_commands_path": "./test/visual/nightwatch/commands",
  "custom_assertions_path": "./test/visual/nightwatch/assertions",
  "page_objects_path": "",
  "globals_path": ""
}
```

#### 3.2. `local.nightwatch.conf.js`
Local에서는 `Chrome` 브라우저로 테스트 할꺼라 `chromedriver`를 `./bin` 경로에 넣어두었다.

```js
/* global require, module */

const extend = require('util')._extend;
const base = require('./base.nightwatch.conf')

module.exports = extend(base, {
  "selenium": {
    "start_process": true,
    "server_path": "./bin/selenium-server-standalone-3.0.1.jar",
    "port": 4444,
    "cli_args": {
      "webdriver.chrome.driver": "./bin/chromedriver"
    }
  },

  "test_settings": {
    "default": {
      "selenium_port": 4444,
      "selenium_host": "localhost",
      "silent": true,
      "desiredCapabilities": {
        "browserName": "chrome"
      }
    }
  }
})
```

#### 3.3. `browserstack.nightwatch.conf.js`
기본 틀은 [browserstack/nightwatch-browserstack](https://github.com/browserstack/nightwatch-browserstack)에 있는 파일들을 그대로 가져와 사용했다.

여러 브라우저 Instance를 병렬로 수행해야 했기에 [browserstack/nightwatch-browserstack/blob/master/conf/parallel_local.conf.js](https://github.com/browserstack/nightwatch-browserstack/blob/master/conf/parallel_local.conf.js) 설정을 사용했다.

```js
/* global require, module */
const package = require('./package.json')
const extend = require('util')._extend;
const base = require('./base.nightwatch.conf')
const launchers = require('./browserstack.launchers.conf')
const testDate = (new Date()).toISOString()

const base_desired_capabilities = {
  'build': `rendering-test-${testDate}`,
  'project': package.name,
  'browserstack.user': process.env.BROWSER_STACK_USERNAME || 'BROWSERSTACK_USERNAME',
  'browserstack.key': process.env.BROWSER_STACK_ACCESS_KEY || 'BROWSERSTACK_ACCESS_KEY',
  'browserstack.debug': true,
  'browserstack.local': true,
  'resolution': '1280x1024'
}

const nightwatch_config = extend(base, {
  selenium: {
    "start_process": false,
    "host": "hub-cloud.browserstack.com",
    "port": 80
  },

  test_settings: {
    default: {}
  }
});

// 브라우저를 추가하는 코드
Object.keys(launchers).forEach(function (key) {
  nightwatch_config.test_settings[key] = {
    'desiredCapabilities': extend(launchers[key], {})
  }
});

// Code to copy seleniumhost/port into test settings
for (let i in nightwatch_config.test_settings) {
  let config = nightwatch_config.test_settings[i];
  config['selenium_host'] = nightwatch_config.selenium.host;
  config['selenium_port'] = nightwatch_config.selenium.port;
  config['desiredCapabilities'] = config['desiredCapabilities'] || {};

  for (let j in base_desired_capabilities) {
    config['desiredCapabilities'][j] = config['desiredCapabilities'][j] || base_desired_capabilities[j];
  }
}

module.exports = nightwatch_config;
```

#### 3.4. 스크린샷을 찍어서 비교하는 코드는 어디에?
이 부분은 [richard-flosi/8a5d2e10b6609ab9d06a](https://gist.github.com/richard-flosi/8a5d2e10b6609ab9d06a) 코드를 사용했다.

다만, 몇 가지 수정을 했는데,

- `resemblejs` -> `blink-diff`로 교체(이건 개취)
- 스샷과 DIFF 파일 이름을 렌덤하게 하도록 파일 이름에 `prefix`를 추가

파일 이름을 수정한 이유는 실제로 여러 브라우저에서 Screenshot을 찍는데, 파일 이름이 동일하면 시점에 따라 다른 브라우저에서 찍힌 파일로 DIFF를 할 가능성이 있기 때문이다.

(실제로도 그런 일이 일어나서 추가했던거죠)

이는 기존의 nightwatch에 있는 기능이라기 보다 확장한 부분으로 자세한 사항은 [가이드 문서](http://nightwatchjs.org/guide#extending)를 살펴보면 되겠다.

위 Gist에서 수정한 파일 최종본은 다음과 같다.

- [kyungw00k/nightwatch-browserstack-example/blob/master/test/visual/nightwatch/assertions/compareScreenshot.js](https://github.com/kyungw00k/nightwatch-browserstack-example/blob/master/test/visual/nightwatch/assertions/compareScreenshot.js)
- [kyungw00k/nightwatch-browserstack-example/blob/master/test/visual/nightwatch/commands/compareScreenshot.js](https://github.com/kyungw00k/nightwatch-browserstack-example/blob/master/test/visual/nightwatch/commands/compareScreenshot.js)

### 4. 후기
- 특정 DOM을 Image Capture 하고 싶은데 Selenium에서 이게 안되는 것 같다(확실하지 않음).
- 최대한 덜 억지스러운 방법을 찾아서 이렇게 저렇게 해봤지만, 여전히 간단해보이지 않음.

이 글을 보는 당신!

> 내가 작업한 게 훨 더 좋네. 흐흐`

그렇다면 한 수 부탁드립니다. 미리 감사드려요.
