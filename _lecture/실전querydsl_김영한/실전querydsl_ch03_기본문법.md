---
layout: default
title: CH 03 기본문법
parent: 실전Querydsl_김영한
nav_order: 3
---

<br>

- jpql vs querydsl
- JPAQueryFactory 를 필드로
- 기본 Q_Type 활용
- 기본 검색 쿼리
- 결과조회
- 정렬
- 페이징
- 집합
  - 집합함수
  - GroupBy
- 조인(기본조인)
- 조인(on 절)
- 조인(페치조인)
- 서브 쿼리
- Case 문

<br>

## jpql vs querydsl

- 컴파일 타임에 오류를 잡아준다.(jpql의 경우 쿼리를 문자열로 만들기 때문에 오류가 있을시 런타임에 알 수 있다.)
- 파라미터 바인딩을 쉽게 처리해준다.(jpql에서는 직접 문자열 쿼리 내에 :username 과 같이 삽입했다.)

```java
    @Test
    public void startJPQL() {
        String qlString =
                "select m from Member m where m.username = :username";

        Member findMember = em.createQuery(qlString, Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

    @Test
    public void startQuerydsl() {
        JPAQueryFactory queryFactory = new JPAQueryFactory(em);

//        QMember m = new QMember("m");
//        Member findMember = queryFactory
//                .select(m)
//                .from(m)
//                .where(m.username.eq("member1"))
//                .fetchOne();

        JPQLQuery<Member> query = queryFactory.selectFrom(member)
                .where(member.username.eq("member1"));
        Member findMember = query.fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
```

<br>

## JPAQueryFactory를 필드로

동시성 문제는 JPAQueryFactory를 생성할 때 제공하는 EntityManager(em)에 달려있다.
스프링 프레임워크는 여러 쓰레드에서 동시에 같은 EntityManager에 접근해도, 트랜잭션 마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.

즉, JPAQueryFactory를 를 필드로 빼고 이를 공유해도 JPAQueryFactory 에 사용되는 EntityManager 에서 알아서 어느 트랜잭션에 걸려있는 처리인지에 따라
다른 영속성 컨텍스트를 제공하기 때문에 동시성 관리가 되고 있어서 괜찮다고 할 수 있다.

> 동시성 문제는 걱정하지 않아도 된다. 왜냐하면 여기서 스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저이다.
> 이 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.
> 더 자세한 내용은 자바 ORM 표준 JPA 책 13.1 트랜잭션 범위의 영속성 컨텍스트를 참고하자.

<br>

## 기본 Q_Type 활용

Q클래스 인스턴스를 사용하는 2가지 방법
```java
QMember qMember = new QMember("m"); // 1. 별칭 직접 지정
QMember qMember = QMember.member; // 2. 기본 인스턴스 사용(내부에 public static final QMember member = new QMember("member1"); 필드가 이미 선언되어 있다.)
```

Q 클래스 내부에 static 필드를 static import 해서 아래와 같이 바로 사용 하는 것을 권장한다.
```java
  @Test
  public void startQuerydsl3() {
      Member findMember = queryFactory
              .select(member)
              .from(member)
              .where(member.username.eq("member1"))
              .fetchOne();
      assertThat(findMember.getUsername()).isEqualTo("member1");
  }
```

<br>

## 기본 검색 쿼리

기본적인 where 조건 검색 쿼리는 아래와 같이 사용 가능하다.
```java
    @Test
    public void search() {
        // Member member1 = new Member("member1", 10, teamA); 위에서 이렇게 만들어 주고 @BeforeEach 로 persist 되고있다.

        Member findMember1 = queryFactory
            .selectFrom(member)
            .where(
                    member.username.eq("member1").and(member.age.eq(10))
                  )
            .fetchOne();

        Member findMember2 = queryFactory
            .selectFrom(member)
            .where(
                    member.username.eq("member1"),
                    (member.age.eq(10))
                  )
            .fetchOne();

        Member findMember3 = queryFactory
            .selectFrom(member)
            .where(
                    member.username.eq("member1"),
                    (member.age.eq(10)),
                    null,
                    null
            )
            .fetchOne();

        assertThat(findMember1.getUsername()).isEqualTo("member1");
        assertThat(findMember2.getUsername()).isEqualTo("member1");
        assertThat(findMember3.getUsername()).isEqualTo("member1");
    }
```

