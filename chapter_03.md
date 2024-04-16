# Chapter 3 자료구조는 적게, 일은 더 많이

> 계산 프로세스는 컴퓨터에 내재하는 추상적인 존재다. 이들이 점점 진화하면서 프로세스는 데이터라는 또 다른 추상적인 존재에 영향을 끼친다.
> 

## 3.1 애플리케이션의 제어 흐름

프로그램이 정답에 이르기까지 거치는 경로는 제어 흐름(control flow)이라고 합니다. 명령형 프로그램은 작업 수행에 필요한 전 단계를 노출하여 흐름이나 경로를 아주 자세히 서술합니다. 보통 작업을 수행하는 단계는 루프와 분기문, 구문마다 값이 바뀌는 변수들로 빼곡히 들어차지요. 명령형 프로그램의 틀을 고수준에서 바라보면 다음 코드와 같습니다.

```tsx
var loop = optC();
while(loop) {
	var condition = optA();
	if(condition) {
		optB1()
	}
	else {
		optB2();
	}
	loop = optC();
}
optD();
```

반면 선언적 프로그램 특히 함수형 프로그램은 독립적인 블랙박스 연산들이 단순하게, 즉 최소한의 제어 구조를 통해 연결되어 추상화 수준이 높다. 이렇게 연결한 연산들은 각자 다음 연산으로 상태를 이동시키는 고계함수에 불과합니다. 실제로 함수형 프로그램은 데이터와 제어 흐름 자체를 고수준 컴포넌트 사이의 단순한 연결로 취급합니다.

덕분에 다음과 같이 코드가 짧아집니다.

```tsx
optA().optB().optC().optB()
```

연산을 체이닝하면 간결하면서 물 흐르는 듯한, 표현적이 형태로 프로그램을 작성할 수 있어 제어 흐름과 계산 로직을 분리할 수 있고 코드와 데이터를 더욱 효과적으로 헤아릴 수 있습니다.

## 3.2 메서드 체이닝

메서드 체이닝은 여러 메서드를 단일 구문으로 호출하는 OOP 패턴입니다. 메서드가 모두 동일한 객체에 속해 있으며 메서드 흘리기라고도 합니다. 대부분 객체지향 프로그램에서 불변 객체에 많이 적용하는 패턴이지만 함수형 프로그래밍에도 잘 맞습니다.

```tsx
'Functional Programming'.substring(0, 10).toLowerCase() + ' is fun';
```

substring과 toLowerCase 메서드는 각자 자신을 소유한 문자열 객체에(this로 접근하여) 어떤 작업을 한 다음 새로운 문자열을 반환합니다. 자바스크립트 문자열에서 플러스(+) 연산자는 문자열을 합친 다음 새 문자열을 반환하도록 오버로드한 간편 구문입니다. 이러한 변환 과정을 거치면 원본 문자열은 전혀 건드리지 않고도 원본과는 무관한 문자열이 생성됩니다. 

```tsx
concat(toLowerCase(substring('Functional Programming', 1, 10)), 'is fun');
```

매개변수는 모두 함수 선언부에 명시해서 부수효과를 없애고 원본 객체를 바꾸지 않아야 한다는 함수형 교리를 충실히 반영한 코드입니다.  그러나 이렇게 함수 코드를 안쪽에서 바깥쪽으로 작성하면 메서드 체이닝 방식만큼 매끄럽지 못합니다. 로직을 파악하려면 가장 안쪽에서 감싼 함수부터 한 꺼풀씩 벗겨내야 하고 가독성도 현저히 떨어지지요.

변이를 일으키지 않는 한 함수형 프로그래밍에서도 단일 객체 인스턴스에 속한 메서드를 체이닝하는 건 나름대로 쓸모가 있습니다. 

## 3.3 함수 체이닝

함수형 프로그래밍은 자료구조를 새로 만들어 어떤 요건을 충족시키는게 아니라, 배열 등의 흔한 자료구조를 이용해 다수의 굵게 나뉜 고계 연산을 적용합니다. 이러한 고계 연산으로 다음과 같은 일을 합니다.

- 작업을 수행하기 위해 무슨 일을 해야 하는지 기술된 함수를 인수로 받습니다.
- 임시 변수의 값을 계속 바꾸면서 부수효과를 일으키는 기존 수동 루프를 대체합니다. 그 결과를 관리할 코드가 줄고 에러가 날 만한 코드 역시 줄어듭니다.

