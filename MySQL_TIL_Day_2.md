# MySQL_TIL_Day_2



- 목차
  - SQL 문의 기능별 분류
    - 데이터 정의어 (DDL)
      - ALTER 문
      - Drop 문
    - 데이터 조작어 (DML)
      - INSERT 문
      - UPDATE 문
      - DELETE 문
      - SELECT 문





## SQL 문의 기능별 분류



### 데이터 정의어 (DDL)

- #### ALTER 문

  - 테이블에 대한 정의 변경

  - 새로운 열 추가, 특정 열의 디폴트 값 변경, 특정 열 삭제 등 수행

    ```mysql
     alter table 테이블명 키워드 (바꿀내용);
      
      add -- 열 추가
      RENAME COLUMN -- 열의 이름 변경
      MODIFY -- 열의 데이터 형식 변경
      CHANGE -- 열의 이름과 데이터 형식 변경
      DROP -- 여러 개의 열 삭제
      DROP COLUMN -- 열 삭제
      DROP PRIMARY KEY -- 기본키 삭제
      DROP CONSTRAINT -- 제약조건 삭제
    ```

  

- #### Drop 문

  - 테이블 구조와 데이터 모두 삭제

  - 데이터 손실이 발생하므로 거의 사용하지 않는게 좋다.

    ```mysql
    drop table 테이블명; -- 테이블 완전 삭제
    ```

  - 테이블 삭제 시 주의

    - 외래키 제약조건이 설정되어 있는 기준 테이블 삭제 시 오류
    - 외래키 제약조건이 설정되어 있는 테이블 먼저 삭제 후
      외래키 제약 조건 삭제하고 기준 테이블 삭제 가능

  - 테이블 삭제 시 외래키 제약 조건 상관없이 삭제 가능하도록 하는 방법

    ```mysql
    set foregin_key_checks = 0; -- 이렇게 설정하면 삭제하기 쉬워서 데이터 손실이 일어날수 있으므								로 안하는게 좋다
    ```

    




### 데이터 조작어 (DML)

- 테이블의 데이터를 검색, 삽입, 수정, 삭제하는데 사용

  

- #### INSERT 문

  - 테이블에 새로운 행을 삽입하는 명령

    ```mysql
    insert into 테이블명 (열이름 리스트) values (값리스트);
    insert into 테이블명 values (값리스트); -- 모든 열에 값을 저장한다.
    ```

  - 데이터 임포트
    - CSV파일을 읽어서 테이블 생성 및 데이터 입력
    - 파일 임포트 시 제약조건 없어짐
      - 기본키와 데이터 타입 변경이 필요하다

  

- #### UPDATE 문

  - 특정열의 값을 수정하는 명령어

  - 조건에 맞는 행을 찾아서 열의 값 수정

    ```mysql
    update 테이블명 set 열 = 값 where 조건;
    
    -- ex)상품번호가 5인 행의 상품명을 'UHD TV'로 수정
    update product set prdname='UHD TV' where prdNo='5';
    ```



- #### DELETE 문

  - 테이블에 있는 기존 행을 삭제하는 명령어

    ```mysql
    delete from 테이블명 where 조건;
    
    -- ex) 상품명이 '그늘막 텐트'인 행 삭제
    delete from product where prdName='그늘막 텐트';
    
    -- 테이블의 모든 행 삭제
    delete from product;
    ```

    

- #### SELECT 문

  - 테이블에서 조건에 맞는 행 검색

    ```mysql
    select (*(all) | distinct) 열이름 리스트 from 테이블명 (추가적인 조건들); 
    
    (추가적인 조건들 리스트)
    where 검색조건 -- 결과에 포함될 행들이 만족해야 할 조건 기술
    order by 열이름 -- 특정 열의 값을 기준으로 결과 정렬
    				-- ASC : 오름차순 DESC : 내림차순
    group by 열이름 
    having 검색조건
    ```

  - 중복 제거 
  
    - 모든 열 출력 
    - distinct
      - 속성값이 중복되는 것이 있으면 한 번만 출력
      - 주문 테이블에서 '홍길동'이 여러 번 주문한 경우 한번이라도 주문한적 이
        있는 고객명 리스트 출력시 distinct 사용하면 '홍길동' 한번만 출력

  - where 조건

    |         종류          |        연산자        | 설명/예                                                      |
    | :-------------------: | :------------------: | :----------------------------------------------------------- |
    |         비교          | =, <, >, <=, >=, !=  | 값의 크기를 비교하여 질의 검색<br />price >= 10000           |
    |         범위          |       between        | 검사 값이 기술된 두 값 사이에 속하는지 검사하여 질의 처리<br />price between 10000 and 20000 |
    |     리스트에 포함     |      in, not in      | 검사 값이 주어진 값의 리스트에 속하는 지 여부를 검사<br />price in (10000, 20000, 30000) |
    |         null          | is null, is not null | price is null                                                |
    |         논리          |       and, or        | 단순 탐색 조건을 결합하여 질의 처리<br />(price <= 10000) and (name like '김%') |
    | 패턴 매칭 문자열 검색 |         like         | 문자열의 일부가 일치하는 데이터 검색 <br />prdName like '%프린터%'<br />% : 0개 이상의 문자를 가진 문자열 <br />_ : 단일 문자(수 만큼의 문자로 구성) |
  
    
  