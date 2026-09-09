# spring-tutorial-24th
CEOS 백엔드 24기 스프링 튜토리얼


# 1. spring이 지원하는 기술들(IoC/DI, AOP, PSA 등)을 자유롭게 조사

## Spring의 핵심 철학

Spring의 핵심 기술인 **IoC/DI, AOP, PSA**는 객체지향적인 애플리케이션을 설계하고 유지보수하기 쉽게 만들어주는 역할을 한다.

Spring을 실무에서 많이 사용하는 이유 중 하나는 객체 간의 의존성을 유연하게 관리할 수 있고, 반복되는 공통 기능을 분리하여 개발자가 비즈니스 로직에 집중할 수 있기 때문이다.

---

## IoC (Inversion of Control)

### IoC란?

**IoC(Inversion of Control, 제어의 역전)** 는 객체를 생성하고 관리하는 제어권을 개발자가 직접 가지는 것이 아니라 Spring과 같은 프레임워크에 맡기는 것을 의미한다.

일반적인 Java 코드에서는 필요한 객체를 직접 생성한다.

```java
public class OrderService {

    private OrderRepository orderRepository = new OrderRepository();
}
```

위 코드에서는 `OrderService`가 직접 `OrderRepository`를 생성하고 관리한다.

하지만 Spring에서는 객체의 생성과 관리 역할을 **Spring Container**가 담당한다.

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

`OrderService`가 직접 객체를 생성하지 않고 Spring이 필요한 객체를 생성하여 전달한다.

### Spring Container

Spring에서는 `ApplicationContext`가 IoC Container의 역할을 한다.

Spring Container는 다음과 같은 역할을 담당한다.

* Bean 생성
* Bean 관리
* 객체 간 의존관계 설정
* Bean 생명주기 관리

즉,

```text
기존 방식

개발자
  ↓
객체 생성
  ↓
객체 연결
```

에서

```text
Spring 방식

Spring Container
  ↓
객체 생성
  ↓
객체 연결
  ↓
개발자에게 제공
```

하는 구조로 제어권이 이동한다.

---

## DI (Dependency Injection)

### DI란?

**DI(Dependency Injection, 의존성 주입)** 는 객체가 필요로 하는 다른 객체를 직접 생성하지 않고 외부에서 전달받는 방식이다.

DI는 **IoC를 구현하는 대표적인 방법**이다.

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

`OrderService`는 `OrderRepository`를 직접 생성하지 않는다.

Spring Container가 `OrderRepository` 객체를 생성하고 `OrderService`에 주입한다.

### DI를 사용하는 이유

DI를 사용하면 객체 사이의 결합도를 낮출 수 있다.

예를 들어 결제 기능을 다음과 같이 인터페이스로 추상화할 수 있다.

```java
public interface PaymentService {

    void pay();
}
```

```java
@Service
public class CashPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("현금 결제");
    }
}
```

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

`OrderService`는 구체적인 `CashPaymentService`에 직접 의존하지 않고 `PaymentService` 인터페이스에 의존한다.

따라서 구현체가 변경되어도 `OrderService`의 수정은 최소화할 수 있다.

### DI의 장점

* 객체 간 결합도를 낮출 수 있다.
* 구현체 변경이 쉬워진다.
* 테스트하기 쉬워진다.
* 객체 생성 및 관리 책임을 Spring Container에 맡길 수 있다.
* 객체의 역할과 책임을 분리하기 쉬워진다.

---

## 생성자 주입

Spring에서 의존성을 주입하는 방법은 크게 다음과 같다.

* 생성자 주입
* Setter 주입
* 필드 주입

이 중 일반적으로 **생성자 주입을 권장한다.**

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 생성자 주입의 장점

#### 1. 의존성을 불변으로 만들 수 있다.

```java
private final UserRepository userRepository;
```

`final`을 사용할 수 있기 때문에 객체가 생성된 이후 의존성이 변경되는 것을 방지할 수 있다.

#### 2. 반드시 필요한 의존성을 보장할 수 있다.

객체 생성 시 필요한 의존성을 생성자를 통해 전달해야 하므로 잘못된 상태의 객체가 생성되는 것을 막을 수 있다.

#### 3. 테스트가 쉽다.

Spring Container 없이도 직접 객체를 생성할 수 있다.

```java
UserRepository repository = new FakeUserRepository();

UserService userService = new UserService(repository);
```

### 필드 주입

```java
@Autowired
private UserRepository userRepository;
```

필드 주입은 코드가 간단하지만 외부에서 의존성을 주입하기 어렵고 테스트가 불편해질 수 있다.

따라서 특별한 이유가 없다면 **생성자 주입을 기본으로 사용하는 것이 좋다.**

---

## AOP (Aspect Oriented Programming)

### AOP란?

**AOP(Aspect Oriented Programming, 관점 지향 프로그래밍)** 는 여러 클래스에서 반복적으로 등장하는 공통 기능을 핵심 비즈니스 로직과 분리하는 프로그래밍 방식이다.

애플리케이션을 개발하면 다음과 같은 기능이 여러 곳에서 반복된다.

* Logging
* Transaction
* 권한 검사
* 실행 시간 측정
* 예외 처리

이러한 기능을 **공통 관심사(Cross-Cutting Concern)** 라고 한다.

---

### AOP를 사용하지 않는 경우

예를 들어 서비스의 실행 시간을 측정한다고 가정한다.

```java
public void createOrder() {

    long start = System.currentTimeMillis();

    // 주문 생성 로직
    orderRepository.save(order);

    long end = System.currentTimeMillis();

    System.out.println("실행 시간 : " + (end - start));
}
```

다른 Service에서도 실행 시간을 측정하려면 동일한 코드를 반복해서 작성해야 한다.

```text
OrderService
 ├─ 실행 시간 측정
 └─ 주문 로직

UserService
 ├─ 실행 시간 측정
 └─ 회원 로직

PaymentService
 ├─ 실행 시간 측정
 └─ 결제 로직
```

이렇게 되면 핵심 비즈니스 로직과 부가 기능이 섞이게 된다.

---

### AOP를 사용하는 경우

AOP를 사용하면 공통 기능을 하나의 영역으로 분리할 수 있다.

```java
@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.example.service..*(..))")
    public Object logExecutionTime(
            ProceedingJoinPoint joinPoint
    ) throws Throwable {

        long start = System.currentTimeMillis();

        Object result = joinPoint.proceed();

        long end = System.currentTimeMillis();

        System.out.println(
                "실행 시간 : " + (end - start)
        );

        return result;
    }
}
```

이제 각각의 Service에서는 핵심 비즈니스 로직만 작성하면 된다.

```text
        Logging Aspect
             ↓

OrderService   UserService   PaymentService
    ↓              ↓              ↓
 주문 로직       회원 로직       결제 로직
```

---

## Spring AOP와 @Transactional

Spring에서 AOP가 사용되는 대표적인 사례가 `@Transactional`이다.

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);
}
```

개발자는 직접 트랜잭션 코드를 작성하지 않는다.

실제로는 개념적으로 다음과 같은 작업이 필요하다.

```java
transaction.begin();

try {

    orderRepository.save(order);

    transaction.commit();

} catch (Exception e) {

    transaction.rollback();
}
```

Spring이 AOP를 이용해 이러한 트랜잭션 처리를 대신 수행한다.

```text
요청
 ↓
Spring Proxy
 ↓
Transaction 시작
 ↓
Service 메서드 실행
 ↓
