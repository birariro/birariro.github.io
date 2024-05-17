# QueryDSL에서 SubQuery 에는 limit 를 사용 할 수 없다. 


SubQuery 종류 중

[from 절에 사용하는 인라인뷰는 지원하지 않으니](https://j-k4keye.tistory.com/68) 당연히 limit를 사용할 수 없고

select의 Scalar SubQuery 나 where 절 SubQuery에 limit 사용이 안된다는 것을 알게 되었다.

Scalar SubQuery에서 limit 분실
--------------------------

    select(
        new QModel(
            orders.orderId,
            JPAExpressions.selectDistinct(orderHistory.message)
                .from(orderHistory)
                .where(orderHistory.orderId.eq(orders.orderId))
                .orderBy(orderHistory.createDateTime.asc())
                .limit(1)
        )
    )
    .from(orders)
    .limit(20)
    .fetch();

위처럼 

QueryDSL에서 select 절에 sub query를 넣는 스칼라 서브쿼리 사용 시

서브쿼리 안에 limit 절이 사라지는 문제가 발생했다.

    select
        tb_order.order_id,
        (select
            distinct tb_order_history.message          
        from
            tb_order_history          
        where
            tb_order_history.order_id=tb_order.order_id          
        order by
            tb_order_history.create_datetime asc)     
    from
        tb_order limit 20

스칼라 서브 쿼리 안에 limit 절이 사라졌으며

스칼라 서브쿼리는 한 번에 한 가지만을 처리하는 특징으로 인해 결과가 하나의 행이어야 하는데

limit 절이 사라져 여러 행이 나올 수 있게 되어 쿼리 에러가 발생할 수 있다.

이문제에 대한 이슈가 있다.

[https://github.com/querydsl/querydsl/issues/3224](https://github.com/querydsl/querydsl/issues/3224)

 [How to use limit() and offset() in subquery? · Issue #3224 · querydsl/querydsl

I want to use limit() in a subquery. limit() seems to work on the main query, but limit() doesn't seem to work on the subquery. offset() is also the same. I guess it's because limit() and offset() ...

github.com](https://github.com/querydsl/querydsl/issues/3224)

QueryDSL 쿼리 변환 과정
-----------------

밖에 있는 Limit는 정상적인 동작을 하고

스칼라 서브쿼리의 limit 가 사라지는 이유는

QueryDSL로 사용한 쿼리 오브젝트가 어떻게 쿼리로 변환되어 데이터베이스로 요청을 하는지 보면 알 수 있다.

QueryDSL 은 

함수를 사용하여 쿼리를 작성할 때 파라미터로 넘긴 값들을 Metadata안에 넣어두고

네이티브 쿼리로 변환 할때 Metadata에서 꺼내어 변환한다.

그중 Limit와 offset는 QueryModifiers 객체로 관리되고 있다

    @Override
    public void setLimit(Long limit) {
        if (modifiers == null || modifiers.getOffset() == null) {
            modifiers = QueryModifiers.limit(limit);
        } else {
            modifiers = new QueryModifiers(limit, modifiers.getOffset());
        }
    }

Metadata에서 필요한 값을 꺼내서 쿼리를 변환 하는 과정은 createQuery메서드에서 확인이 가능하며

select, from, where, group by, having, order by 절을 완성시키는 과정을 처리하는 serialize 메서드를 호출하며 동작한다.

> 💡 중요한 건 이 과정에서 select 절이 완성된다는 것이다.

JPA 가 처리 가능한 쿼리 변환 과정은 createQuery에서 처리하게 되는데

Limit와 Offset  은 처리 하지 않고 Query객체에서 따로 담아 실행 프로세스로 이동한다.

![](01ea1849-6136-4c19-aaaf-77e3b9b1c0d2.png)

![](5664c29a-0cb8-4619-8622-53bfe766fd9a.png)

만들어진 query string 에 limit 가 빠져있고 queryOptions에 limit 절에 넣은 20이 있다

이렇게 만들어진 Query객체는 실행 프로세스에 진입하게 되며

실행 직전에 Limit 작업에 들어간다.

    private List doQuery(
            final SharedSessionContractImplementor session,
            final QueryParameters queryParameters,
            final boolean returnProxies,
            final ResultTransformer forcedResultTransformer) throws SQLException, HibernateException {
    
        final RowSelection selection = queryParameters.getRowSelection();
        final int maxRows = LimitHelper.hasMaxRows( selection ) ?
                selection.getMaxRows() :
                Integer.MAX_VALUE;
    
        final List<AfterLoadAction> afterLoadActions = new ArrayList<AfterLoadAction>();
    
        final SqlStatementWrapper wrapper = executeQueryStatement( queryParameters, false, afterLoadActions, session );
        final ResultSet rs = wrapper.getResultSet();
        final Statement st = wrapper.getStatement();

![](f38941cc-a692-4b15-a8eb-0c803d58c4c1.png)

Limit로 넣어두었던 값을 꺼내서

각 데이터베이스의 Dialect 객체에게 limit 작업을 위임하게 되는데

    public class MySQLDialect extends Dialect {
    
      private static final LimitHandler LIMIT_HANDLER = new AbstractLimitHandler() {
        @Override
        public String processSql(String sql, RowSelection selection) {
          final boolean hasOffset = LimitHelper.hasFirstRow( selection );
          return sql + (hasOffset ? " limit ?, ?" : " limit ?");
        }
      };
    }

MySQL Dialect는 지금까지 만들어진 SQL Query에 "limit" 문자열을 더하게 된다.

![](cc8016cf-3463-4481-992a-ce89e06696e8.png)

이걸 보면 알 수 있는 것은

QueryDSL로 쿼리를 작성하게 되면

공통쿼리 변환(select, from, where, group by, having, order by) -> 방언 변환(limit, offset) -> 실행의 순서로 진행하게 된다.

스칼라 서브쿼리는 select 구문 이기 때문에 공통 쿼리 변환 작업에서 끝이 나게 되고

공통 쿼리 변환 작업은 limit 절을 작업하지 않기 때문에 limit 이 없는 상태로 작업이 끝나게 되는 것이다.

하지만 밖에 붙은 limit 절은 공통 쿼리 변환 과정에서 생략이 되고 방언 변환에서 추가가 되기 때문에 정상적인 처리가 가능했다.

Where 절 SubQuery에서 limit
------------------------

위에서 본 것처럼 QueryDSL의 변환 과정을 보면 이 문제는 Scalar SubQuery에서만 발생하는 것이 아닌

Where 절 SubQuery에서도 발생하는 문제라는 것을 알 수 있다.

    select(new QTestModel( order.orderId, orderHistory.message))
    .from(order)
    .innerJoin(orderHistory).on(order.orderId.eq(orderHistory.orderId))
    .where(
        orderHistory.createDateTime.eq(
            JPAExpressions.selectDistinct(orderHistory.createDateTime)
                .from(orderHistory)
                .orderBy(orderHistory.createDateTime.desc())
                .limit(1)
        )
    )
    .limit(20)
    .fetch();

    select
        tb_order.order_id,
        tb_order_history.message      
    from
        tb_order      
    inner join
        tb_order_history 
        on (tb_order.order_id=tb_order_history.order_id)      
    where
        orderhisto1_.create_datetime=(
            select
                distinct tb_order_history.create_datetime              
            from
                tb_order_history tb_order_history              
            order by
                tb_order_history.create_datetime desc         
        ) limit 20

그럼 어떻게 할까?
----------

limit 가 사용이 불가능하면 쿼리를 변경하여 같은 결과물이 나오게 변경하면 된다.

    query.select(
      new QModel(
        orders.orderId,
        JPAExpressions.selectDistinct(orderHistory.message)
          .from(orderHistory)
          .where(
            orderHistory.orderId.eq(orders.orderId),
            orderHistory.createDateTime.eq(
              JPAExpressions.select(orderHistory.createDateTime.min())
                  .from(orderHistory)
                  .where(orderHistory.orderId.eq(orders.orderId)
                  )
            )
          )
    )
    .from(orders)
    .limit(20)
    .fetch();

스칼라 서브쿼리를 조인을 통해 처리하는 방법도 있을 것이고

    query.select(
        new QModel(orders.orderId, orderHistory.message)
    )
    .from(orders)
    .leftJoin(orderHistory).on(orderHistory.orderId.eq(orders.orderId))
    .where(
      orderHistory.createDateTime.eq(
        JPAExpressions.select(orderHistory.createDateTime.min())
            .from(orderHistory)
            .where(orderHistory.orderId.eq(orders.orderId)
            )
     )
    .limit(20)
    .fetch();

조인하고 where로 필터 하는 방법을 사용해도 된다.