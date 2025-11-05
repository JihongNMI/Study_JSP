# 1. 로그인 

1. 겟어트리뷰트

   ```jsp
   login.jsp
   				<!-- 밑에 부분을 추가  -->
				
				<% if(session.getAttribute("user_id") != null) { %>
					<li style="color:white;"><%=session.getAttribute("user_id")%>님 반갑습니다.</li>
					<li><a href="mypage.jsp">마이페이지</a></li>
					<li><a href="logout.jsp">로그아웃</a></li>
					
						
					<% } else { %>
					
					
					<li><a href="login.jsp">로그인</a></li>
					<li><a href="join.jsp">회원가입</a></li>
					
					<% } %>
					
					
				<!-- 여기까지  -->
   ```

```jsp
   loginProcess.jsp

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>

<%
String id = request.getParameter("id");
String pwd = request.getParameter("pw"); 

if (id.equalsIgnoreCase("must") && pwd.equalsIgnoreCase("1234")) {
	
	//1줄 여기에 추가
	session.setAttribute("user_id", id);
	//이거
	
    response.sendRedirect("index.jsp");
    
//	request.getRequestDispatcher("login.jsp?loginErr=-1")
//    .forward(request, response); 
}
else {
    request.getRequestDispatcher("login.jsp?loginErr=1")
       .forward(request, response); 
}
%>

</body>
</html>


```
   
