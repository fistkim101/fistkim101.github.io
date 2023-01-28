---
layout: default
title: Spring Data Common 2
parent: 스프링데이터JPA_백기선
nav_order: 6
---

- 쿼리 만들기
  - 쿼리가 만들어지는 원리
  - 쿼리를 만드는 방법
- 쿼리 만들기 실습
  - 페이징 적용시 자동 생성되는 쿼리 
- 커스텀 레포지토리
  - 기존 레포지토리 생성 원리 복습
  - 커스텀 레포지토리 생성 원리 및 방법

## 쿼리 만들기
쿼리 생성은 Common 계층이 아니라 JPA 계층에서 만들어 지는 것이다. 강의에서도 JPA 계층에서 만들어진다고 밝히고 있다.
결국 hibernate 이 JDBC를 이용해서 쿼리를 db에 수행하게 될 텐데, 이 때 사용되는 쿼리가 어떻게 만들어지는지를 알아봤다.
다른 표현을 쓰자면 어떻게 해야 쿼리가 의도하는대로 생성되는지에 관한 룰을 학습했다.

기존에 기계적으로 사용했던 방식이지만 뒷단에서 어떤 개념들이 존재했고, 어떤 과정들이 있었는지를 알아보는 시간이었다.

### 쿼리가 만들어지는 원리
스프링 부트에서 자동으로 설정해주어 명시적으로 사용하지 않았던 @EnableJpaRepositories 에서 queryLookupStrategy 를 설정할 수 있다.
아래 코드에 설정한 것은 기본값인데 명시적으로 써 준 것이다.
```java
@EnableJpaRepositories(queryLookupStrategy = QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)
@SpringBootApplication
public class SpringJpaWhiteshipStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringJpaWhiteshipStudyApplication.class, args);
    }

}
```

* CREATE : 메소드 이름을 분석해서 쿼리 만들기
* USE_DECLARED_QUERY : 미리 정의해 둔 쿼리 찾아 사용하기
* CREATE_IF_NOT_FOUND : 미리 정의한 쿼리 찾아보고 없으면 만들기

위와 같이 세 옵션이 존재하고 특별히 정해주지 않을 경우 CREATE_IF_NOT_FOUND 를 기본값으로 동작한다.
그렇기 때문에 @Query 로 직접 설정한 쿼리가 존재할 경우 Repository의 메소드는 직접 설정한 쿼리를 수행하고
특별히 @Query 로 설정해준 쿼리가 없는 경우 메소드 이름을 분석해서 쿼리를 자동 생성하여 해당 쿼리를 수행한다.

결국 쿼리가 만들어지는 방법을 정리하면 아래와 같다. 

- JPA가 메소드 명을 분석해서 쿼리
- @Query 로 직접 선언한 쿼리
- 뒤에 나올 Query DSL 에 의해 생성된 쿼리
- 뒤에 나올 Specification<T> 로 만들어지는 쿼리

더 있을 수 있으나 이 정도면 충분한 것 같다.

```java
public interface PostRepository extends JpaRepository<Post, Long> {
  List<Post> findByNameLike(String name);
}
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {

        Post post = new Post();
        post.setName("포스트이름");
        post.setDescription("포스트상세설명");
        postRepository.save(post);

        List<Post> targets = postRepository.findByNameLike("포스트이름");
        System.out.println(">>> " + targets.size());
    }
```

위와 같은 경우 size는 1 이 나온다. 위의 경우에 @Query를 통해서 메소드와 전혀 일치하지 않는 description 컬럼 검색 쿼리로 설정해주고 이것이 동작이 확인이 되면
실제로 @Query에 우선권을 부여하는 것이 직접 확인이 된다.

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    @Query("select p from Post as p where p.description like :description")
    List<Post> findByNameLike(@Param(value = "description") String description);
}
```

위와 같이 Name으로 찾도록 메소드 명은 지정해줬지만 @Query 로 description 으로 찾도록 쿼리를 직접 설정해줬다.
그리고 아래 코드를 실행하면 원하는 결과를 확인할 수 있다.

```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {

        Post post = new Post();
        post.setName("포스트이름");
        post.setDescription("포스트상세설명");
        postRepository.save(post);

        // List<Post> targets = postRepository.findByNameLike("포스트이름");
        List<Post> targets = postRepository.findByNameLike("포스트상세설명");
        System.out.println(">>> " + targets.size());
    }
