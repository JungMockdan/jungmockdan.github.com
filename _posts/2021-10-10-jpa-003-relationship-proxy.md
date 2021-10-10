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

### 활용

## 영속성 전이 : CASCADE

### 고아 객체

### 고아 객체의 영속성 전이와 그 생명주기

## 실전예제 5.연관관계 정리