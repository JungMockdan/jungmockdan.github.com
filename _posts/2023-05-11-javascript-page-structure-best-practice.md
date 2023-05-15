---
title: "모듈화 유지보수용이 데이터노출 최소화 javascript 구조"
date: 2023-05-11 13:26:28 -0400
categories: javascript
tags: design-pattern javascript


# 목차
toc: true  
toc_sticky: true 
---

## 모듈패턴

### 예시1

```javascript
var pageQM = (function() {
  // 전역으로 사용할 변수를 선언합니다.
  var defaultValues = {},
      events = {},
      initialized = false;

  // 초기화 함수
  function init() {
    if (initialized) {
      return;
    }

    setDefaultValues();
    setEvents();

    initialized = true;
  }

  // 기본값 설정 함수
  function setDefaultValues() {
    // ...
  }

  // 이벤트 설정 함수
  function setEvents() {
    // ...
  }

  // 외부에 노출할 API를 반환합니다.
  return {
    init: init
  };
})();

pageQM.init();//실행
```

### 예시2

아래는 모듈 패턴의 예시입니다.

```javascript

// 모듈 패턴을 사용한 코드 구조 예시
var myModule = (function() {
  // 내부 private 변수와 함수
  var privateVariable = "I am a private variable";
  function privateFunction() {
    console.log("I am a private function");
  }

  // 외부로 공개할 public API
  return {
    publicVariable: "I am a public variable",
    publicFunction: function() {
      console.log("I am a public function");
      // 내부 private 함수 사용 가능
      privateFunction();
    }
  };
})();

// 모듈 사용 예시
console.log(myModule.publicVariable); // "I am a public variable"
myModule.publicFunction(); // "I am a public function"와 "I am a private function" 출력
위 코드에서 myModule 변수는 즉시 실행 함수로 선언되며, 반환된 객체에 대한 참조를 저장합니다. 내부에는 private 변수와 함수, public API가 정의되어 있습니다. private 변수와 함수는 외부에서 접근할 수 없으며, public API만을 통해 접근할 수 있습니다.
```

이러한 모듈 패턴을 사용하면 전역 네임스페이스를 오염시키지 않고도 모듈 간의 의존성 관리와 코드 구조화를 용이하게 할 수 있습니다. 또한, 모듈화를 통해 코드의 재사용성과 유지보수성을 높일 수 있습니다.

## 확장된 모듈 패턴:revealing module pattern

### 예시

```javascript
var module = (function() {
  var privateVar = 'This is private';
  var publicVar = 'This is public';

  function privateFunction() {
    console.log(privateVar);
  }

  function publicFunction() {
    privateFunction();
  }

  return {
    publicVar: publicVar,
    publicFunction: publicFunction
  };
})();

console.log(module.publicVar); // This is public
module.publicFunction(); // This is private

```

위의 코드에서는, var module은 즉시 실행 함수를 할당하는 변수입니다. 이 함수는 두 개의 privateVar와 publicVar 변수와 두 개의 함수 privateFunction과 publicFunction을 선언합니다.

모듈은 return문에서 공개할 인터페이스를 정의합니다. 이 경우, publicVar와 publicFunction이 외부에서 접근 가능합니다. publicFunction 내부에서는 privateFunction을 호출하며, 이 함수는 내부에서만 사용할 수 있는 privateVar에 액세스합니다.

이렇게하면, 모듈의 내부 로직은 외부에서 접근할 수 없으므로, 코드의 안정성과 보안성을 높일 수 있습니다. 동시에 모듈의 공개 인터페이스를 통해 필요한 부분에 대해서만 접근할 수 있기 때문에, 코드의 복잡성을 감소시킬 수 있습니다.

## 싱글톤 패턴

### 예제코드

싱글톤 패턴(Singleton Pattern)은 어떤 클래스가 최초 한 번만 메모리를 할당하고 그 메모리에 인스턴스를 만들어 사용하는 디자인 패턴입니다.

이 패턴의 핵심은 클래스 인스턴스를 하나만 생성하는 것입니다. 이를 통해 메모리 사용을 최적화할 수 있고, 코드 내에서 전역적으로 접근 가능한 객체를 만들 수 있습니다.

```javascript
const Singleton = (function () {
  let instance;

  function createInstance() {
    const object = new Object({name: "Singleton Object"});
    return object;
  }

  return {
    getInstance: function () {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();

console.log(instance1 === instance2); // true

```

위 코드에서는 Singleton이라는 변수에 즉시실행함수를 할당하여, 클로저를 이용하여 instance 변수를 private하게 선언합니다. createInstance 함수는 인스턴스를 생성하는 함수로서, new Object()를 사용하여 객체를 생성하고, 이 객체를 반환합니다.

그리고 getInstance 함수는 싱글톤 패턴에서 가장 중요한 함수로, 인스턴스를 반환합니다. getInstance 함수는 처음 호출되었을 때, instance 변수가 null이므로 createInstance 함수를 호출하여 인스턴스를 생성하고, 이후부터는 instance 변수가 null이 아닌 경우에는 이미 생성된 인스턴스를 반환합니다.

위 코드에서 instance1과 instance2는 동일한 인스턴스를 참조하고 있으므로, true를 출력합니다. 이처럼 싱글톤 패턴을 이용하면 하나의 인스턴스를 여러 곳에서 참조하여 사용할 수 있습니다.