성공 → Commit
실패 → Rollback
```

덕분에 개발자는 트랜잭션 처리보다 핵심 비즈니스 로직에 집중할 수 있다.

---

## PSA (Portable Service Abstraction)

### PSA란?

**PSA(Portable Service Abstraction)** 는 특정 기술에 직접 의존하지 않고 Spring이 제공하는 일관된 방식으로 기술을 사용할 수 있도록 만들어주는 추상화 방식이다.

쉽게 말하면

> 내부 구현 기술이 달라져도 개발자가 사용하는 방식은 최대한 동일하게 유지하도록 만들어주는 것이다.

---

### Transaction을 통한 PSA 예시

데이터베이스를 사용하는 방법에는 여러 가지가 있다.

```text
JDBC
JPA
Hibernate
...
```

각 기술마다 트랜잭션을 처리하는 방법은 다를 수 있다.

하지만 Spring에서는 다음과 같이 동일하게 사용할 수 있다.

```java
@Transactional
public void saveUser() {

    userRepository.save(user);
}
```

내부에서 JDBC를 사용하든 JPA를 사용하든 개발자는 동일한 `@Transactional`을 사용할 수 있다.

이것이 Spring이 제공하는 **서비스 추상화**이다.

---

### PSA의 장점

* 특정 기술에 대한 의존성을 줄일 수 있다.
* 기술을 변경하더라도 코드 변경을 최소화할 수 있다.
* 일관된 방식으로 여러 기술을 사용할 수 있다.
* 개발자가 세부 구현보다 비즈니스 로직에 집중할 수 있다.

Spring에서 PSA가 적용된 대표적인 기술로는 다음과 같은 것들이 있다.

* Spring Transaction
* Spring JDBC
* Spring Data
* Spring MVC

---

## IoC / DI / AOP / PSA 비교

| 기술  | 의미                           | 핵심 역할                        |
| --- | ---------------------------- | ---------------------------- |
| IoC | Inversion of Control         | 객체 생성 및 관리의 제어권을 Spring에게 넘김 |
| DI  | Dependency Injection         | 필요한 객체를 외부에서 주입              |
| AOP | Aspect Oriented Programming  | 반복되는 공통 기능을 핵심 로직과 분리        |
| PSA | Portable Service Abstraction | 특정 기술에 대한 의존성을 낮추는 추상화 제공    |

---

## IoC와 DI의 관계

IoC와 DI는 비슷해 보이지만 개념의 범위가 다르다.

**IoC는 더 큰 개념이고 DI는 IoC를 구현하는 방법 중 하나이다.**

```text
IoC
 │
 └── DI
      │
      └── Spring Container가 객체를 생성하고 주입
```

즉,

> 객체 생성과 관리의 제어권을 Spring에게 넘기는 것이 IoC이고, 그 과정에서 필요한 객체를 외부에서 전달하는 방식이 DI이다.

---

## 정리

Spring의 핵심 기술은 객체지향적인 애플리케이션을 보다 쉽게 설계할 수 있도록 도와준다.

### IoC / DI

객체가 직접 다른 객체를 생성하지 않고 Spring Container가 객체를 관리하고 필요한 의존성을 주입한다.

이를 통해 객체 간 결합도를 낮추고 테스트와 유지보수를 쉽게 만들 수 있다.

### AOP

Logging, Transaction과 같이 여러 곳에서 반복되는 부가 기능을 핵심 비즈니스 로직과 분리한다.

이를 통해 코드 중복을 줄이고 비즈니스 로직에 집중할 수 있다.

### PSA

JDBC, JPA 등의 구체적인 기술을 직접 다루기보다는 Spring이 제공하는 추상화된 인터페이스를 사용하게 한다.

따라서 구현 기술이 변경되어도 애플리케이션 코드의 변경을 최소화할 수 있다.

결국 Spring의 `IoC/DI`, `AOP`, `PSA`는 단순히 편리한 기능이라기보다

> **결합도를 낮추고, 변경에 유연하며, 테스트와 유지보수가 쉬운 객체지향적인 애플리케이션을 만들기 위한 Spring의 핵심 설계 철학**

이라고 이해할 수 있다.


# 2. Spring Bean 이 무엇이고, Bean 의 라이프사이클과 Bean Scope에 대해 조사

## Spring Bean

### Bean이란?

**Spring Bean은 Spring Container가 직접 생성하고 관리하는 객체**이다.

일반 Java에서는 객체가 필요하면 개발자가 직접 `new`를 사용해 생성한다.

```java
UserService userService = new UserService();
```

하지만 Spring에서는 `@Service`, `@Repository`, `@Component` 등의 어노테이션을 붙이면 Spring이 해당 객체를 생성하고 관리한다.

```java
@Service
public class UserService {
}
```

즉,

```text
일반 Java

개발자
  ↓
new
  ↓
객체 생성
```

```text
Spring

Spring Container
  ↓
Bean 생성
  ↓
Bean 관리
  ↓
필요한 곳에 주입(DI)
```

### Bean을 사용하는 이유

객체를 개발자가 직접 생성하고 연결하기 시작하면 객체가 많아질수록 관리가 복잡해진다.

예를 들어

```java
public class PostService {

    private PostRepository postRepository = new PostRepository();
}
```

처럼 작성하면 `PostService`가 `PostRepository`의 생성까지 책임지게 된다.

Spring에서는 다음과 같이 작성할 수 있다.

```java
@Service
public class PostService {

    private final PostRepository postRepository;

    public PostService(PostRepository postRepository) {
        this.postRepository = postRepository;
    }
}
```

```java
@Repository
public class PostRepository {
}
```

`PostService`가 직접 `PostRepository`를 만들지 않는다.

→ Spring이 `PostRepository` Bean을 생성하고
→ 필요한 `PostService`에 넣어준다.

이것이 앞에서 공부한 **DI(Dependency Injection)** 와 연결된다.

### 한 줄 정리

> **Bean = 내가 `new`해서 관리하는 객체가 아니라 Spring Container가 생성하고 관리하는 객체**

---

## Bean은 어떻게 등록될까?

Bean을 등록하는 대표적인 방법은 두 가지가 있다.

### 1. 어노테이션으로 자동 등록

```java
@Component
@Service
@Repository
@Controller
@RestController
```

이러한 어노테이션이 붙은 클래스를 Spring이 찾아 Bean으로 등록한다.

예를 들어

```java
@Service
public class PostService {
}
```

라고 작성하면 Spring이 `PostService`를 발견하고 Bean으로 등록한다.

### @Component와 다른 어노테이션의 관계

`@Service`, `@Repository`, `@Controller`는 내부적으로 `@Component`를 포함하고 있다.

```text
@Component
   │
   ├── @Service
   ├── @Repository
   └── @Controller
```

따라서 모두 Component Scan의 대상이 된다.

다만 역할을 명확하게 표현하기 위해 각각 구분해서 사용한다.

| 어노테이션             | 의미                  |
| ----------------- | ------------------- |
| `@Component`      | 일반적인 Spring 객체      |
| `@Service`        | 비즈니스 로직 담당          |
| `@Repository`     | DB 접근 담당            |
| `@Controller`     | MVC Controller      |
| `@RestController` | REST API Controller |

---

### 2. @Bean으로 직접 등록

직접 Bean으로 등록할 수도 있다.

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

`@Bean`이 붙은 메서드가 반환하는 객체를 Spring이 Bean으로 관리한다.

이 방식은 특히 **내가 직접 어노테이션을 붙일 수 없는 외부 라이브러리 객체를 Spring Bean으로 등록할 때** 많이 사용한다.

```text
@Component 계열
→ 클래스 자체를 Spring이 발견하여 Bean 등록

@Bean
→ 개발자가 직접 어떤 객체를 Bean으로 만들지 정의
```

---

## 어노테이션을 붙이면 Spring에서는 무슨 일이 일어날까?

내가 기존에 공부한 어노테이션과 Bean은 밀접하게 연결되어 있다.

예를 들어

```java
@Service
public class PostService {
}
```

라고 작성했다고 해서 Java 자체가 `PostService` 객체를 자동으로 생성하는 것은 아니다.

`@Service`는 단순히 **"이 클래스는 Service 역할을 하는 Spring Component야"라는 메타데이터**를 붙여주는 것이다.

Spring이 이 정보를 읽고 실제 Bean 등록 작업을 수행한다.

전체 흐름은 다음과 같다.

```text
@Service 작성
      ↓
