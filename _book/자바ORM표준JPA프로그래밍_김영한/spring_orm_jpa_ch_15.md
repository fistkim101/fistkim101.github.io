---
layout: default
title: 고급 주제와 성능 최적화
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 15
---

- 스프링 프레임워크의 JPA 예외 변환
- 엔티티 비교
  - 동일성 vs 동등성
  - 영속성 컨텍스트가 같을때
  - 영속성 컨텍스트가 다를때
- 영속성 컨텍스트와 프록시
- 읽기 전용 쿼리 (@QueryHint, @Transactional(readOnly = true))
  - 읽기 전용 쿼리 의미와 목적
  - @QueryHint, @Transactional(readOnly = true) 의 차이
- 배치처리와 성능 최적화

## 스프링 프레임워크의 JPA 예외 변환
스프링 프레임워크에서 JPA 에서 정의한 예외들을 직접 사용하게 되면 JPA 에 의존하는 형태가 된다. 그래서 JPA 예외들이 발생하면 각 예외마다 이에 맞는
스프링의 예외로 변환해서 throw 할 수 있다.

PersistenceExceptionTranslationPostProcessor 를 bean으로 등록해두면 알아서 이를 변환해준다.
스프링부트에는 이미 이것이 bean으로 등록되어 있다.

[어떤 JPA 예외가 어떤 스프링 예외로 변환되는지 정리된 포스팅](https://milenote.tistory.com/152)이 있어 링크를 남긴다.
그냥 스프링부트를 사용하면서 마주치는 JPA 에서의 에러는 이미 변환기에 의해서 변환된 스프링 예외이며 그 과정은 자동으로 bean으로 등록된
PersistenceExceptionTranslationPostProcessor 가 해준다라는 사실만 기억하면 될 듯 하다.

## 엔티티 비교
### 동일성 vs 동등성 vs 데이터베이스 관점에서의 동등성
* 동일성 : identical, == 비교
* 동등성 : equivalent, equals() 비교
* 데이터베이스 관점에서의 동등성 : pk가 같다

### 영속성 컨텍스트가 같을때
1차 캐시에서 이미 조회된 것이기에 같은 주소값을 할당해준다. 따라서 동일성, 동등성 모두 같다.

### 영속성 컨텍스트가 다를때
동일성은 같지 않다. 주소값이 다르기 때문이다. 하지만 동등성은 같다. 같은 row를 대상으로 한 데이터이기 때문이다.

## 영속성 컨텍스트와 프록시
예제에선 하이버네이트를 사용했기 때문에 이러한 비교를 해본 것인데 실제로는 Spring Data JPA를 사용할 것이라서 이 단계까지 학습할 필요성이 있을지는 모르겠다.

entityManager.find(), entityManager.getReference() 두 가지 메소드를 두고서 무엇을 먼저 실행해서 결과를 가져왔는지에 따라서
처음에 find()로 entity를 가져왔다면 getReference()로 조회해도 entity가 할당된다. 반면에, 처음에 getReference()로 프록시를 가져왔다면
find()로 조회해도 프록시가 할당된다.

{: .point }
영속성 컨텍스트는 자신이 관리하는 영속 엔티티의 동일성을 보장한다. 다시말해 영속성 컨텍스트는 한 번 프록시로 노출한 엔티티는 계속 프록시로 노출한다.
그래야 사용하는 입장에서 프록시인지 아닌지 구분하지 않고 사용할 수 있기 때문이다.

## 읽기 전용 쿼리 (@QueryHint, @Transactional(readOnly = true))
### 읽기 전용 쿼리 의미와 목적
말 그대로 쿼리가 동작할때 '읽기 전용'으로만 동작하도록 하는 것이 읽기 전용 쿼리이다.

그렇다면 읽기만 전용으로 동작한다면 그 외에 무엇이 동작을 안한다는 것인가? <b>1차 캐시 및 스냅샷 저장을 하지 않는다는 것이다.</b>

### @QueryHint, @Transactional(readOnly = true) 의 차이
@QueryHint 를 사용할 경우 스냅샷을 저장하지 않기 때문에 메모리를 절약할 수 있다.
```java
@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Team findByName(String name);
```
```java
@Override
@Transactional
public void run(ApplicationArguments args) throws Exception {
    Team team = teamRepository.findByName("sampleTeamName");
    System.out.println(team.getName()); // sampleTeamName
    team.setName("newNameTest");
}
```

위와 같이 데이터를 변경해도 update 가 발생하지 않는다. findByName 의 @QueryHint 를 삭제하면 Update 문이 수행된다.

@Transactional(readOnly = true) 을 사용할 경우 기본적으로 트랜잭션 종료 시점에 flush()를 사용하지 않아서, dirty checking과 쓰기 지연 SQL의 flush 가 발생하지 않는다.
[이 경우에 대한 인프런에 올라온 질문](https://www.inflearn.com/questions/31497/queryhint%EC%9D%98-readonly-%EC%99%80-transaction%EC%9D%98-readonly-%EC%B0%A8%EC%9D%B4)을 보면
@Transactional(readOnly = true) 을 해주면 스냅샷도 만들지 않는다고 한다.

```java
@Override
@Transactional(readOnly = true)
public void run(ApplicationArguments args) throws Exception {
    Team team = teamRepository.findByName("newNameTest");
    System.out.println(team.getName()); // newNameTest
    team.setName("change!!!");
}
```

findByName 의 @QueryHint 를 삭제한 상태로 위 로직 수행시 Update 문이 수행되지 않는다.

## 배치처리와 성능 최적화
IDENTITY 전략은 persist 시점에 pk 가 필요하기 때문에 쓰기 지연을 할 수 없고 바로 insert 가 실행된다.
그래서 [Jdbctemplate의 batchUpdate()](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html#batchUpdate(java.lang.String...)) 를 이용해서 수행하는 방식이 현재로서는 최선이다.

{: .point }
Issue multiple SQL updates on a single JDBC Statement using batching.
Will fall back to separate updates on a single Statement if the JDBC driver does not support batch updates.
