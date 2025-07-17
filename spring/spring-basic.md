## IoC(Inversion of Control)

**제어의 주인이 프레임워크이다.**
@Configuration으로 등록한 설정파일에 의해 구현체의 의존관계가 설정되므로 개발자 입장에선 어떤 구현체가 사용되는지 모른다.  
우리가 new로 직접 인스턴스를 생성하지 않으니 어떤 구현체가 들어올지 모름. 의존성 주입도 스프링의 생태계에서 정해진 대로 빈 주입이 되는 것이 제어의 흐름이 프레임워크에 있다는 것이다.  
프로그램에 대한 제어흐름을 설정 파일(외부)이 가지고 있다.

### 라이브러리 vs 프레임워크

- 라이브러리: 제어의 흐름이 개발자에게 있다.
- 프레임워크: 제어의 흐름이 프레임워크에 있다.

##

## ⭐️의존관계 주입

> 애플리케이션 실행 시점에 **외부에서 실제 구현 객체를 생성**하고 클라이언트에 **전달**해서 클라이언트와 서버의 실제 **의존관계가 연결** 되는 것을 의존관계 주입(DI)라 고 한다.  
> 객체를 실제로 생성하는 코드가 없이 애플리케이션 실행 시점에 외부에서 생성해주고 주입을 해준다. 따라서 인터페이스에만 의존할 수 있는 것

1. 정적인 클래스 의존관계: import 코드를 보고 판단 가능 -> 애플리케이션을 실행하지 않아도 분석 가능

- intelliJ에서 클래스 다이어그램으로 볼 수 있는 관계를 말함

2. 동적인 클래스 의존관계: 애플리케이션 실행 시점에 실제 생성된 인스턴스와 연결된 의존관계

- 인터페이스는 어떤걸 의존하는지 정적으로 알 수 있으나, 구현체가 스프링이 어떤 걸 선택해서 주입해줄지 실행되지 않는한 모른는 것

### 장점

- 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다. -> 인터페이스를 구현해서 동일한 타입을 가진 클래스로 갈아끼울 수 있다.
- 정적인 클래스 의존관계를 변경하지 않고(=클라이언트 코드를 변경하지 않고), 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다. -> 인터페이스에만 의존하면 된다, 순수 자바 코드로 테스트를 할 경우 의존관계 주입이 불가능하명 NPE가 발생한다.

## IoC 컨테이너, DI 컨테이너

spring 설정파일에서 객체를 생성하고 의존관계를 연결해주는 것을 IoC컨테이너 또는 DI 컨테이너라고 한다.
IoC 용어는 너무 범용적이라 의존관계 주입에 초점을 맞추어 최근에는 주로 DI 컨테이너라고도 한다.

## 의존관계 주입 방법

1. **생성자 주입**: 생성자 호출 시점에 딱 한번 호출되는 것을 보장한다. 불편, 필수 의존관계일 경우 사용한다. 생성자가 하나일 때는 @Autowired를 생략할 수 있다. 빈을 등록하기 위해 객체를 생성해야함. 결국 빈이 등록되는 동시에 의존관계가 주입된다.
2. **수정자 주입(setter)**: setter 메서드로 의존관계를 설정해주는 것이다.
   스프링이 모든 빈을 컨테이너에 등록하고, @Autowired 어노테이션이 붙은 setter 메서드를 인식해서 의존관계를 자동으로 주입해준다. 의존관계를 수정할 수 있으므로 선택, 변겨 가능성이 있는 의존관계에 사용
3. **필드 주입**: 필드에 @Autowired를 붙여서 주입해줄 수 있다. 외부에서 변경이 불가능하다는 단점이 있다. (setter도 없고 생성자도 없어서 의존관계 주입이 @Autowired에만 의존한다. DI프레임워크가 없으면 동적 클래스 의존관계를 맺을 수가 없다.)
4. **일반 메서드 주입**: 메서드에 @Autowired를 붙여서 의존관계 주입 대상이 되도록 한다. 한번에 여러 필드를 주입받을 수 있다는 장점이 있지만, 수정자와 생성자 주입 선에서 다 해결되므로 일반적으로 잘 사용하지 않는다.

   - 의존관계 주입 단계에서 @Autowired가 붙은 메서드를 자동으로 호출하여 의존성을 주입한다.

## @Autowired 동작 방법

1. 스프링 애플리케이션이 시작하면서 빈들을 컨테이너에 등록한다
2. @Autowired 어노테이션을 보고 의존관계를 확인하고, 등록된 빈 중에서 동일한 타입의 빈이 있으면 주입해준다.

## 빈 충돌을 해결하는 방법

1. **@Autowired 필드명 매칭**: 타입 매칭을 시도하고, 이때 여러 빈이 있으면 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다. (타입 매칭 시도 후 필드명(또는 파라미터명)으로 빈이름 찾기)
2. **@Quilifier**: Quilifier를 사용해서 추가 구분자를 붙여주는 방법이 있다. 빈 등록시 @Quilifire를 붙여주고, 주입시에 @Quilifier를 붙여주고 등록한 이름을 적어주면 된다. 이렇게 되면 @Quilifier로 등록된 이름을 가진 빈을 조회해서 주입해준다. 만일 @Quilifier로 등록한 빈을 못찾으면 주입시에 @Quilifier 등록한 이름과 동일한 이름을 가진 빈을 찾는다.
3. **@Primary**: Primary는 말 그대로 우선순위를 정해주는 방법이다. @Autowired 시 여러 빈이 매칭되면 @Primary가 붙은 빈이 우선권을 가진다.

