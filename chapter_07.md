# Chapter 7 함수형 최적화

## 7.1 함수 실행의 내부 작동 원리

### 7.1.1 커링과 함수 콘텍스트 스택

```tsx
// 일반 함수
const logger = function (appender, layout, name, level, message)

// 커리
const logger =
  function (appender) {
    return function (layout) {
      return function (name) {
        return function (level) {
          return function (message) {
        ...
```

중첩 구조는 한 번에 호출하는 것보다 함수 스택을 더 많이 씁니다. logger 함수를 커링 없이 실행하면 자바스크립트는 동기 실행되기 때문에 우선 전역 콘텍스트 실행을 잠시 멈추고 새 활성 콘텍스트를 만든 다음, 변수 해석에 사용할 전역 콘텍스트 레퍼런스를 생성합니다.

logger 함수는 그 안에서 다른 Log4j 연산을 호출하므로 새 함수 콘텍스트가 생성되어 스택에 쌓입니다. 자바스크립트 클로저 때문에 내부 함수 호출로 비롯된 함수 콘텍스트는 다른 콘텍스트 위에 차곡차곡 쌓이며, 각 콘텍스트는 일정 메모리를 차지한 채 scopeChain 레퍼런스를 통해 연결됩니다.

![Untitled](/img/chapter_07/4.png)

아주 강력한 알고리즘이긴 하지만, 함수가 깊이 중첩되면 메모리를 과다하게 점유할 가능성이 있습니다. 그래서 비동기 코드를 다루는 RxJS라는 함수형 라이브러리는 최근 버전 5부터 이전 버전과 완전히 선을 긋고 성능에 올인해서 클로저 수를 줄이는 데 총력을 기울이기도 했습니다.

모든 함수를 커리하면 항상 좋을 것 같지만, 과용하면 메모리가 소모되면서 프로그램 실행속도가 현저히 떨어질 수 있습니다.

### 7.1.2 재귀 코드의 문제점

함수가 자신을 호출할 때에도 새 함수 컨텍스트가 만들어집니다. 하지만 기저 케이스에 도달할 수 없게 잘못 구현된 재귀 코드를 호출하면 스택이 넘칠 수 있습니다. 다만 재귀는 제대로 작동하지 않으면 Range Error : Maximum Call Stack Exceeded 라는 에러를 바로 뱉습니다. 스택 에러가 발생하는 로직은 브라우저마다 다르며, 이 경우 외에 잘못 구현되지 않았더라도 **엄청 큰 용량의 데이터를 재귀로 처리할 때 그 배열의 크기만큼 스택이 커질 수 있습니다.**

이처럼 **커링과 재귀를 함수에 적용하면 명령형 코드보다는 메모리를 더 차지하겠지만 반대로 얻을 수 있는 이점과는 비교해봐야 합니다.**

## 7.2 느긋한 평가로 실행을 늦춤

![Untitled](/img/chapter_07/1.png)

### 7.2.1 대체 함수형 조합으로 계산을 회피

함수를 레퍼런스(또는 이름)으로 전달하고 조건에 따라 한쪽만 호출하여 쓸데 없는 계산을 건너뛰는 것입니다.

```tsx
const alt = R.curry((func1, func2, val) => func1(val) || func2(val));

const showStudent = R.compose(append('#student-info'),
  alt(findStudent, createNewStudent)); // 함수를 레퍼런스로 전달

showStudent('444-44-4444');
```

### 7.2.2 단축 융합을 활용

선언적인 형태로 프로그램을 작성하는 건, 하고 싶은 일을 미리 정의함으로써 함수가 어떻게 작동하든 신경 쓰지 않고 무슨 일을 해야하는지만 밝힌다는 의미입니다. 덕분에 단축 융합이라는 기법으로 로대시JS가 프로그램 실행을 내부적으로 최적화할 수 있습니다. **단축 융합은 몇 개 함수의 실행을 하나로 병합하고 중간 결과를 계산할 때 사용하는 내부 자료구조의 개수를 줄이는 함수 수준의 최적화**입니다. 자료구조가 줄면 대량 데이터를 처리할 때 필요한 과도한 메모리 사용을 낮출 수 있겠죠.

```tsx
const square = x => Math.pow(x, 2);
const isEven = x => x % 2 === 0;
const numbers = _.range(200);

const result =
  _.chain(numbers)
   .map(square)
   .filter(isEven)
   .take(3) // 조건을 만족하는 처음 세 숫자만 처리
   .value(); // -> [0, 4, 16]

result.length; // -> 5
```

