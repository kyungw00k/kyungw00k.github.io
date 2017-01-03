## 시작하기 전에

이 글은 [SauceLabs로 Cross Browser Test 하기](https://kyungw00k.github.io/2016/09/05/SauceLabs%EB%A1%9C-cross-browser-test-%ED%95%98%EA%B8%B0/)와 큰 틀에서 동일한 내용을 다루고 있고,

다른 점을 굳이 적자면

- `SauceLabs`에서 `BrowserStack`으로 바뀌었고
- 기존에 `zuul`을 사용했던걸 [karma](https://karma-runner.github.io/1.0/index.html)로 대체하고
- 렌더링 테스트로 [Nightwatch](http://nightwatchjs.org/)를 사용한

정도라고 할 수 있겠다.

여튼,

- Unit Test는 [karma](https://karma-runner.github.io/1.0/index.html)를 사용하고,
- Rendering Test는 [Nightwatch](http://nightwatchjs.org/)를 사용하고 있다.

여기서는 BrowserStack을 사용해 프로젝트에 `Karma`와 `Nightwatch` 적용하는 것으로 `Cross Browser Test 하기`로 한다.

간단히 환경 설정을 살펴보기 전에, 주의할 점이 있다면

> 최적의 환경이 아닐 수 있습니다.

네, 뭐... 그렇습니다.

### 0. BrowserStack 계정 생성
- 계정 생성: https://www.browserstack.com/users/sign_up 를 통해서 진행하면 되겠다.
- 사용 방법: [BrowserStack 사용기](https://kyungw00k.github.io/2017/01/03/browserstack-사용기/)를 참고하면 되겠다.

> 오픈소스는 뭐 없나요?

![browserstack-free-for-open-source](/images/2017/01/04/browserstack-free-for-open-source.png)

[일단 연락해보세요!](https://www.browserstack.com/contact?ref=open_source#sales)

### 1. Karma로 Unit Test 하기

일단 Karma 설정은 아래와 같이 3개로 두고,

```
base.karma.conf.js
local.karma.conf.js
browserstack.karma.conf.js
```

아래와 같이 grunt에서 `local`과 `ci` 환경을 구분했다.

```js

// Gruntfile.js

module.exports = function (grunt) {
  let currentVersion = require('./package.json').version

  grunt.initConfig({
  // ...
    karma: {
      local: {
        configFile: 'local.karma.conf.js'
      },
      ci: {
        configFile: 'browserstack.karma.conf.js'
      }
    },

  // ...
  })
  grunt.loadNpmTasks('grunt-karma')
}
```

Grunt를 사용하지 않는다면, `package.json`의 `scripts` 항목에 아래와 같이 넣고

```json
  "scripts": {
    "test:local:unit": "./node_modules/.bin/karma start local.karma.conf.js --singleRun true --debug",
    "test:ci:unit": "./node_modules/.bin/karma start browserstack.karma.conf.js --singleRun true --debug",
  },
```

다음과 같은 형태로 실행하면 되겠다.

```bash
npm run test:local:unit
```

일단 위 파일 3개를 대략 살펴보도록 하자.

#### 1.1. `base.karma.conf.js`
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
    'test/test.js'
  ],

  // frameworks to use
  frameworks: ['mocha'],

  preprocessors: {
    // only specify one entry point
    // and require all tests in there
    'src/**/*.js': ['webpack'],
    'test/test.js': ['webpack'],
    'test/**/*.html': ['html2js']
  },

  reporters: ['mocha'],

  coverageReporter: {
    type: 'html',
    dir: 'build/coverage/'
  },

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

#### 1.2. `local.karma.conf.js`
`local` 용이고, 브라우저를 `PhantomJS`과 `Chrome`을 쓰겠다는 정도의 설정이 있다.

```js
/* global require, module */

// local.karma.conf.js

const base = require('./base.karma.conf')

module.exports = function (config) {
  base.browsers = ['PhantomJS', 'Chrome']
  config.set(base);
};
```

#### 1.3. `browserstack.karma.conf.js`
- `base.browserStack` 부분이 `Automate` 화면에서 보이게 된다.
- UI 테스트에서도 동일한 Browser Set을 써야해서 별도로 `browserstack.launchers.conf.js`로 분리했다.

```js
/* global require, module, process */

// browserstack.karma.conf.js

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
  base.browserStack.build = `unit-test-${testDate}`
  base.browserStack.project = 'adfit-web-script'
  base.browserStack.timeout = 1800
  base.browserStack.captureTimeout = 1800
  base.browserStack.startTunnel = true

  base.singleRun = true

  config.set(base);
};
```

#### 1.4. `browserstack.launchers.conf.js`
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
  sl_ie_10: {
    "os": "Windows",
    "os_version": "7",
    "browser": "ie",
    "browser_version": "10.0"
  },
  sl_ie_9: {
    "os": "Windows",
    "os_version": "7",
    "browser": "ie",
    "browser_version": "9.0"
  },
  sl_ie_8: {
    "os": "Windows",
    "os_version": "7",
    "browser": "ie",
    "browser_version": "8.0"
  },
  sl_ie_7: {
    "os": "Windows",
    "os_version": "XP",
    "browser": "ie",
    "browser_version": "7.0"
  },
  sl_ie_6: {
    "os": "Windows",
    "os_version": "XP",
    "browser": "ie",
    "browser_version": "6.0"
  },

  /*
   * iOS
   */

  sl_ios_10: {
    device: 'iPhone 7',
    os: 'ios',
    os_version: '10.0'
  },
  sl_ios_9_3: {
    device: 'iPhone 6S',
    os: 'ios',
    os_version: '9.3'
  },
  sl_ios_8_3: {
    device: 'iPhone 6',
    os: 'ios',
    os_version: '8.3'
  },
  sl_ios_7: {
    device: 'iPhone 5S',
    os: 'ios',
    os_version: '7.0'
  },
  sl_ios_6: {
    device: 'iPhone 4S (6.0)',
    os: 'ios',
    os_version: '6.0'
  },
  sl_ios_5_1: {
    device: 'iPhone 4S',
    os: 'ios',
    os_version: '5.1'
  },

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
  sl_android_4_3: {
    "os": "android",
    "os_version": "4.3",
    "browser": "android",
    'platform': 'ANDROID',
    "device": "Samsung Galaxy Note 3"
  },
  sl_android_4_2: {
    "os": "android",
    "os_version": "4.2",
    "browser": "android",
    'platform': 'ANDROID',
    "device": "Google Nexus 4"
  },
  sl_android_4_1: {
    "os": "android",
    "os_version": "4.1",
    "browser": "android",
    'platform': 'ANDROID',
    "device": "Samsung Galaxy Note 2"
  },
  sl_android_4_0: {
    "os": "android",
    "os_version": "4.0",
    "browser": "android",
    'platform': 'ANDROID',
    "device": "Google Nexus"
  }
};
```

### 2. Nightwatch로 Rendering Test
Nightwatch를 사용하기 전에는 BackstopJS를 사용했다.