### 3.3.1 람다 표현식

함수형 프로그래밍에서 탄생한 람다 표현식(자바 스크립트에서는 두 줄 화살표 함수라고도 함)은 한 줄짜리 익명 함수를 일반 함수 선언보다 단축된 구문으로 나타냅니다. 

```tsx
const name = p => p.fullname;
```

여기서 주목할 점은 일급 함수와 람다 표현식의 관계입니다.  위 예제에서 name은 실재하는 값이 아니라, 그 값을 얻기 위한 (느긋한) 방법을 가리킵니다. 즉 name으로 데이터를 계산하는 로직이 담긴 두 줄 화살표 함수를 가리키는 것입니다. 함수형 프로그램은 이렇게 함수를 마치 값처럼 쓸 수 있습니다. 

함수형 프로그래밍은 람다 표현식과 잘 어울리는 세 주요 고계함수 map, reduce, filter를 적극 사용할 것은 권장합니다. 사실 함수형 자바스크립트는 대부분 자료 리스트를 처리하는 코드입니다.

### 3.3.2 _.map: 데이터를 변환

```tsx
function map(arr, fn) {
	const len = arr.length,
				result = new Array(len);
	
	for(let idx = 0; idx < len; ++idx) {
		result[idx] = fn(arr[idx], idx, arr)
	}
	
	return result;
}
```

map이 반복을 대행하는 덕분에 개발자는 루프 변수를 하나씩 늘리며 경계 조건을 체크하는 등의 따분한 일은 이 함수에게 맡기고 이터레이터 함수에 구현한 비즈니스 로직만 신경쓰면 됩니다.

### 3.3.3 _.reduce: 결과를 수집

```tsx
function reduce(arr, fn, accumulator) {
	let idx = -1,
			len = arr.length;
			
	if(!accumulator && len > 0) {
		accumulator = arr[++idx];
	}
	
	while(++idx < len) {
		accumulator = fn(accumulator, arr[idx], idx, arr);
	}
	
	return accumulator;
}
```

### 3.3.4 _.filter: 원하지 않는 원소를 제거

```tsx
function filter(arr, predicate) {
	let idx = -1,
			len = arr.length,
			result = [];
			
	while(++idx < len) {
		let value = arr[idx];
		if(predicate(value, idx, this)) {
			result.push(value)
		}
	}
	return result;
}
```

## 3.4 코드 헤아리기

‘코드를 헤아린다-reason’는 건 무슨 뜻일까요? 1, 2장에서 필자는 프로그램의 일부만 들여다봐도 무슨 일을 하는 코드인지 멘털 모델을 쉽게 구축할 수 있다는 의미로 이 표현을 사용했습니다. 여기서 멘털 모델이란 전체 변수의 상태와 함수 출력 같은 동적인 부분뿐만 아니라 설계 가독성 및 표현성 같은 정적인 측면까지 포괄하는 개념입니다.

함수형 흐름은 프로그램 로직을 파헤치지 않아도 뭘 하는 프로그램인지 윤곽을 잡기 쉽기 때문에,  개발자는 코드뿐만 아니라, 결과를 내기 위해 서로 다른 단계를 드나드는 데이터의 흐름까지 더 깊이 헤아릴 수 있습니다.

### 3.4.1 선언적 코드와 느긋한 함수 체인

```tsx
var result = [];
for (let i = 0; i < names.length; i++) {
	var n = names[i];
	if (n !== undefined && n !== null) {
		var ns = n.replace(/_/, ' ').split(' ');
		for (let j = 0; j < ns.length; j++) {
			var p = ns[j];
			p = p.charAt(0).toUpperCase() + p.slice(1);
			ns[j] = p;
		}
		
		if(result.indexOf(ns.join(' ')) < 0) {
			result.push(ns.join(' '));
		}
	}
}
result.sort()
```

명령형 코드의 단점은 특정 문제의 해결만을 목표로 한다는 점입니다.  함수형 보다 훨씬 저수준에서 추상한 코드로서 한 가지로 용도로 고정됩니다. 추상화 수준이 낮을수록 코드를 재사용할 기회는 줄어들고 에러 가능성과 코드 복잡성은 증가합니다.

