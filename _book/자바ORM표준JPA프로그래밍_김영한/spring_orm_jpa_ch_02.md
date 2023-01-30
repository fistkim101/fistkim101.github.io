---
layout: default
title: JPA 시작
parent: 자바 ORM 표준 JPA 프로그래밍
nav_order: 2
---

- 직접 EntityManager 생성하여 트랜잭션 관리 코드 샘플 작성
- SpringBoot 에서의 JPA 아키텍처
  - JPA class 생성 흐름
  - JPA class 관계

## 직접 EntityManager 생성하여 트랜잭션 관리 코드 샘플 작성

아래와 같이 EntityManagerFactory 부터 생성해서 궁극적으로 트랜젝션을 시작, 종료하는 샘플 앱을 만들어보는 단원이었다.

```java
        EntityManagerFactory entityManagerFactory=Persistence.createEntityManagerFactory("persistenceUnitName");
        EntityManager entityManager=entityManagerFactory.createEntityManager();
        EntityTransaction transaction=entityManager.getTransaction();
        
        try{
            transaction.begin();
            // business logic
            transaction.commit();
        } catch (Exception exception) {
            transaction.rollback();
        } finally {
            entityManager.close();
        }
```

JPA 를 사용하면 위와 같은 EntityManager 생성 및 트랜잭션 관리가 알아서 되는 것이고,
개발자들은 이를 제대로 활용하기 위해서는 스프링에서 '알아서' 해주는 그런 것들이 어떤 것들이 있는지와 어떤 과정으로 이뤄지는지를 알고 있어야 한다. 

## SpringBoot 에서의 JPA 아키텍처

[이 사이트](https://www.javatpoint.com/spring-boot-jpa) 에 정리가 매우 잘되어 있어서 그대로 가져왔다.

![](/images/concept-spring-jpa-architecture.png)
![](/images/spring-jpa-architecture.png)
![](/images/spring-jpa-class-relationship.png)

여기서 persistence unit은 아래와 같이 Database 접속 정보를 가지고 있는 configuration 이라고 보면 된다. 

```xml
    <persistence-unit name="hello">
        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />
            <property name="javax.persistence.jdbc.user" value="berry"/>
            <property name="javax.persistence.jdbc.password" value="straw"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost/jpa-practice-schema"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>

        </properties>

    </persistence-unit>
```

위 구조에서 entityManagerFactory의 경우 Bean으로 자동 생성된다. Runner 에서 아래와 같이 직접 확인해본다.
```java
    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        String[] beans = applicationContext.getBeanDefinitionNames();

        for (String bean : beans) {
            System.out.println("bean : " + bean);
        }

    }
```
```bash
...(중략)...
bean : entityManagerFactoryBuilder
bean : entityManagerFactory
...(중략)...
```

EntityManager 는 @PersistenceContext 를 통해서 아래와 같이 entityManagerFactory 로 생성해서 주입받아 사용한다.
```java
    @PersistenceContext
    private EntityManager entityManager;
```

책에서도 강조하듯 EntityManager는 데이터 베이스 커넥션과 밀접하게 작동하므로 스레드간에 공유하거나 재사용을 하면 안된다.
아래와 같이 EntityManager 를 변수명을 달리 하여 두 개를 생성해봐도 결국 주소값을 비교해보면 동일한 EntityManager 임을 확인할 수 있다.

```java
@Component
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    private EntityManager entityManager1;

    @PersistenceContext
    private EntityManager entityManager2;


    @Override
    @Transactional
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(">>>>>");
        System.out.println(entityManager1);
        System.out.println(entityManager2);
        System.out.println("<<<<<");
    }

}
```
```bash
>>>>>
Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@103478b8]
Shared EntityManager proxy for target factory [org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean@103478b8]
<<<<<
```