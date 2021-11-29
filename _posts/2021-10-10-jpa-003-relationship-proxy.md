---
title: "JPA 프록시와 연관관계 관리"

date: 2021-10-10 14:16:08 -0400

categories: jpa

tags: JPA 프록시 연관관계 hibernate proxy cascade

# 목차

toc: true  
toc_sticky: true
---
## 프록시
- 왜 써야 하는가? 멤버와 팀 테이블이 있는데, 멤버 조회시에 항상 팀을 조회 해야 할까? 이런 고민을 하란말.
- 멤버만 쓰는 경우 VS 거의 대부분 팀까지 같이 사용해야 하는 경우  이 2CASE가 혼재되었다면?
- EntityManager.getReference() : 프록시 객체를 가져온다...대리 객체. 쿼리는 날라가지 않는다. 사용시점에 쿼리를 날린다.  원래클래스명$HibernateProxy$xxXx 이런 클래스명을 가진다...즉, 하이버네이트에 의해 만들어진 가짜 클래스임.
- 실제 사용은 거의 하지 않으나 프록시의 메커니즘에 대한 이해는 꼭 필요하다.
### 특징
- 프록시 객체는 실제 객체의 참조(target)를 보관
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출함.
- 한 번만 초기화 최초에만
- 프록시 객체가 실제 엔티티로 바뀌는게 아니라 접근 가능한 상태만 되는 것임. 
- 프록시 객체는 원본 엔티티를 상속받음. 따라서 타입체크시에 `==`비교는 실패, 대신  `instanceOf`를 사용할 것.
- 영속성 컨텍스트에 찾는 엔티티(id로 구분하기때문에 같은 인덱스가 있다 그럼 같다고 보는데)가 있으면, em.getRefernce()로 호출해도 프록시객체가 아닌 실제 엔티티를 반환한다. 왜냐하면 jpa는 영속성내의 동일idx  엔티티에대해 동일성을 보장하기 때문이다.

```java
class Test {
    public static void main(String[] args) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
            Member member1 = new Member("member1");
            Member member2 = new Member("member2");
            em.persiste(member1);
            em.persiste(member2);
            em.flush();
            em.clear();
            
            Member m1 = em.find(Member.class, member1.gerId());
            Member m2 = em.find(Member.class, member2.gerId());
            System.out.println("m1.getClass()==m2.getClass() = " + m1.getClass() == m2.getClass());//->true 당연한 결과.
            Member mRef1 = em.find(Member.class, member1.gerId());
            Member mRef2 = em.find(Member.class, member2.gerId());
            System.out.println("m1.getClass()==mRef1.getClass() = " + m1.getClass() == mRef1.getClass());//->false mRef객체는 프록시객체임.
            System.out.println("m1.getClass()==mRef1.getClass() = " + m1.getClass() == mRef2.getClass());//->false mRef객체는 프록시객체임.
            // 그래서 == 비교를 하면 안된다. 어떤 객체로 오는지 알수 없음. 즉, instanceof 를 사용해서 비교해야 한다. 상속관계이기 때문에 비교가 가능하다.
            System.out.println("m1 instanceof mRef2 = " + (m1 instanceof mRef2) );//->false mRef객체는 프록시객체임.
            
            
            // 항상 같은 index 객체임을 보장한다.
            // 프록시를 먼저 조회하면 뒤에 find라고 해도 프록시 객체를
            // find를 통해서 조회하면 뒤에 reference를 이용해 프록시를 호출하려고 해도 실제 엔티티를 반환한다.
            /* member 객체 */
            Member findMember1 = em.find(Member.class, member1.gerId());
            Member refMember1 = em.getReference(Member.class, member1.gerId());
            System.out.println("findMember1.getClass() = " + findMember1.getClass());// Member.class
            System.out.println("refMember1.getClass() = " + refMember1.getClass());//Member.class
            /* member 객체의 프록시 객체 */
            Member refMember2 = em.getReference(Member.class, member1.gerId());
            Member findMember2 = em.find(Member.class, member1.gerId());
            System.out.println("refMember1.getClass() = " + refMember1.getClass());//Member.$HibernateProxy$xxXx
            System.out.println("findMember1.getClass() = " + findMember1.getClass());// Member.$HibernateProxy$xxXx        
            
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            tx.close();
        }
            
    }
}

```

- 준영속 상태-영속성컨텍스트 도움 받을 수 없음(detach, clear, close)-일 때, 프록시를 초기화하면? : org.hibernate.LazyInitializationException 터짐. 이유는? 영속성컨텍스의 도움 받을 수 없기 때문에.

```
//초기화 여부 확인
emf.getPersistanceUnitUtil().isLoaded(객체 ex: refMember);//return true or false

//강제초기화 : 준영속상태에 있을 때 강제로 초기화 하면서 쿼리가 날라감.
// JPA 는 강제표준화 없음.
Hibernate.initialize(refMember);
```

## 즉시 로딩과 지연 로딩
- 지연로딩 (`FetchType.Lazy`) : 연관관계에 있는 엔티티는 프록시객체로 조회한다.

