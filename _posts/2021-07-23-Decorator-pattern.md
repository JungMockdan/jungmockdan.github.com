---
layout: post
sitemap :
    changefreq : daily
    priority : 1.0
title: "Decorator pattern"
date: 2021-07-23 08:26:28 -0400
categories: design-pattern
tags: gof java design-pattern Decorator
# 목차
toc: true  
toc_sticky: true
---

- 상속의 대안으로 활용
- 기존 클래스는 유지하고, 이후 필요한 형태로 꾸밀 때 사용
- SOLID 중에서 개방패쇄원칙(OCP)와 의존역전원칙(DIP)을 따름
- 에스프레소 + 다른 재료 = 다른 커피
## 예제코드 (자동차 등급에 따라 가격이 달라짐)

### 인터페이스와 자동차들
```java
public interface ICar {
    int getPrice();
    void showPrice();
}
```

```java
public class Audi implements ICar {

    private int price;
    public Audi(int price){
        this.price = price;
    }
    @Override
    public int getPrice(){
        return price;
    }
    @Override
    public void showPrice(){
    	System.out.println("audi의 가격은 "+ this.price+"원 입니다.");
    }
    
}
```
### 실행코드
- 데코레이션 패턴 적용전
```java
public class Main {

    public static void main(String[] args){
    	ICar audi = new Audi(1000);
    	audi.showPrice();
    	// 데코레이션 패턴 적용전으로 1000 원이 출력됨.
    }
}
```

### 데코레이터  패턴 구현
#### Decorator
```java
public class AudiDecorator implements ICar {
    protected ICar audi; // 인터페이스를 받아서 확장성을 가짐.
    protected String modelName;
    protected int modelPrice;
    
    public AudiDecorator(ICar audi, String modelName, int modelPrice){
        this.audi = audi;
        this.modelName = modelName;
        this.modelPrice = modelPrice;
    }
    
    @Override
    public int getPrice(){
        return this.audi.getPrice()+this.modelPrice;
    }
    @Override
    public void showPrice(){
        System.out.println(this.modelName+"의 가격은 "+ getPrice() +"원 입니다.");
    }
    
}
```

#### Decorator 상속받은 모델 객체
##### A3
```java
public class A3 extends AudiDecorator{
    public A3(ICar audi, String modelName){
        super(audi, modelName, 1000);
    }
}
```        
##### A4
```java
public class A4 extends AudiDecorator{
    public A4(ICar audi, String modelName){
        super(audi, modelName, 2000);
    }
}
``` 
##### A5
```java
public class A5 extends AudiDecorator{
    public A5(ICar audi, String modelName){
        super(audi, modelName, 3000);
    }
}
```

### 실행코드
- 데코레이션 패턴 적용 후
```java
public class Main {
   
    public static void main(String[] args){
    	ICar audi = new Audi(1000);
    	audi.showPrice();
    	// 데코레이션 패턴 적용전으로 1000 원이 출력됨.
        
        // 등급 a3
        ICar a3 = new A3(audi,"A3");
        a3.showPrice();
        // 등급 a4
        ICar a4 = new A4(audi,"A4");
        a3.showPrice();
        // 등급 a5
        ICar a5 = new A5(audi,"A5");
        a3.showPrice();
    }
    
    
}
```



