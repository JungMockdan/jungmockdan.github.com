---
title: "JPQL 기본문법"

date: 2021-10-18 12:44:27 -0400

categories: jpa

tags: jpa JPQL 기본문법 syntax

# 목차

toc: true  
toc_sticky: true
---

## 기본문법
- JPA에서 사용하는 객체지향 SQL
- JPQL로 작성하면 ANSI SQL(표준 SQL)로 번역된다.
- 쿼리 작성 규칙은 거의 SQL과 동일하지만, 테이블대신 객체를 사용하고, 조회 컬럼대신 객체 자체만 선언해도 된다.
    - e.g `"Select m from Member m where m.name like '%xxx%' "`

## 한계 및 단점
- 단순 문자열이기 때문에 발생할 수 있는 오류. 심지어 빌드시에도 발생하지 않는 오류가 있다.
- 동적쿼리를 만들 때, 스트링을 분기하는 등의 컨트롤이 오류를 발생하기 좋다.

### 프로젝션
- select절에 조회할 대상 지정하는 것(대상 : 엔티티, 임베디드, 스칼라-숫자,문자 등-)
