---
layout: post
read_time: true
show_date: true
title: "javascript의 클로저"
date: 2024-09-18 10:00:20 -0600
description: "코어자바스크립트 클로저 정리"
image:  https://image.yes24.com/goods/78586788/XL
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### 클로저의 의미 및 원리 이해
클로저는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성으로 클로저는 함수와 그 함수가 선언될 당시의 렉시컬 환경의 상호관계에 따른 현상

렉시컬환경은 실행컨텍스트의 outEnvironmentReferenccee에 해당되며 변수의 유효범위인 스코프가 결정되고 스코프 체인이 가능해진다 

어떤 컨텍스트A에서 선언한 내부함수 B의 실행컨텍스트가 활성화된 실행컨텍스트가 활성화된 시점의 outEnvironmentReferenccee가 참조하는 대상인 A의 outEnvironmentReferenccee에도 접근 가능

A에서는 B에 선언된 변수에는 접근이 불가능하지만 B에서는 A에서 선언된 함수에 접근이 가능하다

내부함수 B가 A의 렉시컬환경을 매번 사용하는것은 아니다 내부함수에서 외부변수를 참조하는 경우에만, 선언될 당시의 렉시컬환경의 상호관계가 의미 있다 

```javascript
function outer() {
  let a = 'I am from outer';

  var inner = function () {
    console.log(a); // 내부 함수가 외부 함수의 변수를 참조하는 경우
  }

  inner()
}

const closureFunction = outer();
closureFunction(); // 'I am from outer' 출력
```

outer에서 a를 선언했고 내부함수는 inner에서 a를 선언하지 않아서 환경레코드에서는 찾을 수 없으므로 outEnvironmentReferenced에 지정된 상우컨텍스트의 outer에 접근해서 찾는다 

outer함수의 실행컨텍스트가 종료되면 렉시컬환경에 저장된 실별자들에 대한 참조를 지우고 각 주소에 저장되에 있던 값은 변수가 하나도 없게 되기 때문에 가비지 컬렉터의 수집 대상이 된다

콜스택을 살펴보면 먼저 콜스텍에 전역 컨텍스트가 먼저 들어가게 되고 가장 위에 있는 outer가 쌓이게 되고 그 후 inner도 쌓이게 된다

inner가 사용이 끝나고 나가고 outer도 사용이 끝나고 나가고 다시 전역도 나가 빈 컨텍스트가 된다 

```javascript
function outer() {
  let a = 1

  var inner = function {
    return a++
  }

  return inner()
}

const closureFunction = outer();
closureFunction();
```
이번에도 inner 안에 내부 변수인 a를 사용헸고 inner함수를 return 하고 있어 outer함수의 실행컨텍스트가 종료된 시점에는 a변수 참조 대상이 없어져 a와 inner 둘다 가비지 컬렉터에 사라짐

둘다 outer 함수 실행컨텍스트 삭제 전에 a 참조 대상이 사라져 가비지 컬렉터에 의해 사라진다 

```javascript
function outer() {
  let a = 1

  var inner = function() {
    return a++
  }

  return inner
}

const outer2 = outer();
outer2();
```
이번에는 함수의 실행결과가 아닌 함수 자체를 반환했다 

그러면 outer함수의 실행컨텍스트가 종료될때(const outer2 = outer();)

outer2함수는 outer의 실행결과인 inner함수를 참조할것이다 

그 후 한번더 outer2함수를 호출하면 inner가 실행될것이다 

inner함수의 실행컨텍스트의 환경레코드는 수집할 정보가 없어서 inner 함수가 선언된 위치의 렉시컬 환경이 참조 복사 된다 

### outer2 함수를 호출하면 inner함수만 호출되는것이 아니라 outer함수도 호출되는것이 아닌가?
outer를 호출할때 outer의 실행컨텍스트가 생성되고 a가 선언되도 inner에 익명함수가 할당된다 

그 후 outer는 inner함수를 반환한다 -> outer2는 이제 inner 함수를 가리키고 있다

여기서 outer의 실행이 끝나 더이상 실행되지 않고 inner만 남는다

하지만 클로저 덕분에  inner함수는 outer함수 내부에서 선언된 a에 접근할 수 있다 

const outer2 = outer(); 에서 참조하고 있는 것은 inner함수로 outer2 함수에 저장된것이 outer가 아니라 inner다 

#### 왜 outer가 다시 실행되지 않는지 
outer가 실행된 후 반환된 결과가 inner이기에 outer호출 시 내부의 inner가 실행된다
-> 그러면 함수가 한번 리턴값을 받으면 다시 실행되더라도 리턴값만 사용되는지 

그건 아니다 

함수가 다시 실행될때 매번 새로운 실행컨텍스트가 생성된다 하지만 outer2함수는 처음부터 inner함수를 참조한다 

함수가 호출될때 실행컨텍트가 생성되고 리턴값은 함수가 종료된 후에도 남는다 -> 실행컨텍스트가 종료되어도 반환값은 계속 사용할 수 있다 

outer2 = outer();  에서 outer2는 outer를 실행하고 다시 inner를 반환하고 
outer2() 이렇게 하면 inner만 사용

