# 1. Tourist 개선 방법

## 1. src/main/java/common에 DBConnPool.java를 준비

```java

public class DBConnPool {
    public Connection con;
    public Statement stmt;
    public PreparedStatement psmt;
    public ResultSet rs;


어쩌구저쩌구

```

## 2. 이건 까먹고 있었는데 처음에 이걸 해놔서 모든 서버에 적용된다...Tomcat 설치 폴더 내의 conf/server.xml

<GlobalNamingResources> 태그 내부에 <Resource> 태그로 DB 정보가 정의되어 있을 수 있습니다. 

```xml
  <GlobalNamingResources>
    <!-- Editable user database that can also be used by
         UserDatabaseRealm to authenticate users
    -->
    <!-- <Resource auth="Container" description="User database that can be updated and saved" factory="org.apache.catalina.users.MemoryUserDatabaseFactory" name="UserDatabase" pathname="conf/tomcat-users.xml" type="org.apache.catalina.UserDatabase"/> -->
    
        <Resource auth="Container"
              driverClassName="oracle.jdbc.OracleDriver"
              type="javax.sql.DataSource" 
              initialSize="0"
              minIdle="5"
              maxTotal="20"
              maxIdle="20"
              maxWaitMillis="5000"
              url="jdbc:oracle:thin:@localhost:1521:ORCL"
              name="dbcp_myoracle"
              username="musthave2"
              password="1234" />
    
  </GlobalNamingResources>

```

## 2-1. 야 이것도 까먹고 있었다야 context.xml도 수정


```xml

        <ResourceLink global="dbcp_myoracle" name="dbcp_myoracle" 
                  type="javax.sql.DataSource"/>
    

```


## 3. 회원가입 join.jsp 의 밑준비

<form action="joinProcess.jsp" class="appForm"> 안에 각종 input들, name="emailID", "password" 등을 쳐서 정비

## 4. joinProcess.jsp 작성

```jsp

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@page import="common.DBConnPool"%>    
    
<%
	String emailId = request.getParameter("emailId");
	String emailAddress = request.getParameter("emailAddress");
	String name = request.getParameter("name");
	String password = request.getParameter("password");
	String tel = request.getParameter("tel");
	String gender = request.getParameter("gender");
	String content = request.getParameter("content");

	DBConnPool cp = new DBConnPool();
	
	/* sql구문임 */
	String sql = "INSERT INTO tourist_member(email_Id, email_Address, name, password, tel, gender, content, regidate)"
				+ "VALUES(?,?,?,?,?,?,?,sysdate)";
	cp.psmt = cp.con.prepareStatement(sql);
	
	cp.psmt.setString(1, emailId);
	cp.psmt.setString(2, emailAddress);
	cp.psmt.setString(3, name);
	cp.psmt.setString(4, password);
	cp.psmt.setString(5, tel);
	cp.psmt.setString(6, gender);
	cp.psmt.setString(7, content);
	
	
	int inResult = cp.psmt.executeUpdate();
	out.println(inResult + "행이 입력되었습니다.");
	
	cp.close();
		
%>

```

뭐 이런 식

# 2. 체크박스 처리하는 방법이 있음(나중에 배울려나?)