여기서 where 절이 위와 같이 and 로 체이닝 걸려도 되고, 콤마를 통해서 varargs 로 넘겨줘도 똑같이 and 조건으로 쿼리가 실행된다.
이유는 아래와 같이 내부적으로 구현되어 있기 때문이다.
```java
    /**
 * Add the given filter conditions
 *
 * <p>Skips null arguments</p>
 *
 * @param o filter conditions to be added
 * @return the current object
 */
public Q where(Predicate... o) {
        return queryMixin.where(o);
        }
```

여기서 또 한 가지 주목할 점은 findMember3 처럼 null 을 넣어줘도 알아서 내부적으로 querydsl 이 무시해준다는 것이다.
이걸 활용하면 동적쿼리를 더 편하게 작성할 수 있다. 이 부분은 후에 동적쿼리를 다룰때 더 자세히 다룬다.

아래는 그 외 다양한 조건 표현식들이다.

```java
    member.username.eq("member1") // username = 'member1'
    member.username.ne("member1") //username != 'member1'
    member.username.eq("member1").not() // username != 'member1'
    member.username.isNotNull() //이름이 is not null
    member.age.in(10, 20) // age in (10,20)
    member.age.notIn(10, 20) // age not in (10, 20)
    member.age.between(10,30) //between 10, 30
    member.age.goe(30) // age >= 30  member.age.gt(30) // age > 30
    member.age.loe(30) // age <= 30
    member.age.lt(30) // age < 30
    member.username.like("member%") //like 검색 member.username.contains("member") // like ‘%member%’ 검색 member.username.startsWith("member") //like ‘member%’ 검색
```

<br>

## 결과조회

- fetch() : 리스트 조회, 데이터 없으면 빈 리스트 반환 
- fetchOne() : 단 건 조회
  - 결과가 없으면 : null
  - 결과가 둘 이상이면 : com.querydsl.core.NonUniqueResultException
- fetchFirst() : limit(1).fetchOne()
- fetchResults() : 페이징 정보 포함, total count 쿼리 추가 실행
- fetchCount() : count 쿼리로 변경해서 count 수 조회

<br>

## 정렬

```java
    /**
     * 회원 정렬 순서
     * 1. 회원 나이 내림차순(desc)
     * 2. 회원 이름 올림차순(asc)
     * 단 2에서 회원 이름이 없으면 마지막에 출력(nulls last)
     */
    @Test
    public void sort() {
        em.persist(new Member(null, 100));
        em.persist(new Member("member5", 100));
        em.persist(new Member("member6", 100));
        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(100))
                .orderBy(member.age.desc(), member.username.asc().nullsLast())
                .fetch();
        Member member5 = result.get(0);
        Member member6 = result.get(1);
        Member memberNull = result.get(2);
        assertThat(member5.getUsername()).isEqualTo("member5");
        assertThat(member6.getUsername()).isEqualTo("member6");
        assertThat(memberNull.getUsername()).isNull();
    }
```

<br>

## 페이징

```java
    @Test
    public void paging1() {
        List<Member> result = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc()).offset(1) //0부터 시작(zero index)
                .limit(2) //최대 2건 조회
                .fetch();
        assertThat(result.size()).isEqualTo(2);
    }

    @Test
    public void paging2() {
        QueryResults<Member> queryResults = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
                .offset(1)
                .limit(2)
                .fetchResults();
        assertThat(queryResults.getTotal()).isEqualTo(4);
        assertThat(queryResults.getLimit()).isEqualTo(2);
        assertThat(queryResults.getOffset()).isEqualTo(1);
        assertThat(queryResults.getResults().size()).isEqualTo(2);
    }
```

실무에서 페이징 쿼리를 작성할 때, 데이터를 조회하는 쿼리는 여러 테이블을 조인해야 하지만, count 쿼리는 조인이 필요 없는 경우도 있다.
그런데 이렇게 자동화된 count 쿼리는 원본 쿼리와 같이 모두 조인을 해버리기 때문에 성능이 안나올 수 있다.
count 쿼리에 조인이 필요없는 성능 최적화가 필요하다면, count 전용 쿼리를 별도로 작성해야 한다.