```tsx
_.chain(names)
	.filter(isValid)
	.map(s => s.replace(/_/, ' '))
	.uniq()
	.map(_.startCase)
	.sort()
	.value()
```

반면, 함수형 프로그램은 블랙박스 컴포넌트를 서로 연결만 해주고, 뒷일은 테스트까지 마친 검증된 API에게 모두 맡깁니다. 폭포수 떨어지듯 함수를 연달아 호출하는 모습이 눈에 더 잘 들어오지 않나요?

_.chain 함수는 주어진 입력을 원하는 출력으로 변환하는 연산들을 연결함으로써 입력 객체의 상태를 확장합니다.  _(…) 객체로 단축 표기한 구문과 달리, 이 함수는 임의의 함수를 명시적으로 체이닝 가능한 함수로 만듭니다. 

_.chain을 쓰면 복잡한 프로그램을 느긋하게 작동시키는 장점도 있습니다. 제일 끝에서 value() 함수를 호출하기 전에는 아무것도 실행되지 않으니까요. 결괏값이 필요 없는 함수는 실행을 건너뛸 수 있어서 애플리케이션 성능에 엄청난 영향을 미칩니다.

이 코드가 부드럽게 작동하는 건 FP의 근본 원리인, 부수효과 없는 순수함수 덕분입니다. 체인에 속한 각 함수는 이전 단계의 함수가 제공한 새 배열에 자신의 불변 연산을 적용합니다. _.chain()으로 시작하는 이런 로대시JS의 패턴은 거의 모든 요구를 충족하는 매각이버 칼을 제공합니다. 이런 방식은 함수형 프로그래밍의 독특한 무인수 프로그래밍 스타일로 이어집니다.

프로그램 파이프라인을 느긋하게 정의하면 가독성을 비롯해서 여러모로 이롭습니다. 느긋한 프로그램은 평가 이전에 정의하기 때문에 자료구조를 재사용하거나 메서드를 융합하여 최적화 할 수 있습니다. 

### 3.4.2 유사 SQL 데이터: 데이터로서의 함수

로대시JS가 지원하는 믹스인 기능을 응용하면,  핵심 라이브러리에 함수를 추가하여 확장한 후, 마치 원래 있던 함수처럼 체이닝할 수 있습니다.

```tsx
_.mixin({
	'select': _.map,
	'from': _.chain,
	'where': _.filter,
	'sortBy': _.sortByOrder
})

_.from(persons)
	.where(p => p.birthYear > 1990 && p.address.country !== 'US')
	.sortBy(['firstname'])
	.select(p => firstname)
	.value()
```

## 3.5 재귀적 사고방식

### 3.5.1 재귀란?

재귀는 주어진 문제를 자기 반복적인 문제들로 잘게 분해한 다음, 이들을 다시 조합해 원래 문제의 정답을 찾는 기법입니다. 재귀 함수의 주된 구성 요소는 다음과 같습니다.

- 기저 케이스(종료 조건이라고도 합니다)
- 재귀 케이스

기저 케이스는 재귀 함수가 구체적인 결괏값을 바로 계산할 수 있는 입력 집합입니다. 재귀 케이스는 함수가 자신을 호출할 때 전달한 입력 집합(최초 입력 집합보다 점점 작아집니다)을 처리합니다.  입력 집합이 점점 작이지지 않으면 재귀가 무한 박복되며 결국 프로그램이 뻗겠죠. 함수가 반복될수록 입력 집합은 무조건 작아지며, 제일 마지막에 기저 케이스로 빠지면 하나의 값으로 귀결됩니다.

### 3.5.2 재귀적으로 생각하기

재귀적 사고란, 자기 자신 또는 그 자신을 변형한 버전을 생각하는 겁니다. 재귀적 객체는 스스로를 정의합니다. 

reduce 함수를 쓰면 루프는 물론 리스트 크기조차 신경 쓸 필요가 없습니다. 첫 번째 원소를 나머지 원소들과 순차적으로 더해가며 결괏값을 계산하는 재귀적 사고방식을 적용하는 셈이죠. 이 사고방식을 확장하면 결국 다음과 같이 수평사고라고 불리는 일련의 연산을 수행하는 과정으로 덧셈을 바라보게 됩니다.

