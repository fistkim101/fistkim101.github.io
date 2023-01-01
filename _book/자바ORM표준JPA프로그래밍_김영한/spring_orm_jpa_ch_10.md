---
layout: default
title: 객체지향 쿼리 언어
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 10
---

- JPQL 이란? (의미, 흐름 등)
- fetch join
  - fetch join 이란
  - fetch join 의 장점
  - fetch join 과 일반 join 의 차이
  - fetch join 의 한계와 해결책
- N+1 문제해결
  - N+1 문제란
  - 어떻게 해결하나 (방법은 세 가지)
    - EAGER 전략(아예 사용하지 말자)
    - fetch join  
    - default_batch_fetch_size, @BatchSize
- Criteria(다른 강의에서 이미 정리했기에 생략)
- Query Dsl(다른 강의에서 이미 정리했기에 생략)

이번 단원은 대체로 이전에 백기선님 강의에서 다뤘던 개념들이 대부분이어서 정리할 포인트가 많지 않았다.
다만, fetch join 에 대해서 자세히 다루고 있었어서 그 부분을 중점적으로 봤다.

## JPQL 이란? (의미, 흐름 등)
Java Persistence Query Language의 줄임말로 아래와 같은 특징을 가진다.

{: .point }
*SQL이 데이터베이스 대상이라면 JPQL은 엔티티 객체를 대상으로 한다.<br>
*JPQL은 결국 SQL 로 변환되어 실행된다.<br>
*JPQL은 SQL을 추상화해서 특정 데이터베이스에 의존하지 않기 때문에 여러 데이터베이스의 방언에 맞게 SQL로 변경된다.

책에서 나온 예제에서는 entityManager 를 통해서 JPQL 을 사용하는 예제가 많았다. [책의 예제를 잘 정리해둔 포스팅](https://leveloper.tistory.com/103) 의 링크를 남긴다.

Spring Data JPA 사용시 [@Query](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.at-query) 에서 자주 사용하게 된다.

## fetch join

### fetch join 이란
[fetch 라는 단어 뜻](https://en.dict.naver.com/#/entry/enko/855af04f147b476a88fad35a84e3a5ba) 이 'to go after and bring back (someone or something)'이다.
여태 일을 하며 fetch라는 단어를 많이 접했었는데, 막상 순수하게 단어 뜻을 보니 왜 fetch라고 사용되어 왔었는지 이해가 바로 되었다. (역시 근본적인 이해는 참 중요하다)

책의 표현을 빌리면 '연관된 엔티티나 컬렉션을 한번에 조회하는 기능'이다.

### fetch join 의 장점
fetch 전략을 LAZY로 해두었다고 해도 fetch join 으로 조회를 할 경우 해당 엔티티를 프록시 객체가 아닌 실제 엔티티로 할당한다.
실제 엔티티로 할당되기 때문에 그 엔티티를 반드시 사용할 경우라면 굳이 한번 더 쿼리를 수행할 필요가 없이(LAZY하게 가져올 필요 없이) 최초 한번으로 다 가져올 수 있기에 성능상 유리하다.

{: .warning }
<b>EAGER 전략으로 얻는 이점보다 부작용 및 우려가 크기 때문에 모든 글로벌 fetch 전략을 LAZY로 설정해두고 EAGER 하게 가져와야할 경우에 fetch join 을 사용해주자.</b> 

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member as m join fetch m.team where m.team.name = :teamName")
    List<Member> findAllByTeamNameFetchJoin(@Param(value = "teamName") String teamName);

    List<Member> findAll();

}
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        List<Member> members = memberRepository.findAll();
        System.out.println(members.get(0).getTeam().getName());
        System.out.println(members.get(0).getTeam().getClass().getName());
    }
```
```bash
Hibernate: 
    select
        member0_.id as id1_0_,
        member0_.name as name2_0_,
        member0_.team_id as team_id3_0_ 
    from
        member member0_