페이징을 할 때 여러 개의 join 이 들어간 쿼리가 실행될 일이 많은데, 사실 count 는 단순하게도 가능한 경우가 있는데 이 때 성능 향상을 위해서
count 는 자동생성된 쿼리로 사용하지 말고(join 이 들어간 쿼리) 직접 단순한 쿼리를 만들어서 사용하라는 것.

<br>

## 집합

### 집합함수

```java
    /**
     * JPQL
     * select
     * COUNT(m), //회원수
     * SUM(m.age), //나이 합
     * AVG(m.age), //평균 나이
     * MAX(m.age), //최대 나이
     * MIN(m.age) //최소 나이 * from Member m
     */
    @Test
    public void aggregation() throws Exception {
        List<Tuple> result = queryFactory
                .select(member.count(),
                        member.age.sum(),
                        member.age.avg(),
                        member.age.max(),
                        member.age.min())
                .from(member)
                .fetch();
        Tuple tuple = result.get(0);
        assertThat(tuple.get(member.count())).isEqualTo(4);
        assertThat(tuple.get(member.age.sum())).isEqualTo(100);
        assertThat(tuple.get(member.age.avg())).isEqualTo(25);
        assertThat(tuple.get(member.age.max())).isEqualTo(40);
        assertThat(tuple.get(member.age.min())).isEqualTo(10);
    }
```
```sql
    select
        count(member0_.member_id) as col_0_0_,
        sum(member0_.age) as col_1_0_,
        avg(member0_.age) as col_2_0_,
        max(member0_.age) as col_3_0_,
        min(member0_.age) as col_4_0_ 
    from
        member member0_
```

### GroupBy

```java
    /**
     * 팀의 이름과 각 팀의 평균 연령을 구해라.
     */
    @Test
    public void group() throws Exception {
        List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
                .fetch();
        Tuple teamA = result.get(0);
        Tuple teamB = result.get(1);
        assertThat(teamA.get(team.name)).isEqualTo("teamA");
        assertThat(teamA.get(member.age.avg())).isEqualTo(15);
        assertThat(teamB.get(team.name)).isEqualTo("teamB");
        assertThat(teamB.get(member.age.avg())).isEqualTo(35);
    }
```
```sql
    select
        team1_.name as col_0_0_,
        avg(member0_.age) as col_1_0_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    group by
        team1_.name
```

<br>

## 조인(기본조인)

```java
    /**
     * 팀A에 소속된 모든 회원
     */
    @Test
    public void join() throws Exception {
        QMember member = QMember.member;
        QTeam team = QTeam.team;
        List<Member> result = queryFactory
                .selectFrom(member)
                .join(member.team, team)
                .where(team.name.eq("teamA"))
                .fetch();
        assertThat(result)
                .extracting("username")
                .containsExactly("member1", "member2");
    }
```
```sql
    select
        member0_.member_id as member_i1_1_,
        member0_.age as age2_1_,
        member0_.team_id as team_id4_1_,
        member0_.username as username3_1_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    where
        team1_.name=?
```

- join() , innerJoin() : 내부 조인(inner join)
- leftJoin() : left 외부 조인(left outer join)
- rightJoin() : rigth 외부 조인(rigth outer join)

<br>

## 조인(on 절)

ON절을 활용한 조인(JPA 2.1부터 지원)

```java
    /**
     * 예) 회원과 팀을 조인하면서, 팀 이름이 teamA인 팀만 조인, 회원은 모두 조회
     * JPQL: SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'teamA'
     * SQL: SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID=t.id and
     * t.name='teamA'
     */
    @Test
    public void join_on_filtering() throws Exception {
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team).on(team.name.eq("teamA"))
                .fetch();
        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
```
```sql
    select
        member0_.member_id as member_i1_1_0_,
        team1_.team_id as team_id1_2_1_,
        member0_.age as age2_1_0_,
        member0_.team_id as team_id4_1_0_,
        member0_.username as username3_1_0_,
        team1_.name as name2_2_1_ 
    from
        member member0_ 
    left outer join
        team team1_ 
            on member0_.team_id=team1_.team_id 
            and (
                team1_.name=?
            )
```

