원문: [함수형 자바스크립트](https://www.omin.dev/posts/functional-programming-1)

## 서론

이 글에서는 함수형 프로그래밍에 대해 개괄적으로 소개하고, 함수형 프로그래밍의 중요한 개념인 부수 효과, 순수 함수, 참조 투명성, 불변성, 선언적 프로그래밍 등의 개념을 알아본다. 루이스 아텐시오의 [`함수형 자바스크립트`](https://product.kyobobook.co.kr/detail/S000001810017) 책 1장을 읽고 요약 정리 및 개인적인 생각을 추가하여 작성했다.

## 함수형 프로그래밍이란?

먼저 책에서 정의한 함수형 프로그래밍은 다음과 같다.

> 외부에서 관찰 가능한 부수효과가 제거된 불변 프로그램을 작성하기 위해 순수함수를 선언적으로 평가하는 것.

함수형 프로그래밍에 대해 깊게 이해하고 있는 저자는 위 표현을 간명하다고 하지만 이 문장만 봤을 때는 명확히 어떤 뜻인지 감이 오지 않는다. 지금으로서는 `부수효과`, `불변 프로그램`, `순수함수`, `선언적` 같은 키워드가 눈에 띄는데, 이 키워드들을 이해하면 조금이나마 함수형 프로그래밍에 대한 감을 잡을 수 있을 것 같다.

다행히도 함수 컴포넌트 기반 모던 React로 웹 어플리케이션을 개발하는 개발자라면 `불변 프로그램`을 제외하고는 위 키워드가 이미 익숙한 개념일 것이다. 지금부터 우리가 익숙한 개념들을 활용해 함수형 프로그래밍이 무엇인지 감을 잡아보자. 글 말미에는 함수형 프로그래밍에 대한 본인만의 간명한 정의를 내릴 수 있게 해보자.

### 해결하는 문제

구체적으로 함수형 프로그래밍의 개념을 살펴보기 전에 함수형 프로그래밍 패러다임으로 해결하고자 하는 문제에 대해 알아보자.

함수형 프로그래밍 뿐만 아니라 객체지향 프로그래밍 등 대부분의 프로그래밍 패러다임은 '복잡한 어플리케이션을 어떻게 하면 지속적으로 확장하고 유지보수 할 수 있도록 설계할까' 하는 고민의 결과물이다. 그렇다면 복잡한 어플리케이션을 만들 때 어떤 것을 고려해야 할까? 저자는 다음 5가지 요소를 고려해야 한다고 주장한다.

1. 확장성

   - 추가 기능을 개발하기 위해 계속 코드를 리팩토링 해야 하는가?

2. 모듈화 용이성

   - 파일 하나를 고치면 다른 파일도 영향을 받는가?

3. 재사용성

   - 중복이 많은가?

4. 테스트성

   - 함수를 단위 테스트하기 쉬운가?

5. 헤아리기 쉬움

   - 체계가 없고 따라가기 어려운 코드인가?

함수형 프로그래밍을 적용하면 위 문제들을 해결할 수 있다.

### 해결 방법

위 문제가 함수형 프로그래밍이 해결하는 문제라면 함수형 프로그래밍은 이 문제를 어떤 방법으로 해결할까? 함수형 프로그래밍에서는 `어플리케이션의 부수 효과를 방지하고 상태 변이를 감소하기 위해 데이터의 제어 흐름과 연산을 추상`함으로써 문제를 해결할 수 있다. 아래 예제와 앞으로 살펴볼 기본 개념들을 살펴보면 이 문장이 더 잘 이해될 것이다.

아래 예제를 살펴보자. 아래 코드는 `msg`라는 id를 가진 엘리먼트의 innerHtml을 변경하는 코드다.

```javascript
document.querySelector('#msg').innerHtml = '<h1>Hello world</h1>';
```

의도한 바는 명확히 수행하겠지만, 전혀 재사용할 수 없는 코드다. 만약 tag가 h1이 아니라 h2로 변경되어야 한다면? Hello world가 아닌 Hi there을 출력하고 싶다면? 위 경우 모두 코드 수정이 불가피하다.

보통 위 코드를 재사용 가능한 코드로 변경하기 위해 아래와 같이 수정할 수 있겠다.

```javascript
function printMessage(elementId, format, message) {
  document.querySelector(
    `#${elementId}`
  ).innerHtml = `<${format}>${message}</${format}>`;
}
```

재사용성이 많이 개선되었으나, 만약 DOM이 아닌 파일이나 다른 곳에 출력해야 한다면 어떨까? 이 경우 함수를 다시 수정해야 한다.

함수형 프로그래밍에서는 하나의 작업 단위를 하나의 함수로 분리하고, 함수 자체를 매개변수화 하는 작업을 통해 이 문제를 해결한다. 아래 코드는 동일한 문제를 함수형 프로그래밍으로 해결한 코드다. h1과 echo는 변수가 아닌 함수라는 점을 유의하자.

```javascript
const printMessage = run(addToDOM('msg'), h1, echo);
printMessage('Hello world');
```

run 함수는 addToDOM, h1, echo 함수를 체인처럼 연결하여 한 함수의 반환값이 다른 함수의 입력값으로 전환되게끔 한다. 이렇게 하면 함수 자체를 조합하기 쉬워지고, 각 함수의 재사용성이 높아진다.

만약 DOM이 아닌 console에 출력하고 싶다면 아래와 같이 수정하면 된다.

```javascript
const printMessage = run(console.log, h1, echo);
printMessage('Hello world');
```

마치 작은 함수들을 재료로 새로운 함수를 만든 것 처럼 보이는 위 예시는 프로그램을 더 작은 조각으로 나누고 더 헤아리기 쉬운 형태의 프로그램으로 조합하는 과정이며, 이것이 함수형 프로그래밍의 기본 원리다.

보통 우리가 작성하는 JS 코드와는 결이 매우 다르다는 것을 눈치챘을 것이다. 이는 함수형 프로그래밍 특유의 선언적 프로그래밍 방식 때문이다. 이제 선언적 프로그래밍을 포함해 순수함수, 부수효과, 불변성 등의 개념을 살펴보자.

## 선언적 프로그래밍

선언적 프로그래밍은 '어떻게' 보다 '무엇'을 표현하는 프로그래밍 방식이며, 함수형 프로그래밍은 큰 틀에서 선언적 프로그래밍 패러다임에 속한다.

선언적 프로그래밍과 반대되는 개념은 명령형 또는 절차형 프로그래밍이다. 명령형 프로그래밍은 컴퓨터가 수행해야 할 일을 하나하나 명령하는 방식이다. 배열의 원소를 모두 제곱수로 바꾸는 예제를 살펴보자.

```javascript
const numbers = [1, 2, 3, 4, 5];

