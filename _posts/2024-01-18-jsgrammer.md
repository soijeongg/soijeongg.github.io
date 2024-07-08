---
layout: post
read_time: true
show_date: true
title: "자바스크립트 문법 정리 "
date: 2024-01-18 15:00:20 -0600
description: "자바스크립트 문법 정리"
image: https://hanghae99.spartacodingclub.kr/images/main/og.jpg
tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---

- 내가 보는 용으로 자바스크립트 문법
지속적인 업데이트

01.18 목
 ### 1. indexof
indexof는 문자열 뒤에 붙어 쓰이는 메소드 이다 `string.indexof(문자)`
문자열에서 지정된 부분 문자열을 검색하고, 해당 부분 문자열의 첫 번째 발생 위치를 반환하는 JavaScript의 문자열 메서드
없으면 -1을 반환한다 
만약 문자열이면 그 문자의 시작 부분의 인덱스를 반환한다 

 ### 2.자바스크립트에는 in 연산자는 주로 객체의 속성 존재 여부를 확인하는 데 사용
```javascript
// 예제 객체
var myObject = {
    name: "John",
    age: 25,
    city: "New York"
};

// 객체의 속성 존재 여부 확인
console.log("name" in myObject);  // true
console.log("gender" in myObject);  // false

```
파이썬 처럼 문자열에 직접 사용 불가

### 3.자바스크립트는 문자에다가 숫자를 곱하는 방식으로 문자열이 반복되지 않는다 
반복하는 메소드인 repeat을 써야 한다

2024.01.19
### 4.`Array or string.includes()` 메서드는 배열(문자열)에 특정 요소가 포함되어 있는지 여부를 확인하는 메서드
있으면 true 반환하고 없으면 false을반환한다 


### 5.`Array.from()` 배열을 만든다
문자열등 유사 배열을 배열로 만들어주는데 한글자씩 끊어서 만들어준다 
첫번째 인자로 배열로 만들 유사 배열이 들어가고 두번째 인자로 매핑 함수(mapping function)가 들어간다
이 매핑함수는 배열의 각 요소에 대해 호출되어 해당 요소를 새로운 값으로 변환하는 역할
선택적으로 사용된다 
숫자에 사용할경우 일단 문자열로 배열이 만들어져 두번째 인자로 Number가 들어가야 숫자로 변환된다
문자열이 들어가게 되면 한글자씩 쪼개져서 들어가게 된다
``` javascript
Array.from(arrayLike, (currentValue, index) => {
  // currentValue는 현재 요소의 값
  // index는 현재 요소의 인덱스
  // 여기에서 반환하는 값은 새로운 배열의 해당 요소의 값이 됨
});

```

### 6.Array.map()
```javascript
const newArray = array.map((currentValue, index, array) => {
  // currentValue: 현재 요소의 값
  // index: 현재 요소의 인덱스
  // array: 원본 배열
  // 여기에서 반환하는 값은 새로운 배열의 해당 요소의 값이 됨
});

```
각 요소에 대해 주어진 콜백 함수를 호출, 그 함수가 반환한 결과로 새로운 배열을 생성
기존 배열을 수정하지 않고, 새로운 배열을 반환

### 7. Array.reduce()
```javascript
array.reduce(callback, initialValue);
//callback: 각 요소에 대해 실행되는 함수로, 네 개의 매개변수를 받습니다.

//accumulator: 누적된 결과값 또는 중간 결과값입니다.
//currentValue: 현재 처리 중인 배열의 요소입니다.
//currentIndex: 현재 처리 중인 배열의 요소 인덱스입니다.
//array: reduce가 호출된 배열입니다.
//initialValue (선택적): 첫 번째 콜백 함수 호출 시 사용되는 초깃값입니다.
//만약 이 값이 제공되지 않으면 배열의 첫 번째 요소가 초기 accumulator로 사용됩니다.
```
중간값이 계속 accumulator로 감

### 8. padstart, padend
```javascript
str.padStart(targetLength [, padString]);

//targetLength: 최종 문자열이 가져야 할 길이.
//현재 문자열의 길이가 이 값보다 작으면 채워진다. 
//만약 현재 문자열의 길이가 이미 targetLength와 같거나 더 크면 원래 문자열을 그대로 반환합니다.
//padString (선택적): targetLength에 도달하기 위해 사용할 채워넣을 문자열입니다. 
//기본값은 공백입니다.
str.padEnd(n, 'b') === 'abbbb'//이건 뒤에
```
### 9. 대문자와 소문자로 바꾸기
문자열.toUpperCase()-> 대문자로 바꾸기
문자열.toLowerCase()-> 소문자로 바꾸기

### 10 for of/ for in
for of는 값을 반환하고 for in은 값의 인덱스를 반환