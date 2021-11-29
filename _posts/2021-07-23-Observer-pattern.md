---
title: "Observer pattern"
date: 2021-07-23 09:26:28 -0400
categories: design-pattern
tags: gof java design-pattern Observer


# 목차
toc: true  
toc_sticky: true
---
- 변화가 일어났을 때, 미리 등록된 다른 클래스에 통보해주는 패턴
- 실 사용 예 : Event listener, swing, jwt..etc,.

## 예제코드 (버튼의 이벤트를 전달받아 이벤트를 처리함.)

### 리스너 인터페이스와 객체
```java
public interface IButtonListener {
   void clikcEvent(String event);
}
```

```java
public class Button {

    private Stirng name;
    private IButtonListener buttonListener;
    
    public Button(String name){
        this.name = name;
    }
    
    public void click(String event){
    	buttonListener.clickEvent(event);
    }
    
    public void addListener(IButtonListener buttonListener){
        this.buttonListener = buttonListener;
    }
}
```
### 실행코드
- 옵저버 패턴 적용
```java
public class Main {

    public static void main(String[] args){
        Button button = new Button("버튼");
        // 버튼에 리스너 추가 - 인터페이스 구현(익명클래스로 바로 구현함)
        button.addListener(new IButtonListener() {
            @Override
            public void clikcEvent(String event){
                System.out.println(event);  
            }
        });
        
        //addListener의 clikcEvent()를 호출하게 된다.
        button.click("메세지 전달 : click 1");
        button.click("메세지 전달 : click 2");
        button.click("메세지 전달 : click 3");
        button.click("메세지 전달 : click 4");
    }
}
```





