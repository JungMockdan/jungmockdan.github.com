---
title: "Spring features"
date: 2021-07-25 14:05:28 -0400
categories: spring
tags: java spring framework IoC DI AOP
# 목차
toc: true  
toc_sticky: true
---
흔히 스프링의 특성을 꼽으라고 하면 **IoC**/**DI**, **AOP**를 꼽는다. 그 외에도 POJO와 확장추상화등도 있다. 이글에서는 후자는 빼고 설명한다.
그런데 그것에 대해 설명을 매끄럽게 할 수 있는 초보개발자는 많치 않은 것 같다. 나도 그랬었고, 정기적으로 그런다. 먼저 스프링이 머길래 이렇게 JAVA 개발자들의 사랑을 받을까를 생각해 보면 그 특성들이 왜 필요해졌고(등장배경), 그것이 어떻게 편리한지가 그것을 설명하는 길이 될수 있을 것 같다. 개발자의 겨울이 가고 봄이와서 스프링이..스프링이고..이게 한 4월쯔음이라면 더봄이 와버린 스프링부트는 5월~ 개발자에게 얼마나 큰 편리함을 주는가.... 머지 않은 미래에는 스프링엣지나 프리서머, 6월이나 7월같은 프레임워크가 등장할 수도 있겠죠. 활활타는 여름이 오고, 수확의 가을이 가고 나면 겨울이 오는데....그럼 개발자들은 다시 겨울을 맞이 하는 시나리오인가??? 필터없이 그냥 한번 적어 봤어요.
스프링 이전에 Java 프로그래밍은 테스트가 매우 힘들었다고 한다. 심지어 배포 후 테스트를 반복해야 했다고 하는데, 그것은 높은 결합도 때문이었다고 한다. 그래서 스프링프레임워크는 그 불편함을 해결하는데 목표를 두었다. 느슨한 결합도를 지향하며, 테스트를 용이하게 하는것이다. 그러니까 IoC/DI와 AOP가 뭐든간에 우리는 이것이 결합도를 느슨하게 해주며 결과적으로 테스트를 쉽게 해줌으로서 개발자를 편하게 해준다는 사실을 유추해 볼 수 있다. 물론 스프링등장배경에는 그외에도 개발자간의 기술차이라던가 개발스타일이라던가 여러다른 인격체가 모여 개발하며 통일성은 우주로 날려버리게 되어 여러 어려움을 초래한다던가 하는 그런 이유와 또 다른 이유도 있겠지만 이글에서는 언급하지 않는다.

나는 어떤 개념을 전달하거나 전달받을때, 복잡한 수식어를 거치고, 배경을 말하고 예시를 들고, 먼길을 거쳐 결론에 다다랐지만 출발지가 어디였는지 잊어버리게 되는 설명은 좋아하지 않는다. 그래서 어떤 개념을 설명할때, 단순한 한개의 문장으로 정의하고 살이 붙이는 방법을 선호한다. 그렇기 어려울때는 실물을 한번 보여주는 방법을 선호한다.

IoC는 스프링프레임워크에 제어의 권한을 주는것이다. 어렵게 `제어의 역전-Inversion of Control`이라고 해봤자 그걸 이해할수 있을리가 없다-꼭 일본어 표현같다 그런걸 이해할 수 있을리가 없잖아 OO상-. 단번에 이해해버린 그대는 훌륭한 두뇌의 소유자. 그렇다 스프링 프레임워크 없이는 모든 콘트롤(제어)를 개발자 본인이 해야 한다는 뜻이다. 이 제어는 어떤 제어일까 대체 내가 무엇을 제어하고 있었던 것이며, 대체 무엇을 넘겨준걸까? 
우리가 넘겨준것은 객체의 생성과 소멸에 대한 제어이다. (Bean) 
그리고 사실 이것을 제어하는것은 스프링컨테이너이다.
그리고 사실 IoC자체가 컨테이너라고 보면 된다.

스프링에는 2가지 컨테이너가 있고, 둘 중 BeanFactory 컨테이너가 빈의 생성과 주입에 관여하는 메서드를 가지고 있다. 스프링 컨테이너에 대한 내용은 아래 링크를 참조할 수 있도록 포스트를 작성해보겠다. 어쨌든 이글에서는 대신 해주는 놈정도로 이해하면 된다.
```
// todo
 - 스프링 구조, 구동순서 등에 대한 포스트
```
이정도면 IoC가 머냐고 물어보면 BeanFactory 인터페이스이다 라고 해도 되지 않을까? 실제로 인터페이스에 설계된 메소드들만 봐도 대신제어하는 기능을 충분히 설명해주고 있다. 해당 인터페이스 다음은 BeanFactory안의 인터페이스에대한 주석을 구글번역한 결과이고 중간중간 삭제한 글이다. 다 좋은말이다. 꼭 읽어보시길 바랍니다.
org.springframework.beans.factory.BeanFactory.java를 열어서 직접보시는게 제일 좋다.
```text
# BeanFactory : Spring 빈 컨테이너에 접근하기 위한 루트 인터페이스.

 - 빈 컨테이너의 기본 클라이언트 view 입니다. {@link ListableBeanFactory} 및 {@link org.springframework.beans.factory.config.ConfigurableBeanFactory}와 같은 추가 인터페이스는 특정 목적에 사용할 수 있습니다.

 - 이 인터페이스는 각각 문자열 이름으로 고유하게 식별되는 여러 빈 정의를 보유하는 객체에 의해 구현됩니다. 빈 정의에 따라 팩토리는 포함된 객체의 독립 인스턴스(프로토타입 디자인 패턴) 또는 단일 공유 인스턴스(인스턴스가 범위의 싱글톤인 싱글톤 디자인 패턴에 대한 우수한 대안)를 반환합니다. 공장). 반환될 인스턴스 유형은 빈 팩토리 구성에 따라 다릅니다. API는 동일합니다. Spring 2.0부터 구체적인 애플리케이션 컨텍스트에 따라 추가 범위를 사용할 수 있습니다(예: 웹 환경의 "request" 및 "session" 범위).

 - 이 접근 방식의 요점은 BeanFactory가 응용 프로그램 구성 요소의 중앙 레지스트리이며 응용 프로그램 구성 요소의 구성을 중앙 집중화한다는 것입니다(예: 개별 개체가 더 이상 속성 파일을 읽을 필요가 없음). 이 접근 방식의 이점에 대한 논의는 "Expert One-on-One J2EE Design and Development"의 4장과 11장을 참조하십시오.

 - 일반적으로 BeanFactory 조회와 같은 "pull" 구성 형식을 사용하는 것보다 설정자 또는 생성자를 통해 애플리케이션 객체를 구성하기 위해 종속성 주입("push" 구성)에 의존하는 것이 좋습니다. Spring의 의존성 주입 기능은 이 BeanFactory 인터페이스와 그 하위 인터페이스를 사용하여 구현됩니다.

 - 일반적으로 BeanFactory는 구성 소스(예: XML 문서)에 저장된 빈 정의를 로드하고 {@code org.springframework.beans} 패키지를 사용하여 빈을 구성합니다. 그러나 구현은 필요에 따라 Java 코드에서 직접 생성한 Java 객체를 간단히 반환할 수 있습니다. LDAP, RDBMS, XML, 속성 파일 등 정의를 저장할 수 있는 방법에 대한 제한은 없습니다. 구현은 빈 간의 참조를 지원하도록 권장됩니다(종속성 주입).

 - {@link ListableBeanFactory}의 메소드와 달리 이 인터페이스의 모든 작업은 이것이 {@link HierarchicalBeanFactory}인 경우 상위 팩토리도 확인합니다. 이 팩토리 인스턴스에서 빈이 발견되지 않으면 바로 부모 팩토리가 요청될 것입니다. 이 팩토리 인스턴스의 Bean은 상위 팩토리에 있는 동일한 이름의 Bean을 대체해야 합니다.

 - 빈 팩토리 구현은 가능한 한 표준 빈 라이프사이클 인터페이스를 지원해야 합니다.
 ... 이하 생략
```
정리해보면, 스프링프레임워크의 특징중 IoC란,
- 스프링 컨테이너가 빈 생성/소멸의 제어를 다 가져갔다. --> 제어가 역전되었다. --> IoC
- 스프링컨테이너 중 BeanFactory 컨테이너가 담당이다.
이정도가 되겠다.
## IoC 실제 사용
- IoC의 큰그림을 봤다고 치고, 실제 우리는 IoC를 사용하고는 있을까? 물론이다!! 그것도 너무 자주 많이. 우리가 만드는 콘트롤러, 서비스 그리고 레파지토리 모두 IoC의 특성을 이용한 것이다. 콘트롤러가 호출될 때, new 키워드와 함께 어딘가에서 생성해준적이 있는가??? 없지 않은가? 서비스, 레파지 토리도 마찬가지다. 서비스에서 사용했던 생성자주입방식의 생성자를 만들었다고??? 그래서 그걸 직접 new 키워드와 함께 생성한적은 없을 것 이다.
- 실제 코드에서는 어떻게 적용될 것인가 역시도, 여러분이 콘트롤러, 서비스 그리고 레파지토리를 만들어 서비스하나를 만드는 과정과 다르지 않다.

### 빈을 만드는 어노테이션
- 빈을 만들어야 컨테이너에 생성이 되고, 그 쓰임이 있다. 아래는 빈을 만드는 어노테이션이다.
```markdown
- @Component // 내가 만든 pojo를 빈으로 만들어줌. @Controller, @Service, @Repository가 상속받음.
- @Configuration // 외부라이브러리로 부터(구현 및 커스텀) 1개 이상의 빈을 등록하겠다는 어노테이션, @Component을 가지고 있음
- @Bean// 메소드영역위에 선언하여 Bean으로 등록함. 
 ※ @Configuration과 같이 사용하는 것이 싱글톤이 확실하게 됨. 반대로 말하면, 그냥 @Bean만 사용시 싱글톤 구현에 오류가 생길 수 있음.
```
- 빈의 생성순서는 
    - 커스텀 빈(@Component, @Configuration)클래스의 패키지 위치상의 top-down
    - 그다음 커스텀 @Configuration클래스내의 @Bean 메소드
    - 그다음 스프링부트의 bean들의 순서이다.
    - 스프링부트의 구조를 공부하고 싶으신 분들은 정말 잘 정리된 곳이 있어 링크를 드린다.[배고픈 매튜](https://github.com/hungry-matt/TIL/blob/master/Spring%20Boot/%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%20%EA%B0%9C%EB%85%90%EA%B3%BC%20%ED%99%9C%EC%9A%A9/2.%20%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%20%EC%9B%90%EB%A6%AC/%EC%8A%A4%ED%94%84%EB%A7%81%20%EB%B6%80%ED%8A%B8%20%EC%9B%90%EB%A6%AC.md#%EC%9E%90%EB%8F%99-%EC%84%A4%EC%A0%95-%EC%9D%B4%ED%95%B4)

### 작성된 빈의 확인
- 인텔리제이에서 스프링부트프로젝트를 만들고, 빈객체를 코딩한다. 그리고 Application.java 즉, main()함수 있는 class로 가자. 거기 @SpringBootApplication 좌측에 초록색 돋보기를 클릭하자. 확인가능하다.
- code : 순서도 확인가능하다. 클래스단위 빈뿐만 아니라 메소드단위 빈도 확인 가능하다.

먼저, 테스트할 컴포넌트와 빈을 작성했단는 가정하에 아래 코드로 테스트하면 된다. 테스트 차원에서 컨텍스트 가지고 오는것도 컴포넌트로 작성해봤다.
```java
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class CustomContextProvider implements ApplicationContextAware {
    private static ApplicationContext context;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.context = applicationContext;
    }

    public static ApplicationContext getContext(){
        return context;
    }
}
```
Application.java에서 생성된 빈을 출력해 보는 코드
```java
import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.ApplicationContext;

import java.util.Arrays;

@SpringBootApplication
public class StudyApplication {

    public static void main(String[] args) {

        SpringApplication.run(StudyApplication.class, args);
        ApplicationContext context = CustomContextProvider.getContext();
        
        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        Arrays.stream(beanDefinitionNames)
                .forEach(str-> System.out.println(str));
    }

}
```

## DI
Dependency Injection-의존성 주입.
예를 들어,  A라는 클래스가 있는데, 이 클래스는  B라는 클래스의 멤버변수를 가진다 혹은 어떤 상호작용이 있다. 이럴때 우리는 A 는 B에 의존성을 가진다라고 한다. 전에 설명한 IoC는 결국 DI를 통해 완성이 되는데, 그 과정을 설명해보면, 일단 IoC는 컨테이너가 bean을 관리하고, 이말인 즉슨, DI를 통해서 Bean을 주입받음으로서 IoC가 완성이 되는 것이라고 말할 수 있다. 다시 풀어서. B라는 클래스는 사실 인터페이스였다. B라는 클래스를 구현한 Bean 객체인 B', B'', B'''이 있고, A는 생성자주입으로 B라는 인터페이스를 주입받고 있고, IoC컨테이너가 B', B'', B'''를 제어-구동시 bean 생성-하고, 실제 작동시에 B라는 인터페이스대신 B', B'', B'''가 들어오면 주입하는 형태로 IoC가 DI를 통해 완성된다.
### 빈을 주입받는 방법
- 변수
- 생성자
- set 메서드
