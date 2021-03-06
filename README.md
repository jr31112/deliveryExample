[TOC]

# 다형성

## 역할과 구현을 분리

* 역할과 구현으로 구분하면 단순해지고, 유연해지며 변경도 편리해진다.
* 클라이언트는 대상의 역할(인터페이스)만 알면됨
* 클라이언트는 구현 대상의 내부 구조를 몰라도 됨
* 클라이언트는 구현 대상의 내부 구조가 변경되도 영향을 받지 않음
* 클라이언트는 구현 대상 자체를 변경해도 영향을 받지 않음

## 자바에서 역할 구현 분리

* 자바의 다형성을 활용(interface), 물론 상속도 가능하지만 interface가 더 나음
    * 역할 : 인터페이스
    * 구현 : 인터페이스를 구현한 클래스, 구현 객체
* 객체를 설계할 때 역할과 구현을 명확히 분리
* 객체 설계시 역할을 먼저 부여하고, 그 역할을 수행하는 구현 객체 만듬

### 장단점

* 인터페이스를 구현한 객체 인스턴스를 실행 시점에 유연하게 변경 가능
* 클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경
* 인터페이스 자체가 변한다면 클라이언트 서버 모두에 큰 변경이 발생



# SOLID 5원칙

> SRP 단일 책임 원칙
>
> OCP 개방 폐쇄 원칙
>
> LSP 리스코프 치환 원칙
>
> ISP 인터페이스 분리 원칙
>
> DIP 의존관계 역전 원칙

객체 지향의 핵심은 다형성

다형성 만으로는 쉽게 부품을 갈아 끼우듯 개발할 수 없으며 구현 객체를 변경할 때 클라이언트 코드가 함께 변형된다 -> OCP, DIP 위배

## SRP 단일 책임 원칙

한 클래스는 하나의 책임만 가져야 한다.

변경이 있을 때 파급 효과가 작으면 단일 책임 원칙을 잘 지킨것

## OCP 개방 폐쇄 원칙

소프트웨어 요소는 확장에서는 열려 있으나 변경에는 닫혀 있어야 한다.

다형성을 활용해서 원칙을 지키고자 노력해야함

인터페이스를 구현한 새로운 클래스를 하나 만들어 새로운 기능을 구현

구현 객체를 변경하려면 클라이언트 코드를 변경 -> 객체를 생성하고, 연관관계를 맺어주는 별도의 조립, 설정자가 필요!



## LSP 리스코프 치환 원칙

프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 잇어야 한다.

하위 클래스는 인터페이스 규약을 지켜야 한다는 것 -> 다형성을 지원하기 위한 원칙

단순히 컴파일이 아니라 규약을 지켜서 원래 설계한 기능을 수행할 수 있도록 해야함



## ISP 인터페이스 분리 원칙

특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 보다 나음

인터페이스를 분리하면 어떤 클라이언트가 변해도 다른 클라이언트에 영향을 주지 않는다.

인터페이스가 명확해 지고 대체 가능성이 높아진다.



## DIP 의존관계 역전 원칙

추상화에 의존하며 구체화에 의존하지 않는다.

역할(추상화)에 의존하여 구현체의 변경을 용이하게 수행하게 해야한다.



## 스프링 -> 객체 지향 설계

인터페이스를 도입하면 추상화란 비용이 발생 -> 확장 가능성이 없다면 구체화된 클래스를 직접 사용하고 향후 필요할때 리팩토링해서 인터페이스를 도입.

# Member, Order Serivce 개발
작성하는 코드에 대해서 



# 지금 까지 코드의 문제점

## DIP문제

![image-20210103143500957](./dist/DIP위반.jpg)

코드상으로 `OrderServiceImpl`이 `DiscountPolicy`를 의존하는 것이 아니라 `FixDiscountPolicy`, `RateDiscountPolicy`에 의존하여 구체(구현) 클래스를 의존하고 있다. -> DIP문제

또한, 다음과 같이 코드가 구성될 경우 null point exception이 발생한다.

```java
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private DiscountPolicy discountPolicy;

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);
        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```

