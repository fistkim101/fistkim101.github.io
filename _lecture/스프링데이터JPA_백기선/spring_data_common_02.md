---
layout: default
title: Spring Data Common 2
parent: 스프링데이터JPA_백기선
nav_order: 6
---

- 쿼리 만들기
  - 쿼리가 만들어지는 원리
  - 쿼리를 만드는 방법

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

### 쿼리를 만드는 방법
이미 위에서 어느정도 확인을 했지만 @Query로 직접 설정해주는 방법과 메소드 이름으로 자동 생성되도록 하는 방법이 있다.
직접 생성하는 거야 쿼리를 직접 넣어주는 것만 하면 되니, 내가 인지해야할 포인트는 어떤 규칙으로 메소드 이름을 구성해줘야 의도한대로 쿼리가 생성되도록 하는지에 관한
지식이다.

다 외울 필요는 없고 이런 식의 원리로 동작한다는 것을 인지해둔 채로 강의에 나온 자료를 첨부한다.
![](/images/concept-spring-data-create-query.png)

출처 : 강의자료
