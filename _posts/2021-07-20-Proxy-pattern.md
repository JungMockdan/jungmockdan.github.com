---
title: "Proxy pattern"
date: 2021-07-20 13:26:28 -0400
categories: design-pattern
tags: gof java design-pattern


# 목차
toc: true  
toc_sticky: true
---

- proxy 뜻 : 대리인, 무언가를 대신 처리하는 것.
- Proxy Class 를 통해서 대신 전달하는 형태로 설계되며, 실제  Client 는 Proxy로 부터 결과를 받는다.
- Cache 기능, 스프링의 AOP등이 PROXY 패턴의 예이다.
-  SOLID 중에서 개방폐쇄 원칙(OCP)과 의존 역전 원칙(DIP)을 따른다.
## 예제1 (Cache )

- 브라우져 등에서 캐싱해두고 같은 요청이 있을 때, 값을 준다.
### 프록시 적용되지 않음

```java
public interface IBrowser{
    Html show();
}
```
```java
public Html implements IBrowser{
    private String url;
    public Html(){
    	this.url = url;
    }
}
```

```java
public Browser implements IBrowser{
    private String url;
    public Browser(String url){
    	this.url = url;
    }
    
    @Override
    public Html show(){
    	System.out.println("Browser loading html from"+url);
    	return new Html(url);
    }
}
```



#### 실행코드
```java
public class Main {
   
    public static void main(String[] args){
    	Browser browser = new Browser("www.naver.com");
    	browser.show();
    	browser.show();
    	browser.show();///3번 모두 로딩한다. 아래로 캐싱 지원되는 프록시를 생성하여 진행해 보자.
    	
    }
    
   
}
```
### 프록시 패턴 적용
```java
public BrowserProxy implements IBrowser{
    private String url;
    private Html html;
    public BrowserProxy(String url){
    	this.url = url;
    }
    
    @Override
    public Html show(){
    	if(html==null){
			this.html = new Html(url);
            System.out.println("BrowserProxy loading new html from"+url);
    	}
    	System.out.println("BrowserProxy using cached html");
    	return html
    }
}
```

#### 실행코드
```java
public class Main {
   
    public static void main(String[] args){
    	.
    	.
    	.
    	
    	IBrowser proxyBrowser = new BrowserProxy();
    	browser.show();// new
    	browser.show();// cached
    	browser.show();
        browser.show();
        browser.show();
    }
    
   
}
```
## 예제2 
(스프링 AOP 유사기능... 예제1 클래스 연결)
- HttpClient, restclient의 소요시간 확인
- 트랜잭션에 걸리는 소요타임 확인
- 특정 서비스, 메소드등의 소요시간을 확인하여 어떤 서비스때문에 서버가 느린지 확인할 때 유용.

### AOP기능 구현
```java
public AopBrowser implements IBrowser{
    private String url;
    private Html html;
    private Runnable before;
    private Runnable after;
    
    public AopBrowser(String url, Runnable before, Runnable after){
    	this.url = url;
    	this.before = before;
    	this.after = after;
    }
    
    @Override
    public Html show(){
    before.run()
    	if(html==null){
			this.html = new Html(url);
            System.out.println("AopBrowser html loading from"+url);
            try{
            	Thread.sleep(1000);
            }catch(){
            
            }
    	}
    	System.out.println("AopBrowser using cached html");
    	return html
    }
    after.run()
}
```

#### 실행코드
```java
public class Main {
   
    public static void main(String[] args){

		AtomicLong start = new AtomicLong();
		AtomicLong end = new AtomicLong();
    	IBrowser aopBrowser = new AopBrowser("www.naver.com".
            ()->{
				System.out.println("before");
				start.set(System.currentTimeMillis());
            },
            ()->{
            	long now = System.currentTimeMillis
            	end.set(now - start);
				System.out.println("after");
            }
    	);
    	
    	
    }
    // 첫번째 호출
    aopBrowser.show();
    System.out.println("1st - loading time : "+ end.get());
   
   // cache된 2번째 호출
    aopBrowser.show();
    System.out.println("2nd - loading time : "+ end.get());
}
```