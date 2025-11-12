# Spring Transaction Management

Spring Framework가 제공하는 선언적, 프로그램적 트랜잭션 관리 방식의 특징과 역사적 배경을 정리한 문서입니다.

## 선언적 트랜잭션 관리 (@Transactional)

`@Transactional` 어노테이션을 통해 비즈니스 로직과 트랜잭션 제어 로직을 완전히 분리하는 AOP 기반의 트랜잭션 관리 방식입니다.

### 탄생 배경: EJB의 한계

*   과거 JTA/EJB 기반 트랜잭션 관리는 XML 설정과 컨테이너 종속성으로 인해 복잡했습니다.
*   개발자가 `UserTransaction` 객체를 직접 생성하고 `begin()`, `commit()`, `rollback()`을 수동으로 호출해야 했습니다.
*   이로 인해 비즈니스 로직과 인프라 로직이 혼재되어 유지보수가 어려웠습니다.

### 스프링의 혁신과 생태계 영향

*   Spring은 AOP를 기반으로 `@Transactional` 어노테이션만 선언하면 프록시가 트랜잭션을 자동 관리하는 모델을 도입했습니다.
*   이를 통해 컨테이너 없이 순수 자바 코드로 테스트가 가능해졌고, JDBC, JPA, Hibernate 등 다양한 기술을 일관된 방식으로 지원할 수 있게 되었습니다.
*   이러한 변화는 DDD, TDD, 클린 아키텍처와 같은 모던 개발 패러다임 확산에 결정적인 영향을 미쳤습니다.

> **철학적 의미**: "개발자가 트랜잭션을 관리하는 것이 아니라, 트랜잭션이 개발자를 대신 관리한다."

## 프로그램적 트랜잭션 관리

선언적 방식 외에 코드 기반으로 트랜잭션을 직접 제어하는 방식으로, 세밀한 제어가 필요할 때 유용합니다.

### 1. JDBC 직접 관리

```java
try (Connection con = dataSource.getConnection()) {
  con.setAutoCommit(false);
  // ... 비즈니스 로직 ...
  con.commit();
} catch (Exception e) {
  con.rollback();
}
```
*   가장 낮은 수준의 접근 방식으로, 단일 리소스 제어에 적합하지만 유지보수성이 낮습니다.

### 2. JPA `EntityTransaction`

```java
EntityTransaction tx = em.getTransaction();
tx.begin();
// ... 비즈니스 로직 ...
tx.commit();
```
*   JPA 표준 API로, 단일 데이터베이스 트랜잭션에는 적합하지만 다중 리소스 제어는 제한적입니다.

### 3. JTA `UserTransaction`

```java
@Resource
UserTransaction utx;
utx.begin();
// ... 비즈니스 로직 ...
utx.commit();
```
*   분산 트랜잭션(2PC)을 위한 고수준 API지만, 설정 복잡도와 비용이 큽니다.

### 4. Spring `TransactionTemplate`

```java
transactionTemplate.execute(status -> {
  repo.save(...);
  service.doSomething();
  return result;
});
```
*   Spring이 제공하는 콜백 기반의 프로그램적 트랜잭션 관리 방식으로, 선언적 방식과 혼용하여 조건부 롤백 등 세밀한 제어를 구현할 때 유용합니다.
