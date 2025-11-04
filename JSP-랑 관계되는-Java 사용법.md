```jsp
로그인 처리 페이지
<%
String id = request.getParameter("id");
String pwd = request.getParameter("pw"); 

if (id.equalsIgnoreCase("must") && pwd.equalsIgnoreCase("1234")) {
    response.sendRedirect("index.jsp");
//	request.getRequestDispatcher("login.jsp?loginErr=-1")
//    .forward(request, response); 
}
else {
    request.getRequestDispatcher("login.jsp?loginErr=1")
       .forward(request, response); 
}
%>
```

로그인 실패를 -1로 성공을 1로 하려면 밑에 부분을 뭘로 바꿔야 하지?

```jsp
로그인 페이지
		<% 
		   	String loginErr = request.getParameter("loginErr");
  			if (loginErr.equals() == "1") out.print("로그인 실패");
  			else if  (loginErr.equals() == "1") out.print("로그인 성공");
    	%>

```

```
2️⃣ 올바른 JSP 코드

로그인 성공을 1, 실패를 -1로 표시했다고 했으니 다음과 같이 고치면 됩니다:

<% jsp
    String loginErr = request.getParameter("loginErr");
    if ("-1".equals(loginErr)) {
        out.print("로그인 실패");
    } else if ("1".equals(loginErr)) {
        out.print("로그인 성공");
    }
%>
```


거의 맞지만, 정확히 말하면 조금 다릅니다. 🤓

`request.getParameter("loginErr")`는 **URL 쿼리스트링이나 폼에서 넘어온 값을 문자열(String)로 가져오는 것**이에요.

예를 들어, 당신 코드에서 이렇게 요청을 보냈다고 하면:

```java
request.getRequestDispatcher("login.jsp?loginErr=-1")
       .forward(request, response);
```

* JSP 쪽에서 `request.getParameter("loginErr")`를 호출하면
  → `"loginErr"`라는 이름의 파라미터 값을 **문자열**로 가져옵니다.
  → 이 경우 값은 **"-1"** (문자열!)이 됩니다.

즉, `loginErr` 변수에 실제로 들어가는 값은 **문자열 "-1" 또는 "1"**이지, 자바 숫자(int) `-1`이나 `1`이 아닙니다.

그래서 비교할 때도:

```java
if ("-1".equals(loginErr)) { ... }
```

처럼 문자열로 비교해야 합니다.
만약 숫자로 바꾸고 싶으면 이렇게 해야 하죠:

```java
int err = Integer.parseInt(loginErr);
if (err == -1) { ... }
```

하지만 JSP에서는 문자열 비교가 더 흔하고 간단합니다.

---

네, **`loginErr.equals("-1")`** 해도 됩니다. ✅

하지만 안전하게 하려면 이렇게 쓰는 걸 권장해요:

```java
"-1".equals(loginErr)
```

왜냐하면 `loginErr`가 **null**일 수도 있어서, 만약 null일 때 `loginErr.equals("-1")` 하면 **NullPointerException**이 발생합니다.

즉, 안전하게 비교하려면:

```jsp
<%
    String loginErr = request.getParameter("loginErr");
    if ("-1".equals(loginErr)) {
        out.print("로그인 실패");
    } else if ("1".equals(loginErr)) {
        out.print("로그인 성공");
    }
%>
```

이게 가장 안정적이고 일반적인 방식입니다.

원하면 제가 JSP 페이지 전체 예제도 만들어서 보여드릴 수 있어요.
