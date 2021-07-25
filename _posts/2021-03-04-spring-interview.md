---
title: "스프링 인터뷰 (2014)"
date: 2021-03-04 13:26:28 -0400
categories: interview
tags: spring interview backend 커리어
# 목차
toc: true  
toc_sticky: true 
---


# `SPRING` OVERVIEW 
- [코드긱스 링크](https://www.javacodegeeks.com/spring-interview-questions-and-answers.html)
1. 스프링은 무엇인가?
  - 스프링은 상용자바-엔터프라이즈자바-기반의 오픈포스 개발 프레임워크. ...

2. 스프링의 장점은 무엇인가
  - 가볍다.
  - 제어역전  ioc
  - 관점지향 aop
  - 컨테이너 : 스프링은 어플리케이션 객체의 설정과 라이프사이클을 포함하고 그것을 관리할 수 있다.
  - 트랜잭션의 관리
  - 예외 처리

3. 스프링 모듈에서 는 어떤것들이 있을까?
스프링은 몇개의 일반화된 계층?에 모두 20개가 넘는 모듈이 있다. 
  - Core Container : 스프링의 중심/핵심... 
    - 코어
    - 빈
    - 컨텍스트
    - el - expression language
  
  - Data Access/Integration : 데이터 베이스와 소통하는 역할.
    - JDBC module
    - Object-Relational Mapping (ORM) module
    - Java Messaging Service (JMS) module
    - Object XML Mappers (OXM) module
    - Transaction Management module
  
  - Web : 웹어플리케이션 생성을 지원하는 계층
    - Web module
    - Web-MVC module
    - Web-Socket module
    - Web-Portlet module

  - AOP (Aspect Oriented Programming) : 코드 분리를 위해 `Advices, Pointcuts`등을 사용할 수 있는 계층
  - Instrumentation : 클래스 계측과 클래스로더 구현을 지원하는 계층
  - Test : JUnit과 TestNG로 테스트할 수 있도록 지원하는 계층
  - Messaging : `STOMP-스트리밍 텍스트 지향 메시지 프로토콜` 을 지원하는 모듈. WebSocket 클라이언트에서 STOMP 메시지를 라우팅하고 처리하는 데 사용되는 annotaion 프로그래밍 모델을 지원.
  - Aspects : AspectJ와의 통합을 지원을 제공하는 모듈

4. 코어컨테이너-Application 컨텍스트-모듈을 설명해라.

# `DI(Dependency Injection)`
1. 
# `Spring Annotations`
1. 
# `Spring Data Access`
1. 
# `Spring Aspect Oriented Programming (AOP)`
1. 
# `Spring Model View Controller (MVC)`
1. 
# `Authentication and authorization`
1. 
