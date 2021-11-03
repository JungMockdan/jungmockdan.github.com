---
title: "JPQL 문법"

date: 2021-10-18 12:44:27 -0400

categories: jpa

tags: jpa JPQL 문법 syntax

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
- select절에 조회할 대상 지정하는 것(대상 : 엔티티, 임베디드, 스칼라-숫자,문자 명시적으로 컬럼 다 가져오기  [타입이 없다. ]등-)
- jpql로 가져온 엔티티 프로젝션은 모두 영속성 관리 대상이다.
- 'select m.team from Member m' 이런식 보다는 직관적으로 보이게 'select t from Member m join m.team t '와 같이 하는 것이 좋다. 명시적 조인 추천.
- 리턴 : TypedQuery, Query(return:Object[]-배열-),  타입지정이 안되기 때문에 타입캐스팅이 필요하다 'List result = em.createQuery(...) 혹은 List<Object[]> 이렇게 하거나 new Dto.class를 이용하여 List<XxxDto> result = em.createQuery(..., new 풀패키지.XxxDto.class) 할 수 있다.'
- 바로 DTO클래스로 받는것도 가능.

```
List<MemberDto> result = em.createQuery("
  select new dto.MemberDto(m.name, m.age) from Member m
", MemberDto.class).getResultList();// 전체 패키지명 입력 string  이기때문에...
```

### 페이징
- JPA는 페이징을 다음 두 API로 추상화
  - setFirstResult(int startPosition) : 조회 시작 위치(0부터)
  - setMaxResults(int maxResult) : 조회 할 데이터수 (페이지 사이)

```
// 0페이지 데이터 개수는 20 
em.createQuery(...).setFirstResult(0).setMaxResults(20).getResultList(); 
```

### JOIN (조인)
- 종류
  - inner join
  - outer join
  - cross join (카타시안 곱 일어남) 'select m from Member m, Team t where m.name=t.name'
- 조인 대상 필터링
  - on 절 사용 가능 
    - on 절을 써서 left 조인에서 연관관계없이도 그 대상을 조회 할 수 있다. 이때 연관관계인 이퀄조건이 없는 쿼리가 날라가더라
    
### 서브쿼리
```sql
select m from Member m 
where m.age >(서브쿼리)
```
- 지원됨.
- exists, in 모두 사용가능 not 도 가능.
- all, any도 지원함.

```sql
select m from Member m 
where m.age = any(서브쿼리)
```
- 한계
  - where, having 절에서만 사용가능.
  - 단, select에서는 hibernate계만 지원됨. 즉, 거의 다 된다라고 볼수 있음.
  - 단, from절에서는 안됨. 진짜 안됨. 그러니 join으로 풀어 해결하거나, native 쿼리 쓰거나(비추천). 비즈니스로직으로 풀것.
  
### JPQL 타입 표현과 기타식
- 문자는 싱글쿼테이션으로 묶어서, 싱글쿼테이션 표현은 싱글쿼테이션2개를 쓰면됨.
- 숫자는 long 은 L, double 은 D, float 은 F를 숫자뒤에 붙여주면 된다. e.g) 10L
- boolean : TRUE, FALSE
- ENUM : 풀패키지.Enum.value 이런식으로 `type = your.package.Auth.ADMIN` 이런식으로 써야 한다.
- 엔티티 타입 : `TYPE(i) = Book` 이런식으로 쓰여질수 있다. 예로 상속관계에서 Item 테이블을 조회 하는데 그중에 타입이 Book 인 것을 앞선 예 처럼 where 절에서 사용하면 된다.

```sql
select i from Item i 
where type(i) = Book
```

### 조건식(case 등)
- 