이 문제를 해결하려면 누군가가 클라이언트인 `OrderServiceImpl` 에 `DiscountPolicy` 의 구현 객체를 대신 생성하고 주입해주어야 한다.



## AppConfig 를 활용한 해결

애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 이용하여 관심사를 분리하고 주입하게 한다.

생성자 주입 방법을 이용해 해결

* `MemberRepository` 인터페이스만 의존한다.
* `MemberServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 들어올지(주입될지)는 알 수 없다.
* `MemberServiceImpl` 의 생성자를 통해서 어떤 구현 객체를 주입할지는 오직 외부(`AppConfig`)에서 결정된다.
* `MemberServiceImpl` 은 이제부터 의존관계에 대한 고민은 외부에 맡기고 실행에만 집중하면 된다.

```java
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;
    
    public MemberServiceImpl(MemberRepository memberRepository) {
   		this.memberRepository = memberRepository;
    }
    
    public void join(Member member) {
    	memberRepository.save(member);
    }
    
    public Member findMember(Long memberId) {
    	return memberRepository.findById(memberId);
    }
}
```

![image-20210103174151577](./dist/의존성 주입.jpg)

* 객체의 생성과 연결은 `AppConfig` 가 담당한다.
* DIP 완성: `MemberServiceImpl` 은 `MemberRepository` 인 추상에만 의존하면 된다. 이제 구체 클래스를 몰라도 된다.
* 관심사의 분리: 객체를 생성하고 연결하는 역할과 실행하는 역할이 명확히 분리되었다.
* `appConfig` 객체는 `memoryMemberRepository` 객체를 생성하고 그 참조값
* `MemberServiceImpl`을 생성하면서 생성자로 전달한다.
* 클라이언트인 memberServiceImpl 입장에서 보면 의존관계를 마치 외부에서 주입해주는 것 같다고 해서 `DI`(Dependency Injection) 우리말로 의존관계 주입 또는 의존성 주입이라 한다.



* `AppConfig`는 애플리케이션의 실제 동작에 필요한 구현 객체를 생성한다.
  * `MemberServiceImpl`
  * `MemoryMemberRepository`
  * `OrderServiceImpl`
  * `FixDiscountPolicy`
* `AppConfig`는 생성한 객체 인스턴스의 참조(레퍼런스)를 생성자를 통해서 주입(연결)해준다.
  * `MemberServiceImpl` -> `MemoryMemberRepository`
  * `OrderServiceImpl` -> `MemoryMemberRepository` , `FixDiscountPolicy`



## 리펙토링

중복 코드를 제거하고 역할에 따른 구현을 하기 위해 진행

아래 그림의 모든 모습을 한눈에 보여줘야 하며 각 기능에 대해 중복이 없어야함.

![image-20210103175923851](./dist/AppConfig_Refactoring.jpg)



# 스프링 전환 전 정리

## SOLID 적용

### SRP 단일 책임 원칙
한 클래스는 하나의 책임만 가져야 한다.
클라이언트 객체는 직접 구현 객체를 생성하고, 연결하고, 실행하는 다양한 책임을 가지고 있음
`SRP` 단일 책임 원칙을 따르면서 관심사를 분리함
구현 객체를 생성하고 연결하는 책임은 `AppConfig`가 담당
클라이언트 객체는 실행하는 책임만 담당

### DIP 의존관계 역전 원칙
프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안된다.” 의존성 주입은 이 원칙을 따르는 방법 중
하나다.
새로운 할인 정책을 개발하고, 적용하려고 하니 클라이언트 코드도 함께 변경해야 했다. 왜냐하면 기존 클라이언트 코드(`OrderServiceImpl`)는 `DIP`를 지키며 `DiscountPolicy` 추상화 인터페이스에 의존하는 것 같았지만, `FixDiscountPolicy` 구체화 구현 클래스에도 함께 의존했다.
클라이언트 코드가 `DiscountPolicy` 추상화 인터페이스에만 의존하도록 코드를 변경했다.
하지만 클라이언트 코드는 인터페이스만으로는 아무것도 실행할 수 없다.
`AppConfig`가 `FixDiscountPolicy` 객체 인스턴스를 클라이언트 코드 대신 생성해서 클라이언트 코드에 의존관계를 주입했다. 이렇게해서 `DIP` 원칙을 따르면서 문제도 해결했다.

### OCP
소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다
다형성 사용하고 클라이언트가 DIP를 지킴
애플리케이션을 사용 영역과 구성 영역으로 나눔
AppConfig가 의존관계를 FixDiscountPolicy RateDiscountPolicy 로 변경해서 클라이언트 코
드에 주입하므로 클라이언트 코드는 변경하지 않아도 됨
소프트웨어 요소를 새롭게 확장해도 사용 영역의 변경은 닫혀 있다!

## IoC, DI,  컨테이너
### 제어의 역전 IoC(Inversion of Control)

기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다. 개발자 입장에서는 자연스러운 흐름이다.
반면에 `AppConfig`가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의 제어 흐름은 이제 `AppConfig`가 가져간다. 예를 들어서 `OrderServiceImpl` 은 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다.
프로그램에 대한 제어 흐름에 대한 권한은 모두 `AppConfig`가 가지고 있다. 심지어 `OrderServiceImpl`도 `AppConfig`가 생성한다. 그리고 `AppConfig`는 `OrderServiceImpl` 이 아닌 `OrderService` 인터페이스의 다른 구현 객체를 생성하고 실행할 수 도 있다. 그런 사실도 모른체 `OrderServiceImpl`은 묵묵히 자신의 로직을 실행할 뿐이다.
이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 `제어의 역전(IoC)`이라 한다.

### 프레임워크 vs 라이브러리
프레임워크가 내가 작성한 코드를 제어하고, 대신 실행하면 그것은 프레임워크가 맞다. (JUnit)
반면에 내가 작성한 코드가 직접 제어의 흐름을 담당한다면 그것은 프레임워크가 아니라 라이브러리다.

### 의존관계 주입 DI(Dependency Injection)
OrderServiceImpl 은 DiscountPolicy 인터페이스에 의존한다. 실제 어떤 구현 객체가 사용될지는 모른다.
의존관계는 정적인 클래스 의존 관계와, 실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계 둘을 분리해서 생각해야 한다.
#### 정적인 클래스 의존관계
클래스가 사용하는 import 코드만 보고 의존관계를 쉽게 판단할 수 있다. 정적인 의존관계는 애플리케이션
을 실행하지 않아도 분석할 수 있다.
`OrderServiceImpl`은 `MemberRepository` , `DiscountPolicy`에 의존한다는 것을 알 수 있다.
그런데 이러한 클래스 의존관계 만으로는 실제 어떤 객체가 `OrderServiceImpl` 에 주입 될지 알 수 없다.

#### 동적인 객체 인스턴스 의존 관계

애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다.
객체 다이어그램
애플리케이션 실행 시점(런타임)에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언
트와 서버의 실제 의존관계가 연결 되는 것을 의존관계 주입이라 한다.
객체 인스턴스를 생성하고, 그 참조값을 전달해서 연결된다.
의존관계 주입을 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스
를 변경할 수 있다.
의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽
게 변경할 수 있다.
### IoC 컨테이너, DI 컨테이너
`AppConfig`처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을 `IoC 컨테이너` 또는 `DI 컨테이너`라 한다.
의존관계 주입에 초점을 맞추어 최근에는 주로 **`DI 컨테이너`**라 한다.
또는 어샘블러, 오브젝트 팩토리 등으로 불리기도 한다.

# 스프링 부트 전환

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }

    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
}
```