## BeanFactory와 ApplicationContext

> 둘 다 스프링 컨테이너라고 한다.

### BeanFactory

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리하고 조회하는 역할을 담당
- getBean() 제공 (Authowired 로 빈 주입시 빈타입 찾을때 사용하는 메서드)

### ApplicationContext

- BeanFactory를 확장한 인터페이스
- 관리, 조회 말고도 수많은 부가기능 제공(예를 들어서 환경변수 설정)
- BeanFactory를 사용할 일은 거의 없고 부가기능이 있는 ApplicationContext만 거의 사용

## 스프링 컨테이너의 생성 과정

스프링 컨테이너를 생성하고, 설정 정보를 참고해서 스프링 빈을 등록한 후, 의존관계를 설정한다.

### 자세한 과정

1. `java new AnnotationConfigAppliationContext(AppConfig.class)`
   위 코드를 실행해서 스프링 컨테이너를 만든다(컨테이너 않에 스프링 빈 저장소가 만들어져있음)
2. 구성 정보(AppConfig.class)를 보고 스프링 빈으로 등록한다. (key: 메서드명, value: 반환하는 구현체)
3. 빈 의존관계 설정을 위한 준비를 한다.

- value에 있는 객체를 생성해준다.

4. 빈 의존관계를 설정을 완료한다.

- 스프링 컨테이너가 설정 정보를 참고해서 의존관계를 주입(DI)한다. -> 동적인 클래스 의존관계 설정

## 빈 등록을 위해 작성하는 설정파일

- java와 xml로 작성할 수 있다.

## 빈 생명주기

> 애플리케이션 시작 시점에 필요한 빈을 모두 생성, 연결하고, 종료 시점에 한번에 없애는 작업
> 객체 생성 -> 의존관계 주입 순으로 이루어진다.(Setter, Field Injection 경우)

- 생성자 주입같은 경우는 예외임. 객체를 만들 때 스프링 빈이 들어와야하므로

### 스프링 빈의 이벤트 라이프사이클

스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입(Setter, Field Injenction) -> 초기화 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료

## 빈스코프

스프링 빈이 존재할 수 있는 범위를 말한다.

1. 싱글톤: 기본 스코프로, 스프링 컨테이너 시작과 종료까지 유지되는 가장 넒은 범위의 스코프이다.
2. 프로토타입: 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 짦은 범위

웹 관련 스코프

- **request**: 웹 요청이 들어오고 나갈 때까지 유지
- session: 웹 세션이 생성되고 종료될 때까지 유지
- application: 웹의 서블릿 컨텍스와 같은 범위로 유지

## POJO(Plain Old Java Object) 란 무엇인가?

POJO는 프레임워크 인터페이스, 클래스를 구현하거나 확장하지 않은 단순한 클래스로 Java에서 제공하는 API 외에 종속되지 않습니다. 특정 환경에 종속되지 않아 코드가 간결하고 테스트 자동화에 유리합니다. 스프링에서는 도메인과 비즈니스 로직을 수행하는 대상이 POJO대상이 될 수 있습니다.

### POJO 프로그래밍이 필요한 이유

- 특정 환경이나 기술에 종속적이지 않으면 재사용이 가능하고, 확장 가능한 유연한 코드를 작성할 수 있다.
- 저수순 레벨의 기술과 환경에 종속적인 코드를 제거하여 코드를 간결해지며 디버깅하기에도 상대적으로 쉬워진다.

### Spring 과 POJO의 관계

> 스프링은 IoC/DI, AOP, PSA 같은 기술을 통해 POJO가 잘 동작하도록 지원하는 프레임워크입니다.  
> Spring은 POJO 프로그래밍을 지향하는 프레임워크이다.  
> 최대한 다른 환경이나 기술에 종속적이지 않도록 하기 위한 POJO 프로그래밍 코드를 작성하기 위해 스프링 프레임워크에서는 IoC/DI, AOP, PSA를 지원하고 있다.

스프링이 POJO를 지향한다는 의미는 스프링은 프레임워크에 종속적인 코드 없이 동작할 수 있도록 설계되었다. 즉 extends SpringSomething, implements ContainerInterface 같은 걸 하지 않아도 스프링이 DI, AOP 등을 적용해 줄 수 있다는 뜻이다.

<details>
<summary>
DI컨테이너를 사용해서 빈을 관리하려면 스프링 컨테이너가 필요하지 않나요?
</summary>
-> DI나 AOP를 하기 위해서는 스프링 IoC(DI) 컨테이너가 필요하긴 함. 근데 개발자가 작성한 객체 자체는 스프링에 종속되지 않아도 되도록 설계되었다는 뜻이다.

```java
// 이런 코드는 프레임워크 종속적임
@Component
public class MyService implements ApplicationContextAware {
    ...
}
```

```java
// 이런 코드는 POJO
public class MyService {
    public void work() { ... }
}
```

스프링이 이 객체를 관리할 수는 있지만, 객체 자체는 스프링을 전혀 몰라도 된다.

</details>

## Configuration의 바이트코드 조작

설정 파일에 동일한 구현체를 반환하는 메서드가 두개 이상이여도 반환되는 객체가 싱글톤을 보장해준다.  
Configuration을 사용하면 스프링이 생성해준는 객체인 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 설정 파일을 상속받은 임의의 다른 클래스를 만들고 해당 클래스를 기반으로 빈을 생성해준다.  
그래서 바이트코드 조작으로 생성된 코드에서는 빈을 생성할때 컨테이너에 동일한 빈이 있으면 재사용하고, 없으면 생성하는 로직이 포함되어 있다.

