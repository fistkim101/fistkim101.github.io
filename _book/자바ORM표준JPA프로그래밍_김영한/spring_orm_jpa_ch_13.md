---
layout: default
title: 웹 애플리케이션과 영속성 관리
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 13
---

- 프레젠테이션 레이어에서의 지연로딩
  - 프레젠테이션 레이어에서 지연로딩이 가능할까?
    - 트랜젝션 범위와 영속성 컨텍스트 생명주기의 분리 필요성
    - OSIV 개념과 원리
  - 영속성 컨텍스트가 살아있지 않다면 사용할 수 있는 방식
    - 강제 초기화
    - FACADE
    - 프레젠테이션 제공을 위한 DTO 변환 

## 프레젠테이션 레이어에서의 지연로딩
정답은 없지만 보통(적어도 나는 그러하다) 프레젠테이션 레이어에서 응답을 위한 데이터 컨버팅 작업을 하게 된다. 당연한 말이지만 프레젠테이션 레이어의 책임중 하나이기 때문이다.
서비스 레이어에서 응답을 위한 데이터 변환을 책임지는 것은 근본적인 레이어 분리에 따라 부여하는 책임에 맞지 않는 것 같다.

아무튼 프레젠테이션 레이어에서 응답 형식에 맞는 데이터 컨버팅 작업을 하게 되는데 이 때 LAZY 로딩을 사용할 일이 분명 존재한다.
이 상황에서 인지하고 있어야할 여러 개념들을 정리한다.

### 프레젠테이션 레이어에서 지연로딩이 가능할까? - 트랜젝션 범위와 영속성 컨텍스트 생명주기의 분리 필요성
지연로딩이 가능하냐의 질문은 결국 '영속성 컨텍스트가 살아있는가'와 같은 이야기가 된다. 결론적으로 스프링부트에서는 영속성 컨텍스트가 프레젠테이션 레이어에도 존재하기에 가능하다.

중요한 것은 스프링부트가 알아서 해주니까 신경을 안써도 된다가 아니고, 어떤 의도에서 프레젠테이션 레이어에서 영속성 컨텍스트를 살려두었는지와
만약 이것이 살아있지 않다면 어떤 문제가 있을지에 대해서 이해하고 있어야 한다.

프레젠테이션 레이어에서 지연로딩을 사용하고싶다는 전제하에 트랜젝션의 범위와 영속성 컨텍스트의 생명주기를 똑같이 한다면 아래와 같은 형태가 된다.

![](/images/jpa-persistence-context-transaction-01.png)

출처 : 강의자료

이때 문제가 되는 것은 프레젠테이션 레이어가 데이터를 변경할 수 있다는 것이다. dirty checking 이 발생하여 의도치않게 데이터를 변경할 수 있는 여지가 있다.

그래서 영속성 컨텍스트를 프레젠테이션 레이어에서 열어두되, 데이터 변경 로직이 있는 서비스 레이어에서만 트랜젝션이 열리고 닫히도록 해야한다.
스프링부트에서는 이것이 가능하도록 처리해주는데 OSIV 라는 개념이다. 

### 프레젠테이션 레이어에서 지연로딩이 가능할까? - OSIV(Open Session In View)

아래 그림이 곧 스프링부트가 제공하고 있는 구조이다.

![](/images/jpa-persistence-context-transaction-02.png)

출처 : 강의자료

이게 실제로 맞는지 아래와 같이 확인해보았다.

