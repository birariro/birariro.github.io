# Mysql(Maraidb) 실행 계획 살펴보기 


TL;DR
-----

*   쿼리문 앞에 explain을 붙여 실행하는 것으로 실행 계획을 볼 수 있다.
*   key 항목에 있는 값이 동작한 인덱스이다
*   type 항목의 있는 값은  const(좋음) ->  eq-ref -> ref -> range -> index ->  all(안 좋음) 순이다.
*   extra 항목의 값은 동작 방식을 알려주며 인덱스만을 사용했는지, 디스크까지 접근했는지 등을 알려준다.

![](e67c50df-997d-485d-a46c-7898039f4107.png)

https://dev.mysql.com/doc/refman/8.0/en/pluggable-storage-overview.html

MySQL(Mariadb) 서버는 클라이언트로부터 오는 요청을 처리하기 위해

Parser, Optimizer, Chches 등 수행하는 DB(MySQL, Mariadb) Engines과

실제 데이터를 디스크에 저장하고 조회하는 Storage Engines로 구성되어 있으며

옵티마이저는 요청한 쿼리를 어떤 실행 계획으로 실행했을 때 비용이 얼마나 발생하는지를 계산하여

비용이 가장 적은 방법을 선택한다.

어떠한 쿼리는 Storage Engines에서 실제 디스크에 접근하여 데이터를 DB Engins로 반환하고

DB Engins는 다시 한번 재가공하여 반환하는 경우도 있고

또 어떠한 쿼리는 실제 디스크까지 접근하지 않고 인덱스 만을 통해 데이터를 반환하여 DISK I/O 를 발생시키지 않을 수도 있다.

따라서  DB Engines이 최소한의 일만 하게 하거나

Storage Engines 이 디스크에 접근을 하지 않게 하기 위해 인덱스를 사용하는 방식으로 최적화를 진행한다.

Explain
-------

위와 같은 이유로 현재 내가 작성한 쿼리가 얼마나 빠른 속도로 실행되는지 확인할 필요가 있다.

Explain 명령을 사용하면 MySQL 은 쿼리 실행계획 정보를 옵티마이저에게 얻어 출력한다.

> 💡 Explan 명령어로 확인한 정보는 추상 값이므로 정확하지 않을 수 있다.

![](51433d51-0d3a-42de-84da-7ca15c4a46fa.png)

### Table 

쿼리로 실행되는 테이블이 표기가 되며

조인을 통해 여러 테이블을 함께 사용할 경우 여러 행으로 각 테이블이 표기된다

### possible\_keys

옵티마이저가 최적의 실행계획을 만들기 위해 후보로 선정했던 

사용할 후보 인덱스 목록으로 이 항목이 NULL이라면 연관된 인덱스가 존재하지 않는다는 것이다.

> 💡 사용할 후보였기에 사용하지 않을 수도 있다.

### key

최종으로 실행계획에서 선택된 인덱스가 표기된다.

### key\_len

선택된 인덱스의 길이를 표기하며 길이는 작을수록 좋다

### ref

데이터 레코드를 찾기 위해 key 칼럼에 표시된 인덱스를 어떤 칼럼 또는 상수와 비교하였는지 표기한다.

### rows

쿼리를 처리하기 위해 얼마나 많은 레코드를 읽어야 하는지를 표기한다

통계값으로 계산되는 것이기에 실제 쿼리의 레코드와 일치하지 않는 경우도 많다

### Type

중요한 요소 중 하나로

각 테이블의 레코드를 어떤 접근방식으로 레코드를 읽는지가 표기된다

const, eq-ref, ref, range, index, all 순으로 뒤로 갈수록 안 좋은 값이다.

>  💡 const, eq\_ref, ref는 좋은 접근 방식이니 ref를 const로 바꾸려는 노력을 하지 않아도 괜찮다.

#### Type/const

*   PK나 Unique key를 사용했기에 조건을 만족하는 레코드가 하나인경우이다.

#### Type/eq-ref

*   여러 테이블이 조인되는 쿼리에서 표기
*   PK나 Unique key 인덱스를 사용하는 조인으로 const를 제외한 가장 좋은 형태
*   유니크한 값으로 조인하기에 최대 1개의 행만을 패치하는 것이 특징

    select * from tb_order
    left join tb_member on tb_order.member_id = tb_member.id