Spring 실행
      ↓
@ComponentScan
      ↓
@Service가 붙은 클래스 발견
      ↓
BeanDefinition 생성
      ↓
Spring Container에 등록
      ↓
객체 생성
      ↓
필요한 의존성 주입
      ↓
Bean 사용
```

---

## ComponentScan

### @ComponentScan이란?

`@ComponentScan`은 말 그대로

> **Spring이 어떤 클래스들을 Bean으로 등록할지 찾아보는 과정**

이다.

Spring은 특정 패키지부터 시작해서 하위 패키지를 탐색하며

```java
@Component
@Service
@Repository
@Controller
```

등이 붙은 클래스를 찾는다.

---

### 그런데 @ComponentScan을 작성한 적이 없는데?

Spring Boot 프로젝트에서는 대부분 직접 작성하지 않는다.

우리가 항상 보는

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

의 `@SpringBootApplication` 안에 `@ComponentScan`이 포함되어 있기 때문이다.

`@SpringBootApplication`은 크게 다음 기능을 묶어서 제공한다.

```text
@SpringBootApplication

├── @SpringBootConfiguration
├── @EnableAutoConfiguration
└── @ComponentScan
```

따라서 `@SpringBootApplication`을 작성하는 것만으로 Component Scan이 자동으로 실행된다.

---

## Spring은 어디까지 탐색할까?

`@SpringBootApplication`이 붙은 클래스의 **패키지를 기준으로 하위 패키지까지 탐색한다.**

예를 들어

```text
com.example.project
│
├── ProjectApplication.java
│
├── controller
│   └── PostController.java
│
├── service
│   └── PostService.java
│
└── repository
    └── PostRepository.java
```

와 같은 구조라면

```java
@SpringBootApplication
public class ProjectApplication {
}
```

이 `com.example.project`에 존재하므로

```text
com.example.project.controller
com.example.project.service
com.example.project.repository
```

를 모두 탐색할 수 있다.

그래서 Spring Boot 프로젝트에서는 보통 Application 클래스를 **가장 상위 패키지에 위치시킨다.**

---

### ComponentScan 내부 과정 조금 더 깊게 보기

단순히

> "`@Service`를 찾아서 Bean으로 만든다"

정도로 끝나는 것이 아니라 내부적으로는 다음 과정이 일어난다.

```text
Spring Boot 실행
       ↓
@ComponentScan 확인
       ↓
탐색할 Base Package 결정
       ↓
Classpath의 클래스 탐색
       ↓
어노테이션 Metadata 확인
       ↓
@Component 계열인지 검사
       ↓
BeanDefinition 생성
       ↓
BeanDefinitionRegistry에 등록
       ↓
Bean 객체 생성
```

### BeanDefinition이란?

Spring이 객체를 바로 생성하기 전에

> **"이 Bean을 어떻게 만들고 관리할 것인가?"**

에 대한 정보를 먼저 저장한다.

이 정보를 `BeanDefinition`이라고 한다.

예를 들면 다음과 같은 정보가 들어간다.

```text
Bean 이름
Bean 클래스
Bean Scope
Lazy 여부
초기화 방법
소멸 방법
```

즉,

```text
@Service
   ↓
Component Scan
   ↓
BeanDefinition
   ↓
실제 Bean 생성
```

이라고 이해할 수 있다.

---

## 어노테이션과 Reflection

그렇다면 Spring은 어떻게 클래스에 `@Service`가 붙어 있다는 것을 알 수 있을까?

Java에서는 실행 중 클래스 정보를 확인할 수 있는 **Reflection**이라는 기능을 제공한다.

간단하게 표현하면 다음과 같은 작업이 가능하다.

```java
Class<?> clazz = PostService.class;

if (clazz.isAnnotationPresent(Service.class)) {
    System.out.println("Service입니다.");
}
```

Reflection을 이용하면 실행 중에

* 클래스
* 생성자
* 메서드
* 필드
* 어노테이션

등의 정보를 확인할 수 있다.

Spring은 이러한 Java의 메타데이터 처리 기능을 활용해 Bean 후보를 찾고 여러 기능을 적용한다.

---

## Bean Lifecycle

Bean도 생성되어 사용되고 사라지는 **생명주기(Lifecycle)** 를 가진다.

전체적인 흐름은 다음과 같다.

```text
Spring Container 시작
        ↓
BeanDefinition 등록
        ↓
Bean 객체 생성
        ↓
의존성 주입(DI)
        ↓
초기화
        ↓
Bean 사용
        ↓
소멸
        ↓
Spring Container 종료
```

조금 더 자세하게 보면

```text
1. Bean 객체 생성

2. 필요한 Bean 의존성 주입

3. 초기화 전 BeanPostProcessor 실행

4. @PostConstruct 등 초기화 작업

5. 초기화 후 BeanPostProcessor 실행

6. Bean 사용

7. @PreDestroy 등 종료 작업

8. Bean 소멸
```

---

## @PostConstruct / @PreDestroy

Bean이 생성된 직후나 없어지기 전에 특정 작업을 수행하고 싶을 수 있다.

```java
@Component
public class DatabaseClient {

    @PostConstruct
    public void init() {
        System.out.println("초기 연결");
    }

    @PreDestroy
    public void close() {
        System.out.println("연결 종료");
    }
}
```

## @PostConstruct

Bean이 생성되고 **의존성 주입이 완료된 후** 실행된다.

```text
Bean 생성
   ↓
DI
   ↓
@PostConstruct
   ↓
Bean 사용
```

예를 들어

* 초기 데이터 세팅
* 외부 서버 연결 준비
* 필요한 리소스 초기화

등에 사용할 수 있다.

## @PreDestroy

Bean이 제거되기 직전에 실행된다.

```text
Bean 사용
   ↓
@PreDestroy
   ↓
Bean 소멸
```

연결 종료나 리소스 정리 등에 사용할 수 있다.

---

## BeanPostProcessor

Bean Lifecycle을 공부하면서 이전에 공부한 **Proxy**와도 연결되는 부분이 있었다.

`BeanPostProcessor`는 Bean이 초기화되기 전후에 Spring이 추가 작업을 수행할 수 있게 해주는 기능이다.

```text
Bean 생성
    ↓
DI
    ↓
BeanPostProcessor
    ↓
초기화
    ↓
BeanPostProcessor
    ↓
최종 Bean
```

예를 들어 `@Transactional`과 같은 기능이 필요한 Bean은 Spring이 Bean을 후처리하면서 **Proxy 객체를 만들어 대신 등록**할 수 있다.

```text
실제 Service
     ↓
BeanPostProcessor
     ↓
Proxy 생성
     ↓
Spring Container

Controller
     ↓
Proxy
     ↓
실제 Service
```

그래서 우리가

```java
@Transactional
public void createPost() {
    postRepository.save(post);
}
```

라고만 작성해도 Proxy가 메서드 실행 앞뒤에 트랜잭션 처리를 추가할 수 있다.

```text
Proxy

Transaction 시작
      ↓
실제 createPost()
      ↓
성공 → Commit
실패 → Rollback
```

기존에 공부했던 **AOP / Proxy / Bean Lifecycle이 서로 연결되어 있다는 점이 중요하다.**

---

## Bean Scope

### Scope란?

Bean Scope는

> **Bean 객체를 언제 만들고, 어디까지 공유하고, 얼마나 오래 유지할 것인가**

를 결정한다.

대표적인 Scope는 다음과 같다.

```text
Singleton
Prototype
Request
Session
Application
```

---

## Singleton

Spring Bean의 **기본 Scope**이다.

아무것도 설정하지 않으면 Singleton으로 동작한다.

```java
@Service
public class PostService {
}
```

Spring Container 하나에서 `PostService` 객체 하나를 만들어 여러 곳에서 공유한다.

```text
             PostService Bean
                   ↑

