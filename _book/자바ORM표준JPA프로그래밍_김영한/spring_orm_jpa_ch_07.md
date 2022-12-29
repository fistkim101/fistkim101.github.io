---
layout: default
title: 고급 매핑
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 7
---

- 상속관계 매핑
- @MappedSuperclass
- 복합키 매핑 

## 상속관계 매핑
[상속관계 매핑에 관해 정리된 포스트](https://ict-nroo.tistory.com/128) 가 있어 링크를 남긴다.
객체 패러다임에서 존재하는 상속 관계를 테이블 패러다임에서 어떻게 표현할지에 관한 방법들과 이 방법들을 위해서 JPA를 어떻게 사용해야하는지에 관한 내용이다.

이건 정답이 있는 것은 아니고 상황에 따라 잘 고민해서 사용해야할 것 같다. 결국 그 고민을 하는 시점에 얼마나 많은 옵션들을 머릿속에 두고 골라서
트레이드오프를 고려할 줄 아느냐가 중요한 것 같다. 책도 강의도 조인전략을 기본으로 추천한다.

조인전략으로 가도 테이블이 많아져서 복잡도가 올라가니 개인적으로는 끌리진 않는다. 그냥 단일 테이블로 하되 필요한 커스텀 필드들은 attribute1, attribute2 와 같이
커스텀 필드로 만들어서 json 으로 넣어버리던가(들어갈 내용이 조회에 사용할게 아니라면) 하는 식으로 테이블을 단순하게 하나로 가져가고 종류를 ENUM 으로 구분하는 등의
단순한 방식이 개인적으로 더 좋은 것 같고 실무에서도 그런식으로 많이 했던 것 같다.

조인전략만 가볍게 실습해보았다.
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "type")
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private int price;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }
}
```
```java
@Entity
public class Album extends Item {

    private String singer;

    public String getSinger() {
        return singer;
    }

    public void setSinger(String singer) {
        this.singer = singer;
    }
}
```
```java
@Entity
public class Movie extends Item {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        Album album = new Album();
        album.setName("아이템이름");
        album.setPrice(10000);
        album.setSinger("가수이름");

        entityManager.persist(album);
    }
```
```sql
Hibernate: 
    
    create table album (
       singer varchar(255),
        id bigint not null,
        primary key (id)
    ) engine=InnoDB
Hibernate: 
    
    create table item (
       type varchar(31) not null,
        id bigint not null auto_increment,
        name varchar(255),
        price integer not null,
        primary key (id)
    ) engine=InnoDB

alter table album
    add constraint FKrl4nl1yn7tatob2buih6y9qws
        foreign key (id)
            references item (id)
```

<br>

![](/images/jpa-inheritance.png)
![](/images/jpa-inheritance-01.png)
![](/images/jpa-inheritance-02.png)

출처 : 강의자료

type 에 들어갈 값은 기본적으로 클래스 명을 쓰지만 바꾸고싶다면 아래와 같이 바꿀 수 있다.
```java
@Entity
@DiscriminatorValue("ALBUM")
public class Album extends Item {
// 중략
}
```

## @MappedSuperclass
[@MappedSuperclass 에 관해 정리된 포스트](https://ict-nroo.tistory.com/129) 를 발견해서 링크만 남긴다.
공통필드 제공 클래스인데, 사실 공통 정보가 그리 많지 않으면 굳이 써야하나 싶다.
반복코드가 만들어지긴 하는데, 로직도 아니고 필드인데 굳이 빼야하나 싶다.

## 복합키 매핑
[복합키 매핑에 관해 정리된 포스트](https://velog.io/@sa1341/JPA-%EB%B3%B5%ED%95%A9-%ED%82%A4%EA%B3%BC-%EC%8B%9D%EB%B3%84-%EA%B4%80%EA%B3%84-%EB%A7%A4%ED%95%91) 가 있어 링크를 남긴다.
많이 팔린 책이라 그런지 책 내용을 정리해놓은 포스트가 많다.