![](e41fd75e-d30e-44b5-ade1-385405f185cc.png)

#### Type/ref

*   인덱스의 동등(Equal) 검색에 표기
*   PK나 Unique key 인덱스를 사용하지 않는 조인에도 표기됨
*   조인에 표기 시 유니크하지 않은 값으로 조인하기에 여러 행을 패치할 가능성이 높음

#### Type/range

*   인덱스 레인지 스캔의 접근 방식
*   인덱스를 하나의 값이 아닌 범위로 검색하는 경우 발생(<, >, is null, between, in, like 등)

    EXPLAIN select * from tb_member WHERE id > 10

![](0d4d27db-fc7a-4676-a7b7-6f228b07f0b7.png)

#### index

*   인덱스를 처음부터 끝까지 탐색하는 인덱스 풀스캔이다
*   인덱스는 데이터 파일보다는 크기가 작기 때문에 풀 테이블 스캔보다는 빠르다

    select id from tb_order

![](6eb5a5cb-f864-4175-99a8-0c2990e7ed51.png)

#### Type/all

*   table를 처음부터 끝까지 탐색하는 테이블 풀 스켄이다
*   가장 비효율 적인 방법이다.

>  💡 옵티마이저가 다른 방법을 선택할 수도 있지만 다양한 이유로 풀스캔을 선택하는 경우도 있다 예로 전체 데이터중 대부분의 데이터를 한 번에 가지고 오려할 때 range를 하지 않고 all을 선택하기도 한다.

### Extra

쿼리 실행계획에서 중요한 항목 중 하나로

옵티마이저가 쿼리를 어떻게 해석하고 어떻게 동작하는지에 대한 정보이다.

#### Using index

커버링 인덱스 일 경우 표기

    select id from tb_member where id = 2

위처럼 쿼리를 사용하면  tb\_member의 PK 인 id는 Clustered Index로 구성되어 있기 때문에

쿼리의 결과를 얻기 위해 Storage Engines 이 실제 디스크까지 접근하지 않기에 DISK I/O 를 발생시키지 않고

Clustered Index 테이블 만으로 필요한 정보를 모두 얻을 수 있다.

select, where 뿐만 아니라 group by, order by 등 모든 곳에서 사용되는 필드가 인덱스로 구성되어 있는 것을

커버링 인덱스라 한다.

#### Using where

Storage Engines으로부터 받은 데이터를 DB Engines에서 다시 필터링으로 가공해야 하는 경우 발생한다

      +--------+     m row    +-----------+      n row    +----------------+           
      | client | <----------- | db engine | <-----------  | Storage engine |  
      +--------+              +-----------+               +----------------+

#### Using index condition

*   인덱스 컨디션 푸시다운(ICP, index condition push down)
*   이전에는 커버링인덱스에 부합하는 쿼리이지만 행 검색 시 인덱스를 사용하지 못하는 경우(like절을 사용하거나 멀티인덱스에 순서를 안 맞춘 경우) 조건이 Storage Engines로 넘어가지 못해  using where가 발생 했음
*   MySQL 5.6, MariaDB 5.3 버전 이후에는 최적화하여 Storage Engines으로 전달하도록 개선됨
*   인덱스 조건을 Storage Engines으로 넘겨준다 하여 인덱스 컨디션 푸시 다운이라 부름

#### Using filesort

*   order by 가 사용된 쿼리의 실행계획에서 발생한다
*   인덱스를 통해 정렬을 할 수 없는 경우 발생
*   조회덴 레코드를 정렬용 메모리 버퍼에 복사하여 퀵 정렬 또는 힙 정렬 알고리즘을 사용하여 정렬함
*   많은 부하를 일으키는 작업이기에 튜닝을 하거나 인덱스를 생성하는 것이 좋음

#### Using temporary

*   쿼리를 처리하는 동안에 중간 결과를 담아주기 위해 임시 테이블을 생성한다는 표기

#### Using join buffer

*   인덱스를 사용하여 조인을 할 수 없을 때 버퍼를 사용하여 조인했음을 표기