PostController ────┤
CommentService ────┤
OtherService ──────┘
```

모두 같은 `PostService` Bean을 사용한다.

### 주의점

Singleton Bean은 여러 요청이 같은 객체를 공유하기 때문에 **변경되는 상태를 필드에 저장하지 않는 것이 중요하다.**

잘못된 예

```java
@Service
public class PostService {

    private Long userId;

    public void createPost(Long userId) {
        this.userId = userId;
    }
}
```

사용자 A와 사용자 B가 동시에 요청하면 값이 서로 덮어써질 수 있다.

따라서

```java
public void createPost(Long userId) {
    // 지역 변수로 사용
}
```

처럼 가능한 한 **Stateless하게 설계한다.**

> Spring Service가 Singleton인데 여러 사용자의 요청을 동시에 처리할 수 있는 이유도 요청 정보를 Bean의 필드에 저장하지 않고 메서드의 지역 변수 등으로 처리하기 때문이다.

---

## Prototype

Prototype Scope는 Bean을 요청할 때마다 새로운 객체를 만든다.

```java
@Component
@Scope("prototype")
public class TestBean {
}
```

```text
요청 1 → TestBean A

요청 2 → TestBean B

요청 3 → TestBean C
```

Singleton과 비교하면 다음과 같다.

```text
Singleton

요청 1 ─┐
요청 2 ─┼── 같은 Bean
요청 3 ─┘
```

```text
Prototype

요청 1 ─── Bean A
요청 2 ─── Bean B
요청 3 ─── Bean C
```

### Prototype의 특징

Spring은 Prototype Bean의

* 생성
* 의존성 주입
* 초기화

까지는 관리한다.

하지만 Bean을 전달한 이후의 **소멸까지 완전히 관리하지는 않는다.**

---

## Request Scope

HTTP 요청 하나마다 새로운 Bean을 생성한다.

```java
@Component
@RequestScope
public class RequestBean {
}
```

```text
사용자 요청 1
     ↓
RequestBean A

사용자 요청 2
     ↓
RequestBean B
```

요청이 끝나면 해당 Bean도 함께 종료된다.

---

## Session Scope

HTTP Session 하나마다 Bean 하나를 생성한다.

```java
@Component
@SessionScope
public class SessionBean {
}
```

```text
사용자 A Session
       ↓
SessionBean A

사용자 B Session
       ↓
SessionBean B
```

같은 사용자가 같은 세션을 사용하는 동안에는 동일한 Bean을 사용한다.

---

### Bean Scope 한눈에 보기

| Scope       | 객체 생성 기준         | 특징                |
| ----------- | ---------------- | ----------------- |
| Singleton   | Spring Container | 기본값, 하나의 Bean을 공유 |
| Prototype   | Bean 요청          | 요청할 때마다 새로운 객체    |
| Request     | HTTP Request     | HTTP 요청마다 새로운 객체  |
| Session     | HTTP Session     | 세션마다 새로운 객체       |
| Application | ServletContext   | 웹 애플리케이션 전체에서 공유  |

---

## 하나의 Interface를 구현한 Service가 여러 개라면?

예를 들어 결제 기능의 인터페이스가 있다고 하자.

```java
public interface PaymentService {

    void pay();
}
```

그리고 구현체가 두 개 존재한다.

```java
@Service
public class CashPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("현금 결제");
    }
}
```

```java
@Service
public class CardPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("카드 결제");
    }
}
```

이 상태에서 다음과 같이 주입하면

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring 입장에서는 문제가 발생한다.

```text
PaymentService가 필요함
       ↓

CashPaymentService ?
CardPaymentService ?

둘 중 누구를 넣어야 하지?
```

같은 타입의 Bean이 2개이기 때문에 Spring이 하나를 선택할 수 없고 일반적으로 `NoUniqueBeanDefinitionException`이 발생한다.

---

### @Qualifier

어떤 Bean을 사용할지 직접 지정하는 방법이다.

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(
        @Qualifier("CashPaymentService")
        PaymentService paymentService
    ) {
        this.paymentService = paymentService;
    }
}
```

Spring Bean의 기본 이름은 일반적으로 클래스명의 첫 글자를 소문자로 변경한 형태이다.

```text
CashPaymentService
       ↓
cashPaymentService
```

```text
CardPaymentService
       ↓
cardPaymentService
```

따라서 `@Qualifier`를 이용해서 특정 Bean을 선택할 수 있다.

---

### @Primary

여러 구현체 중에서 **기본으로 사용할 Bean**을 지정할 수도 있다.

```java
@Service
@Primary
public class CashPaymentService implements PaymentService {

    @Override
    public void pay() {
        System.out.println("현금으로 결제");
    }
}
```

그러면

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

라고만 작성해도 기본적으로 `CashPaymentService`가 주입된다.

```text
@Primary
→ 여러 Bean 중 기본값 지정

@Qualifier
→ 내가 원하는 Bean을 명확하게 지정
```

---

## 구현체를 모두 주입받을 수도 있다

하나를 선택하는 것이 아니라 모든 구현체가 필요한 경우도 있다.

이때는 `List`를 사용할 수 있다.

```java
@Service
public class PaymentManager {

    private final List<PaymentService> paymentServices;

    public PaymentManager(List<PaymentService> paymentServices) {
        this.paymentServices = paymentServices;
    }
}
```

그러면 Spring이

```text
CashPaymentService
CardPaymentService
```

를 모두 찾아 넣어준다.

```text
PaymentService 구현체 탐색
         ↓

CashPaymentService
CardPaymentService
         ↓

List<PaymentService>
```

`Map`으로도 받을 수 있다.

```java
private final Map<String, PaymentService> paymentServices;
```

이 경우 Bean 이름을 Key로 사용할 수 있다.

```text
{
    "cashPaymentService" : CashPaymentService,
    "cardPaymentService"  : CardPaymentService
}
```

결제 수단처럼 **사용자의 선택에 따라 구현체를 변경해야 하는 경우** 유용하게 사용할 수 있다.

---

## Bean 전체 흐름 한눈에 보기

Spring Boot 실행부터 Bean을 사용할 때까지 정리하면 다음과 같다.

```text
1. @SpringBootApplication 실행

            ↓

2. @ComponentScan 시작

            ↓

3. @Component / @Service / @Repository 등 탐색

            ↓

4. BeanDefinition 생성

            ↓

5. Spring Container에 등록

            ↓

6. Bean 객체 생성

            ↓

7. 필요한 의존성 확인

            ↓

8. 다른 Bean 주입 (DI)

            ↓

9. @PostConstruct 등 초기화

            ↓

10. BeanPostProcessor 처리
    → 필요한 경우 Proxy 생성

            ↓

11. 애플리케이션에서 Bean 사용

            ↓

12. @PreDestroy

            ↓

13. Spring Container 종료
```

---

## 이번 공부에서 연결된 개념

처음에는 각각 별개의 개념이라고 생각했지만 실제로는 다음처럼 연결되어 있었다.

```text
어노테이션
    ↓
@Component / @Service 등을 표시

@ComponentScan
    ↓
Bean으로 만들 클래스를 탐색

BeanDefinition
    ↓
Bean에 대한 정보 저장

Spring Container
    ↓
Bean 생성 및 관리

DI
    ↓
Bean끼리 연결

BeanPostProcessor
    ↓
추가 처리

Proxy
    ↓
@Transactional / AOP 등 적용
```

즉,