```java

@Entity
class Member{
    //. 생
    //. 략
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name="TEAM_ID")
    private Team team;
}

class Test {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistance.createEntityManagerFactory("learnJPA");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        try {
            Team team = new Team("team");
            Member member = new Member("member",team);
            
            em.persiste(member);
            em.flush();
            em.clear();
            
            Member m = em.find(Member.class, member.getId());
            
            System.out.println("team =  " + m.getTeam().getClass());//Team.$HibernateProxy$xxXx //여기까지는 프록시 객체기 때문에 쿼리가 날라가지는 않는다.
            System.out.println("===============");
            m.getTeam().getName();// 여기서 team 조회 쿼리 날라감.
            System.out.println("===============");
            
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

- 즉시로딩(`FetchType.EAGER') : find 메소드쓰면, 연관관계에 있는 테이블까지 조인해서 가져온다. 
  - 구현체에 따라 다르겠지만 한번에 조인으로 하는 경우가 다수이고, 테이블당 쿼리를 분리해서 날리는 경우는 거의 없다.
  
### 활용 주의 사항-즉시로딩(`FetchType.EAGER`)
- 실무에서는 EAGER 쓰지말자.
- 즉시 로딩 적용시 예상치 못한 SQL이 발생
- JPQL에서 N+1 문제를 일으킴

```java
//예를 들어 Member 테이블에 만개가 있다. 즉 만개의 회원이 있다. 모든 회원은 팀을 가지고 있다. 
List<Member> members = em.createQuery("select m from Member m").getResultList();

//위 코드 실행시 EAGER로 설정된 경우, 결과로 나온 10개의 회원의 TEAM_ID로 TEAM 테이블에 대해 조회가 일어난다, 즉, 10번의 쿼리가 더 날라가게 된다. 그래서 최초의 쿼리1회에 데이터 결과 N으로 N+1의 문제를 일으킨다고 한다.

// 위 문제를 해결하기 위해서는 fetchjoin : 동적으로 join을 걸어서 사용할 수 있다.  다른 방법도 있음 어노테이션(엔티티그래프) 등....
List<Member> members = em.createQuery("select m from Member m left join fetch m.team").getResultList();
```

- @ManyTo**One**, @OneTo**One** default 값 : EAGER 즉, LAZY 로 명시해야 함.
- @OneTo**Many**, @ManyTo**Many** default 값 : LAZY.

## 영속성 전이 : CASCADE
- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
- e.g : 부모 엔티티를 저장할 때, 자식 엔티티도 같이 저장하고 싶다. 
```java
@Data
@Entity
class Parent{
    
    @Id @GeneratedValue
    @Column(name="parent_id")
    private Long id;

  @OneToMany(mappedBy = "parent", cascade= CascadeType.ALL)//
  private List<Child> childList = new ArrayList<>();
    
    public void addChild(Child child){
        childList.add(child);
        child.setParent(this);
    }
}

@Data
@Entity
class Child{

  @Id @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name="parent_id")
  private Parent parent;
}

class Test {
  public static void main(String[] args) {
    EntityManagerFactory emf = Persistance.createEntityManagerFactory("learnJPA");
    EntityManager em = emf.createEntityManager();
    EntityTransaction tx = em.getTransaction();
    tx.begin();
    try {
      Parent parent = new Parent();
      Child child1 = new Child();
      Child child2 = new Child();
      /* cascade가 안걸린상태라면 */
      em.persiste(member);
      em.persiste(child1);
      em.persiste(child2);
      /*=============*/
      /* cascade를 연관관계 매핑의 주인 쪽에  ALL리라고 옵션을 주고 나서는*/
      parent.setChildList(Arrays.asList(child1,child2));

      
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

- ALL(전체), PERSISTE(저장) 주로 실무에서 많이 사용
- 그 외 REMOVE(삭제), MERGE, REFRESH, DETACH
- 하나의 부모만 자식을 관리할 때, e.g 게시물과 첨부파일(소유자가 하나일 때 라고도 한다.) 아래의 경우에만 쓰자 : 
  - 부모 자식의 라이프사이클이 같을 때
  - 소유자가 하나일 때
  
### 고아 객체
- orphanRemoval 옵션 value가 true 일때, 부모와의 연관관계가 끊어진 자식 객체에 대해 delete 쿼리가 나가 삭제됨

```java
@Data
@Entity
class Parent{
    
    @Id @GeneratedValue
    @Column(name="parent_id")
    private Long id;

  @OneToMany(mappedBy = "parent", cascade= CascadeType.ALL, orphanRemoval = true)//
  private List<Child> childList = new ArrayList<>();
    
    public void addChild(Child child){
        childList.add(child);
        child.setParent(this);
    }
}

class Test {
  public static void main(String[] args) {
    //.생
    //.략
    Parent parent = em.find(Paren.class, id);
    parent.getChildList().remove(0);// 엔티티 컬렉션에서 지우면 실제로 delete 쿼리가 나가버림.
    
    //.생
    //.략
  }
}  
```
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나일 때 사용할 것.
- 특정 엔티티가 개인 소유할 때만 사용.
- @OneToMany, @OneToOne 에서만 사용가능함.
- 부모를 제거하면 자식도 삭제된다. == `CascadeType.REMOVE` 과 같은 효과

### 고아 객체의 영속성 전이와 그 생명주기
- `cascade= CascadeType.ALL, orphanRemoval = true` : 옵션을 조합해서 쓴다?! : 부모객체를 통해서 생명주기를 관리한다. 즉, 부모가 완전 관리하기 때문에 코딩할 때, 사실 자식객체에 대한 레파지토리는 아예 만들지 않게 됨. (Aggregate Root개념을 적용한 DDD에서 응용한다.)

