# db column명은 snake_case 가 최고다 


데이터베이스 Column의 네이밍을 snake\_case로 자주 사용하고 있었는데

한번 camelCase로 하면 어떤가 하는 중에 발생한 일이었다.

![](893a0246-c26c-4828-9681-47080828e60f.png)

아래와 같은 Table를 생성해 두고

JPA Entity를 만들어서 조회 쿼리를 호출해 보면

    @Table(name = "test_entity")
    @Entity
    public class TestEntity {
    
      @Id @Column(name = "id")
      private Long id;
    
      @Column(name = "phone_number")
      private String phoneNumber;
    }

    select
        testentity0_.id as id1_0_,
        testentity0_.phone_number as phone_nu2_0_ 
    from
        test_entity testentity0_

예상 가능한 결과를 얻을 수 있다.

하지만 table column의 이름을 camelCase 형식으로 변경하고 같은 행위를 해보면 이상한 결과를 보게 되는데

![](0eb08dac-599c-4b93-ae79-c98f1e3cf373.png)

    @Table(name = "test_entity")
    @Entity
    public class TestEntity {
    
      @Id @Column(name = "id")
      private Long id;
    
      @Column(name = "phoneNumber")
      private String phoneNumber;
    }

![](3c51f189-3459-4058-beb3-ac992077d129.png)

    select
        testentity0_.id as id1_0_,
        testentity0_.phone_number as phone_nu2_0_ 
    from
        test_entity testentity0_
        
    Error: 1054-42S22: Unknown column 'testentity0_.phone_number' in 'field list'

Column의 이름을 camelCase로 변경하고

@Column의 name 속성에도 camelCase로 지정했지만

실제 쿼리가 날아간 것은 snake\_case 방식이다.

그러니 해당 Column을 못 찾는 문제가 발생했다.

![](207056aa-f1d3-4ec1-8ba6-b61d1da5be3d.png)

관련 이슈를 찾아보았더니 예전에 꽤나 뜨거운 감자였던 것 같다

[https://github.com/spring-projects/spring-boot/issues/2129](https://github.com/spring-projects/spring-boot/issues/2129)

 [@Column with name attribute not working property on entities · Issue #2129 · spring-projects/spring-boot

Hi, I'm using spring-boot-starter-parent and spring-boot-starter-data-jpa (version 1.1.9). I found some strange behaviour on entities. When I specify @column(name = "myName") then jpa is generating...

github.com](https://github.com/spring-projects/spring-boot/issues/2129)

![](5e4a5bb3-e42c-4b42-90e0-7a73b5dea3e4.png)

예상 못했다!

![](5ad0f2f7-1185-4de8-bee2-b1b3f11e908a.png)

미친 짓이다!

뭐 사실 지금 사용 중인 데이터베이스는 Column의 이름의 대소문자를 구분하지 않기 때문에

    @Table(name = "test_entity")
    @Entity
    public class TestEntity {
    
      @Id
      @Column(name = "id")
      private Long id;
    
      @Column(name = "phonenumber")
      private String phoneNumber;
    
    }

이렇게 하면 동작하긴 하지만

Column네이밍을 camelCase로 했을 때는 Column명 그대로 @Column name 속성으로 사용할 수 없다는 것이다.

### Hibernate의 이름 전략

위와 같은 동작에는 Hibernate의 명명 전략으로 인해 발생한 것이다

명명 전략에는 암시적(+명시적) 전략과 물리적 전략이 존재한다

[https://docs.jboss.org/hibernate/orm/5.5/userguide/html\_single/Hibernate\_User\_Guide.html#naming](https://docs.jboss.org/hibernate/orm/5.5/userguide/html_single/Hibernate_User_Guide.html#naming)

 [Hibernate ORM 5.5.9.Final User Guide

Fetching, essentially, is the process of grabbing data from the database and making it available to the application. Tuning how an application does fetching is one of the biggest factors in determining how an application will perform. Fetching too much dat

docs.jboss.org](https://docs.jboss.org/hibernate/orm/5.5/userguide/html_single/Hibernate_User_Guide.html#naming)

간단하게 보면

하이버네이트가 java class의 속성과 SQL Table , Column을 매핑하기 위해 논리적 이름을 결정한다.

이때 논리적 이름을 결정하기 위해서는

@Table, @Column 어노테이션을 사용한 명시적 명명 전략(Explicit naming strategy)과

@Table, @Column 어노테이션을 사용하지 않을 때의 암시적 명명 전략(Implicit naming strategy)을 사용한다

이렇게 결정된 논리적 이름을 SQL Table 혹은 Column에 매핑하기 위해 물리적 이름(Physical naming strategy) 명명 전략을 사용하게 되는데

기본 설정이 물리적 이름은 논리적 이름과 동일하게 된다.

Spring Boot는

물리적 명명 전략(Physical naming strategy)을 org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy 로 되어있고

암시적 명명 전략(Implicit naming strategy)을

org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy 로 되어있는데

이 전략의 특징이 아래와 같다

*   테이블 이름은 소문자
*   camelCase를 snake\_case로 변경

이 특징으로 인해 위와 같은 문제가 발생한 것.

다시 entity의 일부를 보았을 때

    @Column(name = "phoneNumber")
    private String phoneNumber;

속성 이름 phoneNumber의 논리적 이름은 phoneNumber이지만

논리적 이름 phoneNumber의 물리적 이름이 물리적 명명 전략에 의해 phone\_number 이 된 것이다

물리적 명명 전략을 변경하면 원하는 결과를 얻을 수 있다.

      jpa:
        hibernate:
          naming:
            physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

    select 
        testentity0_.id as id1_0_,
        testentity0_.phoneNumber as phonenum2_0_ 
    from 
    	test_entity testentity0_

![](a08586bb-d576-4921-a882-574880897f79.png)

결국 Column명을 대소문자 구분해서 사용하지 말자

굳이 사용해야겠다면 그래도 사용하지 말자!

참고

[https://www.baeldung.com/hibernate-naming-strategy](https://www.baeldung.com/hibernate-naming-strategy)

[https://docs.jboss.org/hibernate/orm/5.5/userguide/html\_single/Hibernate\_User\_Guide.html#naming](https://docs.jboss.org/hibernate/orm/5.5/userguide/html_single/Hibernate_User_Guide.html#naming)

[https://stackoverflow.com/questions/25283198/spring-boot-jpa-column-name-annotation-ignored](https://stackoverflow.com/questions/25283198/spring-boot-jpa-column-name-annotation-ignored)