> **Spring은 어노테이션을 통해 객체의 역할을 확인하고, Component Scan을 통해 Bean 후보를 찾은 뒤, BeanDefinition을 등록한다. 이후 Bean을 생성하고 필요한 의존성을 주입하며, BeanPostProcessor와 Proxy 등을 이용해 트랜잭션이나 AOP 같은 추가 기능까지 적용한다.**

### 핵심 정리

* **Bean** : Spring Container가 생성하고 관리하는 객체
* **Component Scan** : `@Component` 계열 클래스를 찾아 Bean 후보로 등록
* **Bean Lifecycle** : 생성 → DI → 초기화 → 사용 → 소멸
* **Bean Scope** : Bean을 생성하고 공유하는 범위를 결정
* **Singleton** : Spring의 기본 Scope
* **BeanPostProcessor** : Bean 생성 과정 중 추가 처리를 담당
* **Proxy** : 실제 객체를 대신하여 트랜잭션, 로깅 등의 공통 기능을 수행
* **@Qualifier** : 같은 타입의 Bean 중 특정 Bean 선택
* **@Primary** : 같은 타입의 Bean 중 기본 Bean 지정



# 3. Spring MVC 심층 분석

앞에서 Spring Container가 Bean을 생성하고 DI를 통해 객체들을 연결하는 과정을 살펴봤다.

그렇다면 실제 사용자가

```text
GET /posts/1
```

과 같은 HTTP 요청을 보내면 Spring에서는 어떤 일이 일어날까?

Spring MVC의 전체 흐름을 크게 보면 다음과 같다.

```text
Client
  ↓
HTTP Request
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
DB
```

이번에는 이 흐름을 이해하기 위해 다음 개념을 정리했다.

* MVC Pattern
* Spring MVC
* Servlet
* Servlet Container
* WAS
* Tomcat
* DispatcherServlet
* HandlerMapping
* HandlerAdapter
* Spring MVC 전체 요청 처리 과정

---

## MVC Pattern

### MVC란?

MVC는 **Model - View - Controller**로 애플리케이션의 역할을 나누는 디자인 패턴이다.

```text
MVC

Model
Controller
View
```

각각의 역할은 다음과 같다.

#### Model

데이터와 비즈니스 로직을 담당한다.

예를 들어 게시글 조회, 회원가입, 주문 처리 등의 로직이 포함될 수 있다.

```java
Post post = postService.findById(postId);
```

### View

사용자에게 보여줄 화면을 담당한다.

대표적으로

```text
HTML
JSP
Thymeleaf
```

등이 있다.

### Controller

사용자의 요청을 받아 적절한 비즈니스 로직을 호출하고 결과를 View에 전달한다.

```java
@GetMapping("/posts/{postId}")
public String getPost(
        @PathVariable Long postId,
        Model model
) {

    Post post = postService.findById(postId);

    model.addAttribute("post", post);

    return "post";
}
```

전체 구조는 다음과 같다.

```text
사용자
  ↓
Controller
  ↓
Model
  ↓
Controller
  ↓
View
  ↓
사용자
```

---

## MVC 패턴과 Spring MVC의 차이

처음에는 MVC와 Spring MVC가 같은 개념처럼 보일 수 있다.

하지만 둘은 다르다.

### MVC

> 애플리케이션을 Model, View, Controller로 분리해서 설계하자는 **디자인 패턴**

### Spring MVC

> MVC 패턴을 Java와 Spring 환경에서 쉽게 구현하도록 도와주는 **웹 프레임워크**

즉,

```text
MVC
→ 설계 방식

Spring MVC
→ MVC 구조를 실제로 구현하기 위한 Spring 기술
```

이라고 볼 수 있다.

Spring MVC는 다음과 같은 기능을 제공한다.

```text
@Controller
@RestController
@RequestMapping
@GetMapping
@PostMapping
@RequestBody
@PathVariable
@RequestParam
```

뿐만 아니라 내부적으로

```text
DispatcherServlet
HandlerMapping
HandlerAdapter
HttpMessageConverter
ViewResolver
```

등을 제공하여 웹 요청 처리를 자동화한다.

---

## Servlet을 이해하기 전에: HTTP 요청은 어떻게 Java 코드까지 올까?

웹 브라우저에서 다음 주소를 입력했다고 해보자.

```text
http://localhost:8080/posts/1
```

브라우저는 서버에게 HTTP 요청을 보낸다.

대략 다음과 같은 형태이다.

```http
GET /posts/1 HTTP/1.1
Host: localhost:8080
```

서버 입장에서는

```text
GET 방식으로 /posts/1 데이터를 주세요.
```

라는 요청을 받은 것이다.

하지만 일반 Java 클래스는 HTTP 요청을 직접 이해하지 못한다.

```java
public class PostService {

    public void getPost() {
        // HTTP 요청은 누가 받아주지?
    }
}
```

누군가는 다음 작업을 해야 한다.

```text
네트워크 요청 받기
      ↓
HTTP 요청 분석
      ↓
GET / POST 구분
      ↓
URL 분석
      ↓
요청 데이터 추출
      ↓
Java 코드 실행
      ↓
결과를 HTTP 응답으로 변환
```

이 역할을 수행하기 위해 등장한 것이 **Servlet**이다.

---

## Servlet

### Servlet이란?

Servlet은

> **Java 서버에서 HTTP 요청을 받아 처리하고 HTTP 응답을 만들어주는 Java 기술**

이다.

쉽게 표현하면

```text
Browser
   ↓
HTTP Request
   ↓
Servlet
   ↓
Java 코드 실행
   ↓
HTTP Response
   ↓
Browser
```

라고 볼 수 있다.

---

## Servlet을 직접 만들어보면?

Servlet은 다음처럼 직접 작성할 수 있다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        response.getWriter().write("Hello Servlet!");
    }
}
```

하나씩 살펴보면 이해하기 쉽다.

---

### @WebServlet("/hello")

```java
@WebServlet("/hello")
```

는

> `/hello` 요청이 들어오면 이 Servlet이 처리한다.

라는 의미이다.

```text
GET /hello
     ↓
HelloServlet
```

---

### HttpServlet

```java
public class HelloServlet extends HttpServlet
```

HTTP 요청을 처리하는 Servlet을 만들 때 일반적으로 `HttpServlet`을 상속한다.

`HttpServlet`은 HTTP Method에 따라 요청을 처리할 수 있도록 메서드를 제공한다.

```text
doGet()
doPost()
doPut()
doDelete()
```

---

### doGet(), doPost()

HTTP Method에 따라 호출되는 메서드가 달라진다.

| HTTP Method | Servlet Method | 대표적인 의미 |
| ----------- | -------------- | ------- |
| GET         | `doGet()`      | 조회      |
| POST        | `doPost()`     | 생성      |
| PUT         | `doPut()`      | 수정      |
| DELETE      | `doDelete()`   | 삭제      |

예를 들어

```http
GET /hello
```

가 들어오면

```java
doGet()
```

이 실행된다.

반대로

```http
POST /posts
```

가 들어오면

```java
doPost()
```

가 실행된다.

---

### HttpServletRequest

Servlet에서 자주 등장하는 객체가 있다.

```java
HttpServletRequest request
```

`HttpServletRequest`는

> **클라이언트가 보낸 HTTP 요청 정보를 Java 객체로 표현한 것**

이다.

예를 들어 요청이 다음과 같다고 해보자.

```http
GET /posts?id=10 HTTP/1.1
Host: localhost:8080
User-Agent: Chrome
```

Servlet에서는 다음처럼 사용할 수 있다.

```java
String method = request.getMethod();
```

결과

```text
GET
```

Query Parameter도 가져올 수 있다.

```java
String id = request.getParameter("id");
```

결과

```text
10
```

즉,

```text
HTTP Request

GET /posts?id=10
      ↓