# AOP

# 프록시, 프록시 패턴

## 프록시

1. 접근 제어, 캐시: 클라이언트가 프록시를 통해 객체 요청했는데 이미 인스턴스가 있다면 생성된거 반환(접근 제어, 실제 객체에 접근하지 않음)
   - 클라이언트의 요청의 권한을 확인하고 권한없으면 실제 서버에 접근하지 못하게한다.
   - 캐싱: 요청한 데이터가 프록시에 있다면 실제 서버까지 가지않고 프록시에서 반환
   - 지연로딩: 프록시 객체를 사용하다가 인스턴스의 데이터가 필요할 때 실제 요청이 나간다
2. 부가기능 추가
   - 원래 서버가 제공하는 기능에 더해서 부가 기능을 수행(요청, 응답 값을 변경, 실행 시간 측정 및 로그 남기기)

서버와 프록시는 대체 가능해야한다 -> 클라이언트 코드를 변경하지 않기 위함
대체 기능: 객체에서 프록시가 되려면 클라이언트는 서버에게 요청한 것인지, 프록시에게 요청한 것인지 조차 몰라야한다. (프록시는 서버와 동일한 인터페이스를 사용해야한다.)
기본적인 의존관계 틀은 아래와 같다.

**클래스 의존관계**

![class dependency](../images/Pasted%20image%2020250702204028.png)  
**런타임 의존관계**

![runtime dependency](../images/Pasted%20image%2020250702203939.png)

위 2가지 모두 프록시가 제공하는 기능으로 프록시가 사용하는 방법이지만 ,GOF 디자인 패턴에서 둘의 의도(intent)에 따라서 프록시 패턴과 데코레이터 패턴(서버와 프록시가 동일한 인터페이스)으로 구분한다.

- 프록시 패턴: 접근 제어가 목적
- 데코레이터 패턴: 새로운 기능 추가가 목적
  - 추상 클래스를 적용하여 꾸밈 대상과 꾸미는 주체를 구분하는게 GoF에서 설명하는 데코레이터 패턴이다

## ⭐️ 프록시는 인터페이스를 꼭 구현해야하는 건 아니다

인터페이스가 없어도 프록시가 가능하다.
해당 타입과 그 타입의 하위 타입 모두 다형성의 대상이 된다.

### 인터페이스 기반 프록시

장점: 인터페이스가 동일한 객체에 프록시를 적용할 수 있다.
단점: 인터페이스를 생성해야한다.

### 구체 클래스 기반 프록시

장점: 인터페이스 없이도 프록시 target을 기반으로 다형성(extends)을 적용하여 프록시를 생성할 수 있다. \* 다형성: 동일한 구조를 가지고 다른 기능을 수행할 수 있다.
단점(상속으로 인한 제약)

- 클래스 기반은 프록시가 상속한 부모 클래스에만 프록시를 적용할 수 있다. -> 특정 객체에 종속된 프록시 객체가 증가
- 부모 클래스의 생성자를 호출해야한다. super(..)
- final이 붙은 클래스는 상속이 불가능하다
- final이 붙은 메서드를 오버라이딩 할 수 없다. -> 프록시에서 target의 기능에 부가기능을 추가하여 대신 호출할 수 없다.

이론적으로는 인터페이스를 사용하여 역할과 구현을 나누는 것이 좋다. 하지만 구현을 거의 변경할 일이 없는 클래스도 있다. 인터페이스를 도입하는 이유는 변경할 가능성이 많을 때 효과적인데, 변경이 없는 코드에 무작정 인터페이스를 사용하면 오히려 사용하는 객체만 늘어나고 번거롭다. -> 인터페이스가 마냥 좋다는 것이 아니라는 것

# 동적 프록시 기술

동적 프록시가 필요한 이유: 프록시에서 수행하는 부가기능은 동일한데 target 객체의 참조 타입만 다르다면 프록시를 2개 생성행한다. 객체의 참조 타입을 런타임에 동적으로 조회하여 한개의 프록시에서 부가기능을 수행할 수 있다.  
그걸 가능하도록 한 기술이 '리플렉션' 이다.

문법은 대략 아래와 같다.

```java
Class clazz = Class.forName("path"); //Class: 클래스 메타정보
Method method = clazz.getMethod("메서드명"); //Method: 메서드 메타정보
Object o = method.invoke(clazz); //결과
```

리플렉션은 런타임에 동작 = 컴파일 시점에 오류를 잡을 수 없다 = 사용자가 직접 사용할 때 오류 발생 가능성이 크다 = 가급적 사용하지말고, 프레임워크 개발이나 매우 일반적인 공통 처리가 필요할 때 부분적으로 주의해서 사용
-> 리플렉션은 런타임 시점에 오류가 안나도록 구성하는 것이 하나의 과제

## 동적 프록시의 실전 예

### JDK 동적 프록시

인터페이스 기반 프록시를 생성해준다. InvocationHandler 인터페이스 구현하여 적용할 로직 구현하면 된다

### CGLIB (Code Generator Library)

CBLIB는 바이트코드(.class)를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리

