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
- 식별자가 없다.

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

## 값 타입과 불변 객체
- 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념으로 단순하고 안전하게 다룰수 있어야 함.
- 임베디드 타입을 여러 엔티티에서 공유하면 위험?! 부작용발생!!!
- 공유하고 싶은 것이라면 엔티티를 사용할 것. 값타입 공유 금지. 복사해서 사용할 것. 새로운 객체로.
- 해결책 : 복사해서 사용해야 한다. 그런데 개발자의 실수는? 막을 수 없다. 그것이 **객체값타입의 한계**이다.
    - 기본타입의 경우 디폴트가 복사이지만, 객체타입은 참조가 전달 되기 때문에 유념하여 의식적으로 복사를 해야 한다. 
    - 불변객체(immutable object)로 설계해야한다. ex) setter 제공하지 않을 것. 생성자로만 초기화.
    - 불변객체의 값은 어떻게 바꿀수가 있나? 임베디드 객체 전체를 새로운 객체로 set 하는 것이 맞다.
    - 제약은 있지만 큰 위험/부작용을 예방할 수 있다.

```java
class Test {
  public static void main(String[] args) {
    EntityManagerFactory emf = Persistance.createEntityManagerFactory("learnJPA");
    EntityManager em = emf.createEntityManager();
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    try {
        Address address = new Adress("Seoul","gazawh","12345");
        
        Member member1 = new Member("member1",address);
        em.persist(member1);
        Member member2 = new Member("member2",address);
        em.persist(member2);
        // 다른 트랙잭션이라 가정하자..여기서부터..
        // member1의 address를 변경해보자!=> 결과는? member2까지 같이 바뀐다.
        member1.getAddress().setCity("Busan");
        // 해결책 : 복사해서 사용해야 한다. 그런데 개발자의 실수는? 막을 수 없다. 
        
        em.flush();
        em.clear();
        tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }
    emf.close();
  }
}
```

## 값 타입의 비교
- 동일성 비교 vs **동등성 비교**
- equals를 적절한 커스터마이징-대개 모든 필드-을 통해 이용하여 **동등성을 비교**하여야 한다.


## 값 타입 컬렉션
- @ElementCollection 값타입컬렉션임을 선언
- @CollectionTable 테이블을 매핑함. 옵션 테이블명 엔티티 조인컬럼 명시함.
  `@CollectionTable(name="FAVORITE_FOOD", joinColums = @JoinColumn(name="MEMBER_ID"))`
- 테이블 매핑은 되지만 엔티티로 선언되지 않으며, id가 아니라 복합키 형식으로 저장된다.
- 생명주기의 의존성이 엔티티에 있다.
- 컬렉션테이블은 디폴트가 LAZY 지연로딩이다.
- 수정의 경우 역시나 값타입이기 때문에 수정을 위해서는 아예 객체를 바꿔끼우고, 
- 삭제후 삽입 전략 등을 이용할 수 밖에 없다. 삭제하기 위해서는 정확히 같은 객체를 찾기 위해서 equals 메소드가 잘 정의되어 있어야 한다.
- 값타입컬렉션 그럼 어디 쓰는거지? 정말정말진짜 간단한것.
  - 판단 : 식별자가 필요하고, 값을 추적/변경 해야 한다면, 그것은 entity로 관리되어야 한다. 그렇다면, 값타입은 안돼!!!

```java
@Entity
class Member{
    @Id @GenerateValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    @Embeded
    private Period period;
    
    @Embeded
    private Address address;

  @ElementCollection
  @CollectionTable(name="FAVORITE_FOOD", joinColums = @JoinColumn(name="MEMBER_ID"))
  private Set<String> favoriteFood = new HashSet<>();
  
  @ElementCollection
  @CollectionTable(name="ADDRESS", joinColums = @JoinColumn(name="MEMBER_ID"))
  private List<Address> addressHistory = new ArrayList<>();
    
}


class Test {
  public static void main(String[] args) {
    EntityManagerFactory emf = Persistance.createEntityManagerFactory("learnJPA");
    EntityManager em = emf.createEntityManager();
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    try {
      Address address = new Adress("Seoul","gazawh","12345");

      Member member = new Member("member1",address);
      member.getFavoriteFoods().add(new HashSet<>(Arrays.asList(("치킨","피자","떡볶이")));
      member.getAddressHistory().add(new Adress("Seoul","old1","12345"));
      member.getAddressHistory().add(new Adress("Seoul","old2","12345"));
      member.getAddressHistory().add(new Adress("Seoul","old3","12345"));
      em.persist(member);//결과는 member, FAVORITE_FOOD, ADDRESS 세개의 테이블에 모두 persist 된다. 즉, 모든 데이터가 memeber 엔티티에 의존한다.
     
      em.flush();
      em.clear();
      
      Member findMember = em.find(Member.class, member.getId());//요기까지는 컬렉션값타입에 대한 쿼리는 나가지 않는다. 즉, 지연로딩이 디폴트이다.
      findMember.getAddressHistory().remove(new Adress("Seoul","old1","12345"));
      findMember.getAddressHistory().add(new Adress("Seoul","new1","12345"));
      //-> 테이블에서 주인엔티티 관계 데이터 모두 삭제 후 영속성에 남아있는 데이터를 모두 insert 한다.
      //---> 이거 써도 되는 건가? 복잡성과 오류 발생가능성때문에 사용하지 말자.
      //-----> 완전 다르게 접근하자. 값 타입 컬렉션 대신 일대다 관계를 고려하자.
      // 참고) 아래 AddressEntity 클래스 처럼.
      

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }
    emf.close();
  }
}

@Entity
@Table("ADDRESS")
class AddressEntity{
  @Id @GenerateValue
  @Column(name="Address_ID")
  private Long id;
  
  private Address address;//값타입
  
  public AddressEntity(String city, String street, String zipcode){
      this.address = new Address(city, street, zipcode);
  }
}

/* 
 * 값 타입 컬렉션 대신 일대다 관계를 고려하여 AddressEntity가 있을 경우 Member엔티티
 */
@Entity
class Member { 
    // 생략
  @OneToMany(cascade = CascadeType.All, orphanRemoval = true)
  @JoinColum(name="MEMBER_ID")
  private List<Address> addressHistory = new ArrayList<>();
    
    // 생략
}
```
