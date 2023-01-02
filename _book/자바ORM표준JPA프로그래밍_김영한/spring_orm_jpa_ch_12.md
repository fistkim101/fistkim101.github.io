---
layout: default
title: 스프링 데이터 JPA
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 12
---

- JpaRepository 계층구조
- Specification 에서의 Join 사용

## JpaRepository 계층구조

![](/images/jpa-interface-hierarchy.png)

출처 : https://backtony.github.io/jpa/2021-04-01-jpa-springdatajpa-1/

이미 백기선님 강의에서 다뤘던 부분이라 계층구조 이미지만 남기고 넘어간다.
기본적인 CRUD 오퍼레이션은 CrudInterface 에서 정의 되어있다. JpaRepository가 아니다.

## Specification 에서의 Join 사용

Specification 자체는 백기선님 강의에서도 그렇고 프로젝트에서도 이미 썼었는데, 이번 챕터에서 공부하면서 여러 테이블을 조인해서 Specification을 사용할 수 있는지가 궁금했고
[이 질문](https://stackoverflow.com/questions/38168108/spring-data-join-with-specifications)에 어느정도 정리가 되어 있었다.

나중에 실제로 사용할 일이 있을때 참고해서 써보고 정리를 다시 해보자.