```java

@RestController
public class SampleController {

    @PersistenceContext
    private EntityManager entityManager;

    private final SampleService sampleService;

    public SampleController(SampleService sampleService) {
        this.sampleService = sampleService;
    }

    @GetMapping("/check")
    public String sampleHandler() {
        System.out.println(">>> controller entityManager : " + entityManager);
        System.out.println(">>> controller before entityManager.isOpen() : " + entityManager.isOpen());

        TransactionStatus status = null;
        try {
            status = TransactionAspectSupport.currentTransactionStatus();
            System.out.println(">>> controller isNewTransaction : " + status.isNewTransaction());
        } catch (Exception exception) {
            System.out.println("before : not in transaction now ...");
        }

        // service logic
        Team team = sampleService.check();

        System.out.println(">>> controller after entityManager.isOpen() : " + entityManager.isOpen());

        System.out.println("================================");
        team.getMembers().forEach(member -> System.out.println(member.getName()));
        System.out.println("================================");

        try {
            status = TransactionAspectSupport.currentTransactionStatus();
            System.out.println(">>> controller isNewTransaction : " + status.isNewTransaction());
        } catch (Exception exception) {
            System.out.println("after : not in transaction now ...");
        }

        return "ok";
    }

}
```
```java
@Service
public class SampleService {

    @PersistenceContext
    private EntityManager entityManager;

    private final TeamRepository teamRepository;

    public SampleService(TeamRepository teamRepository) {
        this.teamRepository = teamRepository;
    }

    @Transactional
    public Team check() {
        System.out.println(">>> service entityManager : " + entityManager);

        TransactionStatus status = TransactionAspectSupport.currentTransactionStatus();
        System.out.println(">>> service isNewTransaction : " + status.isNewTransaction());

        Team team = new Team();
        team.setName("teamName");

        Member member = new Member();
        member.setName("sampleMember");
        member.setTeam(team);

        return teamRepository.findById(1L).get();
    }

}
```
```bash
>>> controller entityManager : Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@7f41d979]
>>> controller before entityManager.isOpen() : true
before : not in transaction now ...
>>> service entityManager : Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@7f41d979]
>>> service isNewTransaction : true
Hibernate: 
    select
        team0_.id as id1_2_0_,
        team0_.name as name2_2_0_ 
    from
        team team0_ 
    where
        team0_.id=?
2023-01-02 10:39:15.453 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
2023-01-02 10:39:15.470 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([name2_2_0_] : [VARCHAR]) - [teamName]
>>> controller after entityManager.isOpen() : true
================================
Hibernate: 
    select
        members0_.team_id as team_id3_1_0_,
        members0_.id as id1_1_0_,
        members0_.id as id1_1_1_,
        members0_.name as name2_1_1_,
        members0_.team_id as team_id3_1_1_,
        locker1_.id as id1_0_2_,
        locker1_.member_id as member_i3_0_2_,
        locker1_.number as number2_0_2_ 
    from
        member members0_ 
    left outer join
        locker locker1_ 
            on members0_.id=locker1_.member_id 
    where
        members0_.team_id=?
2023-01-02 10:39:15.496 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
2023-01-02 10:39:15.522 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([id1_1_1_] : [BIGINT]) - [2]
2023-01-02 10:39:15.522 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([id1_0_2_] : [BIGINT]) - [null]
2023-01-02 10:39:15.523 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([name2_1_1_] : [VARCHAR]) - [sampleMember]
2023-01-02 10:39:15.523 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([team_id3_1_1_] : [BIGINT]) - [1]
2023-01-02 10:39:15.523 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([team_id3_1_0_] : [BIGINT]) - [1]
2023-01-02 10:39:15.523 TRACE 4545 --- [nio-8080-exec-1] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([id1_1_0_] : [BIGINT]) - [2]
sampleMember
================================
after : not in transaction now ...
2023-01-02 10:45:08.863  WARN 4545 --- [l-1 housekeeper] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Retrograde clock change detected (housekeeper delta=29s843ms), soft-evicting connections from pool.
```

1. 클라이언트 요청이 들어오면 서블릿 필터나 스프링 인터셉터에서 영속성 컨텍스트를 생성한다. 단, 이때 트랜젝션은 시작하지 않는다.
2. 서비스 계층에서 @Transactional로 트랜젝션을 시작할 때 1번에서 미리 생성해둔 영속성 컨텍스트를 찾아와서 트랜젝션을 시작한다.
3. 서비스 계층이 끝나면 트랜젝션을 커밋하고 영속성 컨텍스트를 플러시한다. 이때 트랜젝션은 끝나지만 영속성 컨텍스트는 종료하지 않는다.
4. 컨트롤러와 뷰까지 영속성 컨텍스트가 유지되므로 조회한 엔티티는 영속 상태를 유지한다.
5. 서블릿 필터나 스프링 인터셉터로 요청이 돌아오면 영속성 컨텍스트를 종료한다. 이 때 플러시를 호출하지 않고 바로 종료한다.

## 영속성 컨텍스트가 살아있지 않다면 사용할 수 있는 방식

만약 프레젠테이션 레이어에서 영속성 컨텍스트가 생존해있지 않다면(=OSIV 를 off 처리한 상태) 아래와 같은 그림이 될 것이다.
![](/images/jpa-persistence-context-transaction-03.png)

스프링부트를 사용하는 입장에서 위와 같은 상황을 대비할 일은 없겠지만 이해도를 높히기 위해서 위와 같은 상황일때 어떻게 처리를 해야할지에 대해서 알아본다.(책에 정리가 되어있다)

### 강제 초기화
프레젠테이션 레이어로 가기 전에 강제로 프록시 객체를 호출해서 프록시 객체를 강제초기화 하는 방법이다. 이게 큰 틀에서 유일한 방법이다.

### FACADE
중간에 초기화를 담당해주는 계층을 따로 두는 방식으로 책임을 가지는 레이어를 만든다.
![](/images/jpa-persistence-context-facade.png)

### 프레젠테이션 제공을 위한 DTO 변환 
프레젠테이션 레이어에서 사용할 수 있는 DTO로 서비스 레이어가 변환을 해서 넘겨주는 방법을 하면 그 과정에 초기화가 되면서 실제 엔티티의 데이터를 끌어다 쓸 수 있다.
