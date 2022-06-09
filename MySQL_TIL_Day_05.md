# MySQL_TIL_Day_05



- 목차 
  - JDBC (Java Database Connectivity)
    3. Statement 객체 생성
    4. 쿼리 수행
    5. ResultSet 객체로부터 결과 획득
    6. 모든 객체 close()
  - DTO / DAO 사용



3. ### Statement 객체 생성

   - 쿼리문 전송을 위한 Statement 객체 생성
   
   - 또는 PreparedStatement 객체 생성
   
   - statement 인터페이스
     - Connection 인터페이스의 createStatement() 메소드를 사용하여 객체 생성
     
       ```java
       Statement stmt = con.createStatement();
       PreparedStatement pstmt = con.prepareStatement(sql);
       ```
     
   - statement  객체를 통해 SQL 문 전송 가능



4. ### 쿼리 수행

   - SQL 쿼리 전송에 사용되는 메소드

   - executeQuery() / executeUpdate()

   - pstmt.executeUpdate()

   - 2가지로 구분하는 이유

     - SQL 문 실행 결과가 다르기 때문에

       - executeUpdate()

         - 쿼리문이 insert, update, delete 구문인 경우 사용
         - 영향을 받은 행의 수 반환
           - String sql = "insert ......"
           - int result = pstmt.executeUpdate(sql);
           - result가 0보다 크면 insert 성공
           - result가 0이면 insert 되지 않음(실패)

       - executeQuery()

         - select 구문의 경우 사용

         - ResultSet 객체로 반환 (select 문 결과에 해당되는 여러 행(또는 한 개 행) 반환)

           - String sql = "select ..."

           - ResultSet rs = pstmt.executeQuery(sql);

             

5. ### ResultSet 객체로부터 결과 획득
   
   - select 구문의 경우
   
   - executeQuery() 메소드 결과로 ResultSet 반환
   
     
   
   - ResultSet 에서 데이터 추출
     
     - ResultSet의 next() 메소드를 사용해서 논리적 커서를 이동해가며 각 열의 데이터를 바인딩해 옴
     
       ```java
       ResultSet rs = pstmt.executeQuery(sql);
       boolean b = rs.next();
       여러 행인 경우 반복 사용 : while(rs.next){...};
       ```
     
     
     
   - ResultSet 메소드
     
     - next()
       - 커서를 이동하면서 다음 행 지정
       - 시작 시 첫 번째 행 이전에 위치해 있다가 next() 메소드가 호출되면 이동하면서
         첫 데이터를 가리킴
       - 리턴 타입은 boolean
         - 다음 행이 있으면 true, 없으면 false 반환
         - 모든 행을 가져오려면 반복문 사용
       
     - getXXX()
       - 실제 값을 가져오는 메소드
       
       - 데이터 타입에 맞춰 사용
       
       - getString() / getInt() / getDate()
       
         ```java
         String bookNo = rs.getString(1);
         int bookPrice = rs.getInt(2);
         Date bookDate = rs.getDate(3);
         ```
       
         

 

6. ### 모든 객체 close()
   
   - ResultSet 
   
   - Statement(PreparedStatement) 
   
   - Connection
   
   - rs.close() / pstmt.close() / con.close()
     - 사용한 반대 순서로 닫아준다
     
     ```java
     if(rs != null) rs.close(); rs=null;  ......
     ```



- 메소드로 만들어서 객체 생성해서 





### DTO / DAO 사용 



#### DTO (Data Transfer Object)

- 데이터 저장용 클래스
- 데이터베이스에 데이터 저장하거나 데이터베이스로부터
  데이터를 조회할 때 데이터를 담아놓기(저장하기) 위해 사용
- 멤버 필드 / 생성자 / getters/setters



#### DAO (Data Access Object)

- 데이터베이스에 엑세스하는 클래스
- 데이터베이스에 데이터를 저장하거나 데이터베이스로부터 데이터를 가져올 때 사용
- DB연결(DBConnect라고 DB 연결 담당 클래스 사용해도 됨)
- select() / insert() / update() / delete() 기능의 메소드 포함 