HttpServletRequest
      ↓
request.getMethod()
request.getParameter()
request.getHeader()
```

와 같이 처리할 수 있다.

---

### HttpServletResponse

반대로

```java
HttpServletResponse response
```

는

> **서버가 클라이언트에게 보낼 HTTP 응답 정보를 관리하는 객체**

이다.

```java
response.setStatus(200);
response.getWriter().write("Hello");
```

이렇게 작성하면 개념적으로

```http
HTTP/1.1 200 OK

Hello
```

와 같은 응답을 보낼 수 있다.

JSON도 직접 작성할 수 있다.

```java
response.setContentType("application/json");

response.getWriter().write(
        "{\"name\":\"Spring\"}"
);
```

즉,

```text
Java 코드
   ↓
HttpServletResponse
   ↓
HTTP Response
   ↓
Client
```

이다.

---

### Servlet 하나가 요청을 처리하는 전체 흐름

사용자가 다음 요청을 보낸다고 해보자.

```text
GET /hello?name=spring
```

Servlet은 다음과 같다.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        String name = request.getParameter("name");

        response.getWriter()
                .write("Hello " + name);
    }
}
```

요청 흐름은 다음과 같다.

```text
1. Browser

GET /hello?name=spring

        ↓

2. 서버가 HTTP 요청을 받음

        ↓

3. /hello를 담당하는 Servlet 탐색

        ↓

4. HelloServlet 실행

        ↓

5. doGet() 실행

        ↓

6. request에서 name 추출

name = "spring"

        ↓

7. response 작성

"Hello spring"

        ↓

8. Browser에 HTTP Response 전달
```

---

### 그런데 Servlet은 누가 실행할까?

Servlet 객체가 있다고 해서 스스로 HTTP 요청을 받을 수 있는 것은 아니다.

누군가는

```text
HTTP 요청을 기다리고

Servlet 객체를 생성하고

요청에 맞는 Servlet을 찾고

Servlet 메서드를 실행하고

응답을 다시 클라이언트에게 보내야 한다.
```

이 역할을 담당하는 것이 **Servlet Container**이다.

대표적인 Servlet Container가 바로 **Tomcat**이다.

---

### Servlet Container

Servlet Container는

> **Servlet의 생성, 실행, 생명주기와 HTTP 요청 연결을 관리하는 환경**

이다.

쉽게 말하면

> Servlet의 관리자

라고 생각할 수 있다.

Servlet Container는 다음과 같은 일을 한다.

```text
HTTP 요청 받기
       ↓
HttpServletRequest 생성
       ↓
HttpServletResponse 생성
       ↓
어떤 Servlet이 처리할지 탐색
       ↓
Servlet 실행
       ↓
HTTP Response 반환
       ↓
Servlet Lifecycle 관리
```

개발자는 이 덕분에 Socket 통신이나 HTTP 파싱을 직접 구현하지 않아도 된다.

---

## Tomcat

### Tomcat이란?

Tomcat은 Java 웹 애플리케이션에서 많이 사용하는 대표적인 **Servlet Container**이다.

Tomcat은

```text
Servlet 생성
Servlet 초기화
Servlet 실행
Servlet 종료
HTTP 요청 처리
```

등을 담당한다.

구조를 표현하면 다음과 같다.

```text
             Tomcat
        Servlet Container

          /         \
         ↓           ↓

 HelloServlet     PostServlet
    /hello          /posts
```

사용자가

```text
GET /hello
```

요청을 보내면 Tomcat이 `/hello`에 연결된 Servlet을 찾아 실행한다.

---

## Servlet Lifecycle

Servlet도 객체이기 때문에 생명주기를 가진다.

하지만 Servlet 객체는 일반적으로 개발자가

```java
new HelloServlet();
```

로 직접 생성하지 않는다.

Tomcat 같은 Servlet Container가 관리한다.

Servlet의 대표적인 생명주기는 다음과 같다.

```text
Servlet 생성
   ↓
init()
   ↓
service()
   ↓
service()
   ↓
service()
   ↓
destroy()
```

---

### init()

Servlet이 처음 생성될 때 초기화 작업을 수행한다.

```java
@Override
public void init() {
    System.out.println("Servlet 초기화");
}
```

일반적으로 Servlet 하나에 대해 한 번 실행된다.

```text
Tomcat 실행
   ↓
Servlet 생성
   ↓
init()
```

---

### service()

HTTP 요청이 들어오면 실행된다.

```text
HTTP Request
      ↓
service()
```

`HttpServlet`의 `service()`는 HTTP Method를 확인한다.

GET 요청이라면

```text
service()
   ↓
GET 확인
   ↓
doGet()
```

POST 요청이라면

```text
service()
   ↓
POST 확인
   ↓
doPost()
```

로 연결한다.

즉, 개발자가 직접 `doGet()`을 호출하는 것이 아니라 Tomcat이 요청을 받고 `service()`를 호출하면서 적절한 HTTP Method 메서드로 연결해준다.

---

### destroy()

Servlet이 제거되기 전에 실행된다.

```java
@Override
public void destroy() {
    System.out.println("Servlet 종료");
}
```

서버 종료 시 리소스 정리 등에 사용할 수 있다.

---

## Servlet 객체는 요청마다 새로 만들어질까?

아니다.

일반적으로 Servlet 객체 하나가 여러 요청을 처리한다.

```text
             Servlet

        ↑      ↑      ↑
        │      │      │
     Thread1 Thread2 Thread3

        │      │      │
        A      B      C
```

여러 사용자의 요청이 동시에 하나의 Servlet 객체를 사용할 수 있다.

따라서 다음처럼 요청 데이터를 Servlet의 필드에 저장하면 문제가 발생할 수 있다.

```java
public class TestServlet extends HttpServlet {

    private String username;

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) {

        username = request.getParameter("username");
    }
}
```

사용자 A가

```text
username = A
```

를 저장한 직후 사용자 B가

```text
username = B
```

로 덮어쓸 수 있다.

따라서 요청마다 변경되는 값은

```java
String username =
        request.getParameter("username");
```

처럼 **지역 변수로 사용하는 것이 안전하다.**

이 부분은 앞에서 공부한

> Singleton Bean은 Stateless하게 설계해야 한다.

라는 내용과도 연결된다.

---

## Servlet을 직접 사용하면 어떤 불편함이 있을까?

Servlet만으로도 웹 애플리케이션을 만들 수 있다.

하지만 요청이 많아지면 개발자가 직접 처리해야 하는 코드도 많아진다.

예를 들어 게시글 하나를 조회하려면 다음처럼 작성할 수 있다.

```java
@WebServlet("/posts")
public class PostServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response
    ) throws IOException {

        String idParam = request.getParameter("id");

        Long id = Long.parseLong(idParam);

        Post post = postService.findById(id);

        response.setContentType("application/json");

        String json =
                "{\"id\":" + post.getId()
                + ", \"title\":\""
                + post.getTitle()
                + "\"}";

        response.getWriter().write(json);
    }
}
```

개발자가 직접 해야 하는 일이 많다.

```text
Parameter 추출

String → Long 변환

Service 호출

Response Header 설정

Java Object → JSON 변환

HTTP Response 작성
```

API가 많아질수록 이런 코드가 계속 반복된다.

이를 편리하게 만들어준 것이 **Spring MVC**이다.

---

## Spring MVC에서 같은 요청 처리하기

Spring MVC에서는 다음처럼 간단하게 작성할 수 있다.

```java
@RestController
@RequestMapping("/posts")
public class PostController {

    private final PostService postService;

    @GetMapping("/{postId}")
    public PostResponse getPost(
            @PathVariable Long postId
    ) {

        return postService.getPost(postId);
    }
}
```

우리가 직접 하지 않은 일이 많다.

