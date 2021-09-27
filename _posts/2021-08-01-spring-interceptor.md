---
layout: post
sitemap :
    changefreq : daily
    priority : 1.0
title: "2021-08-01-spring-interceptor.md"
date: 2021-08-01 10:02:24 -0400 
categories: spring 
tags: spring interceptor

# 목차

toc: true  
toc_sticky: true
---
filter : logging
interceptor : 인증
aop aspectj :???


SpingContext에 등록됨. interceptor를 구현하여 component를 만들고  컨피겨레이션 만들어서 해당 인터셉터를 등록함. 
interceptor에서 특정  url에 대해서 검사하거나, 반대로 특정url은 검사에 제외하는 방법을 사용하거나, 커스텀 annotation을 작성하여interceptor가 등록된 configuration에서 해당 annotation을 검사하는 방법으로 세션을 검사하는 즉, 인증의 과정으로 사용될 수 있다. 마찬가지로 에러에 대해서는 advice에서 처리해 줄수 있도록 인증실패시 Exception을 던져 처리될 수 있도록 활용하면 된다.