```tsx
sum[1, 2, 3, 4, 5] = 1 + sum[2, 3, 4, 5]
									 = 1 + 2 + sum[3, 4, 5]
```

재귀와 반복은 동전의 앞/뒷면입니다. 재귀는 변이가 없으므로, 더 강력하고 우수하며 표현적인 방식으로 반복을 대체할 수 있습니다. 사실상 순수 함수형 언어는 모든 루프를 재귀로 수행하기 때문에 do, for, while 같은 기본 루프 체계조차 없으며, 재귀를 적용한 코드가 더 이해하기 쉽습니다. 점점 줄어드는 입력 집합에 똑같은 작업을 여러 번 반복한다는 전제하에 작동하기 때문입니다.

```tsx
function sum(arr) {
	if(_.isEmpty(arr)) { // 기저 케이스(종료 조건)
		return 0;
	}
	
	return _.first(arr) + sum(_.rest(arr)); // 재귀 케이스: _.first와 _.rest로 입력을 점점 줄여가며 자신을 호출합니다.
}
```

원소가 포함된 배열은 첫 번쨰 원소를 추출 후 두 번째 이후 원소들과 계속 재귀적으로 더합니다. 이때 내부적으로 재귀 호출 스택이 겹겹이 쌓입니다. 알고리즘이 종료 조건에 이르면 쌓인 스택이 런타임에 의해 즉시 풀리면서 반환문이 모두 실행되고 이 과정에서 실제 덧셈이 이루어집니다. 바로 이런 식으로 재귀를 이용해 언어 런타임에 루프를 맡기는 것입니다.

재귀와 수동 반복, 성능은 어떨까요? 지난 세월 동안 컴파일러는 아주 영리하게 루프를 최적화 할 수 있도록 진화했습니다. ES6부터는 꼬리 호출 최적화까지 추가되어 재귀와 수동 반복의 성능 차이는 미미해졌지요.

### 3.5.3 재귀적으로 정의한 자료구조

**Node형**

```tsx
/**
 * 트리 탐색용 노드 클래스
 */
// 모듈 전용 도우미 함수
const _ = require('lodash');

const isValid = val => !_.isUndefined(val) && !_.isNull(val);

exports.Node = class Node {
    constructor(val) {
        this._val = val;
        this._parent = null;
        this._children = [];
    }

    isRoot() {
        return !isValid(this._parent);
    }

    get children() {
        return this._children;
    }

    hasChildren() {
        return this._children.length > 0;
    }

    get value() {
        return this._val;
    }

    set value(val) {
        this._val = val;
    }

    append(child) {
        child._parent = this;
        this._children.push(child);
        return this;
    }

    toString() {
        return `Node (val: ${this._val}, children:
            ${this._children.length})`;
    }
};
```

**Tree형**

```tsx
/**
 * 간단한 트리 클래스
 * 저자: 루이스 아텐시오
 */

const _ = require('lodash');

class Tree {
    constructor(root) {
        this._root = root;
    }

    static map(node, fn, tree = null) {
        node.value = fn(node.value);
        if(tree === null) {
            tree = new Tree(node);
        }
        if(node.hasChildren()) {
            _.map(node.children, function (child) {
                Tree.map(child, fn, tree);
            });
        }
        return tree;
    }

    get root() {
        return this._root;
    }

    toArray(node = null, arr = []) {
        if(node === null) {
            node = this._root;
        }
        arr.push(node.value);
        // 기저 케이스
        if(node.hasChildren()) {
            var that = this;
            _.map(node.children, function (child) {
                that.toArray(child, arr);
            });
        }
        return arr;
    }
}

exports.Tree = Tree;
```

변이 및 부수효과 없는 자료형을 다룰 때 데이터 자체를 캡슐화하여 데이터에 접근하는 방법을 통제하는 것이 함수형 프로그래밍의 관건입니다. 자료구조 파싱은 소프트웨어에서 가장 기본적인 작업이자, 함수형 프로그래밍의 주특기이기도 합니다. 함수형 프로그래밍은 원하는 결과를 얻기 위한 비즈니스 로직이 담겨 있는 고수준의 연산을 인련의 단계들로 체이닝하는, 간결한 흐름 중심의 모델을 선호합니다.
