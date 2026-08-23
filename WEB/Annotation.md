# Annotation
**Annotation란?** 코드에 붙이는 메타데이터이다. @로 시작하는 표시인데 어노테이션 자체는 아무 로직도 실행하지 않는다는 점이다. 

- 주석(//, /**/) : 컴파일 시점에 통째로 무시된다. 오직 사람이 읽으라고 남기는 것이다.
- 어노테이션(@) : 컴파일된 클래스 파일(.class)에 정보로 그대로 남아있고 프로그램잉 실행되는 동안 누군가 그 정보를 실제로 읽어서 동작을 바꾼다.

# Controller
**@RestController** : @Controller + @ResponseBody가 합쳐진 것이다. 메서드 리턴값이 뷰 이름이 아니라 응답 바디가 된다. REST API 만들 때는 거의 항상 이걸 쓴다.

**@RequesMapping** : 클래스 레벨에 공통 경로를 지정할 때 쓴다.

**@RequestBody** : 요청 바디를 DTO 객체로 자동 변환한다. 주로 POST/PUT에서 쓴다.

**@GetMapping / @PostMapping / @PutMapping / @PatchMapping / @DeleteMapping** : HTTP 메서드별 매핑이다. 실무에서는 거의 이 축약형만 쓴다.

**@PathVariable** : URL 경로에 있는 값을 변수로 받는다.

**@RequestParam** : 쿼리 파라미터를 받는다.

**@Valid / @Validated** : @RequesBody로 받은 DTO에 붙여서 DTO 안의 @NotNull 같은 검증 어노테이션이 실제로 동작하게 만든다. 이거 빼먹으면 DTO에 검증 어노테이션을 아무리 붙여놔도 무시된다.

# DTO
**@Getter / @Setter** : getter/setter 자동 생성 어노테이션이다. 

**@NoArgsConstructor / @AllArgsConstructor / @Builder** : @Builder는 필드가 많은 DTO 때 new Dto(a, b, c, d)처럼 순서를 외워야 하는 실수를 막아주고 어떤 필드에 어떤 값을 넣는지 코드만 보고 바로 알 수 있게 해준다.

# Service
**@Service** : 이 클래스가 비즈니스 로직을 담당하는 스프링 빈이라는 표시이다. @Component의 특수한 형태라 기능은 동일하지만 이건 서비스 계층이라는 의미를 코드에 명시하는 역할이다.

**@Transactional** : 메서드(또는 클래스) 안의 여러 DB 작업을 하나의 트랜잭션으로 묶어서 중간에 실패하면 전부 롤백되게 만든다. **Service 계층에서 가장 중요한 어노테이션이다.** Controller가 아니라 반드시 Service에 붙여야한다. 그리고 조회만 하는 메서드에는 @Transactional(readOnly = true)를 붙여서 성능 최적화를 할 수 있다.

**@Slf4j** : 로깅용 log 객체를 자동 생성한다. Service 계층에서 예외 발생 지점이나 주요 로직 흐름을 로그로 날길 때 쓴다.

# Repository
**@Repository** : 이 인터페이스가 데이터 접근 계층 빈이라는 표시이다. DB 관련 예외를 스프링의 공통 예외로 변환해주는 역할을 하기도 한다.

**extends JpaRepository<Entity, ID>** : 어노테이션은 아니나, 상속하는 순간 save, findById, findAll, delete 같은 기본 CRUD가 구현 없이 바로 생긴다.

**@Query** : 메서드 이름만으로 표현하기 복잡한 쿼리를 JPQL 또는 네이티브 SQL로 직접 작성할 때 쓴다.

**@Param** : @Query 안에서 ':파라미터명'으로 참조할 변수를 메서드 인자와 연결한다.

# Entity
**@Entity** : 이 클래스가 DB 테이블과 매핑되는 JPA 엔티티임을 표시한다.

**Table(name="...")** : 클래스명과 실제 테이블명이 다를 때 명시적으로 지정한다.

**@Id** : 기본키(PK) 필드를 지정한다.

**@Column** : 컬럼명이 필드명과 다르거나 'nullable = false'같은 제약조건을 지정할 때 쓴다.

**@JoinColumn** : 외래키(FK) 컬럼명을 지정한다.