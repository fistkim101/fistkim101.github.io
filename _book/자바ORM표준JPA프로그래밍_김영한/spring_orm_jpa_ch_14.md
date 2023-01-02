---
layout: default
title: 컬렉션과 부가 기능
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 14
---

- JPA와 컬렉션
  - JPA가 제공하는 래퍼 컬렉션
  - 래퍼 컬렉션별 차이점
- @EntityGraph

## JPA와 컬렉션

### JPA가 제공하는 래퍼 컬렉션
엔티티간의 관계에서 일대다 혹은 다대다 등 multiple 의 관계가 있을 수 있다. 이때 자바의 List, Set을 사용하게 되는데 이 컬렉션이 JPA 에서 어떤식으로 동작하는지 알아본다.
대표적으로 사용될 수 있는 List, Set 두 가지만 자세히 살펴보자.
```java
@OneToMany(mappedBy = "team", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
private Set<Member> members = new HashSet<>();
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Team team = new Team();
        System.out.println(team.getMembers().getClass().getName()); // java.util.HashSet
        entityManager.persist(team);
        System.out.println(team.getMembers().getClass().getName()); // org.hibernate.collection.internal.PersistentSet
    }
```

members로 Set 을 사용중인데 위와 같이 영속화 되기 전에 HashSet 이었던 클래스가 PersistentSet 으로 바뀌었다. 하이버네이트가 기존 컬렉션을 래핑한 것이다.
List, Set 각각 다른 래퍼 클래스를 제공한다.

{: .point }
*List -> PersistentBag<br>
*Set -> PersistentSet

### 각 래퍼 컬렉션별 차이점
내가 인지해야할 중요한 사항은 각 래퍼 컬렉션별로 내부적으로 무엇이 다른가이다. 결론부터 정리하자면 중복 허용/비허용 차이로 인한 지연로딩 여부가 큰 차이점이다.

```java
    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<Member> members = new ArrayList<>();
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Team team = entityManager.find(Team.class, 1L);

        Member member = new Member();
        member.setName("sameName");

        System.out.println("========================");
        team.getMembers().add(member);
        System.out.println("========================");

        entityManager.persist(member);
    }
```
```bash
Hibernate: 
    select
        team0_.id as id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        team team0_ 
    where
        team0_.id=?
========================
========================
Hibernate: 
    insert 
    into
        member
        (name, team_id) 
    values
        (?, ?)

```

위와 같이 List 를 사용할 경우 중복을 허용하기 때문에 add 시점에 중복되는 객체인지 검사할 필요가 없어서 프록시 객체의 초기화가 발생하지 않는다.

반면에 아래와 같이 Set을 사용할 경우 중복을 막아야하기 때문에 add 시점에 해당 객체의 중복검사를 위해서 기존 데이터를 알고자 프록시 객체의 초기화가 발생한다.
```java
    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private Set<Member> members = new HashSet<>();
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Team team = entityManager.find(Team.class, 1L);

        Member member = new Member();
        member.setName("sameName");

        System.out.println("========================");
        team.getMembers().add(member);
        System.out.println("========================");

        entityManager.persist(member);
    }
```
```bash
Hibernate: 
    select
        team0_.id as id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        team team0_ 
    where
        team0_.id=?
========================
Hibernate: 
    select
        members0_.team_id as team_id3_0_0_,
        members0_.id as id1_0_0_,
        members0_.id as id1_0_1_,
        members0_.name as name2_0_1_,
        members0_.team_id as team_id3_0_1_ 
    from
        member members0_ 
    where
        members0_.team_id=?
========================
Hibernate: 
    insert 
    into
        member
        (name, team_id) 
    values
        (?, ?)
```

이 과정에서 equals() 와 hashcode() 가 사용된다.

## @EntityGraph
한번 조회 할때 연관관계를 가진 entity를 프록시 객체가 아니라 entity 로 할당하기 위해서는 fetch join 을 사용하면 되었다.
@EntityGraph 를 이용해도 조회 시점에 바로 entity를 가져올 수 있다.

사용할 일이 있으면 [공식문서](https://docs.spring.io/spring-data/data-jpa/docs/current/reference/html/#jpa.entity-graph)를 참고하면 될 것 같다. 
