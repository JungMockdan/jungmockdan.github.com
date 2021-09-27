---
layout: post
sitemap :
    changefreq : daily
    priority : 1.0
title: "Adapter pattern"
date: 2021-07-20 13:26:28 -0400
categories: design-pattern
tags: gof java design-pattern Adapter
# 목차
toc: true  
toc_sticky: true
---

- 호환성이 없는 기존 클래스의 인터페이스를 변환하여 재사용 할 수 있도록 한다. SOLID  중에서 개방패쇄 원칙(OCP)를 따른다.
- 100 V를 220 V로 바꾸는 변환기를 생각하면 이해하기 쉽다.
## 예제코드 (전압 변환 플러그 어댑터)

### 110V 인터페이스와 제품
```java
public interface Electronic110V {
    void powerOn();
    
}
```

```java
public class HairDryer implements Electronic110V {
    @Override
    public void powerOn(){
    	System.out.println("헤어 드라이어 110v on");
    }
    
}
```
### 220V 인터페이스와 제품
```java
public interface Electronic220V {
    void connect();
    
}
```
```java
public class AirConditioner implements Electronic220V {
    @Override
    public void powerOn(){
    	System.out.println("에어컨 220v on");
    }
    
}
```

```java
public class VacuumCleaner implements Electronic220V {
    @Override
    public void powerOn(){
    	System.out.println("청소기 220v on");
    }
    
}
```
### 실행코드
```java
public class Main {
   
    public static void main(String[] args){
    	//110v
    	HairDryer hairDryer = new HairDryer();
    	connect(hairDryer);
    	
    	//220V
    	VacuumCleaner cleaner = new VacuumCleaner();
    	//connect(cleaner);// adater없이는 오류
    	Electronic110V cleanerAdapter = new SocketAdapter(cleaner);
    	connect(cleanerAdapter);
    	
    	AirConditioner airConditioner = new AirConditioner();
    	Electronic110V airConditionerAdapter = new SocketAdapter(airConditionerAdapter);
    	connect(adapter);
    }
    
    // 콘세트 
    public static void connect(Electronic110V electronic110V){
    	electronic110V.powerOn();
    }
}
```

### 어댑터 구현
```java
public class SocketAdapter implements Electronic110V {
	private Electronic220V electronic220V;
	
	public SocketAdapter(Electronic220V electronic220V){
		this.electronic220V = electronic220V;
	}
    @Override
    public void powerOn(){
    	electronic220V.connect();
    	System.out.println("220v on");
    }
    
}
```