for (let i = 0; i < numbers.length; i++) {
  numbers[i] = Math.pow(numbers[i], 2);
}

console.log(numbers); // [1, 4, 9, 16, 25]
```

각 연산이 어떻게 수행되어야 하는지 명시적으로 나열되어 있다. 반면 아래는 같은 일을 수행하는 선언적인 코드다.

```javascript
const numbers = [1, 2, 3, 4, 5];
const squaredNumbers = numbers.map(n => Math.pow(n, 2));

console.log(squaredNumbers); // [1, 4, 9, 16, 25]
```

제어 흐름이나 상태변화를 특정하지 않고 무엇을 해야하는지 표현식으로 나타낸다.

우리가 일상적으로 사용하는 React도 선언적으로 UI를 표현하는 라이브러리다. 모든 상태 변화는 React가 관리하고, 우리는 특정 상태에 따라 어떤 UI를 표현할지 선언한다. 아래는 단순 마크업을 선언적으로 표현한 예시다.

```jsx
<div>
  <div>Hello world</div>
</div>
```

React를 사용하지 않고 명령형으로 작성하면 다음과 같다.

```javascript
const div = document.createElement('div');
const innerDiv = document.createElement('div');
innerDiv.innerHtml = 'Hello world';
div.appendChild(innerDiv);
document.body.appendChild(div);
```

물론 예제와 같이 간단한 html 요소만 추가하는 상황이라면 JS가 필요하지 않지만, 복잡한 사용자 이벤트에 적절히 반응하는 웹 어플리케이션을 구축해야 한다고 상상해보자. 상태 변화에 따라 다른 UI를 표현해야 하거나, 각 요소에 이벤트 핸들러를 추가해야 한다면 명령형 코드는 더욱 복잡해질 것이다.

명령형 프로그래밍에서는 코드의 양이 증가하고 복잡해질수록 한 눈에 어떤 일을 하는지 파악하기 어려워진다. 반면 선언적 프로그래밍에서는 코드 자체가 어떤 일을 하는지 표현하기 쉽다.

React 공식 문서에서 [선언적 프로그래밍과 명령형 프로그래밍의 차이를 설명한 글](https://react.dev/learn/reacting-to-input-with-state#how-declarative-ui-compares-to-imperative)도 함께 살펴보면 좋다.

## 순수함수와 부수효과

함수형 프로그래밍은 순수함수로 구성된 불변 프로그램 구축을 전제로 한다. 불변 프로그램은 이후에 살펴보고, 우선 순수함수와 부수효과의 정의부터 해보자.

### 순수함수

순수함수란 주어진 입력에만 의존하고, 외부 상태를 읽거나 쓰지 않으며, 같은 입력에 대해 항상 같은 결과를 반환하는 함수다.

#### 참조 투명성

참조 투명성은 순수함수를 정의하는 더 공식적인 방법이며, 순수성(purity)이란 함수 인수와 결과 사이의 순수한 맵핑 관계를 의미한다. 함수가 동일한 입력을 받았을 때 항상 동일한 결과를 반환하면 참조 투명한 함수라고 한다.

아래 코드는 주어진 입력 외에 어떤 외부 변수도 참조하지 않고, 외부 상태를 변경하지 않으며, 같은 입력에 대해 항상 같은 결과를 반환하기 때문에 참조 투명한 순수함수다.

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(1, 2)); // 3
console.log(add(1, 2)); // 3
```

