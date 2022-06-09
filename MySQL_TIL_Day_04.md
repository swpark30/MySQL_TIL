# MySQL_TIL_Day_04



- 목차 
  - DML
    - 다중 행 서브 쿼리
      - in / not in
      - exists / not exists
      - in과 exists의 차이
      - all / any, some 
    - 스칼라 서브쿼리 (Scalar Subquery)
    - 인라인 뷰(inline View) 보속 질의
    - MySQL 내장 함수
      - 수학 함수
      - 문자 함수
      - 날짜 함수
      - load_file()
    - MySQL 시스템 변수 : secure-file-priv 옵션 
    - 테이블 복사
  
  - DCL(Data Control Language) : 데이터 제어어
    - 관리 기능
    - 권한 (Privilage)
    - 백업 및 복구(export / import)
  
  - JDBC (Java Database Connectivity)
  
    1. JDBC 드라이버 로드
  
    2. connection 객체 생성



### 다중 행 서브 쿼리

- #### in / not in 

  ```mysql
  -- 한번도 도서를 주문한적이 없는 고객의 고객번호, 고객명 출력
  select clientNo, clientName
  from client 
  where clientNo not in (select clientNo from booksale);
  ```

  

- #### exists / not exists

  - 서브 쿼리의 결과가 행을 반환하면 참이 되는 연산자

  - where 절에 외래키 제약조건 지정

    ```mysql
    -- 도서를 구매한 적이 있는 고객
    -- 1. booksale 테이블에 조건에 해당되는 행이 존재하면 참 반환
    -- 2. client 테이블에서 이 clientNo에 해당되는 고객의
    -- 고객번호, 고객명 출력
    select clientNo, clientName
    from client 
    where exists (select clientNo from booksale where client. clientNo = booksale.clientNo);
    ```

    

- #### in과 exists의 차이

  - in
    - 서브 쿼리에서 조건에 해당되는 행의 열을 비교하여 값 확인
    - 서브 쿼리의 결과 값을 메인 쿼리에 대입하여 조건 비교 후 결과 출력
    - in 쿼리 -> 메인 쿼리
    - null 값이 결과에 포함되지 않음
  - exists
    - 서브쿼리에서 조건에 해당되는 행의 존재 여부만 확인(true / false 반환)
    - in에 비해 성능이 좋다
    - exists 키워드 앞에 속성명, 수식 등 올 수 없다
    - null 값 포함 : null 값도 결과에 포함



- #### all / any, some 

  - 관계 연산자 뒤에 위치
  
  - all
    - 검색 조건이 서브 쿼리의 결과의 모든 값에 만족하면 참이 되는 연산자
    
    - 조건 > ALL (서브 쿼리 결과)
    
      ```mysql
      -- 2번 고객이 주문한 도서의 최고 주문수량 보다 더 많은 도서를 구입한 고객의 
      -- 고객번호, 주문번호, 주문수량 출력
      select clientNo, bsNo, bsQty
      from booksale
      where bsQty > all (select bsQty from booksale where clientNo = '2');
      ```
    
      
    
  - any, some 
    - 검색 조건이 서브 쿼리의 결과 중에서 하나 이상에 만족하면 참이 되는 연산자
    
    - 조건 > ANY (서브 쿼리 결과)
    
      ```mysql
      -- 2번 고객보다 한 번이라도 더 많은 주문을 한 적이 있는 고객의 
      -- 고객번호, 주문번호, 주문수량 출력 
      -- 최소 한 번이라도 크면 됨 
      select clientNo, bsNo, bsQty
      from booksale
      where bsQty > any (select bsQty from booksale where clientNo = '2');
      ```
    
      

### 스칼라 서브쿼리 (Scalar Subquery)

- SELECT 절에서 사용

- 결과 값을 단일 열의 스칼라 값으로 반환

- 스칼라 값이 들어갈 수 있는 모든 곳에서 사용 가능

- 일반적으로 SELECT 문과 UPDATE 문에서 사용

  ```mysql
  select clientNo, 
  	   (select clientName from client where client.clientNo = booksale.clientNo) as "고객명", sum(bsQty) as "총 주문액"
  from booksale
  group by clientNo;
  ```

  

### 인라인 뷰(inline View) 보속 질의

- from 절에서 사용

- 테이블명 대신 인라인 뷰 서브 쿼리 결과 (가상 테이블) 사용

- 쿼리 결과로 반환되는 데이터는 다중 행, 다중 열이어도 상관 없음

- 가상의 뷰 형태로 제공

- 개발 중에 뷰가 필요한 모든 경우에 뷰를 생성하면 관리할 양이 너무 많아 트랜잭션 관리나
  성능 상의 문제가 발생할 수 있는 경우에 인라인 뷰 사용
  
  ```mysql
  -- 도서 가격이 25000원 이상인 도서에 대하여 
  -- 도서별로 도서명, 도서가격, 총판매수량, 총판매액 출력 
  -- 총판매액 기준으로 내림차순 정렬 
  select bookName, bookPrice, sum(bsQty) as "총 판매수량", sum(bookPrice * bsQty) as "총판매액"
  from (select bookNo, bookname, bookPrice from book where bookPrice >= 25000)book, booksale
  where book.bookNo = booksale.bookNo
  group by book.bookNo
  order by 총판매액 desc;
  ```
  
  

