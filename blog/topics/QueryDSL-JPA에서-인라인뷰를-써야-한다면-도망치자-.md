# QueryDSL-JPA에서 인라인뷰를 써야 한다면 도망치자 


Query DSL JPA을 사용도중 select, where에 Sub Query는 자주 사용하였지만

from 절에 Sub Query를 작성하려다가 문제를 만나게 되었다.

![](3cac8d69-6e7d-4181-b9ff-a2e0f13f267c.png)

어디서 찍은거였더라 이거..

QueryDSL로 from절 SubQuery, 즉 inline view를 작성하려 하니 지원하지 않는다는 말을 듣게 되었고

이게 Query DSL 이 제공 안 하는 건지, 하이버네이트가 지원 안 하는 건지 알아보니

[예전 하이버네이트 이슈 중 inline view 관련 이슈](https://hibernate.atlassian.net/browse/HHH-3356)를 확인해 볼 수 있었다.

![](3ec22050-d02e-481a-a721-cb55ae248605.png)

But!

[Hibernate 6.1 버전에서부터는 지원](https://in.relation.to/2022/06/14/orm-61-final/)을 시작하였다.

![](175873c1-7ca3-48a6-8014-aec638b15448.png)

그렇지만..

사용 중인 프로젝트의 Hibernate 버전은 5 버전이었다.

![](2ab4ca86-a7c5-46fb-aa42-663c8812f4c5.png)

즉 이 프로젝트에서는 Hibernate로 인라인뷰를 사용할 수 없었고

즉 이 문제를 해결할 수는 없었고 회피를 했어야 했다.

문제 회피 시도
--------

문제 회피 방법으로 생각한 방법은 여러 가지였다.

1.  from 절 Sub Query를 where로 변경하는 방법
2.  from 절 Sub Query를 그냥 일반 join으로 처리하고 애플리케이션 레벨에서 가공하는 방법 
3.  네이티브 쿼리 작성
4.  MyBartis로 작성
5.  Hibernate 버전 변경
6.  Query를 분리해서 사용

현 상황에서는 1,2번을 사용할 수 없는 상황이었기에

3번 "네이티브 쿼리 작성"부터 검토해 보면

    public class NativeRepositoryImpl {
    
      private final EntityManager em;
    
      public List<Object> findSettlement() {
        String jpql = ""
            + "select  a.* from history a "
            + "INNER JOIN "
            + "( select key, MAX(create_datetime) as last_create_datetime from detail_history"
            + " GROUP by key ) b"
            + " on a.key = b.key and a.create_datetime = b.last_create_datetime";
    
    
        Query nativeQuery = em.createNativeQuery(jpql);
        List<Object> resultList = nativeQuery.getResultList();
    
        return list;
      }
    
    }

여기서도 문제가 발생하게 되는데

네이티브 쿼리의 결과를 DTO로 받는 방법이 너무 불편하다.

DTO를 받는 방법 또한 여러 가지 방법이 있었고 각자 다른 이유로 결국 포기하게 되었다.

*   인터페이스로 받기 (가장 좋아 보이는 방법이 이였으나 단순 DTO 로만 사용가능)
*   SqlResultSetMapping 어노테이션으로 받기 (도메인 클래스로 매핑이 가능했지만 코드가 지저분해짐)
*   qlrm 라이브러리를 사용하기 (도메인 클래스로 매핑이 가능했지만 라이브러리를 써야 깔끔해진다는 것 에서 이상함)

그리고 위의 DTO를 잘 받아 온다고 해도 네이티브 쿼리 사용 및 MyBartis의 경우

내가 칼럼명에 오타를 내거나 띄어쓰기를 안 했을 때

빌드타임에서 확인이 안 되고 런타임에서 문제가 생기는 것이 싫었기에

원하는 방식이 아니었다.

또한 5번 "Hibernate 버전 변경"으로 버전을 올려서 인라인뷰 기능을 제공받더라도

QueryDSL-JPA로 처리를 했어야 했는데

[QueryDSL-JPA에서는 아직 인라인뷰를 지원하지 않는다.](https://github.com/querydsl/querydsl/issues/3438)

![](97597736-c39d-4994-88e0-11614bf8d98f.png)

다양한 문제 회피방법이 있었고 각자의 장점과 단점이 있었다. 

그중

*   from 절 Sub Query를 where로 변경하는 방법
*   from 절 Sub Query를 그냥 일반 join으로 처리하고 애플리케이션 레벨에서 가공하는 방법

위의 두 방식이 가장 좋아 보이며

이 방식을 사용할 수 없는 경우 쿼리를 분리하는 방식이 다음으로 좋다고 느꼈다

하지만 역시 문제에 대해 다른 방법으로 해결하게 되면 결국 해결이 아니다.

문제를 회피는 방법을 다 사용할 수 없다면 다시 원점이다.

문제 해결 시도
--------

![](dcfa93c9-c0ac-4f37-a6e4-b11fcb40ad40.png)

저 인라인뷰가 하고싶어요

그럼 from 절에서 SubQuery를 가능하게 할 수 있는 방법을 찾아야 한다.

*   QueryDSL-SQL
*   JPA SQL Query
*   Blaze-Persistence
*   JOOQ

#### QueryDSL-SQL

먼저 프로젝트에서 데이터베이스에 접근하는 가장 메인은 QueryDSL-JPA였기에

같은 프로젝트인 QueryDSL-SQL로 처리를 하려 했다.

하지만 QueryDSL-SQL 이 Q클래스를 생성하기 위해 데이터베이스에 접근하여 테이블을 스캔하는 과정을 해야 하는데

이미 QueryDSL-JPA를 사용하면서 Q클래스를 가지고 있는데 다시 Q클래스를 생성해서 관리를 하기가 부담스러웠고

    {
      implementation "com.querydsl:querydsl-sql:${queryDslVersion}"
      implementation "com.querydsl:querydsl-sql-spring:${queryDslVersion}"
      implementation "com.querydsl:querydsl-sql-codegen:${queryDslVersion}"
    }
    
    @Bean
    public SQLQueryFactory sqlQueryFactory(DataSource dataSource) {
      MySQLTemplates sqlTemplates = new MySQLTemplates();
      com.querydsl.sql.Configuration configuration = new com.querydsl.sql.Configuration(sqlTemplates);
      return new SQLQueryFactory(configuration, dataSource);
    }
    
    public class QueryDSLSQLTest{
      
      private final SqlQueryFactory<?> sqlQueryFactory
      
      public test(){
      
        Qhistory history = Qhistory.history
        sqlQueryFactory
            .select(history)
            .from(history)
      }
    }

혹시..? 하는 마음에 간절한 기도를 하며 기존에 사용하던 Q클래스로 사용해보려 했지만

    select history from history

문자열 그대로 쿼리가 생성되어서 사용할 수가 없었다. 

#### JPASQLQuery

기존의 JPAQuery는 작성한 질의를 JPQL로 변환하고 다시 JPQL을 데이터베이스의 네이티브 쿼리로 변환한다면

JPASQLQuery는 질의를 바로 데이터베이스의 네이티브 쿼리로 변환하기에 

기존의 사용 중이던 Q클래스를 그대로 사용하면서 네이티브쿼리를 함께 사용할 수 있다.

    @Bean
    public JPASQLQuery<?> jpaSqlQuery() {
      SQLTemplates templates = MySQLTemplates.builder().build();
      return new JPASQLQuery<>(entityManager, templates);
    }
    
    public class JPASQLQueryTest{
    
      private final JPASQLQuery<?> jpaSqlQuery;
    
      public void test(){
         
        StringPath subQueryAlias = Expressions.stringPath("sub_query_alias");
          
        Qhistory history = Qhistory.history;
        QdetailHistory detailHistory = QdetailHistory.detailHistory;
        
        jpaSqlQuery
            .select(history)
            .from(history)
            .innerJoin(JPAExpressions.select(
                        detailHistory.key.as("key"),
                        detailHistory.createDateTime.max().as("lastCreateDatetime"))
                    .from(detailHistory)
                    .groupBy(detailHistory.key)
                , subQueryAlias
            )
            .on(history.key.eq(Expressions.stringPath(subQueryAlias, "key"))
                .and(history.createDateTime.eq(
                    Expressions.datePath(LocalDateTime.class, subQueryAlias, "lastCreateDatetime")))
            )
            .fetch();
       }
    }

기존의 Q클래스를 그대로 사용할 수 있었고

그토록 원하던 inlineView를 사용할 수 있었기에 굉장히 만족했지만

select의 결과를 DTO로 가져오는 과정에서 문제가 발생했다.

테이블에서 데이터에서 시간 데이터를 DTO의 LocalDataTime 변수에 넣으려고 하니

> argument type mismatch

Type 가 맞지 않다는 에러가 발생하는 것.

이와 관련한 이슈들이 존재했고 [이슈1,](https://github.com/querydsl/querydsl/issues/3425) [이슈2](https://github.com/querydsl/querydsl/issues/2382)

![](8799f956-bad6-4dc1-b51e-7c5d263f00fb.png)

그 결과는 JPASQLQuery를 사용하는 것을 권장하지 않는다.(아니 해결 방법을 알려줘)

### Blaze-Persistence

JPA의 확장 라이브러리이며 QueryDSL-JPA의 확장 기능을 지원한다.

따라서 이 라이브러리는 JPA 혹은 QueryDSL-JPA에서 지원을 안 하는 기능을 지원하는데

그중 하나가 인라인뷰이다.

    implementation 'com.blazebit:blaze-persistence-integration-querydsl-expressions:1.5.1'
    implementation 'com.blazebit:blaze-persistence-integration-hibernate-5.6:1.6.5'
    implementation 'com.blazebit:blaze-persistence-core-impl:1.5.1'
    implementation 'com.blazebit:blaze-persistence-core-api:1.5.1'

[가이드가 알려주는 대로](https://persistence.blazebit.com/documentation/1.5/core/manual/en_US/#getting-started-setup) 사용 중인 하이버네이트 버전에 맞춰 환경을 세팅하고

    @Configuration
    public class BlazePersistenceConfiguration {
    
      @PersistenceContext
      private EntityManager entityManager;
    
      @Bean
      @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
      @Lazy(false)
      public CriteriaBuilderFactory createCriteriaBuilderFactory() {
        CriteriaBuilderConfiguration config = Criteria.getDefault();
        return config.createCriteriaBuilderFactory(entityManager.getEntityManagerFactory());
      }
    }

BlazeJPAQuery을 사용하여 처리를 한다.

    QDetailHistory detailHistory = new QDetailHistory("detailHistory");
    QDetailHistory lastDetailHistory = new QDetailHistory("lastDetailHistory");
    
    new BlazeJPAQuery<>(entityManager,blazeQueryFactory)
        .select(lastDetailHistory)
        .from(detailHistory)
        .leftJoin(JPAExpressions.select(lastDetailHistory)
                .from(lastDetailHistory)
                .groupBy(lastDetailHistory.key)
            , lastDetailHistory)
        .on(detailHistory.key.eq(lastDetailHistory.key).and(detailHistory.createDateTime.eq(lastDetailHistory.createDateTime.max())))
        .fetch();

QueryDSL의 QClass를 그대로 사용이 가능했고

기존의 문법을 그대로 작성가능했으며 원하는 서브쿼리가 가능하였지만

from 절에 서브쿼리가 하나 들어간 네이티브 쿼리가 생성될 거라는 예상과는 다르게

실제 생성된 네이티브 쿼리는 from 절 안의 서브쿼리 안에 union 절 안에 또 서브쿼리가 들어가는

너무 다른 쿼리가 발생하였고

예측이 안 되는 변환이라 사용할 수 없었다.

### JOOQ

JOOQ(Java Object oriented querying)는 java object로 쿼리를 작성하게 해주는 도구로

QueryDSL-SQL처럼 테이블 스캔하여 편하게 질의 작성을 도와주지만

테이블 스캔을 하지 않고도 사용은 가능하다

    implementation 'org.springframework.boot:spring-boot-starter-jooq'

테이블 스캔을 하지 않을 것이기에 JOOQ 라이브러리만 받아온다

    public class jooqTest{
    
      private final DSLContext dsl;	
        
      public void test(){
    
        dsl.select()
            .from(table("history"))
            .innerJoin(
                dsl.select(
                        field("key").as("key"),
                        max(field("create_datetime")).as("last_create_datetime"))
                    .from(table("detail_history"))
                    .groupBy(field("key"))
                    .asTable("b")
            )
            .on(field("history.key").eq(field("b.key")))
              .and(field("history.create_datetime").eq(field("b.last_create_datetime"))
              );
        }
    }

완전한 DSL처럼 작성은 할 수 없었지만

원하던 From 절의 SubQuery를 사용할 수 있었고

JOOQ의 지원을 약간 포기하는 대신 테이블 스캔을 따로 하지 않을 수 있었고

MyBartis 보다는 IDE에 도움을 받으면서 작성이 가능해졌다.

각 장단점을 비교하니 각자 trade off 가 있었으며

이러한 문제가 발생하면 가능한 회피하고 만약 회피 할 수 없다면 

최대한 관리 포인트를 줄일 수 있는 JOOQ를 선택하게 되었다.

참고 및 출처

*   [https://persistence.blazebit.com/documentation/1.5/core/manual/en\_US/#getting-started-setup](https://persistence.blazebit.com/documentation/1.5/core/manual/en_US/#getting-started-setup)
*   [https://github.com/querydsl/querydsl/issues/3425](https://github.com/querydsl/querydsl/issues/3425)
*   [https://github.com/querydsl/querydsl/issues/2382](https://github.com/querydsl/querydsl/issues/2382)
*   [https://github.com/querydsl/querydsl/issues/3438](https://github.com/querydsl/querydsl/issues/3438)
*   [https://in.relation.to/2022/06/14/orm-61-final/](https://in.relation.to/2022/06/14/orm-61-final/)
*   [https://hibernate.atlassian.net/browse/HHH-3356](https://hibernate.atlassian.net/browse/HHH-3356)