# MySQL_TIL_Day_03



- 목차
  - SELECT 문
    - order by 절
    - 집계 함수
    - group by 절
    - having 절
    - 트랜잭션(Transaction)
    - 조인(Join)
    - 서브 쿼리 (Subquery)





## SELECT 문



### order by 절

- 특정 열의 값을 기준으로 질의 결과 정렬
- ASC : 오름차순 (생략 가능)
- DESC : 내림차순




### 집계 함수

- sum() : 합계
- avg() : 평균
- count() : 선택된 열의 행 수 (null 값은 제외)
- count(*) : 전체 행의 수
- max() : 최대
- min() : 최소



### group by 절

- group by <속성>

- 그룹 질의를 기술할 때 사용

- 특정 열로 그룹화한 후 각 그룹에 대해 한 행씩 질의 결과 생성

- 주의
  - select 절에는 group by에서 사용한 열과 집계 함수만 나올 수 있음

  ``` mysql
  select bookNo, sum(bsQty) as "주문량 합계"
  from booksale
  group by bookNo;
  ```

  

### having 절

- having <검색 조건>
- group by 절에 의해 구성된 그룹들에 대해 적용할 조건 기술
- sum, avg, max, min, count 등의 집계 함수와 함께 사용
- 주의
  - 반드시 group by 절과 같이 사용
  - where 절 보다 뒤에 사용
  - <검색 조건>에는 sum, avg, max, min, count 등의 집계 함수만 와야 함



### union all 절 

- 2개의 쿼리를 합칠때 쓰인다



### 트랜잭션(Transaction)

- DBMS에서 데이터를 다루는 논리적인 작업의 단위
- 데이터베이스 관리 시스템에서 회복 및 병행 수행 시 처리되는 작업의 논리적 단위로 사용
- 하나의 트랜잭션은 정상적으로 종료될 경우 COMMIT 연산이 수행되고 비정상적으로 종료될 경우 ROLLBACK 연산 수행해서 작업을 취소



- #### commit 연산

  - 트랜잭션 처리가 정상적으로 종료되어 트랜잭션이 수행한 변경 내용을 
    데이터베이스에 반영하는 연산

  - 내용을 변경한 트랜잭션이 완료되면 그 트랜잭션에 의해 데이터베이스는 
    새롭게 일관된 상태로 변경되고 이 상태는 시스템에 오류가 발생되더라도 취소되지 않음



- #### rollback 연산

  - 하나의 트랜잭션 처리가 비정상으로 종료되어 데이터베이스의 일관성이 깨졌을 때
    트랜잭션이 행한 모든 변경 작업을 취소하고 이전 상태로 되돌리는 연산



### 조인(Join)

- 여러 개의 테이블을 결합하여 조건에 맞는 행 검색

