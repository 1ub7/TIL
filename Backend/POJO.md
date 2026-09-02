# POJO
**POJO(Plain Old Java Object)란?** 필드와 Getter, Setter와 같은 기본 기능만을 갖는 기본 객체를 의미한다. 즉, **Java로 생성하는 순수한 객체를 뜻한다.** 프레임워크에 종속되 있는 상태로 개발하지 않는 개념을 위해 등장한 개념이다. 즉, **필요에 따라 재활용될 수 있는 방식으로 설계된 오브젝트이다.**

**종속성**으로 인해 시스템 업그레이드, 모듈 교체와 같은 상황에서 많은 불편함이 있었기에 POJO라는 개념이 등장했다. 

# 사용 이유
**재사용성 & 이식성** : 특정 인터페이스 구현이나 특정 부모 클래스 상속을 강제하지 않기 때문에 이 객체를 다른 프로젝트나 다른 레이어(도메인 로직, DTO, 테스트 코드 등)에 그대로 가져다 써도 문제가 없습니다.

**테스트 용이성** : 프레임워크 컨테이너를 띄우지 않아도 new 키워드로 바로 인스턴스를 만들어 단위 테스트를 돌릴 수 있습니다. **DevOps 관점**에서도 CI 파이프라인에서 빠르게 유닛 테스트를 돌리는 데 이 성질이 직접적으로 도움이 됩니다.

**관심사 분리 & 가독성** : 객체가 데이터와 그 데이터에 관련된 로직만 담당하고, 트랜잭션 처리나 영속성 같은 인프라 로직은 외부(서비스 계층, 프레임워크)에 위임하므로 코드가 깔끔해지고 유지보수가 쉬워집니다.

# POJO 규칙
**특정 프레임워크 클래스를 상속하지 않음** : 프레임워크 제공 클래스를 extends 하면 안된다.

**특정 프레임워크 인터페이스를 구현하도록 강제되지 않음** : 프레임워크가 요구하는 인터페이스를 구현할 필요가 없어야 한다.

**특정 어노테이션에 종속되지 않음** : 원칙상으로는 어노테이션이 붙지 않아야 순수하다고 보지만, 실제로는 **@Entity, @Data** 같은 어노테이션이 붙은 객체도 실무에서는 관례적으로 POJO라고 부른다.

# 코드 예제
아래 코드는 POJO를 잘 지키고 단순한 Getter와 Setter 개념을 아주아주 잘 지킨 예시이다.
```java
public class POJOClass {
    private String name;
    private int age;

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

아래 코드는 POJO를 무시하고 짠 예시이다.
```java
public class POJOClass extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        // 서블릿 컨테이너 없이는 테스트도, 재사용도 못 함
    }
}
```

아래 코드는 Spring의 POJO 개념을 잘 지킨 예시이다. **@Autowired**를 활용해 의존성 주입을 받으면, UserRepository를 상속하거나 구현하지 않고도 필드로 주입받아 그 기능을 사용할 수 있다.
```java
@Service
public class POJOClass {
    @Autowired
    UserRepository userRepository;

    public void test() {
        userRepository.findAll();
    }
}
```