```text
HTTP 요청 분석
URL 분석
Servlet 탐색
String → Long 변환
Java Object → JSON 변환
HTTP Response 작성
```

이러한 공통 작업을 Spring MVC가 대신 처리한다.

---

## 그렇다면 Spring MVC에서는 Servlet이 사라졌을까?

아니다.

**Spring MVC도 Servlet 기반으로 동작한다.**

다만 개발자가 URL마다 Servlet을 직접 만들지 않는다.

대신 Spring MVC에서 제공하는 하나의 대표 Servlet이 대부분의 요청을 먼저 받는다.

바로

```text
DispatcherServlet
```

이다.

---

### DispatcherServlet도 Servlet이다

이름 그대로

```text
Dispatcher + Servlet
```

이다.

DispatcherServlet 역시 Servlet 계층 구조 안에 있다.

개념적으로 보면

```text
Servlet
   ↑
HttpServlet
   ↑
...
   ↑
DispatcherServlet
```

구조를 가진다.

즉,

> Spring MVC가 Servlet을 없앤 것이 아니라 Servlet 기술 위에 더 편리한 구조를 만든 것이다.

---

## 기존 Servlet 방식과 Spring MVC 방식

### Servlet 직접 사용

```text
/posts
   ↓
PostServlet

/users
   ↓
UserServlet

/comments
   ↓
CommentServlet
```

각 URL마다 Servlet을 만들 수 있다.

각 Servlet에서는

```text
요청 분석
Parameter 처리
응답 생성
예외 처리
```

같은 코드가 반복될 수 있다.

---

## Spring MVC

```text
/posts
/users
/comments
    │
    │
    ↓
DispatcherServlet
    │
    ├── PostController
    ├── UserController
    └── CommentController
```

모든 요청을 하나의 Servlet이 먼저 받는다.

이러한 구조를 **Front Controller Pattern**이라고 한다.

---

## Front Controller Pattern

여러 Servlet이 각각 요청을 처리하면 공통 로직이 반복될 수 있다.

```text
PostServlet

요청 처리
로그
인증
예외 처리
```

```text
UserServlet

요청 처리
로그
인증
예외 처리
```

이를 개선하기 위해 하나의 진입점을 만든다.

```text
           모든 HTTP 요청

                 ↓

         DispatcherServlet

         /       |       \
        ↓        ↓        ↓

     Post      User     Comment
  Controller Controller Controller
```

Spring MVC의 DispatcherServlet이 대표적인 Front Controller이다.

---

## WAS란?

### WAS = Web Application Server

WAS는

> **웹 애플리케이션 코드를 실행하여 동적인 요청을 처리하는 서버 환경**

이다.

예를 들어 사용자가

```text
GET /users/1
```

요청을 보낸다면 단순히 파일을 반환하는 것이 아니라

```text
Controller 실행
      ↓
Service 실행
      ↓
DB 조회
      ↓
JSON 생성
```

과 같은 작업이 필요할 수 있다.

이처럼 애플리케이션 로직을 실행하여 응답을 생성하는 역할을 WAS가 담당한다.

---

## Web Server와 WAS의 차이

### Web Server

주로 정적 데이터를 제공한다.

```text
HTML
CSS
JavaScript
Image
```

대표적인 예는

```text
Nginx
Apache HTTP Server
```

이다.

---

### WAS

애플리케이션 코드를 실행하여 동적인 응답을 만든다.

```text
HTTP Request
      ↓
Java Application 실행
      ↓
Database 조회
      ↓
HTTP Response 생성
```

대표적으로

```text
Tomcat
Jetty
Undertow
```

등이 사용된다.

다만 실제 서버 환경에서는 Web Server와 WAS의 기능이 완전히 분리되어 있는 것은 아니며 일부 기능이 겹칠 수 있다.

---

## Tomcat은 WAS일까? Servlet Container일까?

Tomcat을 설명할 때

```text
Servlet Container
WAS
Web Server
```

라는 표현이 함께 등장해서 처음에는 헷갈릴 수 있다.

가장 중요한 핵심은

> **Tomcat은 Servlet을 실행하고 관리하는 Servlet Container이다.**

라는 점이다.

또한 HTTP 서버 기능도 제공하고 Java 웹 애플리케이션을 실행할 수 있기 때문에 실무나 학습에서는 WAS라고 부르는 경우도 많다.

Spring MVC를 이해하는 단계에서는

```text
Tomcat
→ HTTP 요청을 받음
→ Servlet 실행
→ Servlet 생명주기 관리
```

라고 기억하는 것이 가장 이해하기 쉽다.

---

## Spring Boot와 Tomcat

Spring Boot 프로젝트를 실행하면 다음 로그를 본 적이 있다.

```text
Tomcat started on port 8080
```

Spring Boot에는 기본적으로 **Embedded Tomcat**이 포함되어 있다.

따라서 별도의 Tomcat 프로그램을 설치해 실행하지 않아도

```java
SpringApplication.run(Application.class, args);
```

를 실행하면 Tomcat도 함께 실행된다.

```text
Spring Boot 실행
       ↓
Spring Container 생성
       ↓
Embedded Tomcat 실행
       ↓
8080 Port 열림
       ↓
HTTP 요청 대기
```

---

## Tomcat과 Spring의 역할 구분

두 개는 서로 다른 역할을 한다.

### Tomcat

```text
HTTP 요청을 받음
Servlet을 실행
```

### Spring MVC

```text
요청을 어떤 Controller에 보낼지 결정
Controller 실행 지원
응답 변환
```

따라서

```text
Client
   ↓
Tomcat
   ↓
DispatcherServlet
   ↓
Spring MVC
   ↓
Controller
```

구조가 된다.


## DispatcherServlet

### DispatcherServlet이란?

`DispatcherServlet`은 Spring MVC의 **Front Controller** 역할을 하는 핵심 Servlet이다.

클라이언트의 HTTP 요청을 가장 먼저 받아서, 해당 요청을 처리할 Controller를 찾고 실행한 뒤 결과를 다시 응답으로 반환한다.

즉,

> **요청을 직접 처리하기보다 Spring MVC의 여러 구성요소를 연결하고 전체 요청 흐름을 조율하는 역할**

을 한다.

```text
Client
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Response
```

---

### 왜 DispatcherServlet을 사용할까?

Servlet을 직접 사용하면 URL마다 각각의 Servlet을 만들고 공통 로직을 반복해서 작성할 수 있다.

```text
/posts
  ↓
PostServlet

/users
  ↓
UserServlet
```

Spring MVC에서는 모든 요청을 `DispatcherServlet`이 먼저 받는다.

```text
/posts
/users
/comments
    ↓
DispatcherServlet
    ↓
각 Controller
```

이처럼 요청의 진입점을 하나로 모으는 방식을 **Front Controller Pattern**이라고 한다.

덕분에 요청 매핑, 파라미터 변환, 예외 처리, 응답 변환 등의 공통 기능을 Spring MVC가 일관되게 처리할 수 있다.

---

### DispatcherServlet의 핵심 메서드: doDispatch()

DispatcherServlet 내부에서 실제 요청 처리 흐름을 담당하는 핵심 메서드는 `doDispatch()`이다.

`doDispatch()`는 크게 다음 순서로 동작한다.

```text
HTTP Request
    ↓
DispatcherServlet
    ↓
doDispatch()
    ↓
HandlerMapping
    ↓
HandlerAdapter
    ↓
Controller
    ↓
응답 처리
```

---

#### 1. HandlerMapping으로 Handler 찾기

사용자가 다음 요청을 보냈다고 하자.

```text
GET /posts/1
```

DispatcherServlet은 `HandlerMapping`에게 이 요청을 처리할 Handler를 찾도록 요청한다.

```java
@GetMapping("/posts/{postId}")
public PostResponse getPost(
        @PathVariable Long postId
) {
    return postService.getPost(postId);
}
```