### MySQL 내장 함수



- #### 수학 함수

  - round()

  - rank() : 값의 순위 반환 (동일 순위 개수만큼 증가)

  - dense_rank() : 값의 순위 반환 (동일 순위 상관 없이 1증가)

  - row_number() : 행의 순위 반환

    ```mysql
    select bookPrice,
    	rank() over (order by bookPrice desc) "rank",
    	dense_rank() over (order by bookPrice desc) "dense_rank",
    	row_number() over (order by bookPrice desc) "row_number"
    from book;
    ```

    

- #### 문자 함수

  - replace() : 문자열을 치환하는 함수

    - 실제 데이터는 변경되지 않음

      ```mysql
      select bookNo, replace(bookName, '안드로이드', 'Android') bookName, bookAuthor, bookPrice
      from book
      where bookName like '%안드로이드%';
      ```

    

  - char_length() : 글자의 수를 반환하는 함수

  - length() : 바이트 수 반환하는 함수

    ```mysql
    select B.bookName as "도서명",
    	length(B.bookName) as "바이트 수",
    	char_length(B.bookName) as "길이",
    	P.pubName as "출판사"
    from book B
    inner join publisher P on B.pubNo = P.pubNo
    where P.pubName = '서울 출판사';
    ```

    

  - substr() : 지정한 길이만큼의 문자열을 반환하는 함수

    ```mysql
    -- substr(전체문자열, 시작, 길이)
    -- 저자에서 성만 출력
    select substr(bookAuthor, 1, 1) as "성"
    from book;
    -- 저자에서 이름만 출력 
    select substr(bookAuthor, 2, 2) as "이름"
    from book;
    ```

    

- #### 날짜 함수

  - date()

  - now()

  - time()

  - year() / month() / hour() / minute() / second()

  - datediff()

    ```mysql
    select date(now()), time(now());
    
    select year(curdate()), month(curdate()), dayofmonth(curdate());
    
    select datediff('2022-01-01', now()), timediff('23:23:59', '12:11:10');
    ```

    

- #### load_file()

  - txt 파일
  
  - jpg (이미지 파일)
  
  - MP4 (동영상 파일)
  
    ```mysql
    insert into flower values ('f001', '장미', 
    				load_file('D:/ITclass/MySQL/dbworkspace/file/rose.txt'),
                    load_file('D:/ITclass/MySQL/dbworkspace/file/rose.jpg'));
    ```
  
    

### MySQL 시스템 변수 : secure-file-priv 옵션 

- 보안 강화를 위해 지정한 폴더 외에는 파일의 읽기 / 쓰기를 금지 
- my.ini 파일 찾아서 메모장으로 열고 추가
  - secure-file-priv = "C://dbWorkspace/file"
  - 맨 마지막 것이 적용됨
  - ini 파일은 열어서 수정한 후 저장할 때 권한때문에 저장 안됨 (바탕화면으로 가져와서 수정)
    - C:\ProgramData\MySQL\MySQL Server 8.0
  - ini 변경 후 MySQL 서비스 재시작해야 반영됨



#### 파일 용량 초과 문제

- 파일 최대 크기 변수 변경
- max_allowed_packet = 1024M
- my.ini 파일 저장후 MySQL 서비스 재시작 해야함



### 테이블 복사

- 테이블 복사 구문
  
  ```mysql
  create table 새 테이블명 as 
  select 복사할 열 from 원본 테이블명 [where 절]
  ```
  
- 주의 
  - 기본 키 제약조건 복사 안됨
  - 복사 후 기본키 제약조건 설정해야 함





## DCL(Data Control Language) : 데이터 제어어

- 데이터의 사용 권한을 관리하는 데 사용
- 데이터베이스 트랜잭션 명시 (완료/ 취소)
- 권한 부여 및 취소 
- Grant : 데이터베이스 객체에 권한 부여
- revoke : 이미 부여된 데이터베이스 객체 권한 취소



### 관리 기능

- 계정 관리
  - 사용자 계정 생성 / 조회 / 삭제
  
    ```mysql
    -- 사용자 계정 조회 
    select * from user;
    
    -- 계정 생성 
    create user newuser2@'%' identified by '1111';
    
    -- 비밀번호 변경 
    -- set password for '계정명'@'%' = '새 비밀번호';
    set password for 'newuser2'@'%' = '1234';
    
    -- 계정 삭제 : drop user '계정'@'호스트';
    drop user 'newuser2'@'%';
    ```
  
  - 계정 잠금 / 잠금 해제
  