Hibernate: 
    select
        team0_.id as id1_1_0_,
        team0_.name as name2_1_0_ 
    from
        team team0_ 
    where
        team0_.id=?

com.fistkim.springjpawhiteshipstudy.Team$HibernateProxy$4OTJPoST
```

LAZY 를 기본 fetch 전략으로 선택했기 때문에 쿼리가 두번 요청된 것을 볼 수 있고, Team 이 프록시 객체가 할당 된 것을 알 수 있다.
반면 fetch join을 사용하면 아래와 같이 Entity 가 나온다.

```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        List<Member> members = memberRepository.findAllByTeamNameFetchJoin("NewTeam");
        System.out.println(members.get(0).getTeam().getName());
        System.out.println(members.get(0).getTeam().getClass().getName());
    }
```
```bash
Hibernate: 
    select
        member0_.id as id1_0_0_,
        team1_.id as id1_1_1_,
        member0_.name as name2_0_0_,
        member0_.team_id as team_id3_0_0_,
        team1_.name as name2_1_1_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.id 
    where
        team1_.name=?

com.fistkim.springjpawhiteshipstudy.Team
```

inner join 으로 한 번의 쿼리로 Team 까지 가져왔고, Team 객체 역시 프록시 객체가 아니고 엔티티임을 확인할 수 있다.
fetch 라는 말 그대로 가서 엔티티를 가져온 것이다.

책에서는 아래와 같이 entityManager를 이용해서 가져오는 예제를 보여주고 있다. spring data jpa 를 이용해서 fetch join을 사용한 것과
동일한 쿼리가 수행된다.
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        String jpql = "select m from Member as m join fetch m.team where m.team.name = :teamName";
        List<Member> members = entityManager.createQuery(jpql, Member.class)
                .setParameter("teamName", "NewTeam")
                .getResultList();
        System.out.println(members.get(0).getTeam().getName());
        System.out.println(members.get(0).getTeam().getClass().getName());
    }
```
```bash
Hibernate: 
    select
        member0_.id as id1_0_0_,
        team1_.id as id1_1_1_,
        member0_.name as name2_0_0_,
        member0_.team_id as team_id3_0_0_,
        team1_.name as name2_1_1_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.id 
    where
        team1_.name=?

com.fistkim.springjpawhiteshipstudy.Team
```

### fetch join 과 일반 join 의 차이
fetch join 을 하면 inner join이 실행되는데, 이걸 참고해서 그러면 fetch join을 사용하지 않되 sql을 join 이 나가게 설정하면 어떻게 될까?

결과적으로 sql은 join 문이 실행되지만 엔티티에는 프록시 객체가 할당된다.

![](/images/jpa-fetch-join-vs-normal-join.png)

### fetch join 의 한계와 해결책
책에서는 아래와 같이 언급하고 있다.

* 페치 조인 대상에는 별칭을 줄 수 없다.
* 둘 이상의 컬렉션을 페치할 수 없다.
* 컬렉션을 페치 조인 하면 페이징 API를 사용할 수 없다.

페치 조인 대상에는 별칭을 줄 수 없다는 제약은 하이버네이트의 경우 별칭이 가능하다. 하지만 별칭 사용은 객체 탐색의 뎁스가 더 깊어져야할 때에만 사용하도록 한다.

여기서 가장 크리티컬한 것이 페이징을 사용할 수 없다는 것이다. 사용할 수는 있으나 전체 데이터를 메모리에 올려서 페이징 처리를 하므로 문제가 있을 수 있다.
```java
@Query(value = "select t from Team as t join fetch t.members", countQuery = "select count(t) from Team as t inner join t.members")
Page<Team> findAllFetchJoin(Pageable pageable);
```
```bash
.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
Hibernate: 
    select
        team0_.id as id1_1_0_,
        members1_.id as id1_0_1_,
        team0_.name as name2_1_0_,
        members1_.name as name2_0_1_,
        members1_.team_id as team_id3_0_1_,
        members1_.team_id as team_id3_0_0__,
        members1_.id as id1_0_0__ 
    from
        team team0_ 
    inner join
        member members1_ 
            on team0_.id=members1_.team_id
```

