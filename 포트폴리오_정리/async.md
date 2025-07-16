## @Async vs CompletableFutre

스프링에서 비동기 방식으로 처리하는 방법이 @Aysnc와 CompletableFuture 두가지인 줄 알고 있었다.
정확히는 "스프링"이 제공하는 비동기 방식은 @Aysnc이고, "자바"에서 제공하는 방식이 CompletableFuture인 것이다.
@Async는 AOP 개념을 활용하여 구현된것이고, 반환값으로 void, Future, CompletableFuture가 있다.
즉, 스프링에서 비동기 처리를 스프링이 제공하는 기능을 잘 활용하면서 사용하고 싶다면 @Async를 사용하는 걸로 이해했다.

> 서비스 확장을 위한 한 가지 접근 방식은 백그라운드에서 값비싼 작업을 실행하고 Java의 CompleteFuture 인터페이스를 사용하여 결과를 기다리는 것입니다. Java의 CompleteFuture는 일반적인 Future에서 발전한 제품입니다. 여러 개의 비동기 연산을 쉽게 파이프라인화하여 하나의 비동기 연산으로 병합할 수 있습니다.

메서드에 @Async 붙이면 개별 스레드로 실행된다. 호출자(caller)가 호출되어진(callee) 메서드의 완료여부를 기다리지 않는다(호출자는 이전 작업 완료여부를 신경안쓰고 계속 호출하는것)

## 비동기 활성화

자바에서 비동기 프로세스 활성화하는법
설정파일에 @EnableAsync 를 붙여준다.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```

@EnableAsync 를 사용하는 것으로 충분하지만, 다른 옵션으로도 설정할 수 있다.

1. @EncableAsync 지금 보고 있는거 @Async 어노테이션을 감지하기 위해 설정해야함.
2. mode: JDK 프록시 기반 어드바이스(부가기능) 또는 aspectJ위빙 타입의 어드바이스를 감지한다.
3. proxyTargetClass: CGLIB와 JDK 플록시를 감지한다. 이 속성은 AdviceMode.PROXY인 경우에만 효력이 있다.
4. order: AsyncAnnotationBeanPostProcesser(빈후처리)가 적용되는 순서를 결정한다. 디폴트값은 모든 기존 프록시를 고려할 수 있도록 마지막에 실행됩니다.

## @Async

@Async가 적용되기 위한 조건

1. 공개 메서드에만 적용
2. 동일한 클래스내부에서 @Async가 적용된 메서드(비동기로 호출하려는 메서드)를 호출하면 적용이 안된다.
   - 이유는 프록시로 적용되기 때문에 public 메서드에 적용되어야하고(프록시 호출은 오버라이드를 해야하므로), 자체 호출은 프록시를 우회하고 기본 메서드를 직접 호출하기 때문에 동작하지 않는다.
   - 프록시 객체를 빈으로 등록해서 메서드 호출 시 부가기능을 먼저 수행 후 진짜 객체 호출 (내부 호출은 this를 참조해야하므로 실제 인스턴스의 메서드를 직접 호출 , 프록시가 가로챌 수없음. 왜? 런타임 객체 의존성을 보면 호출 -> 프록시(빈으로 등록되어 있으니까) -> 실제 객체 여야하는데 내부 호출은 호출 -> 실제 인스턴스 로 가니까)

### void 반환하는 메서드에 비동기 적용

```java
@Async public void asyncMethodWithVoidReturnType() { System.out.println("Execute method asynchronously. " + Thread.currentThread().getName()); }
```

### 리턴 타입이 있는 메서드에 비동기 적용

Future로 Wrapping(감싸서) 반환

```java
@Async public Future<String> asyncMethodWithReturnType() { System.out.println("Execute method asynchronously - " + Thread.currentThread().getName()); try { Thread.sleep(5000); return new AsyncResult<String>("hello world !!!!"); } catch (InterruptedException e) { // } return null; }
```

spring은 Future로 구현된 AsyncResult라는걸 제공한다. 비동기 메소드의 실행 결과를 추적할 수 있다.

```java
public void testAsyncAnnotationForMethodsWithReturnType() throws InterruptedException, ExecutionException { System.out.println("Invoking an asynchronous method. " + Thread.currentThread().getName()); Future<String> future = asyncAnnotationExample.asyncMethodWithReturnType(); while (true) { if (future.isDone()) { System.out.println("Result from asynchronous process - " + future.get()); break; } System.out.println("Continue doing something else. "); Thread.sleep(1000); } }
```

## Executor

spring은 비동기적으로 메소드를 실행할때 SimpleAsyncTaskExecuror 를 사용한다. 하지만 전역적(애플리케이션) 레벨, 메소드 레벨로 오버라이드할 수 있다.

### 메소드 레벨

설정 클래스에 필요한 executor를 정의한다.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {

    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```

@Async에 executor 이름을 속성값으로 넣어준다.

```java
@Async("threadPoolTaskExecutor")
public void asyncMethodWithConfiguredExecutor() {
    System.out.println("Execute method with configured executor - "
      + Thread.currentThread().getName());
}
```

### 전역(애플리케이션) 레벨

설정 클래스를 AsyncConfigurer인터페이스를 구현하면된다.  
getAsyncExecutor() 메소드를 구현해서 적용한 executor를 반환하면 된다. 그러면 여기서 반환된 executor가 @Async 어노테이션이 붙은 메소드에 전역적으로 적용된다.

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig implements AsyncConfigurer {
```

## 예외 처리

메소드 리턴타입이 Future이면 예외 처리가 쉬워진다. Future.get()으로 예외를 던질 수 있다.
하지만 void를 반환하는 메소드는 호출자인 스레드에 예외가 전파되지 않는다. 그래서 예외를 처리하기 위해 추가적인 설정이 필요하다.  
AsyncUncaughtExceptionHandler 인터페이스를 구현해서 커스텀 비동기 예외 핸들러를 만들어야한다. handleUncaughtException() 메소드는 처리하지 못한 비동기 예외가 발생하면 실행된다(invoke).

```java
public class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(
      Throwable throwable, Method method, Object... obj) {

        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }

}
```

이전에 AsyncConfiguration 인터페이스를 구현한 설정파일에 getAsuncUncaughtExceptionHandler() 메소드를 구현해서 사용할 수있다.

```java
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return new CustomAsyncExceptionHandler();
}
```

[출처](https://www.baeldung.com/spring-async)
