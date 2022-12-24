---
layout: default
title: Spring Data JPA 2
parent: 스프링데이터JPA_백기선
nav_order: 10
---

- Specification
- Auditing

## Specification
query DSL 과 유사하게 query 를 프로그램으로 작성할 수 있도록 해준다. 아래는 [JPA 공식 문서에서 Specification] 에 대해 설명한 글 일부다.

{: .point }
JPA 2 introduces a criteria API that you can use to build queries programmatically.
By writing a criteria, you define the where clause of a query for a domain class.
Taking another step back, these criteria can be regarded as a predicate over the entity that is described by the JPA criteria API constraints.

cf. [criteria] 라는 단어의 뜻 자체가 '기준'의 이미이다.

query dsl 과 차이나는 점은 다이나믹 쿼리의 형태로도 Specification 을 사용할 수 있다는 것이다.
나는 개인적으로 이 부분이 제일 마음에 들었다.

사용 방법은 공식문서에도 나와있지만 간단하다. repository 에서 JpaSpecificationExecutor<T> 를 상속 받고
findAll() 에 파라미터로 Specification 을 넘겨준다. pageable 도 파라미터로 넘겨주면 페이징 처리도 가능하다.

```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository, QuerydslPredicateExecutor<Post>, JpaSpecificationExecutor<Post> {
}
```
```java
    @Test
    void specificationTest() {
        Post post = new Post();
        final String NAME = "specificationPostName";
        final String DESCRIPTION = "specificationPostDescription";
        post.setName(NAME);
        post.setDescription(DESCRIPTION);
        postRepository.save(post);

        List<Post> targets = postRepository.findAll(Post.getPostsByCustomSpecification(NAME, DESCRIPTION));
        Assertions.assertThat(targets.size()).isEqualTo(1);
        Assertions.assertThat(targets.get(0).getName()).isEqualTo(NAME);
        Assertions.assertThat(targets.get(0).getDescription()).isEqualTo(DESCRIPTION);
    }
```
```sql
    select
        post0_.id as id1_0_,
        post0_.description as descript2_0_,
        post0_.name as name3_0_ 
    from
        post post0_ 
    where
        post0_.name=? 
        and post0_.description=?
# binding parameter [1] as [VARCHAR] - [specificationPostName]
# binding parameter [2] as [VARCHAR] - [specificationPostDescription]
```

만약 여기서 아래와 같이 DESCRIPTION 에 null 을 넣을 경우 쿼리가 의도한대로 달라진다.
```java
    @Test
    void specificationTest() {
        Post post = new Post();
        final String NAME = "specificationPostName";
        final String DESCRIPTION = "specificationPostDescription";
        post.setName(NAME);
        post.setDescription(DESCRIPTION);
        postRepository.save(post);

        List<Post> targets = postRepository.findAll(Post.getPostsByCustomSpecification(NAME, null));
        Assertions.assertThat(targets.size()).isEqualTo(1);
        Assertions.assertThat(targets.get(0).getName()).isEqualTo(NAME);
        Assertions.assertThat(targets.get(0).getDescription()).isEqualTo(DESCRIPTION);
    }
```
```sql
    select
        post0_.id as id1_0_,
        post0_.description as descript2_0_,
        post0_.name as name3_0_ 
    from
        post post0_ 
    where
        post0_.name=?
```

## Auditing
특별한 내용은 없어서 강의자료만 첨부한다.

사용법
- 메인 애플리케이션 위에 @EnableJpaAuditing 추가 (스프링부트가 자동설정 해주지 않는다)
- 엔티티 클래스 위에 @EntityListeners(AuditingEntityListener.class) 추가

```java
    @CreatedDate
    private Date created;

    @LastModifiedDate
    private Date updated;

    @CreatedBy
    @ManyToOne
    private Account createdBy;

    @LastModifiedBy
    @ManyToOne
    private Account updatedBy;
```

[JPA 공식 문서에서 Specification]: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#specifications
[criteria]: https://en.dict.naver.com/#/entry/enko/950022ddab1a4485941a8cb66ec2a958
[Auditing]: https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#auditing