이렇게 limit, offet 같은게 하나도 없이 모두 조회되어서 메모리상에 올라간다.
그래서 'firstResult/maxResults specified with collection fetch; applying in memory!' 와 같은 경고 문구가 뜬다.

해결책으로는 fetch join을 사용하지 않고 LAZY 로딩을 그대로 사용하면서 대신 default_batch_fetch_size 를 이용하는 방법이 있다.
나의 예제 코드에 적용해보면 아래와 같이 나오게 된다.
```java
@Query(value = "select t from Team as t", countQuery = "select count(t) from Team as t")
Page<Team> findAllPage(Pageable pageable);

@Override
@Transactional
public void run(ApplicationArguments args) throws Exception {
    Page<Team> teams = teamRepository.findAllPage(PageRequest.of(0, 2));
    teams.forEach(team -> System.out.println(team.getMembers().size()));
}
```
```bash
# paging query. page 번호를 0으로 넣어서 limit 만 들어간 모습이다.
Hibernate: 
    select
        team0_.id as id1_1_,
        team0_.name as name2_1_ 
    from
        team team0_ limit ?

# 내가 설정해준 count query.
Hibernate: 
    select
        count(team0_.id) as col_0_0_ 
    from
        team team0_

# 지연로딩시 모아서 한번에 in 절로 나간 모습.
Hibernate: 
    select
        members0_.team_id as team_id3_0_1_,
        members0_.id as id1_0_1_,
        members0_.id as id1_0_0_,
        members0_.name as name2_0_0_,
        members0_.team_id as team_id3_0_0_ 
    from
        member members0_ 
    where
        members0_.team_id in (
            ?, ?
        )
```