```java
public class DeliveryApplication {

	public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
        OrderService orderService2 = applicationContext.getBean("orderService", OrderService.class);

        System.out.println(memberService);
        System.out.println(orderService);
        System.out.println(orderService2); // 싱글턴 확인해 보고 싶어 넣어봄
    
    }
}
```

`ApplicationContext`를 스프링 컨테이너라 한다.
기존에는 개발자가 `AppConfig`를 사용해서 직접 객체를 생성하고 `DI`를 했지만, 이제부터는 스프링 컨테이
너를 통해서 사용한다.
스프링 컨테이너는 `@Configuration`이 붙은 `AppConfig`를 설정(구성) 정보로 사용한다. 여기서 `@Bean`이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 `스프링 빈`이라 한다.
스프링 빈은 `@Bean` 이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다. (`memberService`, `orderService`)
이전에는 개발자가 필요한 객체를 `AppConfig` 를 사용해서 직접 조회했지만, 이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다. 스프링 빈은 `applicationContext.getBean()`메서드를 사용해서 찾을 수 있다.
기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 `스프링 컨테이너`에 객체를 `스프링 빈`으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.



# 빈 조회 방법

스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법

* `getBean(빈이름, 타입)`
* `getBean(타입)`

조회 대상 스프링 빈이 없으면 예외 발생

