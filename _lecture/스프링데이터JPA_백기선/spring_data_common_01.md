---
layout: default
title: Spring Data Common 1
parent: 스프링데이터JPA_백기선
nav_order: 5
---

- Spring Data 구성 및 학습순서
- Spring Data Repository 

## Spring Data 구성 및 학습순서

![](/images/concept-spring-data-common.png)

{: .careful }
Spring Data Common 의 자원은 위 그림에서 확인할 수 있듯이 JPA보다 상위 개념으로
MongoDB, Redis 등 어디에도 사용이 가능하다.

## Spring Data Repository

![](/images/concept-spring-data-repository-hierarchy.png)

출처 : 강의자료

지금까지 얼마나 무지하고 무심하게 JPA를 사용했는지 이번 시간에 다시 한번 느낄 수 있었다.
자동으로 이뤄지는 일들에 대해서 깊게 파보고 이해하고 있는 것이 아니면
그냥 그 기술은 그냥 '모르는' 기술이라고 판단하기로 했다.

위 그림처럼 Repository 도 계층이 구분되어 있다. 그래서 'Spring Data 구성 및 학습순서' 에서도 다뤘듯이
Spring Data Common 계층의 Repository 는 DB가 바뀌어도 사용이 가능한 것이다.

이러한 계층을 인지하고 있는 상태에서 Repository 를 구성 및 사용할 수 있어야 한다.

```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
```
```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {

	/**
	 * Returns all entities sorted by the given options.
	 *
	 * @param sort the {@link Sort} specification to sort the results by, can be {@link Sort#unsorted()}, must not be
	 *          {@literal null}.
	 * @return all entities sorted by the given options
	 */
	Iterable<T> findAll(Sort sort);

	/**
	 * Returns a {@link Page} of entities meeting the paging restriction provided in the {@link Pageable} object.
	 *
	 * @param pageable the pageable to request a paged result, can be {@link Pageable#unpaged()}, must not be
	 *          {@literal null}.
	 * @return a page of entities
	 */
	Page<T> findAll(Pageable pageable);
}
```
```java
/**
 * Interface for generic CRUD operations on a repository for a specific type.
 *
 * @author Oliver Gierke
 * @author Eberhard Wolff
 * @author Jens Schauder
 */
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
```
```java
/**
 * Central repository marker interface. Captures the domain type to manage as well as the domain type's id type. General
 * purpose is to hold type information as well as being able to discover interfaces that extend this one during
 * classpath scanning for easy Spring bean creation.
 * <p>
 * Domain repositories extending this interface can selectively expose CRUD methods by simply declaring methods of the
 * same signature as those declared in {@link CrudRepository}.
 * 
 * @see CrudRepository
 * @param <T> the domain type the repository manages
 * @param <ID> the type of the id of the entity the repository manages
 * @author Oliver Gierke
 */
@Indexed
public interface Repository<T, ID> {

}
```

여기서 중간 계층의 Repository 에는 @NoRepositoryBean 가 있음으로 인해서 핵심개념이해에서 다뤘던 Repository를 bean으로 등록해주는 과정에서
bean 등록이 이뤄지지 않고 넘어가는 것 같다. 이는 나중에 커스텀 레포지토리를 다룰 때 자세하게 다룰 수 있을 것 같아서 지금은 넘어가기로 한다.
