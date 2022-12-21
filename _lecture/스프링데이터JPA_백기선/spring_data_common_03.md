---
layout: default
title: Spring Data Common 3
parent: 스프링데이터JPA_백기선
nav_order: 7
---

- AbstractAggregateRoot 이용하여 도메인 이벤트 발생시키기
- Query DSL
  - Query DSL 이란
  - Query DSL 작동 원리

## AbstractAggregateRoot 이용하여 도메인 이벤트 발생시키기
기존에 ApplicationEventPublisher 를 통해서 수행한 Event 발행 방식을 AbstractAggregateRoot 를 이용해서 도메인 내에서 바로 발생시킬 수 있다.
```java
/**
 * Convenience base class for aggregate roots that exposes a {@link #registerEvent(Object)} to capture domain events and
 * expose them via {@link #domainEvents()}. The implementation is using the general event publication mechanism implied
 * by {@link DomainEvents} and {@link AfterDomainEventPublication}. If in doubt or need to customize anything here,
 * rather build your own base class and use the annotations directly.
 *
 * @author Oliver Gierke
 * @author Christoph Strobl
 * @since 1.13
 */
public class AbstractAggregateRoot<A extends AbstractAggregateRoot<A>> {

	private transient final @Transient List<Object> domainEvents = new ArrayList<>();

	/**
	 * Registers the given event object for publication on a call to a Spring Data repository's save methods.
	 *
	 * @param event must not be {@literal null}.
	 * @return the event that has been added.
	 * @see #andEvent(Object)
	 */
	protected <T> T registerEvent(T event) {

		Assert.notNull(event, "Domain event must not be null");

		this.domainEvents.add(event);
		return event;
	}

	/**
	 * Clears all domain events currently held. Usually invoked by the infrastructure in place in Spring Data
	 * repositories.
	 */
	@AfterDomainEventPublication
	protected void clearDomainEvents() {
		this.domainEvents.clear();
	}

	/**
	 * All domain events currently captured by the aggregate.
	 */
	@DomainEvents
	protected Collection<Object> domainEvents() {
		return Collections.unmodifiableList(domainEvents);
	}

	/**
	 * Adds all events contained in the given aggregate to the current one.
	 *
	 * @param aggregate must not be {@literal null}.
	 * @return the aggregate
	 */
	@SuppressWarnings("unchecked")
	protected final A andEventsFrom(A aggregate) {

		Assert.notNull(aggregate, "Aggregate must not be null");

		this.domainEvents.addAll(aggregate.domainEvents());

		return (A) this;
	}

	/**
	 * Adds the given event to the aggregate for later publication when calling a Spring Data repository's save-method.
	 * Does the same as {@link #registerEvent(Object)} but returns the aggregate instead of the event.
	 *
	 * @param event must not be {@literal null}.
	 * @return the aggregate
	 * @see #registerEvent(Object)
	 */
	@SuppressWarnings("unchecked")
	protected final A andEvent(Object event) {

		registerEvent(event);

		return (A) this;
	}
}
```
{: .point }
Convenience base class for aggregate roots that exposes a {@link #registerEvent(Object)} to capture domain events and
expose them via {@link #domainEvents()}.

수행하는 과정에 domainEvents() 에 임시로 담겼다가 이벤트 발생 이후에는 메모리 누수 방지를 위해서 clearDomainEvents() 를 통해 제거된다.
위 모든 과정은 save() 와 동시에 발생된다.

```java
@Entity
public class Post extends AbstractAggregateRoot<Post> {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String description;

    public void setName(String name) {
        this.name = name;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    // 여기서 event를 AbstractAggregateRoot를 통해서 등록한다. 
    public void registerEvent() {
        PostPublishedEvent postPublishedEvent = new PostPublishedEvent(this);
        this.registerEvent(postPublishedEvent);
    }
}
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Post post = new Post();
        post.setName("event publish test");
        post.setDescription("event description");

        post.registerEvent(); // <-- 도메인 내에서 선언한 event publish 처리를 해준다.
        postRepository.save(post);
    }
```

## Query DSL 

### Query DSL 이란
[Query DSL 공식 홈페이지] 에 있는 [Querydsl Reference Guide] 에 보면 탄생 배경은 [HQL] 을 타입 세이프 하게 쓰기 위해 생겼고, 지금은 여러 모듈을 지원한다고 나와있다.

{: .point }
Querydsl was born out of the need to maintain HQL queries in a typesafe way.
HQL for Hibernate was the first target language for Querydsl, but nowadays it supports JPA, JDO, JDBC, Lucene, Hibernate Search, MongoDB, Collections and RDFBean as backends.

### Query DSL 작동 원리
Query DSL 사용을 위해서는 Predicate 을 만들 수 있어야 하고 그렇기 때문에 Entity 별로 Q 클래스가 필요하다.
설정을 통해 추가해주는 라이브러리와 플러그인에 따라 컴파일 타임에 개입하여 이러한 Q 클래스들을 만들어 준다.

Query DSL 공식 문서에는 아예 gradle 세팅 정보가 없다. 그래서 여러 블로그를 봤는데 [이 블로그]가 정리가 가장 잘 되어있는 것 같았다.
```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.6'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'com.ewerk.gradle.plugins.querydsl' version "1.0.10"
}

group = 'com.fistkim'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
    mavenCentral()
}

dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'mysql:mysql-connector-java:8.0.30'
    implementation 'com.querydsl:querydsl-jpa'
    implementation 'com.querydsl:querydsl-apt:5.0.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}

// querydsl 사용할 경로 지정합니다. 현재 지정한 부분은 .gitignore에 포함되므로 git에 올라가지 않습니다.
def querydslDir = "$buildDir/generated/'querydsl'"

// JPA 사용여부 및 사용 경로 설정
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

// build시 사용할 sourceSet 추가 설정
sourceSets {
    main.java.srcDir querydslDir
}

// querydsl 컴파일 시 사용할 옵션 설정
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}

// querydsl이 compileClassPath를 상속하도록 설정
configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
    querydsl.extendsFrom compileClasspath
}
```

위와 같이 라이브러리 및 플러그인을 지정해주고 설정을 잡아줘야한다.
그래야 컴파일타임에 설정해둔 소스폴더로 Q클래스들이 들어가고 이것이 source 폴더로 포함되어서 코드에서 참조할 수 있다.
그리고 Repository 에서는 아래와 같이 QuerydslPredicateExecutor<T> 를 상속해줘야한다.

```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository, QuerydslPredicateExecutor<Post> {
}
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Post post = new Post();
        post.setName("queryDslTestName");
        post.setDescription("queryDslTestDescription");
        postRepository.save(post);

        QPost targetPost = QPost.post;
        Predicate predicate = targetPost
                .name.contains("DslTestN")
                .and(targetPost.description.contains("DslTestDes"));
        Optional<Post> target = postRepository.findOne(predicate);

        System.out.println("-----------------------");
        System.out.println(target.get().getName());
        System.out.println(target.get().getDescription());
        System.out.println("-----------------------");
    }
```

위와 같이 타입세이프 하게 Predicate 을 사용할 수 있다. 다시 query DSL 의 필요성 및 장점을 인지하고 넘어가자면
위와 같은 조건식의 쿼리를 직접 만든다고 했을 때, 내가 sql 을 @Query 내에서 작성을 해야하는데
그 과정에서 오타가 난다던가 잘못된 문법을 쓴다던가 하는 식의 오류가 발생할 수 있는 것이다. 

[Query DSL 공식 홈페이지]: http://querydsl.com/
[Querydsl Reference Guide]: http://querydsl.com/static/querydsl/5.0.0/reference/html_single/#d0e97
[HQL]: https://www.digitalocean.com/community/tutorials/hibernate-query-language-hql-example-tutorial
[이 블로그]: https://data-make.tistory.com/728