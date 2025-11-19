# 그러니까 구조가...
Link.jsp -> <a href="../06/LoginForm.jsp">로그인</a> -> 로그인폼

<a href="../08/List.jsp">게시판(페이징X)</a> -> 리스트

에서

Link_servlet.jsp -> <a href="/MustHaveJSP/login.do">로그인</a> -> 

<a href="/MustHaveJSP/boardlist.do">게시판(페이징X)</a> -> BoardListController.java 안에 있는 @WebServlet("/boardlist.do")
: 게시물 리스트 표시

ListResult.jsp 안에 있는 <c:if test="${not empty boardLists}"> 안에 있는 <a href="board.do?num=${dto.num}">${dto.title }</a> -> BoardViewController.java 안에 있는 @WebServlet("/board.do")
: 게시물 상세 보기 뷰
