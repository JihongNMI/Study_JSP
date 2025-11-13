# 1. 성공과 실패 확인

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

---

맞아요, JSP나 자바에서 문자열 비교를 배울 때 보통 처음 배우는 방식이 **`null` 체크 후 equals** 또는 **`equals`로 비교**였어요. 그런데 실무에서는 **항상 null 안전하게 비교**하는 게 관례입니다.

즉, 두 가지 접근을 정리하면:

---

### 1️⃣ 전통적 방식 (null 체크)

```java
if (loginErr != null && loginErr.equals("-1")) {
    out.print("로그인 실패");
}
```

* loginErr가 null이면 equals 호출 시 NPE 방지.
* 이해하기 쉽지만 조금 길어짐.

---

### 2️⃣ 실무 안전 방식 (상수를 앞에)

```java
if ("-1".equals(loginErr)) {
    out.print("로그인 실패");
}
```

* null이 들어와도 안전하게 비교 가능.
* 간단하고 깔끔해서 JSP/Servlet에서 많이 사용됨.

---

즉, 네가 배운 null 체크 방식도 맞지만, 실무에서는 **상수 앞에 두고 equals** 하는 방법이 더 흔합니다.
사실 이건 자바에서 “null-safe 문자열 비교”의 대표적인 패턴이에요.

원하면 내가 **성공/실패 메시지를 JSP에서 더 깔끔하게 표시하는 예제**로 보여줄 수도 있어요. 그러면 바로 써먹기 좋습니다.


# 2. 아니 java resources랑 src/java 안의 내용은 같잖아. 근데 듣기로는 후자보다 전자에서 파일을 다루라고 하더라고?

결론부터 말씀드리면, Java 파일 작업은 이클립스에서 제공하는 논리적 뷰인 Java Resources를 통해 하는 것을 압도적으로 추천합니다.

이클립스를 사용하는 주된 이유 중 하나가 바로 이 개발 편의 기능을 활용하기 위함입니다.

🤷‍♀️패키지를 이름 등을 알아서 바꿔주는 기능이 있다구?

✨네, 맞습니다! **패키지 이름 등을 알아서 바꿔주는 기능**이 이클립스의 `Java Resources` 뷰를 통해 작업할 때 얻을 수 있는 **가장 큰 이점**입니다.

이것이 바로 **'리팩토링(Refactoring)'** 기능입니다.

### 🛠️ 리팩토링(Refactoring) 기능의 마법

일반적으로 Java에서 클래스의 위치(패키지)를 바꾸면 다음과 같은 복잡한 작업이 필요합니다.

1.  **물리적 파일 이동:** 해당 클래스 파일(`.java`)을 새 폴더로 옮긴다.
2.  **소스 코드 수정 (1):** 옮긴 파일 내부의 첫 줄에 있는 `package` 선언문을 새 패키지 이름으로 바꾼다.
3.  **소스 코드 수정 (2):** 이 클래스를 사용하던 **모든 다른 파일**을 찾아서, 그 파일들 내부에 있는 `import` 구문도 모두 새 패키지 이름에 맞춰 수정한다. (이게 제일 골치 아픈 작업이죠!)

하지만 이클립스의 `Java Resources` 뷰에서 클래스나 패키지를 드래그 앤 드롭으로 이동시키거나, 이름 변경(Refactor > Rename) 메뉴를 사용하면 이 모든 3단계를 **이클립스가 자동으로 처리**해 줍니다. 

### 💡 왜 `Java Resources`를 써야 하는가?

`Java Resources`는 이클립스가 **Java 개발 환경에 최적화된 논리적 뷰**를 제공하기 때문입니다. 일반 폴더 구조(예: `Project Explorer`에서 보는 단순 디렉토리)에서 작업하면 이클립스는 파일 시스템을 다루는 것처럼 행동하여 단순 이동만 시키고, **'리팩토링'이라는 마법은 부려주지 않습니다.**

따라서, Java 개발 시에는 **Java의 문법적 구조(패키지, 클래스)**에 맞춰 편리하게 작업할 수 있도록 설계된 `Java Resources` 뷰를 사용하시는 것이 훨씬 효율적이고 실수도 줄일 수 있습니다. 수동으로 `import` 구문 수십 개를 고치는 것보다는 AI의 도움을 받는 게 낫죠! 😉
