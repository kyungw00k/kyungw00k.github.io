---
layout: post
title: Managing Scope in JavaScript
date: 2010-09-17 12:08:41.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
- JavaScript
---

### Backgrounds
JavaScript에서의 Data 접근은 Data 형태에 따라 크게 4가지 종류가 있다.

* Literal values
* Variables
* Array items
* Object members

![wpid-f2-1-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-1-2010-09-17-13-08.png)

위의 도표는 종류별로 각각 200,000개를 접근하는 속도를 브라우저 별로 테스트한 자료다. Literal value와 Variable의 접근 속도가 대체적으로 유사하고, Array item과 Object member에 접근하는 속도가 상대적으로 늦다는것을 보여준다.


속도에 민감한 작업시에는 Array와 Object 형태 보다 Literal과 local variable을 사용하는게 성능 향상하는게 도움이 된다고 한다.

### Managing Scope

#### Scope Chains and Identifier Resolution

JavaScript에서의 function은 Object의 형태 구조를 따른다. Function object의 내부 property 중에 `[[Scope]]`가 있는데 이는 ECMA-262 Third Edition에 정의되어있다.


이 `[[Scope]]` 에는 function이 생성되는 시점과 관련된 여러 Object Reference 정보들을 가지고 있는데, 이를 Function Scope Chain이라 한다. 이 Chain에서 function이 접근할 수 있는 Data 범위를 결정한다.

Scope Chain에 있는 각각의 Object를 Variable Object라 하고, 이는 key-value 형태 자료구조를 갖는다. 함수가 생생될 시점에 Scope Chain 또한 생성되는데 이는 함수가 생성된 Scope를 기준으로 만들어진다.

```js
function add(num1, num2){
    var sum = num1 + num2;
    retur sum;
}
```

위의 예제에서 보면 add()가 생성될 때, add()의 Scope Chain이 형성되고 이는 Global Object를 가리키게 된다. 아래 그림을 보면 쉽게 이해할 수 있다.

![wpid-f2-2-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-2-2010-09-17-13-08.png)

**The Global Object**

보통 Global Context라고도 하는데, 이 Context 또한 Object이므로 Global Object라고도 한다. 웹브라우저에서 Global Object는 자기 자신을 가리키는 Property로, 보통은 window를 쓴다. Global Context에서 아래의 표현은 모두 같은 의미이다.

```js
foo = ‘bar’;
var foo = ‘bar’;
this.foo = ‘bar’;
window.foo = ‘bar’;
this.window.foo = ‘bar’;
```

함수 add()의 Scope Chain은 실제로 add()가 호출이 될때 사용하게 된다.

```js
var total = add(5, 10);
```

위와 같이 함수 add() 실행시 실행 환경을 결졍하는 Context 안에서 실행하게 되는데 이를 Execution Context라고 한다. Execution Context는 함수 실행 시점에 생성되고, 함수가 종료되는 시점에 사라진다.


생성 단위는 하나의 함수 실행이고, 여러개의 함수가 실행되면 그때마다 각 함수에 해당하는 Context가 생성된다.

![wpid-f2-3-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-3-2010-09-17-13-08.png)

add() 함수를 실행하는 시점에서 add() 함수 내부에서 num1, num2가 각각 5, 10으로 연결이 되는건, 이 값들이 Scope Chain의 첫번째인 Activation Object에 존재하기 떄문에 가능하다.

만약 add() 함수 내부에서 document를 썼다면, Activation Scope내에 document가 존재하지 않기 때문에 그 다음 Chain으로 이동해 document를 찾게 된다.(Scope Chain에서 찾지 못하게 되면 undefined가 되겠다)

이렇게 Chain을 따라 특정 key 값으로 value를 찾는 작업을 Identifier Resolution이라고 한다.

**The Execution Context**

모든 JavaScript 코드는 Execution Context안에서 실행된다. 이 Context는 보통 JavaScript에서 "scope"란 의미로 종종 사용된다. 보통 함수를 호출함으로서 Execution Context가 형성되는데, 함수 안이 아닌 밖에서 실행되는 전역함수와 같은 경우는 scope가 아닌 global execution context란 의미의 "Program"이라는 단어를 사용한다고 한다.


```js
function add(num1, num2){
  var sum = num1 + num2;
  return sum;
}

add(5,3); // Global Context

function hello() {
  add(5,3); // "hello" Context
}
```

#### Identifier Resolution Performance
아래 두 도표는 Scope Chain이 깊어질수록 key를 탐색하는 속도를 보여주는 그래프이다.(Depth 1은 Local variable이다.)

![wpid-f2-4-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-4-2010-09-17-13-08.png)

![wpid-f2-5-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-5-2010-09-17-13-08.png)

Global Variable은 항상 Scope Chain의 마지막에 있기 때문에, Global Variable을 사용하는 코드들을 먼저 Local Variable 영역으로 옮겨오는게 우선으로 해야할 작업이라고 한다.
(예, document.getElementById 사용시 document 대신, document를 local variable로 담은 등의 작업)

