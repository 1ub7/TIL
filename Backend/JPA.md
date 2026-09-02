# JPA
**JPA(Java Persistence API)란?** Java EE(Jakarta EE) 표준 스펙으로 스프링과 무관하게 독립적으로 존재하는 인터페이스이다. 스프링은 이 JPA를 가져다 쓰면서 그 위에 Spring Data JPA라는 추상화 계층을 얹어 PSA 철학
(구현체가 바뀌어도 코드는 동일하게 유지)을 실현한 것뿐이다.

ORM이기 때문에 자바 클래스와 DB테이블을 매핑한다(SQL을 매핑하지 않음). **ORM을 사용하기 위한 인터페이스를 모아둔 것**이며 JPA를 사용하기 위해서는 JPA를 구현한 Hibernate, EclipseLink, DataNucleus같은 ORM 프레임워크를 사용해야한다.

## Entity
**Entity란?** 데이터베이스 테이블 하나와 매핑되는 자바 클래스이다. 그냥 매핑만 되는 게 아니라 JPA가 이 객체를 실제로 관리하겠다고 인식하는 대상이 되는 것이다. 

**Entity의 최소 조건** : 
- **@Entity 어노테이션 필수** : 이게 있어야 이 클래스가 테이블이랑 매핑이 되는지 안되는지를 안다.
- **@Id로 기본키를 반드시 지정** : 어떤 필드가 PK 역할을 하는지 표시해줘야한다. 이게 없다면 JPA가 예외를 던진다. 
- **클래스 자체가 final이면 안되고, 필드도 final이면 안됨** : final 클래스는 상속이 안되기 때문에 프록시를 만들 수가 없다.

## Entity Manager
**Entity Manager란?** JPA에서 Entity를 관리하는 핵심 인터페이스이다. JPA를 통해 데이터베이스와 상호작용하며 **Entity의 생명주기를 관리하고 쿼리를 실행하는 역할**을 수행한다.

## Persistence Context
**Persistence Context(영속성)이란?** JPA에서 Entity를 효율적으로 관리하기 위해 제공하는 핵심 개념이다. 즉, Entity 객체들을 관리하는 **논리적인 캐시** 또는 **작업 공간**이다.

# ORM
**ORM(Object-Relational Mapping)이란?** 객체(Class)와 (R)DB(Relational DataBase)의 테이블이 매핑을 이루는 것을 말한다. 즉, **객체가 테이블이 되도록 자동으로 영속화 해주고 매핑 시켜주는 것**을 말한다. ORM을 이용하면 SQL Query가 아닌 직관적인 코드(메서드)로서 데이터를 조작할 수 있습니다.

예를 들어, User 테이블의 데이터를 출력하기 위해서 MySQL에서는 `SELECT * FROM user;` 라는 query를 실행해야하지만, ORM을 사용하면 User 테이블과 매핑된 객체를 user라 할 때, `user.findAll()` 라는 메서드 호출로 데이터를 조회 가능하다.

# JPA를 사용해야하는 이유
생산성 향상 : SQL을 직접 작성하지 않고 **자바 객체(Entity) 중심으로 코드를 짤 수 있어서**, 반복적인 CRUD 코드 작성 부담이 크게 줄어든다.
유지보수성 향상 : **Entity와 테이블 컬럼이 자동으로 매핑되기 때문**에 테이블 구조가 바뀌어도 관련 코드 수정 범위가 최소화한다.
DB 종속성 감소 : JPA는 특정 DB에 종속된 기술이 아니라 **표준 인터페이스**라서 DB 종류가 바뀌어도 코드 변경 없이 그대로 사용할 수 있다.
