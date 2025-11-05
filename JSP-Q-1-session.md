# 1. 말로 정리

```
음...... 함수? 메소드? jsp 측면에서 일단 내가 생각한걸 말해볼테니 fact check등 해서 좀 다듬어볼래 1. [로그인했거나 안한 분기가 있는 메인] : session.getAtrribute("user_id")가 null인지 아닌지로 분기를 해서 이걸로 ~님 반갑습니다를 출력합니다. 2. [로그인] : get이나 post를 통해서 정보를 받는데 보통 form action="loginProcess.jsp"의 post방식으로, 여기서의 name으로 id, pw를 받습니다. 그리고 이 페이지에서 request.getParameter("loginErr")의 값을 스트링에 받아와서, 이 값을 비교해서 로그인 성공 실패를 검증합니다. loginErr는 loginProcess.jsp에서 다룹니다. 3. [로그인프로세스] : 여기에선 스트링에 2에서 받아온 id, pw를 request.getParameter("id"), request.getParameter("pw")로 저장합니다. 그리고 맞는지 확인 한 후에 맞다면, session.setAttribute("user_id", id);를 통해서 1에서 쓸 '세션의 user_id'를 채워 넣고, response.sendRedirect("index.jsp");를 통해서 페이지를 돌려줍니다. 만약 틀렸다면, request.getRequestDispatcher("login.jsp?loginErr=1") .forward(request, response);를 써서 돌려줍니다
```

아~ 오케이 👍
그럼 지금처럼 “말로 로그인 절차를 설명하는 문장”을 자연스럽게 정리해주면 되는 거구나.
너가 쓴 1~3번 내용을 기반으로 문장으로 정리하면 이렇게 표현할 수 있어요 👇

---

1. **메인 페이지에서는** `session.getAttribute("user_id")`가 `null`인지 아닌지를 확인해서 로그인 여부를 판단합니다. 값이 있으면 로그인된 상태로 간주하고, “○○님 반갑습니다” 같은 문구를 출력합니다.

2. **로그인 페이지에서는** 일반적으로 `form` 태그를 통해 `POST` 방식으로 아이디(`id`)와 비밀번호(`pw`)를 `loginProcess.jsp`로 전송합니다.
   로그인 페이지에서는 `request.getParameter("loginErr")` 값을 받아서, 이 값이 있으면 로그인 실패 메시지를 띄워주는 식으로 처리할 수 있습니다. `loginErr` 값은 로그인 프로세스 페이지에서 설정됩니다.

3. **로그인 프로세스 페이지(`loginProcess.jsp`)에서는** `request.getParameter("id")`, `request.getParameter("pw")`로 전달받은 아이디와 비밀번호를 검증합니다.
   만약 일치한다면 `session.setAttribute("user_id", id)`로 세션에 사용자 정보를 저장하고, `response.sendRedirect("index.jsp")`로 메인 페이지로 이동시킵니다.
   반대로 일치하지 않으면 `request.getRequestDispatcher("login.jsp?loginErr=1").forward(request, response);`를 사용해 로그인 페이지로 되돌아가며, 실패 정보를 함께 전달합니다.

---

이런 식으로 말로 풀면 JSP 로그인 절차를 깔끔하게 설명할 수 있어요.
원하면 좀 더 기술 발표나 문서용 톤으로 다듬어드릴 수도 있는데, 그렇게 해드릴까?




# 2. 로그인 

1. 로그인페이지의 겟어트리뷰트

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


2. 로그인 프로세스 페이지의 셋어트리뷰트


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
   