inner함수의 실행컨텍스트의 환경레코드(실행컨텍스트의 렉시컬환경안에 존재, 현재 스코프 내의 변수, 함수의 정보 저장) 에는 수집할 정보가 없어서 Outer Environment Reference(상위 스코프에 대한 참조)에는 inner함수가 선언된 위치의 렉시컬 환경이 참조된다  

inner함수는 outer함수 내부에서 실행되었으므로 outer함수의 렉시컬 환경이 담겨 스코프 체이닝에 따라 outer에서 선언된 변수에 접근 할 수 있다 

### inner함수의 실행컨텍스트의 환경레코드애 수집할 정보가 없는 이유 

실행컨텍스트의 환경레코드에는 현재 스코프내에 선언된 변수 함수의 정보가 담기는데 현재 스코프 inner 안에는 선언된 변수 함수의 정보가 없다 

### inner함수의 실행 시점에서 outer는 이미 종료되었지만 어떻게 outer의 렉시컬 환경에 접근 가능한지 

**가비지 컬렉터의 동작 방식 때문**

가비지 컬렉터는 참조가 없어진 변수를 메모리에서 삭제하는데 이는 참조하는 변수가 있다면 그 값을 수집대상에 포함시키지 않는다 

outer함수는 inner를 반환하고 외부함수인 outer가 종료되더라도 inner 함수의 실행컨텍스트가 활성화되면 Outer Environment Reference가 outer의 렉시컬환경이 필요로 해 수집대상에 제외된다 

**어떤 함수에서 선언된 변수를 참조하는 내부 함수에서만 발생하는 현상 -> 외부 함수의 렉시컬환경이 가비지 컬렉팅이 되지 않는 현상**


클로저란 **어떤 함수 A에 선언한 변수 a를 참조하는 내부 함수 B를 외부로 전달할 경우 A의 실행컨텍스트가 종료된 이후에도 a가 사라지지 않는 현상

외부의 전달이 return만인것은 아니다 

### 클로저와 메모리 관리 
메모리 누수는 개발자의 의도와 달리 참조 카운트가 0이 되지 않아 가비지 컬렉터의 수거 대상이 되지 않는것이다 

하지만 클로저 사용은 개발자가 의도적으로 참조 카운트를 0이 되지 않게 셀계한 것이여서 '누수'가 아니고 의도대로 설계한 메모리 소모에 대한 관리법만 파악해서 적용하면 된다

클로저는 의도적으로 함수의 지역변수, 메모리를 소모하도록 함으로써 발생 -> 참조 카운트를 0으로 만들어 가비지 컬렉터가 수거 하게 한다 

참조카운트를 0으로 만드는 방법은 식병자에 참조형이 하닌 기본형데이터(null, undefined)를 할당하면 된다 

```javascript
function outer() {
  let a = 1

  var inner = function() {
    return a++
  }

  return inner;
}

const outer2 = outer();
outer2();
outer2 =nul;
```

### 클로저 활용 사례
1. 콜백함수 내부에서 외부 데이터를 사용하고자 할떄 

```javascript
var fruits =['apple', 'banana', 'peach']
var $ul = document.createElement(ul)

fruits.forEach(function(fruit) {
  var$li = document.createElement('li')
  $li.addEventListener('click', function() {
    alert('your choice is '+fruit)
  });
  $ul.appendChild($li)

})
document.body.appendChild($ul)
```
fruit 변수를 순회하면서 li를 생성하고 각 li를 클릭하면 해당 리슽네에 기억된 콜백함수 실행됨

forEach 메서드에게 넘겨준 익명의 콜백 함수는 내부에서 외부함수를 사용하고 있지 않아 클로저가 없지만 addEventListener에게 넘겨준 익명함수는 fruit이라는 외부 함수를 참조하고 있어 클로저가 있다 

A의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 B가 실행될때는 B의 Outer Environment Reference에 A의 렉시컬 환경을 참조한다

2. 접근 권한 제어(정보은닉)
정보은닉은 내부 오직에 대해 외부로의 노출을 최소화해 모듈 간의 접합도를 낮추고 유연성을 높이고자 하는 개념으로 public, private, protected 3종류가 있다 

자바스크립트느 ㄴ기본적으로 변수 차체에 접근권한을 직접 부여하도록 설계 되어 있지 않짐반 클로저를 사용하면 public값과 private값을 구분하는 것이 가능하다 

```javascript
var outer = function() {
  var a =1;
  var inner = function() {
    return ++a
  };
  return inner
}
var outer2 = outer()
console.log(outer2())
```
outer 함수의 지역변수인 a의 값을 외부에도 읽을 수 있게 되어 이처럼 클로ㅓ저를 사용하면 외부 스코프에서 선택적으로 일부 변수에 대한 접근 권한을 부여 할 수 있다 

외부에서는 외부공간에 노출되어 있는 outer룰 통해 outer 함수 실행할 수 있지만 내부에는 어떤한 개입이 불가능하고 오직 리턴한 정보에만 접근 할 수 있다 

외부에 제공하고자 하는 정보를 모아 리턴하고 내부에서만 사용할 정보는 리턴하지 않는다 