- 조인의 종류

  

  - #### inner join (내부 조인) - 가장 많이 사용

    - 공통되는 열이 있을 때

    - 공통 속성의 속성값이 동일한 튜플만 반환

    - 조인 기본 형식

      ```mysql
      select 열리스트 
      from 테이블명1 
      inner join 테이블명2 on 조인조건(기본키 = 외래키);
      ```

      

    - 다양한 조인 문 표기 방식 (1)

      - where 조건 사용
      - 양쪽 테이블에 공통되는 열 이름 앞에 테이블명 표기 (모호성을 없애기 위해서)
      - 테이블명 없으면 오류 발생

      ```mysql
      select client.clientNo, clientName, bsQty 
      from client, booksale 
      where client.clientNo = booksale.clientNo;
      ```

      - 2개의 테이블을 조인할 경우 : 제약조건은 1개
      - 3개의 테이블을 조인할 경우 : 제약조건은 2개

      

    - 다양한 조인 문 표기 방식 (2)

      - 양쪽 테이블에 공통되지는 않지만 모든 열 이름에 테이블명 붙임
      - 양쪽 테이블에 공통되지는 않지만 모든 열 이름에 테이블명 붙임
      - 서버에게 찾고자 하는 열의 정확한 위치를 알려주므로 성능이 향상

      ```mysql
      select client.clientNo, client.clientName, booksale.bsQty 
      from client, booksale 
      where client.clientNo = booksale.clientNo;
      ```

      

    - 다양한 조인 문 표기 방식 (3)

      - 테이블명 대신 별칭(Alias)을 사용

      ```mysql
      select C.clientNo, C.clientName, BS.bsQty
      from client C, booksale BS 
      where C.clientNo = BS.clientNo;
      ```

      

    - 다양한 조인 문 표기 방식 (4)

      - join on 명시

      ```mysql
      select C.clientNo, C.clientName, BS.bsQty 
      from booksale BS 
      join client C on C.clientNo = BS.clientNo;
      ```

      

    - 다양한 조인 문 표기 방식 (5)

      - inner join on 명시 : 가장 많이 사용되는 방법으로 우리는 이 방법 사용

      ```mysql
      select C.clientNo, C.clientName, BS.bsQty 
      from booksale BS 
      inner join client C on C.clientNo = BS.clientNo;
      ```

    - 3개 테이블 결합

      - 조인 조건 2개

      ```mysql
      select C.clientName, B.bookName 
      from booksale BS 
      inner join client C on C.clientNo = BS.clientNo
      inner join book B on B.bookNo = BS.bookNo;
      ```

  

  - #### outter join (외부 조인) 

    - 공통되는 열이 없을 때

    - 공통된 속성을 매개로 하는 정보가 없더라도 버리지 않고 연산의 결과를 
      릴레이션에 표시

    - 값이 없는 대응 속성에는 null 값을 채워서 반환

      

    - 좌측 외부 조인 (Left Outer join)

      - 좌측 릴레이션의 정보 유지
      - 왼쪽 (Left) 테이블 기준
      - 왼쪽 테이블 (client) 데이터 모두 출력
      - client에는 있지만 오른쪽 테이블(booksale)에 없는 값은 null로 출력
      - 의미 : 고객 중에서 한 번도 구매한 적이 없는 고객은 null로 출력

      ```mysql
      select * 
      from client C
      	left outer join booksale BS
          on C.clientNo = BS.clientNo
      order by C.clientNo;
      ```

      

    - 우측 외부 조인 (Right Outer join)

      - 우측 릴레이션의 정보 유지
      - 오른쪽 (right) 테이블 기준
      - 오른쪽 테이블(booksale) 데이터 모두 출력
      - 왼쪽 테이블에서는  출력되는 고객의 의미 : 한번이라도 주문한 적이 있는 고객

      ```mysql
      select *
      from client C
      	right outer join booksale BS
      	on C.clientNo = BS.clientNo
      order by C.clientNo;
      ```

      

    - 완전 외부 조인 (Full Outer join)

      - 두 릴레이션의 정보 유지
      - 완전 외부 조인 (full outer join)
      - mysql에서는 full outer join이 지원하지 않음
      - left join과 right join을 union해서 사용

      ```mysql
      select *
      from client C
      	left join booksale BS
      on C.clientNo = BS.clientNo
      
      union
      
      select *
      from client C
      	right join booksale BS
      on C.clientNo = BS.clientNo;
      ```

    

### 서브 쿼리 (Subquery)

- 하위 질의 또는 부속 질의라고도 함
- 하나의 SQL 문 안에 다른 SQL문이 중첩
- 쿼리를 1차 수행한 다음, 반환값(결과)을 다음 쿼리에 포함시켜서 사용
- 다른 테이블에서 가져온 데이터로 현재 테이블에 있는 정보를 찾거나 가공할 때 사용



- #### 서브 쿼리의 구성 및 형식

  - 메인 쿼리와 서브 쿼리로 구성
  
    
  
  - 단일 행 서브 쿼리
    
    - 서브 쿼리 결과 값이 단일 행
    
    - = 연산자 사용
    
      ```mysql
      -- 고객 손흥민의 주문 수량 조회
      -- 1. client 테이블에서 고객명 '손흥민'의 clientNo를 찾아서
      -- 2. booksale 테이블에서 이 clientNo에 해당되는 주문에 대해
      -- 3. 주문일, 주문 수량 출력
      
      select bsDate, bsQty
      from booksale
      where clientNo = (select clientNo
      				  from client
      				  where clientName = '손흥민'); 
      -- 단일 행일지 다중 행일지 헷갈리다면 '='보다 'in'을 쓰는게 더 낫다                  
      ```
    
      
    
  - 다중 행 서브 쿼리
    
    - 서브 쿼리 결과 값이 여러 행
    - in, any, all, exists 연산자 사용
    
    
    
  - 서브 쿼리 연산자
    
    - where 절에서 사용
    
    - 데이터를 선택하는 조건 또는 술어와 같이 사용
    
      | 연산 |             연산자             | 반환 행 |
      | :--: | :----------------------------: | :-----: |
      | 비교 |      =, >, <, >=, <=, !=       |  단일   |
      | 집합 |           in, not in           |  다중   |
      | 존재 |       exists, not exists       |  다중   |
      | 한정 | all(모두), any (최소 하나라도) |  다중   |
    
      



### 조인 vs 서브 쿼리

- 조인
  - 여러 테이블의 데이터를 모두 합쳐서 연산
  
  - 카티전곱(15 x 3 = 45 행 반환) 후 조건에 맞는 행 검색
    - 데이터가 15행이 있는 테이블과 3행이 있는 테이블의 카티전곱 결과는 45행
    
  - 카티전곱 연산 + select 연산
  
    
  
- 서브 쿼리
  - 필요한 데이터만 찾아서 제공
  - 15행을 검색한 결과를 가지고 3행에서 검색
    - 총 18행 검색
  - 경우에 따라 조인보다 성능이 더 좋을 수도 있지만 대량의 데이터에서 서브쿼리를 
    수행할 때 성능이 더 나쁠 수 있다.