- 인터페이스 없이 구체 클래스만으로 프록시 생성가능
- 원래 외부 라이브러리였으나 스프링 프레임워크가 스프링 내부 소스코드에 포함. 스프링을 사용한다면 별도의 라이브러리를 추가하지 않아도된다.
  > 우리가 CGLIB를 직접 사용할 일은 거의 없다. ProxyFactory가 이 기술을 편리하게 사용하도록 도와줌. 그래서 대략 개념만 잡자
  > 부가기능 로직을 MethodIntercepor를 구현하면 된다.

CGLIB는 클래스 기반 프록시이므로 제약 존재(앞서 배운 구체 클래스 기반 프록시 생성 기술에서의 제약조건을 떠올려보자)

- 부모 클래스의 생성자를 체크해야한다 -> CGLIB는 자식 클래스를 **동적으로 생성**하기 때문에 부모 클래스에 기본 생성자가 필요함(@Entity가 붙은 클래스에 기본 생성자가 필요한 이유임) -> 동적으로 생성하려면 컴파일러가 자동으로 추가할 수 있는 기본생성자가 필요(기본 생성자 아니면 개발자가 명시적으로 선언해주어야함)
- 클래스에 final 키워드가 붙으면 상속 불가능 (프록시 생성 불가능)-> CGLIB에서 예외 발생
- 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩 할 수 없다.(프록시에서 오버라이딩해야하는데 못함)

# 스프링이 지원하는 프록시

- 프록시 팩토리
- 포인터컷, 어드바이스, 어드바이저
- 스프링이 제공하는 포인트컷
- 여러 어드바이저 함께 적용

인터페이스 기반 프록시와 구체 클래스 기반 프록시가 각각 동적으로 생성하여 실행하는 것을 해결했다.
이제 인터페이스 기반과 구체 클래스 기반을 구분하지 않고 동적 프록시를 생성하기 위해 어떻게 해야할까?
스프링의 **프록시 팩토리** 라는 기능을 사용하면됨
jdk, CGLIB로 인터페이스, 구체 클래스 기반 프록시인지 구분하지 않고 동적 프록시를 생성할 수 있다.

프록시 팩토리가 인터페이스가 있다면 JDK 동적 프록시를 생성하고, 구체 클래스가 있다면 CGLIB를 사용함. -> 이 설정 변경 가능

부가 기능을 수행하는 핸들러는 어떻게 처리했는가? (InvocaationHandler, MethodInterceptor)

- Advice라는 개념 도입 -> 이게 위 두개 핸들러를 대신함(팩토리에서 advice를 사용)
- 두개의 핸들러는 모두 Advice라는 걸 호출 (핸들러를 Advice라는 것으로 추상화하였다고 생각)

Pointcut

- 특정 조건이 맞을 때 프록시를 적용하는 로직이 적용되도록 하는 개념
- 특정 메서드 이름의 조건에 맞을 때만 프록시 부가 기능이 적용되는 코드

## 프록시 팩토리

큰 흐름은 다음과 같다.  
proxyFactory에 프록시로 생성할 인스턴스를 넘겨준다.  
-> 팩토리는 이 인스턴스를 기반으로 인터페이스가 있으면 JDK동적 프록시를, 없으면 CGLIB 동적 프록시를 생성한다  
-> 팩토리.addAdvice()로 target에 적용할 부가기능을 구현한 advice 객체를 적용해준다. (팩토리 내부에서 target에 부가기능을 수행해줄 핸들러를 설정해주는 것이다. 이전에 JDK또는 CGBLI를 통해 생성된 프록시가 핸들러를 수행하면 이 핸들러는 우리가 설정한 Advice에게 책임을 인가해준다. 그래서 팩토리 내부의 advice가 실행)  
-> 팩토리.getProxy() 하면 여기서 실질적으로 클라리언트 -> 프록시 -> target이 호출된다.

### 이 모든게 프록시 팩토리의 서비스 추상화 덕분

> 스프링 부트는 AOP를 적용할 때 기본적으로 proxyTargetClass = True(인터페이스가 있어도 강제로 CGLIB로 동적 프록시를 생성하겠다)로 설정해서 사용한다. 따라서 인터페이스가 있어도 항상 CGLIB를 사용해서 구체 클래스를 기반으로 프록시를 생성

## 포인트컷, 어드바잇, 어드바이저

어드바이저(what): 어떤 메소드(포인트컷)에 어떤 애스펙트(어드바이스)를 적용해야하는 지 알고 있는것, 어떤 포인터컷에 어떤 어드바이스를 적용행해야하는지 알고 있는것. (하나의 포인트 컷과 하나의 어드바이저를 가짐)
포인트컷(where): 어떤 메서드 요청을 가로채서 애스펙트를 실행해야하는지 정의한 것 -> 어디에 부가기능을 적용할지 설정해주는 개념(주로 클래스와 메서드 이름으로 필터링)
어드바이스(what + where): 언제 이 애스펙트 로직을 실행해야하는 지 정의한 것(프록시가 실행하는 부가 로직)

포인트컷, 어드바이스, 어드바이저로 구분한 이유: 역할과 책임 분리

- 포인트컷: 대상 여부를 확인하는 필터 역할만 담당
- 어드바이스는 깔끔하게 부가 기능 로직만 담당
- 둘을 합치면 어드바이저 -> 스프링의 어드바이저는 하나의 포인트컷 + 하나의 어드바이스로 구성
  즉 프록시는 어드바이저(포인터컷, 어드바이스)를 가지고 있고, 이를 참조하여 프록시에서 target에 대해 필터링(포인트컷)하고 부가기능 수행(어드바이스)하는 것이다.

