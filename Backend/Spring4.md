# IoC
**IoC(Inversion of Control)란?** 객체의 생성, 생명주기 관리, 의존관계 설정을 개발자가 직접 코드로 하지 않고 프레임워크(컨테이너)가 대신 처리하는 원칙이다.

기존 방식
```java
public class OrderService {
    private UserRepository userRepository = new UserRepositoryImpl();
}
```

IoC 적용
```java
public class OrderService {
    private UserRepository userRepository;
}
```

# DI
**DI(Dependency Injection)이란?** 객체가 필요로 하는 의존 객체를 스스로 생성하지 않고 외부로부터 주입받는 방식이다. 즉, IoC를 실현하는 구체적 기법이다.

```java
// 1. 필드 주입 - 간단하지만 안티패턴에 가까움
@Service
public class OrderService {
    @Autowired
    private UserRepository userRepository;
}
```

```java
// 2. 세터 주입 - 선택적 의존성에 적합, 런타임에 변경 가능
@Service
public class OrderService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

```java
// 3. 생성자 주입 - 현재 스프링 공식 권장 방식
@Service
public class OrderService {
    private final UserRepository userRepository;

    @Autowired
    public OrderService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

## 왜 생성자 주입인가?
**final 사용 가능** : 생성자에서 값이 채워지므로 필드에 final을 붙일 수 있어 불변성이 보장되고 재할당이 방지된다.

**순환 참조 문제** : 필드/세터 주입은 실행 후에야 오류가 터지지만, 생성자 주입은 앱이 뜨는 시점에 바로 감지되어 빨리 잡힌다.

**테스트하기 편함** : new로 바로 인스턴스를 생성할 수 있어 스프링 없이도 단위 테스트가 가능하다.

# AOP
**AOP(Aspect Oriented Programming)이란?** 로깅, 트랜잭션, 보안 검사처럼 여러 비즈니스 로직에 공통으로 걸쳐있는 관심사(cross-cutting concern)를 핵심 로직과 분리해서 모듈화하는 방법이다.

AOP 없이 짜면 생기는 문제 : 
```java
public void createOrder() {
    log.info("시작"); 
    long start = System.currentTimeMillis(); 

    orderRepository.save(order);
    
    log.info("종료, 소요시간: " + (System.currentTimeMillis() - start)); // 로깅
}
```

```java
@Aspect
@Component
public class LoggingAspect {
    @Around("execution(* com.example.service.*.*(..))")
    public Object logging(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("시작: " + joinPoint.getSignature());
        long start = System.currentTimeMillis();
        
        Object result = joinPoint.proceed(); // 실제 메서드 실행
        
        log.info("종료, 소요시간: " + (System.currentTimeMillis() - start));
        return result;
    }
}
```

## 핵심 용어
**Aspect** : 공통 관심사를 하나로 묶어놓은 모듈이다. 로깅 기능을 모아놓은 클래스, 트랜잭션 처리를 모아놓은 클래스도 포함된다.

**Advice** : 실제로 실행되는 부가 로직 코드를 말한다. 

**Pointcut** : "이 Advice를 어디에다 적용할 거냐"를 지정하는 표현식이다. 

**JoinPoint** : Advice가 낄 수 있는 지점들 자체를 말한다. 메서드가 호출되는 시점이라던지 예외가 던져지는 시점 같은 것들을 말한다.

# PSA
**PSA(Portable Service Abstraction)이란?** 구체적인 기술 구현체가 바뀌어도 개발자가 사용하는 인터페이스(API)는 동일하게 유지되도록 스프링이 추상화 계층을 제공하는 것이다.

**@Transactional** : JDBC든 JPA든 MyBatis든 상관없이 @Transactional만 붙이면 동일하게 트랜잭션 처리가 된다. 내부적으로 구현체만 갈아 끼워진다.

**@RequestMapping / @GetMapping** : 서블릿(Tomcat)이든 리액티브(WebFlux)든 컨트롤러 코드 작성 방식이 동일하다.