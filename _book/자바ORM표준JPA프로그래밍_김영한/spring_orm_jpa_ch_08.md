---
layout: default
title: 프록시와 연관관계 관리
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 8
---

- 프록시
  - 하이버네이트에서의 프록시
  - 프록시 객체의 초기화
  - 컬렉션 타입의 프록시
- 결국 fetch 전략을 어떻게 선택할까
- 컬렉션 타입에 EAGER 사용시 주의할 점 두 가지
- 영속성 전이와 DDD

## 프록시
프록시는 프록시서버에서 많이 접한 개념인데 [프록시라는 영어단어의 뜻](https://en.dict.naver.com/#/entry/enko/2ba09ebcdb6a4c22bd17e98caae1f4c5) 자체가 '대리인', '대리물' 이다.
디자인 패턴에서도 [프록시 패턴](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9D%EC%8B%9C_%ED%8C%A8%ED%84%B4) 이 존재하는데 하이버네이트에서도 프록시 패턴과 같은 방식으로 지연로딩을 제공한다.
위 링크에 있는 프록시 패턴의 코드를 남긴다.

쉽게 표현하면 일단은 겉보기에는 실제와 일치하는 번듯한 것을 제공하고, 실제로 이걸 까보려 하면 이제야 실제 자원을 가져와 이를 통해서 필요한 것을 제공하는 것이다. 

![](/images/concept-proxy-pattern.png)

```java
import java.util.*;

interface Image {
    public void displayImage();
}

//on System A
class RealImage implements Image {
    private String filename;
    public RealImage(String filename) {
        this.filename = filename;
        loadImageFromDisk();
    }

    private void loadImageFromDisk() {
        System.out.println("Loading   " + filename);
    }

    @Override
    public void displayImage() {
        System.out.println("Displaying " + filename);
    }
}

//on System B
class ProxyImage implements Image {
    private String filename;
    private Image image;

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    @Override
    public void displayImage() {
        if (image == null)
           image = new RealImage(filename);

        image.displayImage();
    }
}

class ProxyExample {
    public static void main(String[] args) {
        Image image1 = new ProxyImage("HiRes_10MB_Photo1");
        Image image2 = new ProxyImage("HiRes_10MB_Photo2");

        image1.displayImage(); // loading necessary
        image2.displayImage(); // loading necessary
    }
}
```

### 하이버네이트에서의 프록시
하이버네이트에서 LAZY 하게 가져오는 방식과 아래 방식은 동일하다. entityManager.getReference(T, PK) 를 이용해서 프록시 객체를 가져올 수 있다.
fetch 전략이 LAZY 옵션이라면 이미 첫 행에서 team 에 프록시 객체를 가져온 상황이긴 하지만 이해를 위해 아래와 같이 세팅해본다.

```java
Member member = entityManager.find(Member.class, 1L);
Team team = entityManager.getReference(Team.class, member.getTeam().getId()); // SQL 실행하지 않음
member.setTeam(team);
System.out.println(team.getClass().getName());
```
```bash
com.fistkim.springjpawhiteshipstudy.Team$HibernateProxy$4ZcChNbx
```

EAGER 전략을 택하면 프록시 객체가 아닌 entity 가 return 된다.
```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(referencedColumnName = "id")
private Team team;
```
```bash
com.fistkim.springjpawhiteshipstudy.Team
```

### 프록시 객체의 초기화

아래 흐름을 잘 기억해두자. 실제로 프록시 객체를 통해서 realEntity 를 참조할 일이 생기면 비로소 영속성 컨텍스트를 조회해보고 없으면 DB를 조회하여
Entity 객체를 생성한다.

이 때도 프록시 객체가 Entity 로 바뀌는 것이 아니라 프록시 객체 내에 realEntity 를 바라보는 참조가 채워지는 것이다.

![](/images/concep-jpa-proxy-initialize.png)

### 컬렉션 타입의 프록시

하이버네이트는 엔티티에 컬렉션이 있는 경우 이를 추적하고 관리할 목적으로 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 이걸 '컬렉션 래퍼' 라고 한다.
```java
Team team = entityManager.find(Team.class, 1L);
System.out.println(team.getMembers().getClass().getName());
```
```bash
org.hibernate.collection.internal.PersistentSet
```

컬랙션 레퍼도 프록시와 동일한 역할을 수행한다.

## 결국 fetch 전략을 어떻게 선택할까

너무 뻔한 말이지만 연관관계에 있되 거의 항상 자주 사용한다 싶으면 EAGER 전략을 취하고 그렇지 않으면 LAZY를 택한다.
책과 강의에서는 일단 모두 LAZY 로 가져오도록 처리하라고 권고한다.

{: .warning }
LAZY로 인해서 발생하는 문제는 fetch join, batch size, 어노테이션 등 차후에 나올 개념으로 해결한다.

{: .warning }
무조건 그냥 지연 로딩만 사용. 즉시 로딩은 절대 사용하지 않는다.

{: .careful }
*무조건 그냥 지연 로딩만 사용. 즉시 로딩은 절대 사용하지 않는다.<br>
*즉시 로딩은 JPQL 에서 N+1 문제를 일으킨다.(드라이빙 테이블에서 가져오고 보니 필드가 EAGER 인게 있어서 그것의 개수만큼 모두 조회 또 한다)<br>
*@ManyToOne, @OneToOne 의 기본값이 EAGER니까 그건 직접 LAZY로 변경<br>

![](/images/jpa-lazy-eager-careful.png)

## 컬렉션 타입에 EAGER 사용시 주의할 점 두 가지

1. 컬렉션 하나를 초과하여 즉시 로딩은 삼가한다.
N과 M개 두 컬렉션이 있는 경우 이를 둘다 EAGER 로 가져오게 되면 N * M 의 경우의 수를 가져와야해서 성능에 악영향을 주므로 삼가한다.
   
2. 컬렉션 즉시 로딩은 되도록 외부조인을 사용한다. 
   
optional 에 따라서 내부 조인, 외부 조인 이 다르게 선택된다.
@OneToMany, @ManyToMany의 경우 optional 에 상관없이 외부 조인으로 동작하고,
@ManyToOne, @OneToOne의 경우 optional이 false 일때만 내부 조인으로 동작한다.

성능을 위해서는 내부 조인이 더 좋겠지만 내부조인을 만족하지 않는 경우 아예 조회로 엔티티가 잡히지 않는 경우가 있어 신중하게 접근해야 한다.

```java
Member member = entityManager.find(Member.class, 1L);
```
```java
@ManyToOne(fetch = FetchType.EAGER, optional = true)
@JoinColumn(referencedColumnName = "id")
private Team team;

@ManyToOne(fetch = FetchType.EAGER, optional = false)
@JoinColumn(referencedColumnName = "id")
private Team team;
```
```sql
Hibernate:
select
  member0_.id as id1_0_0_,
  member0_.name as name2_0_0_,
  member0_.team_id as team_id3_0_0_,
  team1_.id as id1_1_1_,
  team1_.name as name2_1_1_
from
  member member0_
    left outer join
  team team1_
  on member0_.team_id=team1_.id
where
  member0_.id=?

Hibernate:
select
  member0_.id as id1_0_0_,
  member0_.name as name2_0_0_,
  member0_.team_id as team_id3_0_0_,
  team1_.id as id1_1_1_,
  team1_.name as name2_1_1_
from
  member member0_
    inner join
  team team1_
  on member0_.team_id=team1_.id
where
  member0_.id=?
```

## [영속성 전이와 DDD](https://www.popit.kr/%EC%97%90%EA%B7%B8%EB%A6%AC%EA%B2%8C%EC%9E%87-%ED%95%98%EB%82%98%EC%97%90-%EB%A6%AC%ED%8C%8C%EC%A7%80%ED%86%A0%EB%A6%AC-%ED%95%98%EB%82%98/)
DDD 책을 읽을때 참고해서 봤던 포스팅인데 당시에 도움이 많이 되었었다. 영속성 전이 전략과 엮어서 내용을 이해해두면 좋을 것 같다.  

또 한 가지로 강의에서 cascade 사용시 주의할 점을 아래와 같이 강조한다.

{: .careful }
*단일 소유자가 분명한 child 에만 영속성 전이<br>
*라이프사이클이 똑같은 child 에만 영속성 전이