### 어드바이저

어드바이저는 하나의 포인트컷과 하나의 어드바이스를 가지고 있다. 프록시 팩토리를 통해 프록시를 생성할 때 어드바저를 넣어주면 어디에 어떤 기능을 제공하는 지 알 수 있다.

```java
DefalutPointcutAdvisor adsvisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new TimeAdvice());
proxyFactory.addAdvisor(advisor); //프록시 팩토리에 어드바이저 넣어줌
```

이전에는 그냥 factory.adAdvice(advice) 해서 어드바스를 어드바이저 만들지 않고 넣어줬는데, 스프링이 이 코드 내부에서 어드바이저를 생서하고 그 안에 advice를 넣어준다. 프록시 팩토리를 사용할 때는 어드바이저가 필수이다.

### 직접 포인트컷 만들기

![](../images/Pasted%20image%2020250707230719.png)
스프링이 제공하는 포인트 컷을 위와 같다 ClassFitler: 클래스가 일치하는지, MehtodMacher: 메서드가 일치하는지, 둘다 true를 반환해야 어드바이스를 적용할 수 있다.

위 Pointcut를 구현해서 직접 포인트 컷을 구현할 수 있다.

## 스프링이 제공하는 Pointcut

![](../images/Pasted%20image%2020250707230942.png)
스프링은 위 사진과 같이 23개의 구현체를 제공한다.
그중에서 NameMatchMethodPointcut을 사용해서 메서드 이름으로 필터링을 해보자

```java
NameMathcMethodPointcut poitcut = new NameMatchMethodPintcu();
pointcut.setMappedName("save"); //메서드 명이 save인 것만 필터링해서 advice를 적용하겠다.
```

스프링이 제공하는 pointcut중 중요한게있다. aspectJ표현식을 기반으로 사용하는 AspctJExpressionPointcut
aspectJ 표현식과 사용밥법은 중요하므로 AOP를 설명할 때 자세히 다룬다.

## 여러개의 어드바이저 적용

프록시 팩토리에 여러 어드바이저를 다를 수 있다
위에서 어드바이저를 생성해서 프록시에 적용하기 위해 DefalutPointcutAdvisor를 생성해서 factor.addAdvisor(어드바이저) 했다.
이런 방식으로 하나의 타겟에 여러 프록시를 적용하여 체인을 이룬다면 다음과 같은 코드가 된다. -> 자료 참고
N개의 어드바이저를 생성하기 위해 N개의 프록시가 생성된다.
factory.addAdvisor()로 추가하면 이 문제를 해결된다. -> 순서대로 advisor가 적용된다.

> 스프링 AOP를 처음공부하게 되면 AOP 적용 수만큼 프록시가 생성된다고 착각한다. 스프링은 AOP를 적용할때, 최적화를 진행해서 지금처럼 프록시를 하나만 만들고, 하나의 프록시에 어려 어드바이저를 적용한다.
> ⭐️ 하나의 target에 여러 AOP가 동시에 적용되어도, 스프링의 AOP는 target 마다 하나의 프록시만 생성한다

하나의 프록시에 어드바이저를 리스트로 관리
![](../images/Pasted%20image%2020250707233833.png)

# 빈 후처리기 -> 스프링이 프록시를 어떻게 자동으로 등록하는지 이해하기 위해 알아야하는 개념

## BeanPostProcessor

스프링 빈 저장소에 등록할 목적으로 생성한 객체를 빈 저장소에 등록하기 직전에 조작하고 싶다면 빈 후처리기를 사용하면된다.

빈 후처리기가 어떻게 개입하는 지 빈 등록과정과 같이 살펴보자.

1. 생성: 스프링 빈 대상이 되는 객체를 생성한다.
2. 전달: 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달한다.
3. 후 처리 작업: 빈 후처리기는 전달된 스프링 빈 객체를 조작하거나 다른 객체로 바꿔치기 할 수 있다.
4. 등록: 빈 후처리기는 빈을 반환한다. 전달된 빈을 그대로 반환하면 해당 빈이 등록되고, 바꿔치기하면 다른객체가 빈 저장소에 등록된다.

## 빈 바꿔치기 해보기

스프링이 제공하는 BeanPostProcessor 인터페이스를 구현하면 된다. 이 인터페이스를 구현하고 스프링 빈으로 등록하면 된다.
![](../images/Pasted%20image%2020250717191631.png)

- `postProcessBeforeInitialization`: 객체 생성 이후에 `@PostConstuct` 같은 **초기화가 발생하기 전**에 호출되는 포스트 프로세서
- `postProcessAfterInitialization`: 객체 생성 이후에 @PostConstruct 같은 **초기화가 발생한 다음**에 호출되는 포스트 프로세서

```java
B b = applicationContext.getBean("beanA", B.class); //beanA이름으로 B 객체가 등록된다.

@Configuration
static class BeanProcessorConfig {
	@Bean(name="beanA")
	public A a() {
		return new A();
	}
	@Bean
	public AToBPostProcessor helloPostProcessor() {
		return new AToBPostProcessor();
	}
}

static class AToBPostProcessor implements BeanPostProcessor {
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		if (bean instanceof A) {
			return new B(); //레퍼런스타입이 A이면 B반환하여 빈으로 등록
		}
		return bean;
	}
}
```

빈 후처리기를 사용하면 빈 객체를 프록시로 교체하는 것도 가능하다는 말