[이 해결책을 코드에 적용한 것이 정리가 잘 된 포스트](https://junhyunny.github.io/spring-boot/jpa/jpa-fetch-join-paging-problem/)가 있어 링크를 남긴다.

{: .point}
*결론적으로 무조건 글로벌 fetch 전략으로 LAZY를 사용하고, EAGER하게 가져와야할 경우 전부 fetch join으로 해결하자.<br>
*다만, 페이징처리가 필요한 경우에는 default_batch_fetch_size를 사용하자!

## N+1 문제해결

### N+1 문제란?
[N+1 문제](https://junhyunny.github.io/spring-boot/jpa/jpa-one-plus-n-problem/) 는 LAZY 로딩을 사용하게 되면 매우 흔히 겪을 수 있는 문제이다. 
이직을 할때 면접에서도 자주 질문을 받았었다. 이는 fetch join 을 이용하면 해결할 수 있다. 또다른 방법으로는 default_batch_fetch_size 설정이 있다.

### <b>어떻게 해결하나? - fetch join</b><br>
먼저 fetch join을 살펴보자.
```java
public interface TeamRepository extends JpaRepository<Team, Long> {

    @Query(value = "select t from Team as t join fetch t.members")
    List<Team> findAllFetchJoin();

}
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        // List<Team> teams = teamRepository.findAll(); <-- N+1 문제 발생
        List<Team> teams = teamRepository.findAllFetchJoin();
        teams.forEach(team -> team.getMembers().forEach(member -> System.out.println(member.getName())));
    }
```

LAZY 로딩을 사용할 경우 team.getMembers() 를 호출할 때 member 테이블에 대해서 team 의 id 를 이용해서 조회하는 쿼리가 발생한다.
그래서 teams 의 사이즈 만큼 member 테이블에 대한 조회가 발생하게 된다.
최초로 teams 에 대한 조회 1회와 teams 를 순회하며 각 team element 에서 참조하고 있는 member들 컬렉션을 조회하는 N회를 합해서 1+N 쿼리가 수행되는 것이다.

하지만 fetch join 을 사용할 경우 아래와 같이 join 을 활용해서 1회의 쿼리로 조회를 끝낼 수 있다.
```sql
Hibernate: 
    select
        team0_.id as id1_1_0_,
        members1_.id as id1_0_1_,
        team0_.name as name2_1_0_,
        members1_.name as name2_0_1_,
        members1_.team_id as team_id3_0_1_,
        members1_.team_id as team_id3_0_0__,
        members1_.id as id1_0_0__ 
    from
        team team0_ 
    inner join
        member members1_ 
            on team0_.id=members1_.team_id
```

하지만 문제가 있는데, 1:N 관계인 team과 member 의 경우 team 을 조회하게 되면 team의 수보다 더 많은 row가 return 될 수 있다.
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        List<Team> teams = teamRepository.findAllFetchJoin();
        System.out.println(">>>> " + teams.size());
        teams.forEach(team -> System.out.println(team.getId()));
    }
```
```bash
Hibernate: 
    select
        team0_.id as id1_1_0_,
        members1_.id as id1_0_1_,
        team0_.name as name2_1_0_,
        members1_.name as name2_0_1_,
        members1_.team_id as team_id3_0_1_,
        members1_.team_id as team_id3_0_0__,
        members1_.id as id1_0_0__ 
    from
        team team0_ 
    inner join
        member members1_ 
            on team0_.id=members1_.team_id

>>>> 5
1
1
2
2
2
```

결과를 보면 team은 아이디가 1, 2 밖에 없는데 (총 2개) row가 5개가 나온 것을 볼 수 있다. 이는 쿼리를 보면 inner join 의 결과인 것이다.

해결책으로는 jpql 에서 제공하는 distinct 를 사용하면 된다.
sql의 distinct 는 중복된 결과를 제거하는 명령이지만 jpql의 distinct 는 아래 두 가지 기능을 수행한다.

* 중복된 결과를 제거.
* 어플리케이션에 올린뒤 중복되는 객체를 제거.

![](/images/jpa-fetch-join-distinct.png)

{: .warning }
*일대다 페치 조인은 결과 수가 증가할 수 있다.<br>
*일대다 페치 조인은 distinct를 사용하자.<br>
*일대일, 다대일 페치 조인은 결과가 증가하지 않는다.

<br>

### <b>어떻게 해결하나? - default_batch_fetch_size,  @BatchSize</b><br>
default_batch_fetch_size 의 원리는 지연 로딩으로 할당된 프록시 객체가 호출될 때 바로 호출하지 않고 모아서 in 절로 호출하는 원리이다.
[이 포스팅](https://velog.io/@jadenkim5179/Spring-defaultbatchfetchsize%EC%9D%98-%EC%9E%91%EB%8F%99%EC%9B%90%EB%A6%AC)에 정리가 잘 되어있다.

코드에 적용한 예시와 결과는 아래와 같다.
```yml
spring:  
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```
```java
@Override
@Transactional
public void run(ApplicationArguments args) throws Exception {
    List<Team> teams = teamRepository.findAll();
    teams.forEach(team -> System.out.println(team.getMembers().size()));
}
```
```bash
Hibernate: 
    select
        team0_.id as id1_1_,
        team0_.name as name2_1_ 
    from
        team team0_

Hibernate: 
    select
        members0_.team_id as team_id3_0_1_,
        members0_.id as id1_0_1_,
        members0_.id as id1_0_0_,
        members0_.name as name2_0_0_,
        members0_.team_id as team_id3_0_0_ 
    from
        member members0_ 
    where
        members0_.team_id in (
            ?, ?
        )
```

개별 엔티티에 @BatchSize 를 이용해서 사이즈를 지정해주는 방식도 있다.
```java
    @BatchSize(size = 100)
    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private Set<Member> members = new HashSet<>();
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        List<Team> teams = teamRepository.findAll();
        teams.forEach(team -> System.out.println(team.getMembers().size()));
    }
```
