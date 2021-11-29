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

### types
- Value Types(프리미티브 타입) : String, Number, Boolean, undefined, null(Object), `Symbol`(ES6에서 추가 됨.)
- Reference Types : Object, Array, Function

### 스크립트는 다이나믹 랭귀지! 
let quiz = 'xxx';
quiz = 1;
--> 에러 안남. 그냥 타입이 바뀐것으로...

## Object
### new 키워드를 사용하면,
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
모든 object에 constructor가 있다. 그 오브젝트를 생성하는 함수를 참조한다. (해당 오브젝트는 함수를 이용해 생성되었다.)
위 코드에서 생성된 각 object의 constructor를 접근해보면 각각을 생성한 함수를 리턴한다.

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


### 혼란스럽겠지만, 자바스크립트에서는 function이 object라는 것.
위 코드상에서 function 그 자체에 대한 컨스트럭터를 접근해 보면 아래와 같이 나온다. 

```javascript
ConstructorCircle.constructor
ƒ Function() { [native code] }

```

즉, 이 함수를 만드는 것은 자바스크립트의 Function() 이라는 말이다.

Fuction 함수를 이용해 함수객체를 만들수도 있다. 아래와 같다.
```javascript

//native Function을 이용해서 함수만들기.
const useNativeFuncFunction = new Function('radius',
`
    this.radius = radius;
    this.draw = function(){
        console.log('draw');
    }
`// 함수내용을 스트링으로.
);

//funcFuntionObj1, funcFuntionObj2 는 같다.
// new == {} 와 같고, 함수 안에서 this는 {}를 가르킨다.
// 객체 생성시 'new'가 없다면, this는 글로벌에 접근, 즉, window객체를 가르키게 된다.
const funcFuntionObj1 = new useNativeFuncFunction(1);
const funcFuntionObj2 = ConstructorCircle.call({},1);

```

### 프리미타입과 레퍼런스타입의 동작
- 밸류타입에서 참조를 해도, 참조형태가 아니라, 딥카피 형태로 완전 독립된다.
   
```javascript
let x = 20; // x에 값을 셋팅하고,
let y = x; // y에 x를 세팅하고,

x = 99999; //다시 x값을 바꾸면, 

// 결과적으로 x값만 바뀌고, y에는 20이라는 기존 값이 그대로 남아있다 즉, 완전히 독립된다는 의미이다.

```

그러나 object 타입의 경우는 참조형태로 서로를 참조한다.

```javascript
let a ={value : 20}// object
let b = a;
a.value = 8888;
console.log(a);
// console.log --> VM920:1 {value: 8888}
console.log(b);
// console.log --> VM938:1 {value: 8888}
```

### 프로퍼티 추가하기
```javascript
fnCircle.logcation = {x:1,y:2};
console.log(fnCircle);
// console.log
//      {radius: 1, logcation: {…}, draw: ƒ}
//      draw: ƒ ()
//      logcation: {x: 1, y: 2}
//      radius: 1
//      [[Prototype]]: Object
```

- 브라켓을 이용해 스트링으로 프로퍼티 추가하기. 동적일 때 유용.

```javascript
// 위에 소스 이어서.
const propertyName = 'tag';
fnCircle[propertyName] = {value:"동그라미"};
```

### object 반복하기

- 모든 인자 반복
```javascript
// 위에 소스 이어서.
for(let key in fnCircle){
    if(typeof fnCircle[key] !== typeof 'function')
    console.log(key, fnCircle[key]);
}
```

- 객체의 key 접근 및 Array 에 담기
```javascript
// 위에 소스 이어서.
const keys = Object.keys(fnCircle);
console.log(keys);
```

- 응용 : 객체에 프로퍼티가 존재하는가? 'in'을 이용하라
```javascript
// 위에 소스 이어서.
if('radius' in fnCircle){
    console.log('Yes~ It does!!!');
}
```

