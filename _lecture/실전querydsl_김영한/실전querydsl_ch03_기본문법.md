---
layout: default
title: CH 03 기본문법
parent: 실전Querydsl_김영한
nav_order: 3
---

- jpql vs querydsl
- JPAQueryFactory 를 필드로
- 기본 Q_Type 활용
- 기본 검색 쿼리

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