```

{: .point }
@Query 로 등록되는 쿼리는 사실상 하이버네이트의 @NamedQuery 와 동일하다.

### 쿼리를 만드는 방법
이미 위에서 어느정도 확인을 했지만 @Query로 직접 설정해주는 방법과 메소드 이름으로 자동 생성되도록 하는 방법이 있다.
직접 생성하는 거야 쿼리를 직접 넣어주는 것만 하면 되니, 내가 인지해야할 포인트는 어떤 규칙으로 메소드 이름을 구성해줘야 의도한대로 쿼리가 생성되도록 하는지에 관한
지식이다.

다 외울 필요는 없고 이런 식의 원리로 동작한다는 것을 인지해둔 채로 강의에 나온 자료를 첨부한다.
![](/images/concept-spring-data-create-query.png)

출처 : 강의자료

## 쿼리 만들기 실습

### 페이징 적용시 자동 생성되는 쿼리
페이징시 limit N, N 의 쿼리가 생소하여 찾아봤더니 아래와 같이 offset 의 의미였다.
```java
PageRequest pageRequest = PageRequest.of(1, 5);
Page<Post> postPage = postRepository.findAll(pageRequest);
```
```sql
select post0_.id          as id1_0_,
       post0_.description as descript2_0_,
       post0_.name        as name3_0_
from post post0_
limit 5 offset 5;

select post0_.id          as id1_0_,
       post0_.description as descript2_0_,
       post0_.name        as name3_0_
from post post0_
limit 5,5;
```

## 커스텀 레포지토리

### 기존 레포지토리 생성 원리 복습
자동 생성되는 쿼리 외에 직접 커스텀하게 쿼리를 구성해야할 때가 있다. 이 때 커스텀 레포지토리를 만들고 이를 기존 레포지토리가 상속하게 만들어서
기존 레포지토리로 커스텀 정의한 쿼리를 사용할 수 있도록 할 수 있다.

이 방법을 정리하기 전에 먼저 기존 레포지토리를 Bean 으로 어떻게 사용하는지 그 원리를 다시 짚어본다.(`핵심개념이해 4` 에서 이미 다뤘다.)

{: .point }
Repository<T, ID><br>
<- CrudRepository<T, ID><br>
<- PagingAndSortingRepository<T, ID><br> 
<- JpaRepository<Post, Long><br> 
<- PostRepository

위와 같은 방향으로 상속이 이뤄지고 있으며, 최종적으로 최하단의 PostRepository 가 bean 으로 등록이 될때 @NoRepositoryBean 가 있는 중간 계층의 Repository들의 자원을 모두 갖게 된다.
그리고 @NoRepositoryBean 이 있는 중간 계층의 Repository 는 정의된 메소드들만 제공하고 bean 으로 등록되지 않는다.

###  커스텀 레포지토리 생성 원리 및 방법
커스텀 레포지토리 생성 원리도 이와 크게 다르지 않다.

* 커스텀 메소드들을 정의해주고,
* 최하단 Repository 가 해당 메소드들을 사용할 수 있게 한다. 

결국 이렇게만 해주면 되는데, 내부적으로 지켜줘야할 규칙이 있다.

* 인터페이스로 다중 상속의 형태로 상속하게 만들어야한다.
* 구현체의 Postfix 를 규칙대로 지정해줘야한다.

```java
public interface PostCustomRepository {
    Post findMyPost();
}
```
```java
@Repository
@Transactional
public class PostCustomRepositoryImpl implements PostCustomRepository {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public Post findMyPost() {
        return entityManager.find(Post.class, 1L);
    }

}
```
```java
@SpringBootApplication
@EnableJpaRepositories(repositoryImplementationPostfix = "Impl")
public class SpringJpaWhiteshipStudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringJpaWhiteshipStudyApplication.class, args);
    }

}
```
Impl 이 default 값이고 이를 바꾸고 싶으면 위 설정에서 따로 바꿔 주어야 한다.

최종적으로 아래와 같이 최하단의 Repository가 커스텀 레포지토리 인터페이스를 상속하도록 해준다.
```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository {
}
```
