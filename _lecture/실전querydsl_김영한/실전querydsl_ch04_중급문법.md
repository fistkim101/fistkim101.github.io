---
layout: default
title: CH 04 중급문법
parent: 실전Querydsl_김영한
nav_order: 4
---

<br>

- 프로젝션
- 프로젝션과 결과 반환 (기본)
- 프로젝션과 결과 반환 (DTO 조회)
  - 순수 JPA 에서 DTO 조회
  - Querydsl 빈 생성(Bean population)
- 프로젝션과 결과 반환(@QueryProjection)
- 동적 쿼리
  - BooleanBuilder
  - Where 다중 파라미터 사용
- 수정, 삭제 벌크 연산
- SQL function 호출하기

<br>

## 프로젝션

프로젝션(Projection) 은 sql 용어이다. [IBM document](https://www.ibm.com/docs/en/informix-servers/14.10?topic=concepts-selection-projection) 에서는 아래와 같이 정의되어 있다.

> In relational terminology, projection is defined as taking a vertical subset from the columns of a single table that retains the unique rows.
> This kind of SELECT statement returns some of the columns and all the rows in a table.

쉽게 column 이라고 봐도 사실 의미는 통한다. 어원은 아무래도 entity(row) 의 일부니까 일부라는 것이 곧 투영(projection) 이라서 projection 이 된게 아닐까 싶다.

<br>

## 프로젝션과 결과 반환 (기본)

### 프로젝션 대상이 하나

```java
    @Test
    void projectionSingle() {
        List<String> result = queryFactory
                .select(member.username)
                .from(member)
                .fetch();

        result.forEach(System.out::println);
    }
```
```sql
    select
        member0_.username as col_0_0_ 
    from
        member member0_
```

<br>

### 프로젝션 대상이 둘 이상

com.querydsl.core.Tuple 를 사용한다.

```java
    @Test
    void projectionTuple() {
        List<Tuple> result = queryFactory
                .select(member.username, member.age)
                .from(member)
                .fetch();
        for (Tuple tuple : result) {
            String username = tuple.get(member.username);
            Integer age = tuple.get(member.age);
            System.out.println("username=" + username);
            System.out.println("age=" + age);
        }
    }
```
```sql
    select
        member0_.username as col_0_0_,
        member0_.age as col_1_0_ 
    from
        member member0_
```

Tuple 은 com.querydsl.core.Tuple 이다. 따라서 이 type 을 infra 계층을 넘어버리면(service 나 application 계층으로 넘어가서까지 사용하면)
특정 기술에 어플리케이션이 의존하게 되어버리므로 유의해서 사용하자. 필요시 일반 DTO 로 변환하여 넘기던가 해야한다.

<br>

## 프로젝션과 결과 반환 (DTO 조회)

### 순수 JPA 에서 DTO 조회
```java
    @Test
    void projectionDtoSimple() {
        List<MemberDto> result = em.createQuery(
                        "select new com.example.querydslstudy.MemberDto(m.username, m.age) " +
                                "from Member m", MemberDto.class)
                .getResultList();

        result.forEach(memberDto -> System.out.println(memberDto.toString()));
    }
```

- 순수 JPA에서 DTO를 조회할 때는 new 명령어를 사용해야함
- DTO의 package이름을 다 적어줘야해서 지저분함
- 생성자 방식만 지원함

<br>

### Querydsl 빈 생성(Bean population)

프로퍼티 접근, 필드 직접 접근, 생성자 사용 하는 방식이 있다.

```java
    @Test
    void projectionDtoQueryDsl() {
//        List<MemberDto> result = queryFactory
//                .select(Projections.bean(MemberDto.class,
//                        member.username,
//                        member.age))
//                .from(member)
//                .fetch();

//        List<MemberDto> result = queryFactory
//                .select(Projections.fields(MemberDto.class,
//                        member.username,
//                        member.age))
//                .from(member)
//                .fetch();

        List<MemberDto> result = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        result.forEach(memberDto -> System.out.println(memberDto.toString()));
    }
```

<br>

필드명이 다른 DTO 로 조회 결과를 가져오고 싶다면 아래와 같이 사용한다.

```java
@Data
@NoArgsConstructor
public class UserDto {
    private String name;
    private int age;

    public UserDto(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```
```java
    @Test
    void fieldNameTest() {
        List<UserDto> result = queryFactory
                .select(Projections.fields(UserDto.class,
                        member.username.as("name"),
                        member.age))
                .from(member)
                .fetch();

        result.forEach(memberDto -> System.out.println(memberDto.toString()));
    }
```

<br>

## 프로젝션과 결과 반환(@QueryProjection)

```java
package study.querydsl.dto;
  import com.querydsl.core.annotations.QueryProjection;
  import lombok.Data;
  @Data
  public class MemberDto {
      private String username;
      private int age;
      public MemberDto() {
      }
      @QueryProjection
      public MemberDto(String username, int age) {
          this.username = username;
          this.age = age;
      }
  }
```
```java
List<MemberDto> result = queryFactory
          .select(new QMemberDto(member.username, member.age))
          .from(member)
          .fetch();
```

@QueryProjection 을 사용할 경우 위와 같이 처리해준 뒤 compileQuerydsl 으로 Q class 를 생성해서 사용해야한다.

이 방식은 굳이 Q class 를 만들어줘야하고, DTO 가 POJO 하지 못하고 특정 기술에 의존하게 되므로 썩 좋지는 않은 것 같다.

<br>

## 동적 쿼리

### BooleanBuilder

```java
    @Test
    public void 동적쿼리_BooleanBuilder() throws Exception {
        String usernameParam = "member1";
        Integer ageParam = 10;
        List<Member> result = searchMember1(usernameParam, ageParam);
        Assertions.assertThat(result.size()).isEqualTo(1);
    }

    private List<Member> searchMember1(String usernameCond, Integer ageCond) {
        BooleanBuilder builder = new BooleanBuilder();
        if (usernameCond != null) {
            builder.and(member.username.eq(usernameCond));
        }
        if (ageCond != null) {
            builder.and(member.age.eq(ageCond));
        }
        return queryFactory
                .selectFrom(member)
                .where(builder)
                .fetch();
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
        and member0_.age=?
```

위와 같이 사용하면 된다. 아래는 내가 이메탈 프로젝트시 사용했던 코드 일부이다.

```java
    @Override
    public Page<Auction> findAllByAuctionTypeAndMetalAndIsDisplayed(AuctionType auctionType, Metal targetMetal, YesNoType isDisplayed, Pageable pageable) {
        BooleanBuilder builder = new BooleanBuilder();

        JPQLQuery<Auction> query = queryFactory.selectFrom(auction)
                .innerJoin(auction.hostUser, user)
                .fetchJoin()
                .innerJoin(auction.auctionItem.metal, metal)
                .fetchJoin()
                .innerJoin(auction.auctionItem.metalOption, metalOption)
                .fetchJoin();

        if (auctionType != null) {
            builder.and(auction.auctionType.eq(auctionType));
        }

        if (targetMetal != null) {
            builder.and(auction.auctionItem.metal.eq(targetMetal));
        }

        if (isDisplayed != null) {
            builder.and(auction.isDisplayed.eq(isDisplayed));
        }

        query = query.where(builder);
        long totalCount = query.fetchCount();
        List<Auction> results = getQuerydsl().applyPagination(pageable, query).fetch();

        return new PageImpl<>(results, pageable, totalCount);
    }
```

<br>

### Where 다중 파라미터 사용

```java
    @Test
    public void 동적쿼리_WhereParam() throws Exception {
        String usernameParam = "member1";
        Integer ageParam = 10;
        List<Member> result = searchMember2(usernameParam, ageParam);
        Assertions.assertThat(result.size()).isEqualTo(1);
    }

    private List<Member> searchMember2(String usernameCond, Integer ageCond) {
        return queryFactory
                .selectFrom(member)
                .where(usernameEq(usernameCond), ageEq(ageCond))
                .fetch();
    }

    private BooleanExpression usernameEq(String usernameCond) {
        return usernameCond != null ? member.username.eq(usernameCond) : null;
    }

    private BooleanExpression ageEq(Integer ageCond) {
        return ageCond != null ? member.age.eq(ageCond) : null;
    }
```

CH03 기본문법에서 이미 알아봤지만 where 절이 아래와 같이 구성되어 있기 때문에 위와 같은 처리가 가능하다.

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

null 이 반환이 되어도 아무 문제가 없는 것이다. 위와 같은 방식으로 하면 BooleanBuilder 를 사용했을 때보다 가독성 및 코드 재활용성이 좋아진다.

- BooleanExpression 를 반환하는 개별 private 함수를 다른 곳에서 재활용 할 수 있고,
- 동적쿼리를 수행하는 메인 로직에서 쿼리를 먼저 읽게 되기 때문이다.(BooleanExpression 를 사용하는 방식에선 쿼리를 가장 나중에 보게 된다는 것과 비교해보면 이해하기 쉽다.)

아래와 같이 조합도 가능하다. 조합을 하면 좋은 장점이 아래 예시처럼 함수명이 allEq 이 아니라 isAvailable() 과 같이 도메인 용어나 도메인 지식이 들어간
의미 있는 함수명을 써서 코드의 가독성을 높힐 수 있기 때문이다.

```java
 private BooleanExpression allEq(String usernameCond, Integer ageCond) {
        return usernameEq(usernameCond).and(ageEq(ageCond));
 }
```

<br>

## 수정, 삭제 벌크 연산

```java
    @Test
    void batchUpdateTest() {
        long count = queryFactory
                .update(member)
                .set(member.username, "비회원")
                .where(member.age.lt(28)).execute();

        System.out.println("count : " + count);
    }
```
```bash
    update
        member 
    set
        username=? 
    where
        age<?

2023-03-31 16:34:03.221 TRACE 32760 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [비회원]
2023-03-31 16:34:03.224 TRACE 32760 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [INTEGER] - [28]
```

<br>

```java
    @Test
    void batchDeleteTest() {
        long count = queryFactory
                .delete(member)
                .where(member.age.gt(18))
                .execute();

        System.out.println("count : " + count);
    }
```
```bash
    delete 
    from
        member 
    where
        age>?
2023-03-31 16:37:44.503 TRACE 32784 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [INTEGER] - [18]
```

주의 할 점은 영속성 컨텍스트에 있는 엔티티를 무시하고 실행되기 때문에 배치 쿼리를 실행하고 나면 영속성 컨텍스트를 초기화 해줘야 안전하다.
즉, DML 실행 전 혹은 후에 entityManager.flush(), entityManager.clear() 처리를 해줘야 한다는 것이다.

그렇게 해주지 않으면 하나의 트랜잭션 내에서 해당 DML 이 실행되고 나서 DB와 1차 캐시 간에 데이터 불일치가 발생할 수 있다.
이 불일치는 후에 어떠한 처리를 하느냐에 따라 의도되지 않은 결과를 가져올 수 있다.

<br>

## SQL function 호출하기

SQL function은 JPA와 같이 Dialect에 등록된 내용만 호출할 수 있다.

```java
String result = queryFactory
            .select(Expressions.stringTemplate("function('replace', {0}, {1},
            {2})", member.username, "member", "M"))
            .from(member)
            .fetchFirst();
```

사용하는 database 의 Dialect 를 queryDsl 내에서 호출해서 사용이 가능하다는 것을 인지하고 있는 정도만 해도 충분하다.
쓸 일이 개인적으로 없었고, 앞으로도 굳이 있을까 싶다. 데이터 조작 자체는 어플리케이션 내에서 처리하는 것을 지향하는 것이 좋을 것 같다.