on 절을 활용해 조인 대상을 필터링 할 때, 외부조인이 아니라 내부조인(inner join)을 사용하면, where 절에서 필터링 하는 것과 기능이 동일하다.
따라서 on 절을 활용한 조인 대상 필터링을 사용할 때, 내부조인 이면 익숙한 where 절로 해결하고, 정말 외부조인이 필요한 경우에만 이 기능을 사용하자.

<br>

## 조인(페치조인)

```java
    @Test
    public void fetchJoinNo() throws Exception {
        em.flush();
        em.clear();
        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"))
                .fetchOne();
        boolean loaded =
                emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        assertThat(loaded).as("페치 조인 미적용").isFalse();
    }
```
```sql
    select
        member0_.member_id as member_i1_1_,
        member0_.age as age2_1_,
        member0_.team_id as team_id4_1_,
        member0_.username as username3_1_ 
    from
        member member0_ 
    where
        member0_.username=?
```

<br>

```java
    @Test
    public void fetchJoinUse() throws Exception {
        em.flush();
        em.clear();
        Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team, team).fetchJoin()
                .where(member.username.eq("member1"))
                .fetchOne();
        boolean loaded =
                emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
        assertThat(loaded).as("페치 조인 적용").isTrue();
    }
```
```sql
    select
        member0_.member_id as member_i1_1_0_,
        team1_.team_id as team_id1_2_1_,
        member0_.age as age2_1_0_,
        member0_.team_id as team_id4_1_0_,
        member0_.username as username3_1_0_,
        team1_.name as name2_2_1_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    where
        member0_.username=?
```

<br>

아래는 내가 프로젝트에서 사용했던 fetch join 코드 예시
```java
    @Override
    public Page<Auction> findAllByHostUserAndIsDisplayed(User targetUser, YesNoType isDisplayed, Pageable pageable) {
        JPQLQuery<Auction> query = queryFactory.selectFrom(auction)
                .innerJoin(auction.hostUser, user)
                .fetchJoin()
                .innerJoin(auction.auctionItem.metal, metal)
                .fetchJoin()
                .innerJoin(auction.auctionItem.metalOption, metalOption)
                .fetchJoin()
                .where(auction.hostUser.eq(targetUser).and(auction.isDisplayed.eq(isDisplayed)));

        long totalCount = query.fetchCount();
        List<Auction> results = getQuerydsl().applyPagination(pageable, query).fetch();

        return new PageImpl<>(results, pageable, totalCount);
    }
```

<br>

## 서브 쿼리

```java
    /**
     * 나이가 가장 많은 회원 조회
     */
    @Test
    public void subQuery() throws Exception {
        QMember memberSub = new QMember("memberSub");
        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(
                        JPAExpressions
                                .select(memberSub.age.max())
                                .from(memberSub)
                )).fetch();
        assertThat(result).extracting("age")
                .containsExactly(40);
    }
```
```sql
    select
        member0_.member_id as member_i1_1_,
        member0_.age as age2_1_,
        member0_.team_id as team_id4_1_,
        member0_.username as username3_1_ 
    from
        member member0_ 
    where
        member0_.age=(
            select
                max(member1_.age) 
            from
                member member1_
        )
```

서브쿼리 예시는 더 많은데 (in 절 등) 정리는 생략한다.

from 절의 서브쿼리 한계

JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다.
당연히 Querydsl 도 지원하지 않는다. 하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다.
Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다.

<br>

## Case 문

```java
     List<String> result = queryFactory
             .select(member.age
             .when(10).then("열살") .when(20).then("스무살") .otherwise("기타"))
             .from(member)
             .fetch();
```
```java
List<String> result = queryFactory
             .select(new CaseBuilder()
             .when(member.age.between(0, 20)).then("0~20살") .when(member.age.between(21, 30)).then("21~30살") .otherwise("기타"))
             .from(member)
             .fetch();
```

querydsl 을 사용하여 case 문을 사용할 수는 있으나, 저렇게 데이터를 변환하는 식의 로직을 DB 단에서 처리하는 것은 옳지 못하다.
database 에서는 데이터를 조회하고 가져가는 것만 하고 저러한 데이터 가공 및 조작은 어플리케이션에서 처리하자.