> 참고 @PostConstruct의 비밀
> @PostConstructor는 스프링 빈 생성 이후에 빈을 초기화 하는 역할을 한다. 빈 초기화는 g해당 어노테이션이 붙ㅇㄴ 초기화 메서드를 한번 호출만 하면 된다. -> 빈을 한번 조작하는 것
> 스프링은 CommonAnnotationBeanPostProcessor라는 빈 후처리기를 자동으로 등록하는데, 여기에서 @PostConstruct 애노테이션이 붙은 메서드를 호출한다. 따라서 스프링 스스로도 스프링 내부의 기능을 확장하기 위해 빈 후처리기를 사용한다.

## 빈 호처리기 - 적용

실제 객체 대신 프록시를 빈으로 등록해보자. 이렇게 하면 수동으로 등록하는 빈, 컴포넌트 스캔을 사용하는 빈 모두 프록시 적용 가능, 설정 파일에 있는 수많은 프록시 생성 코드도 한번에 제거할 수 있다.

이제는 ProxyFactoryConfig에서 팩토리를 사용해서 프록시 생성하고 어드바이저 등록하는 설정 파일이 필요가 없고, 빈 호추리기가 다 해주게 되었다.
(빈으로 등록되는 모든 "빈" 에 대한 "프록시"를 생성해주는 빈 후처리기를 우리가 구현하고 적용해보았다.)

빈 후처리기에 대상을 지정하지 않으면 우리가 등록한 스프링 빈 뿐 아니라 스프링 부트가 기본으로 등록하는 수 많은 빈들이 빈 후처리기로 넘어온다. 그래서 빈을 프록시로 만들 것인지 기준이 필요해서 여기서는 basePackage를 사용해서 특정 패키지를 기준으로 해당 패키지와 그 하위 패키지의 빈들을 프록시로 만든다.

- 스프링 부트가 기본으로 제공하는 빈 중에 프록시로 만들 수 없는 빈이 있어서 모든 빈을 대상으로 빈 후처리를 적용하면 오류 발생

## 빈 후처리기 사용 전 문제

### 문제1

프록시를 직접 스프링 빈으로 등록하는 ProxyFactoryConfig 설정파일을 보면 프록시 관련 설정이 지나치게 많아진다. 예를 들어 스프링 빈 100개가 있고 해당 객체에 모두 부가기능을 추가하기 위해 프록시를 적용해야한다면 설정 파일에 100개의 빈에 프록시 팩토리를 사용해서 프록시를 만들고 빈으로 등록하는 과정이 필요하다.

```java
@Slf4j
@Configuration
public class ProxyFactoryConfigV2 {

	@Bean
	public OrderControllerV2 orderControllerV2(LogTrace logTrace) {

		OrderControllerV2 orderController = new

		OrderControllerV2(orderServiceV2(logTrace));

		ProxyFactory factory = new ProxyFactory(orderController);

		factory.addAdvisor(getAdvisor(logTrace));

		OrderControllerV2 proxy = (OrderControllerV2) factory.getProxy();log.info("ProxyFactory proxy={}, target={}", proxy.getClass(),

		orderController.getClass());

	return proxy;

	}

	@Bean
	public OrderServiceV2 orderServiceV2(LogTrace logTrace) {
		OrderServiceV2 orderService = new OrderServiceV2(orderRepositoryV2(logTrace));
		ProxyFactory factory = new ProxyFactory(orderService);
		factory.addAdvisor(getAdvisor(logTrace));
		OrderServiceV2 proxy = (OrderServiceV2) factory.getProxy();
		log.info("ProxyFactory proxy={}, target={}", proxy.getClass(),

		orderService.getClass());
	return proxy;

	}

	@Bean
	public OrderRepositoryV2 orderRepositoryV2(LogTrace logTrace) {

		OrderRepositoryV2 orderRepository = new OrderRepositoryV2();

		ProxyFactory factory = new ProxyFactory(orderRepository);

		factory.addAdvisor(getAdvisor(logTrace));

		OrderRepositoryV2 proxy = (OrderRepositoryV2) factory.getProxy();

		log.info("ProxyFactory proxy={}, target={}", proxy.getClass(),

		orderRepository.getClass());

	return proxy;

	}

	private Advisor getAdvisor(LogTrace logTrace) {
	//pointcut
		NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();

		pointcut.setMappedNames("request*", "order*", "save*");
		//advice
		LogTraceAdvice advice = new LogTraceAdvice(logTrace);
		//advisor = pointcut + advice
		return new DefaultPointcutAdvisor(pointcut, advice);

	}

}
```

### 문제2 컴포넌트 스캔

컴포넌트 스캔을 사용하는 경우 프록시 팩토리를 통해서 빈을 생성하지 못하므로 프록시 적용이 불가능 했다. 빈 후처리기를 사용해서 빈 초기화 후에 프록시로 바꿔서 프록시를 빈으로 등록해주므로 컴포넌트 스캔하는 빈들도 프록시가 적용된다.

이렇게 빈 후처리기 덕분에 프록시를 생성하는 부분을 한곳에서 관리할 수 있다.(BeanPostProcessor) 덕분에 애플리케이션에 수 많은 스프링 빈이 추가되어도 프록시와 관련된 코드는 전혀 변경하지 않아도 된다.
**-> 스프링은 프록시를 생성하기 위한 빈 후처리기를 이미 만들어서 제공하고 있다.**