* `NoSuchBeanDefinitionException`

타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정하자.
`ac.getBeansOfType()` 을 사용하면 해당 타입의 모든 빈을 조회할 수 있다.

부모 타입으로 조회하면, 자식 타입도 함께 조회한다.

## BeanFactory와 ApplicationContext

### BeanFactory

![image-20210110020255946](./dist/beanFactory.jpg)

스프링 컨테이너의 최상위 인터페이스로 스프링 빈을 관리하고 조회하는 역할을 담당한다.
getBean() 을 제공한다.
지금까지 우리가 사용했던 대부분의 기능은 BeanFactory가 제공하는 기능이다.

### ApplicationContext

![image-20210110020510060](./dist/application_context.jpg)

BeanFactory 기능을 모두 상속받아서 ApplicationContext는 빈 관리기능 + 편리한 부가 기능을 제공한다.

BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다.

ApplicatonContext가 제공하는 부가기능

* 메시지소스를 활용한 국제화 기능(예를 들어서 한국에서 들어면 한국어로, 영어권에서 들어오면 영어로 출력)
* 환경변수
  * 로컬, 개발, 운영등을 구분해서 처리
* 애플리케이션 이벤트
  * 이벤트를 발행하고 구독하는 모델을 편리하게 지원
* 편리한 리소스 조회
  * 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

정리
ApplicationContext는 BeanFactory의 기능을 상속받는다.
ApplicationContext는 빈 관리기능 + 편리한 부가 기능을 제공한다.
BeanFactory를 직접 사용할 일은 거의 없다. 부가기능이 포함된 ApplicationContext를 사용한다.
BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다.

## 다양한 설정 형식 지원 - 자바 코드, XML

스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연하게 설계되어 있다.
![image-20210110020826332](./dist/config.jpg)

* `AnnotationConfigApplicationContext` 는 `AnnotatedBeanDefinitionReader` 를 사용해서`AppConfig.class`를 읽고 `BeanDefinition` 을 생성한다.
* `GenericXmlApplicationContext` 는 `XmlBeanDefinitionReader` 를 사용해서 `appConfig.xml` 설정 정보를 읽고 `BeanDefinition` 을 생성한다.

* 새로운 형식의 설정 정보가 추가되면, `XxxBeanDefinitionReader`를 만들어서 `BeanDefinition` 을 생성하면 된다.

## BeanDefinition

* `BeanClassName`: 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)

* `factoryBeanName`: 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig

* `factoryMethodName`: 빈을 생성할 팩토리 메서드 지정, 예) memberService
* `Scope`: 싱글톤(기본값)
* `lazyInit` : 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연처리 하는지 여부
* `InitMethodName`: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
* `DestroyMethodName` : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
* `Constructor arguments`, `Properties`: 의존관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)

# 싱글톤 컨테이너

> * 순수한 DI 컨테이너인 AppConfig는 요청을 할 때 마다 객체를 새로 생성한다.
> * 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸된다! 메모리 낭비가 심하다.
> * 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 설계하면 된다. 싱글톤 패턴

## 싱글톤 패턴
클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.

그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.

private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.

* 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new 키워드로 객체 인스턴스가 생성되는 것을 막는다.