```js
// (수정 전)
var bd = document.body,
links = document.getElementsByTagName(“a”),
length = links.length;

// (수정 후)
var doc = document,
  bd = doc.body,
  links = doc.getElementsByTagName(“a”),
  length = links.length;
```

### Scope Chain Augmentation

*with 사용시*

```js
function initUI(){
  with (document){        //avoid!
    var bd = body,
    links = getElementsByTagName(“a”),
    i= 0, len = links.length;

    while(i &lt; len){
      update(links[i++]);
    }

    getElementById(“go-btn”).onclick = function(){
      start();
    };
    bd.className = “active”;
  }
}
```

위의 예제에서 initUI가 실행되면 with 실행 시점에, 인자로 document가 넘어오게 된다.

이때 Scope Chain은 local variable인 bd, links 등등 보다 document의 Scope를 먼저 보게 되므로, with 구문 내의 코드 실행시 물론 편리한점은 있지만 Scope Chain을 탐색하는데 있어 항상 document의 Property들을 먼저 본다는 점에서 성능적인 이슈가 있다. 하여 위의 예제와 같은 경우에서는 with을 쓰기 보다는 local variable을 활용하는게 좋겠다.

![wpid-f2-6-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-6-2010-09-17-13-08.png)

*try-catch 사용시*

```js
try {
  methodThatMightCauseAnError();
} catch (ex) {
  alert(ex.message); //scope chain is augmented here
}
```

위의 예제의 경우, methodThatMightCauseAnError()에서 뭔가 문제가 발생하면 Exeception Object가 Scope chain의 첫번째로 push가 된다.

그리고 catch 블럭의 처리가 끝나면 cope chain은 이전 상태로 되돌아 온다.

만약 위의 경우, catch 블럭에서 local variable을 접근하려 한다면 먼저 Exeception Object를 찾게 되기 때문에 비효율적인 코드가 된다. 이를 아래와 같이 바꿔보자.

```js
try {
  methodThatMightCauseAnError();
} catch (ex) {
  handleError(ex) //delegate to handler method
}
```

위의 코드는 해당 Exception Object를 다른 함수에서 처리하게 하는 코드다.

이럴 경우, catch 블럭 안에서 오류를 처리하기 위해 추가적인 Chain 탐색이 이루어지지 않고 handleError()의 Scope에서 처리가 되기 때문에 첫번째에 비해 효율적이라 할 수 있다.

### Dynamic Scopes

Dynamic Scope는 단순 코드 정적 분석으로는 그 결과를 예측하기 힘든 경우에 해당한다. 아래의 코드를 보자.

```js
function execute(code) {
  eval(code);

  function subroutine(){
    return window;
  }

  var w = subroutine(); //what value is w?
}

// 위의 코드가 있다고 할 때, 다음과 같이 execute()를 실행한다고 하자.

execute("var window = {};")
```

이 경우, eval()은 인자로 넘어오는 `var window = {};`를 가지고 Local Variable을 생성한다.

Scope Chain의 첫번째인 Activation Object에 window = {}가 위치하게 되므로, subroutine()이 리턴하는 window는 빈 Object가 된다.

다시말해 execute내에 window는 인자로 주어지는 코드에 따라 결정이 된다.

이러한 Dynamic Scope의 사용은 자칫 의도하지 않는 결과를 낳을 수 있으므로, 사용할때 주의해야한다.

### Closures, Scope, and Memory

Closure와 관련하여 아래의 코드를 살펴보자.

```js
function assignEvents(){
  var id = "xdi9592";
  document.getElementById("save-btn").onclick = function(event) {
    saveDocument(id);
  };
}
```

assignEvents()는 하나의 DOM Element에 이벤트 핸들러를 할당하는 함수다.

이때 이 이벤트 헨들러가 Closure가 되는데, 이는 assignEvent() 함수가 실행되는 시점에 만들어지고, assignEvents()가 종료가 되어도 그 당시의 Chain Scope를 가지고 있다.

일반적으로 함수가 실행되는 시점에 Activation Object가 생성되고, 실행이 종료되면 Activation Object가 메모리에서 해제가 되는데, Closure를 포함한 경우는 그렇지 않다.

그 이유는 Closure의 Scope Chain이 함수의 실행 당시 Activation Object를 참조하고 있기 때문이다.

이는 Closure가 Closure가 아닌 일반적인 함수에 비해 Memory Overhead가 크다는 것을 의미한다.

![wpid-f2-7-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-7-2010-09-17-13-08.png)

Closure가 실행되면 Closure 역시 함수이므로, 자기 자신의 Activation Object가 생기게 된다.


(Figure 2-8 참조)

![wpid-f2-8-2010-09-17-13-08.png](/images/2010/09/17/wpid-f2-8-2010-09-17-13-08.png)

### References
* [http://tore.darell.no/pages/scope_in_javascript](http://tore.darell.no/pages/scope_in_javascript)
* High Performance JavaScript(Chapter 2)