- take(3)을 호출하여 map, filter를 통과한 처음 세 값만 신경 쓰고 나머지 195개 값들에 대해선 괜히 에너지를 낭비하지 말라고 로대시JS에게 지시
- 단축 융합을 이용해서 이후 map/filter 호출은 compose(filter(isEven), map(square))  속으로 융합시킵니다.

## 7.3 ‘필요할 때 부르리’ 전략

반환 시 유일하게 참조할 수 있는 키를 돌려주고 키/쌍 값을 캐시에 보관하는 것입니다. 캐시란 값비싼 연산을 하기 전에 일단 질의하는 중간 저장소/메모리입니다.

이것을 함수로 감싸는 로직을 만들면 전역 공유 캐시 객체에 의존하는 부수효과가 있고 가독성도 떨어진다.

### 7.3.1 메모화

메모화 배후의 캐시 전략도 함수 인수를 키값을 만들고 이 키로 계산 결과를 캐시에 보관해두었다가, 이후 다시 같은 인수로 함수를 호출하면 보관된 결과를 즉시 반환한다는 로직은 방금전 코드와 같습니다. 함수의 결과를 해당 입력과 연관시키는 일, 즉 다시 말해, **함수의 입력을 어떤 값으로 계산해내는 건 어떤 함수형 프로그래밍의 원리 덕분에 가능할까요? 네, 그렇습니다, 바로 참조 투명성이죠.** 먼저, 단순 함수 호출에 메모화를 적용하고 어떤 효과가 있는지 살펴보겠습니다.

### 7.3.2 계산량이 많은 함수를 메모화

메모화를 하면 동일한 입력으로 함수를 재호출할 때 내부 캐시가 히트되어 즉시 결과가 반환됩니다. 이러한 메모화는 자바스크립트에서 자동 메모화를 지원하진 않지만, 아래와 같이 Function 객체에 보강해 사용 가능합니다.

```tsx
Function.prototype.memoized = function () { 
// 이 함수 인스턴스에 특화된 캐시 로직이 담긴 내부 도우미 메서드
    let key = JSON.stringify(arguments); 
// 입력을 문자열화해서 함수 식별자를 얻습니다. 
// 입력 형식을 감지해서 키를 생성하는 체계가 있으면 더 짜임새 있는 코드가 되겠지만 이 예제로 충분합니다.
    
    this._cache = this._cache || {};
// 이 함수의 내부 지역 캐시를 만듭니다.

    this._cache[key] = this._cache[key] || this.apply(this, arguments);
// 어떤 입력으로 이전에 함수를 실행한 적 있는지 캐시를 먼저 읽습니다
// 값이 있으면 함수를 건너뛰고 결과를 반환하며, 값이 없으면 계산을 합니다.    

    return this._cache[key];
}

Function.prototype.memoize = function () {
    let fn = this;
    if(fn.length === 0 || fn.length > 1) {
        return fn;
    }

    return function () {
        return fn.memoized.apply(fn, arguments);
    }
}
```

![Untitled](/img/chapter_07/2.png)

### 7.3.3 커링과 메모화를 사용

인수가 여러 개인 함수는 아무리 순수함수라 해도 캐시하기가 어렵습니다. 캐시 계층에서 추가 오버헤드가 안 생기게 하려면 키값 생성 연산이 단순해야 하는데 외려 더 복잡해지기 때문입니다. 한가지 해결 방법은 커링입니다. 다항 함수를 커리하려면 단항 함수로 바꿀 수 있다고 했습니다. 

```tsx
const safeFindObject = R.curry(function (db, ssn){
	// ... 복잡한 IO
})

const findStudent = safeFindObject(DB('students')).memoize();
```

### 7.3.4 분해하여 메모화를 극대화

코드를 잘게 나눌수록 메모화 효과는 더욱 커집니다. 여러 함수가 조합된 함수 중 일부만 메모된 함수로 바꿔도 큰 속도 향상의 효과를 볼 수 있습니다.

### 7.3.5 재귀 호출에도 메모화를 사용

재귀는 때때로 브라우저를 죽음으로 몰고 가거나 예외를 던지게 합니다. 이는 엄청 큰 입력을 처리하면서 스택이 비정상적으로 커질 때 일어날 수 있으며, 메모화를 이용하면 문제가 해결될 수 있습니다.