호출할 때 마다 같은 객체 인스턴스를 반환하는 것을 확인할 수 있다.

### 싱글톤 패턴 문제점

* 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
* 의존관계상 클라이언트가 구체 클래스에 의존한다. DIP를 위반한다.
* 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
* 테스트하기 어렵다.
* 내부 속성을 변경하거나 초기화 하기 어렵다.
* private 생성자로 자식 클래스를 만들기 어렵다.
* 결론적으로 유연성이 떨어진다.
* 안티패턴으로 불리기도 한다.

## 싱글톤 컨테이너

> 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.
> 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.



**싱글톤 컨테이너**

스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.

* 컨테이너는 객체를 하나만 생성해서 관리한다.
  * 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다.
* 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤레지스트리라 한다.
* 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
  * 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  * DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.



스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

* 참고: 스프링의 기본 빈 등록 방식은 싱글톤이지만, 싱글톤 방식만 지원하는 것은 아니다. 요청할 때 마다 새로운 객체를 생성해서 반환하는 기능도 제공한다. 자세한 내용은 뒤에 빈 스코프에서 설명하겠다.



### 주의점

싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.

무상태(stateless)로 설계해야 한다!

* 특정 클라이언트에 의존적인 필드가 있으면 안된다.
* 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다!
* 가급적 읽기만 가능해야 한다.
* 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!!!

```java
public class StatefulService {
    
    private int price; //상태를 유지하는 필드
    
    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        this.price = price; //여기가 문제!
    }
    
    public int getPrice() {
        return price;
    }
}
```

`StatefulService`의 price 필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.

주문금액이 공유가 되어 문제가 발생한다.

무에서 이런 경우를 종종 보는데, 이로인해 정말 해결하기 어려운 큰 문제들이 터진다.(몇년에 한번씩 꼭 만난다.)

진짜 공유필드는 조심해야 한다! 스프링 빈은 항상 무상태(stateless)로 설계하자.



### AppConfig 싱글톤 확인

`memberService`빈을 만드는 코드를 보면 memberRepository() 를 호출한다. 

* 이 메서드를 호출하면 new MemoryMemberRepository() 를 호출한다.

orderService 빈을 만드는 코드도 동일하게 memberRepository() 를 호출한다.

* 이 메서드를 호출하면 new MemoryMemberRepository() 를 호출한다.

결과적으로 각각 다른 2개의 MemoryMemberRepository 가 생성되면서 싱글톤이 깨지는 것 처럼 보인다.

확인해보면 memberRepository 인스턴스는 모두 같은 인스턴스가 공유되어 사용된다. -> 싱글톤 보장

```java
@Test
void configurationDeep() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    // AppConfig도 스프링 빈으로 등록된다.
    AppConfig bean = ac.getBean(AppConfig.class);
    // 출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$6e4766f1
    System.out.println("bean = " + bean.getClass());
}
```

순수한 클래스라면 다음과 같이 출력되어야 한다.`class hello.core.AppConfig`
-> 예상과는 다르게 클래스 명에 xxxCGLIB가 붙으면서 상당히 복잡해진 것을 볼 수 있다. 이것은 내가 만든 클래스가 아니라 스프링이 `CGLIB`라는 바이트코드 조작 라이브러리를 사용해서 `AppConfig` 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것이다!

![image-20210110180208868](./dist/appConfig_CGLIB.jpg)

`AppConfig@CGLIB`클래스가 바로 싱글톤이 보장되도록 해준다. 아마도 다음과 같이 바이트 코드를 조작해서 @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.

* 참고 AppConfig@CGLIB는 AppConfig의 자식 타입이므로, AppConfig 타입으로 조회 할 수 있다.

@Configuration 을 붙이면 바이트코드를 조작하는 CGLIB 기술을 사용해서 싱글톤을 보장하지만, 만약 @Bean만 적용하면 싱글톤이 보장되지 않는다.

* `AppConfig`가 `CGLIB`기술 없이 순수한 `AppConfig`로 스프링 빈에 등록된 것을 확인할 수 있다.
* @Bean만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않는다.
* memberRepository() 처럼 의존관계 주입이 필요해서 메서드를 직접 호출할 때 싱글톤을 보장하지 않는다.
* **스프링 설정 정보는 항상 @Configuration 을 사용하자.**

