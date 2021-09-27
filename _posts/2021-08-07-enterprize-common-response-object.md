---
layout: post
sitemap :
    changefreq : daily
    priority : 1.0
title: "공통응답객체.md"
date: 2021-08-07 16:52:39 -0400 
categories: design
tags: java design

# 목차

toc: true  
toc_sticky: true
---
# 상용에서 공통응답 객체의 설계
ResponseEntity가 있어도 실제 그 사용에 한계를 두고, 예상할 수 있는 응답을 원한다.
예를 들면 실제 httpstatus 응답은 언제나 200(ok)을 주고, 실제 결과는 응답 페이로드 안에 코드와 메세지로 따로 리턴하는 식으로 사용하는 경우가 정말 많다.

case 1 : 상속구조

case 2 : 제네릭을 이용한 구조
```java
@NoArgsConstructor
@AllArgsConstructor
@Data
public class CmmResponse<T>{
    private ResultInfo resultInfo;
    private T resultBody;
    
    @NoArgsConstructor
    @AllArgsConstructor
    @Data
    public static class ResultInfo{
        @NotNull
        private String resultCode;
        private String resultMessage;
        private ErrorResponse errorResponse;
    }
}
```