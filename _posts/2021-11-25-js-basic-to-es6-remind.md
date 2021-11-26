---
title: "자바스크립트 다시 보기"
date: 2021-07-20 13:26:28 -0400
categories: javascript
tags: javascript ecmascript es6


# 목차
toc: true  
toc_sticky: true
---

## intro
- node는 c++에 자바스크립트(구글의 v8)엔진을 포함한 프로그램. => 브라우저가 아니라도 자바스크립트를 실행할 수 있게 된다.
- 이크마스크립트는 규격이고, 자바스크립트는 그 규격을 준수한 프로그램랭귀지.

## types
프리미티브 타입 : String, Number, Boolean, undefined, null(Object)
레퍼런스 타입 : Object, Array, Function

스크립트는 다이나믹 랭귀지! 
let quiz = 'xxx';
quiz = 1;
--> 에러 안남. 그냥 타입이 바뀐것으로...

## new 키워드를 사용하면,
1. {}   빈 객체가 생성된다.
2. this가 {}빈 객체를 가르킨다.
3. {} 객체를 반환한다.

- 아래 2가지 형식으로 객체를 생성하는 방법에 대한 소스이다.


```javascript
//Factory Fuction
function circleByfunciton(radius){
    return{
        radius,//이름이 같은 파라미터 있으면 그대로 매핑 된다.
        draw: function(){
            console.log('function circle');
        }
    }
}
const fnCircle = circleByfunction(1);

//Constructor Function
function ConstructorCircle(radius){
    this.radius = radius;
    this.draw = function(){
        console.log('draw');
    }
}
const cnstCircle = new ConstructorCircle(1);

```
### object의 constructor.
모든 object에 constructor가 있다. 그 오브젝트를 생성하는 함수를 참조한다.
위 코드의 각 object의 constructor를 접근해보면 각각의 함수를 리턴한다.

todo github 이미지 업로드 후 참조하기
!(자바스크립트 오브젝트의 콘스트럭터)[js-obj-constructor.png]

e.g 프리미브타입도 마찬가지이다.

```javascript
var x = 'xxxx'; // 문자열
----- 
x.constructor
ƒ String() { [native code] }

```

컨스터럭터 형식으로 생성한 객체의 컨스트럭터를 보면 컨스트럭터가


## 혼란스럽겠지만, 자바스크립트에서는 function이 object라는 것.