- 권한 관리
  - 권한 및 롤 부여 / 제거
  
    ```mysql
    -- 권한 조회
    show grants for newuser2;
    
    -- 권한 부여 grant 권한 on 데이터베이스.테이블 to 계정@호스트; 
    -- newuser2 모든 권한 부여
    grant all privileges on *.* to newuser2@'%';
    
    -- select 권한 삭제 
    revoke select on *.* from newuser2@'%';
    ```
  
- 테이블 복사

- 백업 및 복구 (export / import)



### 권한 (Privilage)

- 특정 유형의 SQL 문을 실행하거나 다른 사용자의 객체를 사용할 수 있는 권리

- 권한의 종류
  - 시스템 권한
  - 객체 권한
    - 특정 객체를 조작할 수 있는 권한
    - DML 사용 권한



### 백업 및 복구(export / import)

- DB를 주기적으로 백업해 두거나 다른 서버로 이관할 때 사용

  

- #### 백업 (export)

  - 메뉴 Server / Data Export 
  - 백업할 데이터베이스 / 테이블 선택
  - export to self-contained file 선택하고 
    - 파일 경로 및 파일명 지정 
  - Create Dump in a Single Transaction 체크
  - [Start Export] 버튼 클릭
  - 백업 방법
    - 전체 단위 백업
    - 사용자 단위 백업
    - 테이블 단위 백업

  

- #### 복구 (import)

  - 메뉴 Server / Data import
  - import from self- contained file 선택하고 
  - 복구할 파일 지정
  - [New] 버튼 누르고 스키마 이름 지정 (주의 - 없는 이름으로 지정)
  - [Start import]



## JDBC (Java Database Connectivity)

- 다양한 종류의 관계형 데이터베이스에 접근할 때 사용하는 자바 표준 SQL 인터페이스
- 자바 프로그램이 DBMS에 접근하여 작업할 수 있게 해주는 API를 제공하는 클래스 모음
- 모든 DBMS에서 공통적으로 사용할 수 있는 인터페이스와 클래스로 구성
- 실제 구현 클래스는 각 DBMS 벤더가 구현했기 때문에 거의 모든 벤더가 JDBC 드라이버 제공
- 각 DBMS에 맞는 JDBC 드라이버 사용



### JDBC 드라이버

- JDBC 인터페이스를 구현한 클래스 파일 모음 (jar 파일)
- 각 DBMS 벤더에서 제공되는 구현 클래스 



### JDBC 구성

- JDBC 인터페이스 
- JDBC 드라이버



### JDBC 역할

- 응용프로그램과 DBMS 사이에서 연결 역할
- SQL문을 DBMS에 전달하고 그 결과값을 응용프로그램에 전달하는 역할



### JDBC를 사용했을 때의 효용성

- 사용하는 RDBMS에 독립적인 프로그래밍이 가능
- 쉽게 RDBMS의 교체가 가능
- 자바는 단순히 문자열로 QUERY를 전달할 뿐이고 해석은 각 벤더가 구현한 드라이버에서 
  담당하기 때문에 표준 SQL 뿐 아니라 각 JDBC 드라이버를 제공하는 DBMS 벤더별로 최적의
  성능을 발휘할 수 있는 벤더 종속적인 SQL에 대한 처리도 가능



### JDBC를 이용한 연결 과정

1. #### JDBC 드라이버 로드
   
   - java에서 MySQL 드라이버를 사용하기 위해 드라이버를 JVM에 로딩하는 과정
   
   - 동적으로 MySQL  JDBC Driver 클래스의 객체를 생성해서 런타임 시 메모리에 로딩
   
     ```java
     Class.forName(“com.mysql.cj.jdbc.Driver”);
     ```
   
     
   
2. #### connection 객체 생성
   
   - DriverManger 클래스의 static 메소드인 getConnection() 메소드를 
     이용해서 Connection 객체를 얻어 옴
     
   - MySQL 서버 실제 연결
   
   - Connection 객체가 생성되면 DBMS 접속 성공
   
     ``` java
     1. - String url = “jdbc:mysql://localhost:3306/sqldb3?serverTimezone-UTC”;
            /* 
            - jdbc:mysql : JDBC 드라이버
            - jdbc : JDBC URL의 프로토콜 이름
            - mysql : MySQL JDBC 드라이버
            - localhost : MySQL이 설치된 IP (호스트 이름)
            - 3306 : MySQL 접속 포트
            - sqldb3 : 사용하는 데이터베이스(스키마) 이름
            */
        - String user = "root";
        - String pwd = "비밀번호";
        - DriverManager.getConnection(url, user, pwd)
     ```
   
     
   
3. #### statement 객체 생성

4. #### SQL문에 결과 반환이 있는 경우 resultSet 객체 생성 ( 결과 획득)

5. #### 쿼리 수행

6. #### 모든 객체 (자원) close()
   
   - resultSet
   - Statement
   - Connection (접속 종료)



### 패키지 import 

- JDBC는 java.sql 패키지에 포함되어 있음
- JDBC는 데이터베이스에 접속하기 위해 한개의 클래스(java.sql.DriverManager)와
  두 개의 인터페이스(java.sql.Driver, java.sql.Connection)를 사용