### 부수효과

부수효과는 함수가 외부 변수를 읽거나 쓰면서 발생하는 예기치 못한 결과를 말한다. 부수효과가 있는 함수는 같은 입력에 대해 항상 같은 결과를 반환하지 않을 수 있다. 부수효과가 있는 함수는 참조 투명하지 않으며, 순수함수가 아니다.

대표적으로 Date.now() 함수는 부수효과가 있는 함수다.

```javascript
function logNow() {
  console.log(Date.now());
}

logNow(); // 1711862683285
logNow(); // 1711862683895
```

아래 함수도 외부 변수를 읽고 쓰기 때문에 부수효과가 있는 함수다.

```javascript
let counter = 0;

function increment() {
  return ++counter; // 외부 변수에 쓴다.
}
```

이 외에도 웹 개발에서 부수효과가 발생하는 상황은 다음과 같다.

1. 전역 범위에서 변수, 속성, 자료구조 변경
2. 함수의 원래 인수 값을 변경
3. 사용자 입력을 처리
4. 예외를 일으킨 해당 함수가 catch하지 않고 그대로 throw
5. 화면에 출력 또는 파일에 쓰기
6. HTML 문서, 브라우저 쿠기, DB에 질의

### 순수함수의 이점

함수형 프로그래밍은 순수함수로 구성된 불변 프로그램 구축을 전제로 한다고 했다. 그렇다면 순수함수에는 어떤 이점이 있을까? 순수함수는 재작성하거나 치환하기 쉽다. 동일한 입력에 동일한 값을 반환하기 때문에 이를 값으로 치환할 수 있고, 그 덕분에 코드를 예측 가능하게 한다.

아래는 순수함수로 구성된 예제다.