### 객체 안의 존재하는 함수에서 다른 형제함수를 호출하는데 따른 복잡성에 대한 이해
- 내부 함수의 추상화와 클로져


#### 문제상황 
- 아래와 같은 함수가 있다.
```javascript
// 
function Circle(radius){
    this.radius = radius;
    this.defaultLocation = {x:0, y:0};
    this.computeOptimumLocation = function(){
        // do something.....magic
    };

    this.draw = function(){
        this.computeOptimumLocation(); // 형제함수 부름.

        console.log('draw');
    }
}
```
만약 아래처럼 한다면, 
```javascript
circle.defaultLocation = false; // 외부에서 값/타입 변경사례
circle.computeOptimumLocation(); // inside 가 아니라 밖에서 호출하면
circle.draw();
```
- 모든 프로퍼티가 접근 가능해야 하는가에 대한 고려가 필요해짐.
- 해결 : 그래서 위 상황에서는 defaultLocation, computeOptimumLocation 를 밖에서는 접근할 수 없게 해야 한다.(디테일은 숨기고, 필수만 퍼블릭으로)

#### 클로져가 필요해짐.
클로져 : scope이 함수 내로 접근가능한 것이 제한되어 있는 내부 함수.
scope은 부르고나서 죽지만, 클로져는 거기 있다.
만약, draw() 함수 호출하면, 내부의 x,y는 죽지만, 클로져 안에서 부른 computeOptimumLocation는 아직 거기 있다. 다만, 해당 내부 함수를 가진 object의 프로퍼티를 사용하고 싶다면 역시 내부 함수 내에 선언해야 한다. 

```javascript
function Circle_better(radius){
    this.radius = radius;
    // 어떻게 abstraction을 구현할 것인가? 함수 내의 지역 변수를 사용하라.
    let defaultLocation = {x:0, y:0};
    let computeOptimumLocation = function(){
        // do something.....magic
    };
    
    // 외부에서 지역변수를 접근하고자 할때, 아래 더 나은 방법이 있다.
    this.getDefaultLocation = function(){
        return defaultLocation;
    }
    
    this.draw = function(){
        let x,y;// 지역변수로서 호출후에는 스코프 사라짐.
        computeOptimumLocation(); // 형제함수 부름. 클로져로서 계속 존재함.
        // this.radius; 
        // defaultLocation;
        
        console.log('draw');
    }
    //네이티브에서 제공하는 함수를 구현하라!!
    Object.defineProperty(this,'defaultLocation',{
        get : function(){// this.getDefaultLocation = function(){.... 보다 더 나은 방법. 밖에서 접근하기도 좋다.
            return defaultLocation;

        },
        set : function(value){
            if(!value.x || !value.y) throw new Error('Invalid location!!');

            defaultLocation = value;
        }
    });
}

```
- 호출해보기.

```javascript

const circle_better = new Circle_better(10);
circle_better.defaultLocation = 1;
circle_better.draw();
//아래로 콘솔에 circle_better를 확인하면
// Circle_better {radius: 10, getDefaultLocation: ƒ, draw: ƒ}
// draw: ƒ ()
// arguments: null
// caller: null
// length: 0
// name: ""
// prototype: {constructor: ƒ}
// [[FunctionLocation]]: index.js:124
// [[Prototype]]: ƒ ()
// [[Scopes]]: Scopes[3]
// **0: Closure (Circle_better) {defaultLocation: {…}, computeOptimumLocation: ƒ}**
// 1: Script {fnCircle: {…}, cnstCircle: ConstructorCircle, funcFuntionObj1: {…}, funcFuntionObj2: undefined, useNativeFuncFunction: ƒ, …}
// 2: Global {window: Window, self: Window, document: document, name: '', location: Location, …}
// getDefaultLocation: ƒ ()
// radius: 10
// defaultLocation: (...)
// get defaultLocation: ƒ ()
// set defaultLocation: ƒ (value)
// [[Prototype]]: Object
// 위처럼 영역(Scope에 객체 내의 지역변수가 있는 것을 확인할 수 있다.)
```