이 경우 `PostController.getPost()`가 Handler로 선택된다.

```text
GET /posts/1
      ↓
HandlerMapping
      ↓
PostController.getPost()
```

쉽게 말하면,

> **HandlerMapping = 어떤 Controller 메서드가 처리할지 찾는 역할**

이다.

---

#### 2. HandlerAdapter로 Handler 실행

Handler를 찾은 뒤에는 `HandlerAdapter`가 해당 Handler를 실행한다.

```text
DispatcherServlet
      ↓
HandlerAdapter
      ↓
Controller Method 실행
```

HandlerAdapter는 Controller를 실행하면서 필요한 파라미터 처리도 함께 수행한다.

예를 들어

```java
@PathVariable Long postId
```

는

```text
/posts/1
   ↓
postId = 1L
```

로 변환된다.

`@RequestBody`가 있다면 JSON 데이터를 Java 객체로 변환해서 Controller에 전달한다.

즉,

> **HandlerAdapter = 찾은 Handler를 실제로 실행하는 역할**

이다.

---

#### 3. Controller 실행

HandlerAdapter를 통해 Controller 메서드가 실행된다.

```java
@GetMapping("/posts/{postId}")
public PostResponse getPost(
        @PathVariable Long postId
) {
    return postService.getPost(postId);
}
```

Controller는 일반적으로 Service를 호출하고, Service는 Repository를 통해 DB에 접근한다.

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

---

#### 4. 결과를 HTTP Response로 변환

Controller가 Java 객체를 반환하면 Spring MVC가 이를 HTTP 응답으로 변환한다.

`@RestController`에서는 주로 `HttpMessageConverter`를 사용해 Java 객체를 JSON으로 변환한다.

```java
return new PostResponse(
        1L,
        "Spring MVC"
);
```

↓

```json
{
  "id": 1,
  "title": "Spring MVC"
}
```

전체 흐름은 다음과 같다.

```text
Controller
    ↓
Java Object
    ↓
HttpMessageConverter
    ↓
JSON
    ↓
HTTP Response
```

---

### DispatcherServlet 전체 동작 흐름

```text
1. Client

GET /posts/1

      ↓

2. Tomcat

HTTP 요청 수신

      ↓

3. DispatcherServlet

요청을 가장 먼저 받음

      ↓

4. doDispatch()

요청 처리 시작

      ↓

5. HandlerMapping

요청을 처리할 Controller Method 탐색

      ↓

6. HandlerAdapter

Handler 실행

      ↓

7. Controller

Service 호출

      ↓

8. Service

비즈니스 로직 수행

      ↓

9. Repository

DB 접근

      ↓

10. Controller 결과 반환

      ↓

11. HttpMessageConverter

Java Object → JSON

      ↓

12. DispatcherServlet

응답 반환

      ↓

13. Tomcat

      ↓

14. Client
```

---

### 핵심 역할 정리

| 구성요소                   | 역할                                 |
| ---------------------- | ---------------------------------- |
| `DispatcherServlet`    | Spring MVC 요청의 진입점이자 전체 흐름을 조율     |
| `doDispatch()`         | DispatcherServlet 내부의 핵심 요청 처리 메서드 |
| `HandlerMapping`       | 요청을 처리할 Controller Method 탐색       |
| `HandlerAdapter`       | 선택된 Handler를 실제 실행                 |
| `Controller`           | 요청에 맞는 로직 수행                       |
| `HttpMessageConverter` | Java Object ↔ JSON 변환              |

---

### 한 줄 정리

> **DispatcherServlet은 Spring MVC의 Front Controller로서 모든 요청을 먼저 받고, `doDispatch()` 내부에서 HandlerMapping으로 Controller를 찾고 HandlerAdapter를 통해 실행한 뒤, 결과를 HTTP Response로 변환하여 반환한다.**



## MVC Pattern과 Spring MVC 비교

| 구분         | MVC Pattern   | Spring MVC                       |
| ---------- | ------------- | -------------------------------- |
| 의미         | 디자인 패턴        | Spring Web Framework             |
| 목적         | 역할 분리         | MVC 기반 웹 요청 처리                   |
| Model      | 데이터 / 비즈니스 영역 | Domain, Service 등                |
| View       | 사용자 화면        | JSP, Thymeleaf 등                 |
| Controller | 사용자 요청 처리     | `@Controller`, `@RestController` |
| 요청 처리 도구   | 규정하지 않음       | DispatcherServlet 등 제공           |

---

## Web Server / WAS / Tomcat 비교

| 개념                | 역할                                      |
| ----------------- | --------------------------------------- |
| Web Server        | 주로 정적 파일과 HTTP 요청 처리                    |
| WAS               | 애플리케이션 코드를 실행하여 동적인 요청 처리               |
| Servlet Container | Servlet 생성, 실행, Lifecycle 관리            |
| Tomcat            | 대표적인 Servlet Container이며 HTTP 서버 기능도 제공 |
| Spring MVC        | Servlet 기술 위에서 동작하는 Web MVC Framework   |

---

## Servlet을 이해한 뒤 보는 DispatcherServlet

처음에는

```text
Controller가 HTTP 요청을 직접 받는다.
```

라고 생각하기 쉽다.

하지만 실제로는

```text
Browser
   ↓
Tomcat
   ↓
DispatcherServlet
   ↓
Controller
```

이다.

따라서 더 정확하게 표현하면

> **Controller가 직접 네트워크의 HTTP 요청을 받는 것이 아니라, Tomcat이 요청을 받고 DispatcherServlet이 이를 Spring MVC의 적절한 Controller에 전달한다.**

라고 볼 수 있다.

---

## 전체 학습 내용 연결하기

지금까지 공부한 Spring 개념을 하나의 요청 흐름으로 연결하면 다음과 같다.

```text
@SpringBootApplication
        ↓
Spring Container 생성
        ↓
@ComponentScan
        ↓
BeanDefinition 등록
        ↓
Controller / Service / Repository
Bean 생성
        ↓
DI
        ↓
필요한 Bean에 Proxy 적용
        ↓
Embedded Tomcat 실행
        ↓
HTTP Request
        ↓
DispatcherServlet
        ↓
HandlerMapping
        ↓
HandlerAdapter
        ↓
Controller
        ↓
Service
        ↓
@Transactional Proxy
        ↓
Repository
        ↓
Database
        ↓
HttpMessageConverter
        ↓
HTTP Response
```

즉,

```text
IoC / DI
Bean
Annotation
ComponentScan
Bean Lifecycle
Proxy
AOP
Servlet
Tomcat
Spring MVC
```

는 각각 따로 존재하는 개념이 아니다.

**Spring Boot Application 하나가 실행되고 HTTP 요청 하나를 처리하는 과정 속에서 모두 연결되어 동작한다.**



## 한 줄 정리

> **클라이언트가 HTTP 요청을 보내면 Tomcat이 요청을 받아 `HttpServletRequest`와 `HttpServletResponse`를 준비하고 Spring MVC의 `DispatcherServlet`을 실행한다. DispatcherServlet은 HandlerMapping과 HandlerAdapter 등을 이용해 적절한 Controller를 호출하고, 이후 Service와 Repository를 거쳐 비즈니스 로직과 DB 작업을 수행한다. 결과는 HttpMessageConverter 등을 통해 HTTP 응답으로 변환되어 다시 클라이언트에게 전달된다.**

Servlet은 Spring MVC 때문에 새롭게 등장한 기술이 아니라 **Java 웹 개발의 기본 HTTP 요청 처리 기술**이고, Spring MVC는 Servlet 위에서 복잡한 요청 처리 과정을 추상화해 개발자가 Controller와 비즈니스 로직에 집중할 수 있도록 만들어준 구조라고 이해할 수 있다.