> 자료에서는 패키지를 기준으로 프록시 대상 여부를 결정하였는데, 포인터컷을 사용하면 더 깔끔할 것같다.
> 포인트컷은 이미 클래스, 메서드 단위의 필터 기능을 가지고 있기 때문에, 프록시 적용 대상 여부를 정밀하게 설정할 수 있다.
> 결과적으로 포인터 컷을 다음 두 곳에서 사용된다.
>
> 1.  프록시 적용 대상 여부를 체크해서 필요한 곳에만 프록시 적용(빈 후처리기 - 자동 프록시 생성)
> 2.  프록시의 어떤 메서드가 호출되었을 때 어드바이스를 적용할 지 판단한다(프록시 내부)

## 스프링이 제공하는 빈 후처리기 1

spring aop라이브러리 추가 -> 이 라이브러리를 추가하면 aspectjweaver 라는 aspectJ관련 라이브러리를 등록하고, 스프링 부트가 AOP 관련 클래스를 자동으로 스프링 빈에 등록한다. 스프링 부트가 활성화하는 빈은 AopAutoConfiguration을 참고

### 자동 프록시 생성기 - AutoProxyCreator

앞서 이야기한 스프링 부트 자동 설정으로 AnnotationAwareAspectJAutoProxyCreator라는 빈 후처리기가 스프링 빈에 자동으로 등록된다.  
이름 그래도 자동으로 프록시를 생성해주는 빈 후처리기이다.  
이 빈 후처리기는 스프링 빈으로 등록된 Advisor를 찾아서 프록시가 필요한 곳에 자동으로 프록시를 등록해준다. ->Advisor 에 pointcur과 Advice를 가지고 있으므로 어디에(pointcut), 무엇을(advice)를 적용할 지알고 있다. 즉 Pointcut으로 어떤 스프링 빈에 프록시를 적용해야하는지 알 수 있고, 거기에 부가기능(Advice)를 적용하면 된다.

> AnnotatinAwareAspectJAutoProxyCreator는 @AspectJ와 관련된 AOP 기능도 자동으로 찾아서 처리해 준다.
> Advisor는 물론이고, @Aspect도 자동으로 인식해서 프록시를 만들고 AOP를 적용해준다. @Aspect에 대한 자세한 내용은 뒤에 설명한다.

### 자동 프록시 생성기의 작동 과정

1. 생성: 스프링이 스프링 빈 대상이 되는 객체를 생성한다.(@Bean, 컴포넌트 스캔 모두 포함)
2. 전달: 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달
3. 모든 Advisor 빈 조회: 자동 프록시 생성(빈 후처리기)하기 위해 스프링 컨테이너에서 모든 Advisor를 조회한다.
4. 프록시 적용 대상 체크: 앞서 조회한 Advisor에 포함되어 있는 포인트컷을 사용해서 해당객체가 프록시를 적용할 대상인지 아닌지 판단한다. 이때 객체의 클래스 정보는 물론이고, 해당 객체의 모든 메서드를 포인트컷에 하나하나 모두 매칭해본다. 그래서 조건이 하나라도 만족하면 프록시 적용 대상이 된다.
5. 프록시 생성: 프록시 적용 대상이면 프록시를 생성하고 반환해서 프록시를 스프링 빈으로 등록한다. 만일 대상이 아니라면 원본 객체를 반환해서 원본 객체를 스프링 빈으로 등록한다.
6. 빈 등록: 반환된 객체는 빈으로 등록된다.

advisor1 이라는 어드바이저만 빈으로 등록하면 된다.

- 스프링은 자동 프록시 생성기라는 (AnnotationAwareAspectJAutoProxyCreator) 빈 후처리기가를 자동으로 등록해준다. -> 이 자동으로 등록된 빈 후처리기를 통해서 빈으로 등록된 어드바이저를 찾아서 프록시 대상 객체를 가지고 프록시를 빈으로 등록해준다.

```java
@Configuration
public class AutoProxyConfig{
	@Bean
	public Advisor advisor1(LogTrace logTrace) {
		NameMatchMethodPointcut pointcut = new NameMathchMethodPointcut();
		pointcut.setMappedNames("request*");
		LogTranceAdvice advice = new LogTraceAdvice(logTrace);
		return new DefaultPointcutAdvisor(pointcut, advice);
	}
}
```

### 조 금 더 정밀한 포인트컷을 설정해보자

지금은 메서드 이름에 request만 포함 되어도 매칭되므로 스프링이 기본적으로 등록하는 빈에도 request를 사용하는 메서드의 빈에 프록시가 적용될 것이다.
AspectJ 포인트컷 표현식을 사용해서 좀 더 정밀하게 표현해보자

```java
AsepctJExpressionPoincut pointcut = new AspectJExpressionPointcut();
pointcut.setExpression("execution(* heelo.proxy.app..*(..))");
```

- AspectJExpressionPointcut: AspectJ 포인트컷 표현식을 적용할 수 있다.
- `execution(* hello.proxy.app..*(..))`: AspectJ가 제공하는 포인트컷 표현식이다. - `*`: 모든 반환타입 - `hello.proxy.app`: 해당 패키지와 그 하위 패키지 - `*(..)`: `*`모든 메서드 이름, `(..)` 파라미터는 상관없음.
  = hello.proxy.app 패키지와 그 하위 패키지의 모든 메서드는 포인트컷의 매칭 대상이 된다.

## 하나의 프록시, 여러 Advisor 적용

