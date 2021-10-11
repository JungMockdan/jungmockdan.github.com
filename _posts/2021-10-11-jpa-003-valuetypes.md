---
title: "JPA 값 타입"

date: 2021-10-11 09:26:44 -0400

categories: jpa

tags: JPA 값타입

# 목차

toc: true  
toc_sticky: true
---

- jpa 타입은 크게 2가지 엔티티 타입과 값 타입임.

## 기본값 타입


## 임베디드 타입(복합 값 타입)
- 여전히 엔티티의 생명주기에 의존함.
- @Embeddable   :  값 타입을 정의하는 곳에 표시
- @Embeded : 값 타입을 사용하는 곳에 표시
- 기본생성자 필수!!

### 장점
- 재사용
- 높은 응집도
- Period.isWork() 처럼 해당 값 타입만 사용하는 의미있는 메소드를 만들 수 있음.

```java
@Entity
class Member{
    @Id @GenerateValue
    private Long id;
    
    @Embeded
    private Period period;
    
    @Embeded
    private Address address;
}

@Embeddable
class Period{
    private LocalDate startDate;
    private LocalDate endDate;
    public boolean isWork(){
        return true;
    }
}

@Embeddable
class Address{
    private String zipcode;
    private String city;
    private String street;
}

```

### 임베디드 타입과 테이블 매핑
- 여전히 값타입!!!
- 테이블설계상 다른 점 없다. 똑같다.
- 객체와 테이블을 아주 세밀하게(fine-grained) 매핑하는 것이 가능
- 잘 설계한 ORM  애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음???
- 임베디드 타입은 엔티티를 멤버로 가질 수 있다?!
!!!  todo

```java
@Entity
class Member{
    @Id @GenerateValue
    private Long id;
    
    @Embeded
    private Period period;
    
    @Embeded
    private Address address;
    @Embeded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE")),
            @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET"))
    })
    private Address work_address;
}
```

### 임베디드 타입과 테이블 매핑

## 값 타입과 불변 객체


## 값 타입의 비교


## 값 타입 컬렉션