# 컴포넌트 스캔 자동화

```java
@Configuration
@ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class))
public class AutoAppConfig {
    
}
```

> `AppConfig`, `TestConfig` 등 앞서 만들어두었던 설정 정보도 함께 등록되고, 실행되어 버린다. 그래서 `excludeFilters`를 이용해서 설정정보는 컴포넌트 스캔 대상에서 제외했다. 보통 설정 정보를 컴포넌트 스캔 대상에서 제외하지는 않지만, 기존 예제 코드를 최대한 남기고 유지하기 위해서 이 방법을 선택했다.
>
> * includeFilters : 컴포넌트 스캔 대상을 추가로 지정한다.
> * excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정한다

다음과 같이 `AutoAppconfig`클래스를 만들어 준 후 각 구현체에 `@Component` 어노테이션을 부착하여 스프링 빈으로 등록 될 수 있도록 한다.

이전에 AppConfig에서는 @Bean 으로 직접 설정 정보를 작성했고, 의존관계도 직접 명시했다. 이제는 이런 설정 정보 자체가 없기 때문에, 의존관계 주입도 이 클래스 안에서 해결해야 한다.

* @Autowired 는 의존관계를 자동으로 주입해준다.

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
```

기존과 다르게 스프링 컨테이너를 실행 하기 위해서는 `AnnotationConfigApplicationContext`를 사용하며 설정 정보로 `AutoAppConfig`클래스를 넘겨준다.

* 주의 할점!
  * 현재 고정, 변동 할인 요소가 들어가있으며 이 때문에 충돌이 날 수 있다.(NoUniqueBeanDefinitionException)
  * 때문에 등록은 한개만 할 수 있도록 하자.

1. `@ComponentScan`

  ![image-20210110220525401](./dist/component.jpg)
  `@ComponentScan` 은 `@Component` 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
  이때 스프링 빈의 기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용한다.
  빈 이름 기본 전략: `MemberServiceImpl` 클래스 `memberServiceImpl`
  빈 이름 직접 지정: 만약 스프링 빈의 이름을 직접 지정하고 싶으면
  `@Component("memberService2")` 이런식으로 이름을 부여하면 된다.

2. `@Autowired` 의존관계 자동 주입

  ![image-20210110220457076](./dist/autowired.jpg)
  생성자에 `@Autowired` 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
  이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
  `getBean(MemberRepository.class)` 와 동일하다고 이해하면 된다.
  생성자에 파라미터가 많아도 다 찾아서 자동으로 주입한다.

## 탐색 위치와 기본 스캔 대상

> 모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서  꼭 필요한 위치부터 탐색하도록 시작위치를 지정할 수 있다.

탐색할 패키지의 시작 위치 지정방법

`basePackages` : 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.

* basePackages = {"hello.core", "hello.service"} 이렇게 여러 시작 위치를 지정할 수도있다.
* basePackageClasses : 지정한 클래스의 패키지를 탐색 시작 위치로 지정한다.
* 만약 지정하지 않으면 @ComponentScan 이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

## 권장하는 방법

> 참고로 스프링 부트를 사용하면 스프링 부트의 대표 시작 정보인 `@SpringBootApplication`를 이 프로젝트 시작 루트 위치에 두는 것이 관례이다. (그리고 이 설정안에 바로 `@ComponentScan`이 들어있다!)

개인적으로 즐겨 사용하는 방법은 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것이다. 최근 스프링 부트도 이 방법을 기본으로 제공한다.

프로젝트 시작 루트, 여기에 `AppConfig` 같은 메인 설정 정보를 두고, `@ComponentScan` 애노테이션을 붙이고, `basePackages` 지정은 생략한다.

하위는 모두 자동으로 컴포넌트 스캔의 대상이 된다. 그리고 프로젝트 메인 설정 정보는 프로젝트를 대표하는 정보이기 때문에 프로젝트 시작 루트 위치에 두는 것이 좋다.

## 컴포넌트 스캔 기본 대상

> 참고: 사실 애노테이션에는 상속관계라는 것이 없다. 그래서 이렇게 애노테이션이 특정 애노테이션을 들고 있는 것을 인식할 수 있는 것은 자바 언어가 지원하는 기능은 아니고, 스프링이 지원하는 기능이다.

컴포넌트 스캔은 @Component 뿐만 아니라 다음과 내용도 추가로 대상에 포함한다.

* `@Component` : 컴포넌트 스캔에서 사용
* `@Controlller` : 스프링 MVC 컨트롤러에서 사용(스프링 MVC 컨트롤러로 인식)
* `@Service` : 스프링 비즈니스 로직에서 사용(사실 @Service 는 특별한 처리를 하지 않는다. 대신 개발자들이 핵심 비즈니스 로직이 여기에 있겠구나 라고 비즈니스 계층을 인식하는데 도움을 줌)
* `@Repository` : 스프링 데이터 접근 계층에서 사용(스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환)
* `@Configuration` : 스프링 설정 정보에서 사용(앞서 보았듯이 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처
  리를 한다.)

해당 클래스의 소스 코드를 보면 `@Component` 를 포함하고 있는 것을 알 수 있다.



## 필터

>참고: `@Component`면 충분하기 때문에, `includeFilters`를 사용할 일은 거의 없다. `excludeFilters`는 여러가지 이유로 간혹 사용할 때가 있지만 많지는 않다.
>특히 최근 스프링 부트는 컴포넌트 스캔을 기본으로 제공하는데, 개인적으로는 옵션을 변경하면서 사용하기보다는 스프링의 기본 설정에 최대한 맞추어 사용하는 것을 권장하고, 선호하는 편이다.

`includeFilters` : 컴포넌트 스캔 대상을 추가로 지정한다.
`excludeFilters` : 컴포넌트 스캔에서 제외할 대상을 지정한다.

**필터 타입 옵션**

* `ANNOTATION `: 기본값, 애노테이션을 인식해서 동작한다.(`org.example.SomeAnnotation`)
* `SIGNABLE_TYPE` : 지정한 타입과 자식 타입을 인식해서 동작한다.(`org.example.SomeClass`)
* `ASPECTJ` : AspectJ 패턴 사용(`org.example..*Service+*`)
* `REGEX` : 정규 표현식(`org\.example\.Default.*`)
* `CUSTOM` : TypeFilter 이라는 인터페이스를 구현해서 처리(`org.example.MyTypeFilter`)



## 중복 등록과 충돌

1. 자동 빈 등록 vs 자동 빈 등록

   컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 그 이름이 같은 경우 스프링은 오류를 발생시킨다.

   `ConflictingBeanDefinitionException` 예외 발생

2. 수동 빈 등록 vs 자동 빈 등록

  만약 수동 빈 등록과 자동 빈 등록에서 빈 이름이 충돌되면 어떻게 될까?

  ```java
  @Component
  public class MemoryMemberRepository implements MemberRepository {
      
  }
  
  @Configuration
  @ComponentScan(excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class))
  public class AutoAppConfig {
      
      @Bean(name = "memoryMemberRepository")
      public MemberRepository memberRepository() {
          return new MemoryMemberRepository();
      }
      
  }
  ```


  이 경우 수동 빈 등록이 우선권을 가진다.(수동 빈이 자동 빈을 오버라이딩 해버린다.)

  ```bash
  Overriding bean definition for bean 'memoryMemberRepository' with a different definition: replacing
  ```

  물론 개발자가 의도적으로 이런 결과를 기대했다면, 자동보다는 수동이 우선권을 가지는 것이 좋다. 하지만 현실은 개발자가 의도적으로 설정해서 이런 결과가 만들어지기 보다는 여러 설정들이 꼬여서 이런 결과가 만들어지는 경우가 대부분이다!

  그래서 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생하도록 기본 값을 바꾸었다.

  ```bash
  Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
  ```

  스프링 부트를 실행해보면 위와 같은 오류를 볼 수 있다.