```javascript
const total = arr => arr.reduce((acc, cur) => acc + cur, 0);
const size = arr => arr.length;
const devide = (a, b) => a / b;
const average = arr => devide(total(arr), size(arr));

console.log(average([1, 2, 3, 4, 5])); // 3
```

위 예제에서 average는 다음과 같이 치환할 수 있다.

```javascript
average = devide(15, 5); // 3
average = 15 / 5; // 3
```

이와 같이 함수를 치환할 수 있다면 시스템의 상태를 머릿속으로 쉽게 그려볼 수 있기 때문에 코드를 이해하기 쉽다. 주어진 입력을 처리해서 결과를 내는 일련의 함수로 프로그램을 작성한다면 아래와 같이 나타낼 수 있다.

`Program = Input -> Func1 -> Func2 -> Func3 -> Output`

Func1, Func2, Func3이 순수함수라면 주어진 Input에 대해 항상 동일한 결과를 반환하므로 Output도 항상 동일하다.

### React에서의 순수함수

이미 우리는 React에서 순수함수라는 개념을 적극 활용하고 있다. 모든 React 컴포넌트는 렌더링 과정에서 동일한 입력에 대해 동일한 결과를 도출하는 [순수함수여야 한다](https://react.dev/learn/keeping-components-pure).

아래와 같이 컴포넌트가 순수함수가 아니라면 UI가 예측 불가능해질 것이다.

```jsx
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

위 예제를 순수함수로 변경하면 다음과 같다.

```jsx
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

React [StrictMode](https://react.dev/learn/keeping-components-pure#detecting-impure-calculations-with-strict-mode)는 순수함수가 아닌 컴포넌트를 찾아내도록 도움을 준다.

물론 React에서도 불가피하게 부수효과가 필요할 수 있다. 사용자의 입력을 처리하거나, 외부 데이터 소스와 연결하는 등은 UI 컴포넌트 관점에서는 부수효과인데, 이런 부수효과는 [event handler와 useEffect를 통해 처리](https://react.dev/learn/keeping-components-pure#where-you-_can_-cause-side-effects)한다.

useEffect hook의 이름이 use 'Effect'인 것은 우연이 아니다.

## 불변성과 상태 변이

마지막으로 불변성과 상태 변이의 개념에 대해 살펴보자.

JS에서 모든 원시 자료형은 불변하다. 즉, 생성된 이후 변하지 않는다. 하지만 배열 등의 객체는 그렇지 않다. 따라서 함수 인수로 객체의 참조를 전달했을 때 함수 내부에서 해당 객체의 상태를 변화시킨다면(상태 변이) 함수 외부 변수에 변경을 가하는 것이기 때문에 이는 부수효과다.

함수형 프로그래밍에서는 객체의 불변성을 지킴으로써 의도하지 않은 상태변이를 감소하여 프로그램을 예측 가능하게 만드는 것을 지향한다.

아래 예제는 배열의 원소를 정렬하는 함수이다. 얼핏 보기에 문제가 없어 보이지만 [Array.prototype.sort()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) 메소드는 원본 배열을 변경하기 때문에 부수효과가 발생하며, 불변성이 지켜지지 않는다.

```javascript
function sortDesc(arr) {
  return arr.sort((a, b) => b - a);
}

const numbers = [3, 1, 2, 5, 4];
console.log(sortDesc(numbers)); // [5, 4, 3, 2, 1]
console.log(numbers); // [5, 4, 3, 2, 1] - 원본 배열이 변경됨
```

불변성을 지키기 위해 간단한 방법으로는 다음과 같이 코드를 수정할 수 있다.

```javascript
function sortDesc(arr) {
  return [...arr].sort((a, b) => b - a);
}
```

## 함수형 프로그래밍의 장점

함수형 프로그래밍을 활용하면 다음과 같은 장점이 있다.

1. 간단한 함수들로 작업을 분해한다.
2. 흐름 체인으로 데이터를 처리한다.
3. 리액티브 패러다임을 실현하여 이벤트 중심 코드의 복잡성을 줄인다.

하나씩 살펴보자.

### 분해와 합성

함수형 프로그래밍은 작은 함수들로 작업을 분해하고, 이 함수들을 합성하는 과정의 연속이라고 볼 수 있다. 따라서 모듈적이고 효율적으로 동작한다. 함수형 사고는 대체로 어떤 작업을 논리적 하위 작업으로 분할하는 것부터 시작된다.

분해와 합성을 적절히 수행하면 함수가 하나의 역할만을 수행하는 것 뿐만 아니라 함수 내 동일한 추상화 수준을 유지할 수 있어, 코드를 이해하기 쉬워진다.

```javascript
// addToDOM, h1, echo 함수를 합성하여 printMessage 함수를 만들었다.
const printMessage = compose(addToDOM('msg'), h1, echo);
printMessage('Hello world');
```

React에서 컴포넌트를 적절히 분리하고, 분리한 컴포넌트를 조합하여 UI를 구성하는 것을 떠올리면 이해하기 쉽다. React 커뮤니티의 컴포넌트 분리, 컴포넌트 합성이라는 기법은 이러한 함수형 사고를 반영한 것이다.

### 체이닝

체인은 같은 객체를 반환하는 순차적인 함수 호출이다. 명령형 프로그래밍에서 순차적으로 연산을 표현했다면, 함수형 프로그래밍에서는 각 작업 단위를 체이닝하여 표현한다.

아래는 lodash를 이용한 체이닝 예제다.

```javascript
_.chain([1, 2, 3, 4, 5])
  .map(n => n * 2)
  .filter(n => n > 5)
  .value(); // [6, 8, 10]
```

덕분에 반복문이나 if문을 사용하지 않고도 어떤 작업이 수행되어야 하는지 선언적으로 표현할 수 있다.

### 리액티브 프로그래밍의 실현

최근에는 `async`와 `await`의 활용으로 어플리케이션 코드베이스에서 수 단계의 중첩이 생기는 callback 지옥을 찾아보기란 어렵지만, 단순히 callback 지옥이 아니라도, 함수형 프로그래밍에 기반한 리액티브 프로그래밍을 활용하면 사용자 이벤트, 데이터 저장소 연결 등 비동기 작업을 효율적이고 간결하게 처리할 수 있다.

아래는 학생의 SSN 번호를 통해 학생 정보를 가져오는 예제다. 명령형 프로그래밍과 함수형에 기반한 리액티브 프로그래밍의 차이를 중점적으로 보자.

```javascript
let valid = false;
const elem = document.querySelector('#student-ssn');
elem.addEventListener('keyup', e => {
  const value = e.target.value;
  const ssn = value.replace(/\D/g, '');
  // some validation
  if (valid) {
    // some async operation
  } else {
    console.error('Invalid input');
  }
});
```

```javascript
Rx.Observable.fromEvent(document.querySelector("#student-ssn"), "keyup")
  .pluck("srcElement", "value")
  .map((ssn) => ssn.replace(/\D/g, ""))
  .filter((value) => /* some validation */ )
  .subscribe((value) => {
    // some async operation
  });
```

위와 같이 함수형에 기반한 리액티브 프로그래밍을 활용하면 비동기 작업을 더 깔끔하게 처리할 수 있다.

## 정리

글 초반에 함수형 프로그래밍에 대해 더 명확한 정의를 내릴 수 있게 해보자고 했다. 본인은 함수형 프로그래밍을 다음과 같이 정의할 수 있을 것 같다.

> 순수함수를 선언적으로 작성해 부수효과와 상태 변이를 줄이고, 프로그램을 예측 가능하게 만드는 프로그래밍 패러다임

## 참고자료

**도서**  
[함수형 자바스크립트](https://product.kyobobook.co.kr/detail/S000001810017)

**공식 문서**  
[Keeping Components Pure](https://react.dev/learn/keeping-components-pure)  
[Reacting to Input with State](https://react.dev/learn/reacting-to-input-with-state)