스프링 빈에 등록된 어떤 빈이 모든 advisor가 제공하는 포인트컷의 조건을 모두 만족한다면 프록시 자동 생성기(빈 후처리기)는 프록시를 몇개 생성할까?
프록시 자동 생성기는 프록시 한개를 생서하고, 프록시 내부에 여러개의 어드바이저를 등록한다.

---

참고: [스프링 교과서](https://product.kyobobook.co.kr/detail/S000213355775)

IoC원칙 기반의 DI를 사용했다.
DI란 애플리케이션 실행 시점에 외부에서 객체를 생성해서 클라이언트에 전달하여 클라이언트와 서버의 의존관계가 설정되는 것을 의미한다.
DI를 사용하면 프레임워크가 사용자가 정의한 객체를 관리하고, 필요한 곳에서 그 객체를 사용하도록 요청할 수 있다.
스프링 컨텍스트에서 @Autowired 어노테이션을 사용하여 객체를 '주입한다'고 한다고 하는데, IoC 원칙에 기반을 둔 다른 기술인 애스팩트(Aspects)에 대해 알아보겠다.

애스펙스는 프레임워크가 메서드 호출을 가로채고 그 메서드의 실행을 변경할 수 있는 방법으로, 사용자가 선택 특정 메서드를 호출 실행에 영향을 줄 수 있다. -> 실행 중인 메서드에 속한 로직 일부를 추출할 수 있다. = 공통 관심사를 외부로 추출할 수 있다.  
이것으로 개발자는 메서드 로직을 읽을 때 비즈니스 로직에만 집중할 수 있다.(비즈니스 로직과 다른 코드가 분리되어 있기때문에)
이런 접근 방식을 **애스펙트(관점) 지향 프로그래밍(Ascpect-Oriented Programming, AOP)**이라고 한다.

## Aspect 작동 방식

애스펙트는 메서드의 공통 관심사를 추출한 것으로, 특정 메서드를 호출할 때 프레임워크가 실행하는 로직의 일부이다. 애스펙트를 설계할 때 다음 사항을 정의한다.

- 특정 메서드를 호출할 때 스프링이 실행하길 원하는 코드가 무엇인지(what) 정의한다. 이를 **애스펙트(aspects)**라고 한다.
- 앱이 언제(메서드 호출 전 또는 후)이 애스펙트 로직을 실행해야 하는지 정의한다. 이를 **어드바이스(advice)**라고 한다.
- 프레임워크가 어떤(which) 메서드를 가로채기(intercept)해서 해당 애스펙트를 실행해야하는 지 정의한다. 이를 **포인트컷(pointcut)**이라고 한다.

애스펙트 용어와 함께 애스펙트 실행을 트리거하는 이벤트를 정의해주는 **조인트 포인트(joint point)** 개념도 발견할 것이다. 스프링에서 이런 이벤트는 항상 메서드 호출이다.  
 애스펙트를 적용하려는 객체는 빈으로 관리되어야 한다. 스프링 컨테이너가 애스펙트를 적용하려는 객체를 빈으로 관리하고 프레임워크가 빈을 제어해서 사용자가 정의한 애스펙트를 적용할 수 있다. 애스펙트가 가로챈 메서드를 선언하는 빈 이름은 대상 객체(target object)라고 한다.

```java
@Service
public class CommentService {
  public void publishComment(Comment comment) {}
  //do something
}
```

_CommentService 빈_(대상 객체)의 _public void publicshComment()메서드_(포인트 컷)가 _실행_(조인트 포인트)되기 _전에_(어드바이스) _일부 로직_(애스펙트)이 실행되길 원한다.

애스펙트를 적용하는 객체는 스프링 빈이어야 한다고 했다. 하지만 애플리케이션 컨텍스트에 애스펙트가 적용된 객체를 요청할 때 실제 인스턴스 참조를 제공하지 않고 프록시 객체를 제공한다.  
이렇게 감싸는 방식을 위빙(weaving)이라고 한다.
빈이 애스펙트 대상이면 스프링은 실제 객체에 대한 참조를 제공하지 않는 대신, 프록시 객체에 대한 참조를 제공한다. 이 프록시 객체는 가로챈 메서드에 대한 호출을 모두 관리하고 애스펙트 로직을 적용할 수 있다.

---

여기서 부터는 코드만 자료 참고 후 내개 생각한 내용을 정리한 것이다. 내용 검증 필요..

```java
@Configuration //특수한 에노테이션이다. 이 에노테이션이 붙은 클래스에서 빈으로 등록하면 빈이 싱글톤임을 보장해준다. -> 애스펙트로 싱글톤으로 관리하는 부가 기능을 수행한다?
public class ProjectConfig {
   @Bean
   public CommentService commentService() {
      return new CommentService();
   }
}
```

위 코드에서 @Configuration 에노테이션을 사용해서 ProjectConfig 객체를 애스펙트 대상 객체로 지정하였다. 애플리케이션이 실행되면 ProjectConfig를 상속받은 프록시가 생성이되고 실제고 아래처럼 콘솔에 객체 타입을 찍어보면 프록시 객체가 나온다.

```java
var c = new AnnotationConfigApplicationContext(ProjectConfig.class);
var service = c.getBean(CommentService.class);
System.out.println(service.getClass()); //XxxCGLIB...
```

메서드에 대한 애스펙트를 정의하면 누군가가 스프링이 제공하는 프록시로 메서드를 호출한다. 프록시는 애스펙트 로직을 적용한 후 실제 메서드로 호출을 위임한다.
