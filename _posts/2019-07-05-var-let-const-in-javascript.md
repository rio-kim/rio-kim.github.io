---
layout: post
title: var, let, const in javascript
categories: Javascript
tags: javascript codingrule
---
vue 프로젝트 진행시 coding rule을 정하면서 정리한 글. __핵심은 var를 사용하지 말고 let과 const를 사용하자는 것이다.__

---
### var의 문제점
var의 사용은 개발자들에게 javascript 코드를 혼란스럽게 만드는 이유가 된다.
아래는 var 사용으로 나타나는 문제점과 let, const를 사용해야 하는 이유이다.

##### 1. var로 중복 변수를 만들어도 문제없이 동작한다.
~~~js
var token = "test token";
console.log(token)
var token = "other token"; // 위에서 선언한 test token 값 유실
console.log(token)
~~~
javascript의 var 변수는 hoisting이라는 행위를 통해 최상단에 선언이 된다. hoist는 끌어올리기라는 뜻이고 이 행위 덕분에 var 변수는 자바스크립트 엔진 구동시 맨 처음에 선언되는 형태가 된다. __하지만 선언과 동시에 할당된 값이 끌어올려지지 않아서 문제를 발생한다.__  
만약 이런 코드가 있다고 치자.
~~~js
console.log(deviceName)
var deviceName = "갤럭시S8";
console.log(deviceName)
~~~
위 코드를 엔진이 해석할 때는 아래와 같다.
~~~js
var deviceName; // 할당된 값은 버리고 선언만 끌어올림
console.log(deviceName)
deviceName = "갤럭시S8";
console.log(deviceName)
~~~
위의 코드는 실제로 결과가 어떻게 나올까.
```
undefined
갤럭시S8
```
var의 이 호이스팅 동작은 개발자가 코드의 동작을 예측하기 어렵게 만드는 이유가 된다. 이런 var의 호이스팅을 막기 위해서는 자바스크립트 scope 내에(최상단 또는 함수 내부) `use strict`을 선언하여 방지할 수 있다. `use strict`를 사용하면 호이스팅은 동작하지 않으며 선언되지 않은 변수를 사용한다는 오류를 발생시킨다. __let을 사용하면 `use strict` 선언 없이도 정상적으로 오류가 발생한다.__
~~~js
'use strict'
console.log(deviceName) // error 발생
var deviceName = "갤럭시S8";
console.log(deviceName)
~~~
##### 2. var 변수는 scope를 무시한다.
__정확히는 var 변수는 function-scoped를 가진다.__ 이는 다시 말해 function 내부에 var 변수를 가두지 않는다면 global scope를 오염시킬 수 있다는 소리이다.
~~~js
var token = "test token";

if (token !== null) {
  var token = "other token";
  console.log(token) // other token
}
console.log(token); // other token
~~~
다른 언어들과 달리 자바스크립트의 중괄호 블록 영역 내부(예시의 if)의 var 키워드는 외부 스코프와 고립되지 않는다. 이는 코드 량이 많아지고, 많은 사람들이 함께하는 대규모 프로젝트가 될 수록 소스의 잠재적인 위험도를 높인다.  
이를 해결하기 위해 아래와 같이 function 으로 고립시키는 방법이 있다.
~~~js
(function() {
  var token = "test token";
  console.log(token);
})();
console.log(token); // error 발생
~~~
그런데 여기서도 function에 고립된 변수가 var 없이 선언된다면 자바스크립트는 global 영역에서 변수를 찾아나간다는 것이 문제.
~~~js
(function() {
  token = "test token"; // var 선언 없이 처음 등장
  console.log(token);
})();
console.log(token); // test token
~~~
위 코드는 개발자의 의도와 상관없이 token 변수를 gloabl scope에 선언되었다.

### 그럼 어떻게?
##### 3. var 대신 let을 사용하면 된다.
__let과 const는 모두 block-scoped 변수로 외부 스코프에 영향을 주지 않는다.__ 같은 스코프 내에서는 중복된 변수 선언을 당연히 할 수 없고 global scope에 선언된 동일한 이름의 변수는 내부 스코프에 영향을 주지 않는다. (일반적으로 개발자들이 원하던 변수의 스코프 범위)
~~~js
let token = "test token";
let token = "other token"; // syntax error 발생. ide에서는 모두 compile error로 인식
~~~
~~~js
let token = "test token";
token = "other token"; // 재할당 가능
~~~
~~~js
let token = "test token";
if (utils.isEmpty(user)) {
  let token = "other token";
}
console.log(token); // test token, 전역 스코프에 선언된 변수 값 유지
~~~
##### 4. const는 값의 재 할당이 불가능하다.
한번 선언되면 값을 재할당 할 수 없는 const의 특성 때문에 상수 선언에 사용되고, 코드 흐름상 추후 값 변경이 없는 변수의 경우는 const로 선언하는 것을 권장한다. 또한 모르는 사이에 코드 진행상 값이 재할당 되는 것을 방지하기 위해 const를 선언하는 습관을 들이는 것을 자바스크립트 개발서에서 추천 하곤 한다.  
let은 값 할당 없이 선언만 하는 것이 가능하지만 const는 그렇지 않다는 것에 주의한다.
~~~js
const REST_URL = "http://localhost:5000/rest";
REST_URL = "other url"; // 값 재할당 시도시 error 발생

const IAM_SERVER_URL; // 값 할당 없이 선언만 하는 경우 error 발생
~~~
코드 내에서 const를 상수 용도로 쓰는 경우와 변수 용도로 쓰는 경우를 구분해야 가독성이 높아진다. 일반적으로 상수는 대문자와 언더스코어를 사용하여 작성하고 변수는 카멜케이스로 작성한다. 절대적인 규칙은 아니다.
~~~js
const RELAY_SERVER_URL = "http://relay-server-url" // 상수 선언

const result = otherService.method(); // 변수 용도로 선언
~~~
---
##### 3줄 요약.
1. 이제 var는 쓰지말자.
2. 대신에 let과 const를 쓰자.
3. 값의 재할당이 없을 것 같다고 여겨지면 const를 쓰는 버릇을 들이자.
