---
layout: default
title: Spring Data JPA 1
parent: 스프링데이터JPA_백기선
nav_order: 9
---

- JpaRepository.save()
  - persist
  - merge
- Projection

## JpaRepository.save()
save() 는 단순히 엔티티를 저장해주는 기능을 수행하는 것이 아님. 경우에 따라 persist 또는 merge 로 동작한다.
- Transient 상태의 객체라면 EntityManager.persist()
- Detached 상태의 객체라면 EntityManager.merge()

### persist
Persist() 메소드에 파라미터로 넘긴(save 대상) 그 엔티티 객체를 Persistent 상태로 변경한다.
save() 결과로 반환받은 saved entity 가 곧 파라미터로 넘긴 그 save 대상과 같다.

```java
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @PersistenceContext
    EntityManager entityManager;

    @Test
    void saveTest() {
        Post post = new Post();
        post.setName("name_1");
        post.setDescription("description_1");
        Post savedPost = postRepository.save(post);

        Assertions.assertThat(entityManager.contains(post)).isTrue();
        Assertions.assertThat(entityManager.contains(savedPost)).isTrue();
        Assertions.assertThat(post).isEqualTo(savedPost);
    }

}
```

### merge
주석 표시한 것들을 잘 기억하자.
update 시 파라미터로 넘긴 entity는 영속화되지 않고 그것의 복사본이 영속화 된다.
그리고 영속화된 객체가 return 된다.

이 포인트가 주는 중요한 시사점은 update 로직 처리 이후 후속 작업을 해야할 경우 반드시 return 된 그 값을 사용해야 한다는 것이다.
왜냐하면 그 객체가 persistent 된 객체이기 때문이다. 반대로 말하면 save() 의 파라미터로 넘긴 객체를 사용해서는 안된다는 것이다.

왜 사용하면 안되냐? managed 객체가 아니기 때문이다. 즉, persistent 상태가 아니기 때문에 상태가 추적이 되지 않아서
dirty check 등 JPA의 이점을 누리지 못하기 때문이다.
```java
    @Test
    void updateTest() {
        Post post = new Post();
        post.setName("name_1");
        post.setDescription("description_1");
        postRepository.save(post);

        Post newPost = new Post();
        newPost.setId(1L);
        newPost.setName("new_name_1");
        newPost.setDescription("new_description_1");
        Assertions.assertThat(entityManager.contains(newPost)).isFalse();

        // merge 발생. 이 때 newPost의 복사본이 영속화된다.
        Post updatedPost = postRepository.save(newPost);

        // 파라미터로 넘긴 entity인 newPost 는 detached
        Assertions.assertThat(entityManager.contains(newPost)).isFalse();

        // update 결과로 받은 entity인 updatedPost는 persistent
        Assertions.assertThat(entityManager.contains(updatedPost)).isTrue();
    }
```

## [Projection](https://www.baeldung.com/spring-data-jpa-projections)
entity의 일부 컬럼만 가져오는 기능인데 나는 실무에서 써본적이 없거니와 얻는 성능 효율 대비 코드 복잡도만 더 커지는 느낌이다.
일단은 이런게 있다 정도만 인지해둔다.

아래는 강의 노트 그대로 발췌.
- 인터페이스 기반 프로젝션
  - Nested 프로젝션 가능.
  - Closed 프로젝션
    - 쿼리를 최적화 할 수 있다. 가져오려는 애트리뷰트가 뭔지 알고 있으니까.
    - Java 8의 디폴트 메소드를 사용해서 연산을 할 수 있다. 
  - Open 프로젝션
    - @Value(SpEL)을 사용해서 연산을 할 수 있다. 스프링 빈의 메소드도 호출 가능.
    -쿼리 최적화를 할 수 없다. SpEL을 엔티티 대상으로 사용하기 때문에.

- 클래스 기반 프로젝션
  - DTO
  - 롬복 @Value로 코드 줄일 수 있음
  
