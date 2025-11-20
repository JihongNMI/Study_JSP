# 서블릿 : 우리가 해왔던 MemberDAO DTO BoardDAO BoardDTO 즉 자바파일
근데 음~ 한곳에서 모여서 했던걸 

컨트롤러, 서비스, DAO (중간매개체 DTO)

| 구조                 | Controller     | Service        | DAO        | JSP           |
| ------------------ | -------------- | -------------- | ---------- | ------------- |
| 1번 (JSP scriptlet) | JSP 안          | 없음             | 없음         | JSP = 화면 + 로직 |
| 2번 (Servlet + DAO) | Servlet(.java) | 없음             | DAO(.java) | JSP = 화면      |
| 3번 (MVC + Service) | Servlet(.java) | Service(.java) | DAO(.java) | JSP = 화면      |


# 그러니까 구조가...
Link.jsp -> <a href="../06/LoginForm.jsp">로그인</a> -> 로그인폼

<a href="../08/List.jsp">게시판(페이징X)</a> -> 리스트

에서

Link_servlet.jsp 안에 있는 <a href="/MustHaveJSP/boardlist.do">게시판(페이징X)</a> -> BoardListController.java 안에 있는 @WebServlet("/boardlist.do")
> 게시물 리스트 표시

ListResult.jsp 안에 있는 <c:if test="${not empty boardLists}"> 안에 있는 <a href="board.do?num=${dto.num}">${dto.title }</a> -> BoardViewController.java 안에 있는 @WebServlet("/board.do")
> 게시물 상세 보기 뷰

Link_servlet.jsp -> <a href="/MustHaveJSP/login.do">로그인</a> -> 할 차례


---
# WriteProcess.jsp를  write.do로 고치면서


좋아요, 지금까지 우리가 다룬 내용을 **정리 + 체크리스트** 형태로 깔끔하게 정리해드릴게요.
이거 그대로 점검하면 놓친 부분 없이 MVC 구조가 안정됩니다.

---

# 1️⃣ JSP 직접 호출 → 서블릿(.do) 호출로 변경 ✅

* **이전 문제**: write.jsp, loginform.jsp 등 JSP를 직접 브라우저에서 호출함 → Model2 구조에서는 잘 안 됨
* **수정 완료**:

  * 글쓰기: `Write.jsp` → `write.do`
  * 로그인: `LoginForm.jsp` → `login.do`
  * 목록보기: `List.jsp` → `list.do`

**원리**: 모든 요청은 **Controller(서블릿) → JSP forward** 흐름으로 처리

--


# 2️⃣ WriteProcess.jsp → BoardService로 옮김 ✅

* **이전 문제**: 글쓰기 처리 후 WriteProcess.jsp를 호출 → JSP 파일이 없어서 404 발생
* **수정 완료**:

  * WriteProcess.jsp 삭제
  * BoardWriteController.doPost()에서 DB insert 수행
  * 성공 시 `resp.sendRedirect("boardList.do")`, 실패 시 alert + history.back()

**원리**: MVC 구조에서는 **로직은 서블릿/서비스, 화면은 JSP**로 분리

---

# 3️⃣ sendRedirect("/boardList.do") 경로 확인 ✅

* **문제**: 대소문자 틀림, 컨텍스트 패스 미반영 → “가용하지 않음” 에러
* **수정 완료**:

  ```java
  resp.sendRedirect(req.getContextPath() + "/boardList.do");
  ```
* BoardListController 매핑 확인:

  ```java
  @WebServlet("/boardList.do")
  ```
* **원리**: 톰캣은 URL 대소문자를 구분, 프로젝트 컨텍스트까지 포함해야 안정적

---

# 4️⃣ write.jsp 폼 action, 목록 버튼 수정 ✅

* **이전 문제**:

  ```jsp
  <form action="WriteProcess.jsp">
  <button onclick="List.jsp">
  ```
* **수정 완료**:

  ```jsp
  <form action="write.do" method="post">
  <button onclick="list.do">
  ```
* **원리**: JSP 직접 접근 금지 → Controller(.do) 경유

---

# 5️⃣ 기타 확인 사항

* 서버 재시작: 이전 클래스 캐시 문제 해결
* JSP include 경로 확인:

  ```jsp
  <jsp:include page="../common/Link_servlet.jsp"/>
  ```

  * 상대경로 깨질 수 있으니 실제 경로 확인
* 세션 확인: 글쓰기, 수정 등 사용자 정보 필요 시 session.getAttribute("UserId") 체크

---

# ✅ 정리: 지금 구조 체크리스트

| 단계     | 요청 URL         | 처리 서블릿                                     | JSP 화면                |
| ------ | -------------- | ------------------------------------------ | --------------------- |
| 글쓰기 화면 | write.do(GET)  | BoardWriteController.doGet                 | Write.jsp             |
| 글쓰기 제출 | write.do(POST) | BoardWriteController.doPost → BoardService | redirect boardList.do |
| 목록 보기  | list.do        | BoardListController                        | listresult.jsp        |
| 로그인    | login.do       | LoginController                            | LoginForm.jsp         |

---

💡 **빠트린 것 확인**:

* [x] write.jsp form action 수정 ✅
* [x] 목록보기 버튼 수정 ✅
* [x] WriteProcess.jsp 삭제 / doPost redirect로 변경 ✅
* [x] boardList.do 경로 대소문자, 컨텍스트 포함 ✅
* [x] 서버 캐시 제거 및 재시작 ✅

즉, 지금 구조라면 **MVC 흐름 완전히 정상**입니다.

---

원하면 제가 이 내용을 **그림으로 흐름**을 그려서
“write → DB → list”까지 한눈에 보여주고, Model1 구조와 비교해서 이해 쉽게 만들어줄 수도 있습니다.

그림으로 보고 싶나요?