재귀 호출은 기저 케이스에 도달할 때까지 하위 문제들을 풀어 마지막에 호출 스택이 풀리며 최종 결과를 내는데, 하위 문제들의 결과를 캐시하면 성능을 끌어올릴 수 있습니다.

```tsx
const factorial = ((n) => (n === 0) ? 1 : (n * factorial(n - 1)).memoize();

factorial(100); // => .299밀리초
factorial(101); // => .021밀리초 / 이전에 캐시한 값으로 101*100!를 계산하면 실행시간이 단축됩니다
```

![Untitled](/img/chapter_07/3.png)

## 7.4 재귀와 꼬리 호출 최적화

위에서 봤던 예제처럼 입력이 동일한 경우가 아닌, 입력이 계속 바뀌어 내부 캐시 계층이 별 역할을 할 수 없다면 메모화도 도움이 되지 않습니다. 재귀를 일반 루프만큼 실행을 최적화하기 위해선, **꼬리 재귀 호출** - tail call optimization (TCO) - 을 수행하게끔 재귀 알고리즘을 구현하면 됩니다.

팩토리얼 함수를 꼬리 위치에서 재귀하도록 바꾸는 아래 예시입니다.

```jsx
const factorial = (n, current = 1) =>
  n === 1 ? current : factorial(n - 1, n * current); // 함수의 마지막 구문에서 재귀 단계를 밟음
```

TCO 는 **꼬리 호출 제거** 라고도 하는데, 재귀 프로그램이 제일 마지막에 다른 함수를 호출할 경우에만 TCO 가 일어납니다. 이때 마지막 호출이 **꼬리 위치**에 있다고 부릅니다.

이러한 방법이 최적인 이유는, 재귀 함수가 가장 마지막에 함수를 호출하면 자바스크립트 런타임은 남은 할 일이 없기 때문에 현재 스택 프레임을 들고 있을 이유가 없어 폐기합니다. 이처럼 재귀를 반복할 때마다 스택에 새 프레임은 쌓이지 않고 버려진 프레임을 재활용할 수 있습니다.

```jsx
// 일반 재귀
factorial(4)
  4 * factorial(3)
    4 * 3 * factorial(2)
      4 * 3 * 2 * factorial(1)
        4 * 3 * 2 * 1 * factorial(0)
          4 * 3 * 2 * 1 * 1
      4 * 3 * 2 * 1
    4 * 3 * 2
  4 * 6
return 24

// 꼬리 재귀 호출
factorial(4)
  factorial(3,4)
  factorial(2, 12)
  factorial(1, 24)
  factorial(0, 24)
  return 24
return 24
```

### 7.4.1 비꼬리 호출을 꼬리 호출로 전환

자바스크립트의 TCO 체제를 활용해 계승 함수를 최적화 해보겠습니다.

```jsx
const factorial = n => n === 1 ? 1 : n * factorial(n-1));
// 마지막에 재귀 단계와 숫자 n 을 곱한 결과를 반환하므로 꼬리 호출이 아님
// 이 부분을 바꿔야 TCO 활용 가능

// 1. 곱셈 부분을 함수의 매개변수로 추가해 현재 곱셈을 추적
// 2. ES6 기본 매개변수로 기본 인수 값을 미리 정함
const factorial = (n, current = 1) =>
  n === 1 ? current : factorial(n - 1, n * current);
```

또 다른 예시도 보겠습니다.

```jsx
// 비꼬리 호출
function sum(arr) {
  if (_.isEmpty(arr)) {
    return 0;
  }
  return _.first(arr) + sum(_.rest(arr)); // 이 부분을 수정
}

// 꼬리 호출
function sum(arr, acc = 0) {
  if (_.isEmpty(arr)) {
    return acc;
  }
  return sum(_.rest(arr), arr + _.first(arr));
}
```

꼬리 재귀는 재귀 루프의 성능을 수동 루프급으로 끌어올립니다. 다만 TCO 는 ES4 시절 처음 나온 후로 자바스크립트 표준이 되었지만 이를 지원하지 않는 브라우저가 더 많아 바벨을 같이 사용해야 합니다.

빡빡한 그래픽 렌더링 루프를 돌리거나, 짧은 시간 내에 대량 데이터를 처리할 경우 성능이 관건입니다. 처리 속도를 높이는 일이 우선이므로 우아하고 확장 가능한 코드는 차치하고 절충점을 찾아야 하며, 최적화는 제일 마지막에 하더라도 밀리초 단위의 성능을 뽑아내야 하는 경우라면 이번 6장에서 본 기법들을 적용해 볼 법 합니다.