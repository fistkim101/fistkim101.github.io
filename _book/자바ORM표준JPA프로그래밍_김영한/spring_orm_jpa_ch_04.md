---
layout: default
title: 엔티티 매핑
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 4
---

- @Entity 객체 설계시 기본 생성자(no args constructor)는 반드시 필요할까?
- @Entity, @Table name 필드 기본값은 무엇이며 어떻게 활용되는지
- @Column
  - unique 필드를 사용해야할까?
- 자연키, 대체키

## @Entity 객체 설계시 기본 생성자(no args constructor)는 반드시 필요할까?

[똑같은 질문](https://stackoverflow.com/questions/69082992/is-it-mandatory-to-use-no-args-constructor-using-entity-annotation-in-java-spri)이 스택 오버 플로우에 이미 올라와 있다.

결론부터 말하자면 하이버네이트가 @Entity 로 명시된 객체를 리플렉션을 사용해서 만들기 때문이다.[(하이버네이트 공식문서 참고)](https://docs.jboss.org/hibernate/orm/5.5/quickstart/html_single/#hibernate-gsg-tutorial-basic-entity)

{: .careful }
The no-argument constructor, which is also a JavaBean convention, is a requirement for all persistent classes.
Hibernate needs to create objects for you, using Java Reflection. The constructor can be private.
However, package or public visibility is required for runtime proxy generation and efficient data retrieval without bytecode instrumentation.

## @Entity, @Table name 필드 기본값은 무엇이며 어떻게 활용되는지

백기선님 강의에서도 이미 다뤘던 주제이지만 그냥 한번 더 정리한다.

@Entity의 name 은 하이버네이트가 entity 객체를 관리하는 이름인데 기본적으로 클래스 이름을 사용한다.

그리고 @Table 의 name 은 해당 Entity 객체를 어떤 테이블과 매핑할지에 관한 정보인데 기본값으로 Entity 의 name을 가지고 매핑전략에 맞춰서 매핑한다.

결국 @Entity 만 달아주고 그 외에 아무 것도 추가해주지 않으면 클래스 이름을 가지고 하이버네이트의 매핑전략에 맞춰서 해당 객체가 테이블과 매핑되는 것이다.

## @Column

### unique 필드를 사용해야할까?
필드의 컬럼 속성에서 unique 를 사용할 경우 아래와 같이 유니크 명이 랜덤하게 사용된다.
```bash
    alter table post 
       add constraint UK_lnx7ylm6ckohautp1gua177et unique (name)
```

보통 회사 컨벤션이 있기 마련이므로 위와 같은 방식의 생성은 옳지 못하다.

따라서 @Column 내 unique 필드 사용은 지양하고, 아래와 같이 @Table 의 uniqueConstraints 를 사용한다.
이렇게 해줄 경우 이름을 따로 지정해줄 수 있을 뿐만 아니라 복합키로 Unique 제약조건을 걸 수도 있기 때문에 이점이 더 크다. 
```java
@Entity
@Table(uniqueConstraints = {@UniqueConstraint(name = "UQ_NAME", columnNames = "name")})
public class Post {
  // 중략
}
```
```bash
    alter table post 
       add constraint UQ_NAME unique (name)
```

## 자연키, 대체키
개념은 쉬운데 용어자체가 생소해서 [레퍼런스](https://www.mssqltips.com/sqlservertip/5431/surrogate-key-vs-natural-key-differences-and-when-to-use-in-sql-server/)를 남겨둔다.

아무래도 변경 가능성이 1%라도 존재하는 자연키 보다는 대체키를 사용하는 것이 맞고 책에서도 그렇게 강조하고 있다.