#### 클로저(Closure)
1. 함수 실행이 끝나도 함수 내의 변수에 접근할 수 있다. 사용할 수 있다.
    - 즉, 데이터를 그 영역(scope)내에 가두어 폐쇄한다(closure).
2. 정보에 대한 접근을 제어한다.
3. 모듈화에 유리하다.
   
클로져 == 함수, 그 함수와 그 선언된 환경의 관계.
OR 
클로져 == 현상,  그리고 함수는 그 현상이 일어나기 위한 조건이다.

중요한 것은 클로져를 이용해서 데이터/변수등의 접근을 제어(패쇄,개방)하는 것이 핵심이다.
함수안에 함수가 있다. 편의상 바깥함수, 내부함수라고 하자. 내부함수가, 외부함수의 변수등을 사용한다. 그리고 외부함수 바깥에서 참조한 외부함수의 변수등을 수정할 수 없는 형태이다.

##### 클로저 실습
- 예제 조건
1. start된 상태에서 start를 호출하면 error 'Stopwatch has already started.'
2. stop된 상태에서 stop을 호출하면 error 'Stopwatch has already stopped.'
3. 최초 스탑워치 생성시점 duration 값은 '0'
4. stop이 된 후 duration값은 변경된 값.
5. reset호출했을 때 duration 값은 '0'

```javascript
function Stopwatch_mine(){
    let isActivate = false; 
    let duration = 0;
    let interval = 0;

    let timePlus = setInterval(function(){
        interval += 0.01;
    },10);
    this.start = function(){
        if(isActivate){
            throw new Error('Stopwatch has already started.');
        }
        isActivate = true;
        timePlus;
    };
    
    this.stop = function(){
        if(!isActivate) throw new Error('Stopwatch has already stopped.');
        isActivate = false;
        clearInterval(timePlus);
        duration += interval;
    };

    this.reset = function(){
        if(isActivate){
            this.stop();
        }
        isActivate = true;
        duration = 0;
    };


    Object.defineProperty(this, 'duration', {
        get:function(){return duration;}
    });
};

function Stopwatch_mosh(){
    let startTime, endTime, running, duration =0;
   
    this.start = function(){
        if(running){
            throw new Error('Stopwatch has already started.');
        }
        running = true;
        startTime = new Date();
    };
    
    this.stop = function(){
        if(!running) throw new Error('Stopwatch has already stopped.');
        running = false;
        endTime = new Date();

        const seconds =(endTime.getTime()-startTime.getTime())/1000;
        duration += seconds;
    };

    this.reset = function(){
        startTime = null;
        endTime = null;
        running = false;
        duration = 0;
    };


    Object.defineProperty(this, 'duration', {
        get:function(){
            return duration;
        }
    });

    Object.defineProperty(this, 'startTime', {
        get:function(){
            return startTime;
        }
    });
    Object.defineProperty(this, 'endTime', {
        get:function(){
            return endTime;
        }
    });
};

const sw = new Stopwatch_mine();
console.log(sw.duration);
```

- 테스트 결과

```javascript
0
sw.start()
undefined
sw.stop();
undefined
sw.duration
12.919999999999769
sw.start()
undefined
sw.stop();
undefined
sw.duration
25.839999999999538
sw.stop();
watch.js:17 Uncaught Error: Stopwatch has already stopped.
    at Stopwatch_.stop (watch.js:17)
    at <anonymous>:1:4
Stopwatch_.stop @ watch.js:17
(anonymous) @ VM2847:1
sw.start()
undefined
sw.stop();
undefined
sw.duration
38.75999999999931
sw.start()
undefined
sw.start()
watch.js:10 Uncaught Error: Stopwatch has already started.
    at Stopwatch_.start (watch.js:10)
    at <anonymous>:1:4
Stopwatch_.start @ watch.js:10
(anonymous) @ VM2894:1
sw.stop();
undefined
sw.reset();
undefined
sw.duration
0
```