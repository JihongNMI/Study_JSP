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
