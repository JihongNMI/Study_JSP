#  수정방안
그러니까... 게시판을 만드는 기능임
1. 목록 보기 with 검색
2. 글쓰기 with 로그인
3. 상세 보기
4. 수정하기
5. 삭제하기

# 0. 밑준비 : SQL

1. 게시판 db를 SQL에 만든다
```sql
CREATE TABLE TOURIST_BOARD(
	num NUMBER PRIMARY KEY,
	title VARCHAR2(200) NOT NULL,
	content VARCHAR2(2000) NOT NULL,
	postdate DATE DEFAULT sysdate NOT NULL,
	visitcount NUMBER(6),
	id VARCHAR2(100) NOT NULL
);

```
2. 외부키를 설정한다(회원정보db의 id(주키)에 외래키로서 연결함 -> 이 안에 있는 아이디들만 가능하게 만듬)
```sql
ALTER TABLE TOURIST_BOARD
	ADD CONSTRAINT tourist_board_mem_fk FOREIGN KEY(id)
	REFERENCES tourist_member(email_id);
```
3. 게시판번호용 시퀀스 생성
```sql
CREATE SEQUENCE seq_tourist_board_num
	INCREMENT BY 1
	START WITH 1
	MINVALUE 1
	NOMAXVALUE
	NOCYCLE
	NOCACHE;
```
4. 적당한 값을 집어넣어 둔다
```sql
INSERT INTO tourist_board(num, title, content, id, postdate, visitcount)
VALUES (seq_tourist_board_num.nextval, '이번 여름 휴가 제주 갈까? 미션 투어 (여행경비 50만원 지원)','내용1','aaa',sysdate, 0);

INSERT INTO tourist_board(num, title, content, id, postdate, visitcount)
VALUES (seq_tourist_board_num.nextval, '박물관 미션 투어 응모 당첨자 발표','내용2','aaa',sysdate, 0);

INSERT INTO tourist_board(num, title, content, id, postdate, visitcount)
VALUES (seq_tourist_board_num.nextval, '추석 연휴 티켓/투어 배송 및 직접 수령 안내','내용3','aaa',sysdate, 0);
```


# 0. 밑준비 : jsp
java 밑에 model1 패키지 만들고
1. BoardDTO 를 복붙
2. BoardDAO 를 복붙

1. DTO : 에서는 쓰는 변수 정리(게시판DB 구성 칼럼)하면 된다. 복사한건 name 쓰는데 여긴 없어서 name 관련해서 지웠다.
2. DAO : DAO안에
   - selectCount : 게시물 개수 세기. 왜 세느냐? 10개 있다고 치면 검색했을 때 5개가 나오게 다시 세야하기 때문
   - selectList : 게시물 목록 가져오기. 얘도 검색하면 따로 SQL 쿼리로 가져오게 하는 기능이 포함됨, jsp에 for문써서 테이블 만드는 것도 해야함
   - insertWrite
   - selectView
   - updateVisitCount
   - updateEdit
   - deletePost
  
   가 있다.(많이도 있다...)

   원래 SQL쿼리문에 FROM board를 여기서 쓰는 보드DB 이름으로 바꿔야 한다.(from tourist_board)

3. notice_list.jsp : 해왔던거
   - <jsp:include page="header.jsp" />,랑 <jsp:include page="footer.jsp" /> 를 대체해서 넣는다(<jsp:include page="quicklink.jsp" />)도 있어요~)

# 1-1. 목록 보기 with 검색
notice_list.jsp
   - ✔문제 : 도데체 표를 어떻게 생성했는지? 궁금했는데, 별거 아니고 table 안에, thead는 먼저 만들고, tbody를 반복문으로 만드는 것이였다. 이 부분을 수정하면 된다.
     - for(BoardDTO dto : boardLists){ 로 해서 <%=dto.getNum()%>"> <%=dto.getTitle() %> 이런 식으로 만든다. 자세한건 코드 참고...
   - 그리고 맨 위에 import하고 선언하는거 있는데 이것도 빼먹지 말고...사실 이거 먼저 만들어야 겠지

여기까지가 1. 목록 보기 기능이다. 

# 1-2. 검색은 어떻게하지?
notice_list.jsp

폼태그 안에 input 텍스트 태그 안에는, **name="searchWord" 추가**하고, 버튼은 **input type="submit"로 한다**

✔문제 : 음 왜 검색이 안돼지... : 제목검색만 하지말고 예제 그대로 제목&내용 검색으로 ㄱㄱ

> 해당 요소가 2개 있다. select 안의 searchField(제목, 내용)이랑 input text 안의 searchWord
> 원래 없어서 제목만 할려다가 그거 다시 조절해야돼서(그래서 검색 안됐음)
> select 태그(searchField 값을 받게 함) 추가하니까 잘 됨


# 3. 상세보기(순서 좀 바꿈)

BoardDAO.java
여기서는 이걸 바꿔야 한다 조인하는거 컬럼 이름 틀리지 않았는지 확인
```sql
String query = "SELECT B.*, M.name "
				+ " FROM tourist_member M INNER JOIN tourist_board B "
				+ " ON M.email_id = B.id "  여기 email_id로
				+ " WHERE num=?";
```
>✨여기서 유추할 수 있는 것 : "SELECT B.*, M.name " 가 출력 순서고 결과임(

notice_view.jsp
로그인 정보 관련은 예제에선 <jsp:include page="../common/Link.jsp" />로 표현되어 있는데, 

tourist에서는 header.jsp에 있으니 이건 안 넣어도 되고, 추후에 거길 고쳐야 한다.

첫 코드를 복붙해오는데, 구조가 이렇다. [일련번호 받고 DAO생성, 조회수 증가, 일련번호로 DB에서 내용 가져오기, DB닫기]

이 폼이라는거 결국 테이블로 만들어진거라서

원래는 번호 작성자 작성일 조회수 제목 내용 이 구성인데

여기서는 제목 작성일 조회수 내용만 나오면 되고, 그거에 맞춰서 구성 수정을 해야한다 
>✔문제 발생 : 덜 고친 상태에선 조회수 : aaa 이렇게 나옴
>>조회수 : <span> <%=dto.getVisitcount() %> </span> 했는데도 이렇게 나오는데??
>>>그건 BoardDAO를 손봐야되는데
```java

    // 결과셋에 어떤 값이 들어있는지 확인
    System.out.println("=== ResultSet 컬럼 값 테스트 ===");
    for (int i = 1; i <= rs.getMetaData().getColumnCount(); i++) {
        String columnName = rs.getMetaData().getColumnName(i);
        String columnValue = rs.getString(i);
        System.out.println(i + "번 컬럼 (" + columnName + "): " + columnValue);
    }
    System.out.println("================================");

```
로 확인

```
1번 컬럼 (NUM): 3
2번 컬럼 (TITLE): 추석 연휴 티켓/투어 배송 및 직접 수령 안내
3번 컬럼 (CONTENT): 내용3
4번 컬럼 (POSTDATE): 2025-11-11 14:32:36
5번 컬럼 (VISITCOUNT): 7
6번 컬럼 (ID): aaa
7번 컬럼 (NAME): AAA
```

그래서 뭐가 문제였냐면
```java
public BoardDTO selectView(String num) {
		BoardDTO dto = new BoardDTO();
		String query = "SELECT B.*, M.name "
				+ " FROM tourist_member M INNER JOIN tourist_board B "
				+ " ON M.email_id = B.id "
				+ " WHERE num=?";
		try {
			psmt = con.prepareStatement(query);
			psmt.setString(1, num);
			rs = psmt.executeQuery();
			if(rs.next()) {
				dto.setNum(rs.getString(1));
				dto.setTitle(rs.getString(2));
				dto.setContent(rs.getString("content"));
				dto.setPostdate(rs.getDate("postdate"));
				dto.setId(rs.getString("id"));
				dto.setVisitcount(rs.getString(6));
				// dto.setName(rs.getString("name"));
				
			}
			
			
			// 확인차
			
			    System.out.println("=== ResultSet 컬럼 값 테스트 ===");
			    for (int i = 1; i <= rs.getMetaData().getColumnCount(); i++) {
			        String columnName = rs.getMetaData().getColumnName(i);
			        String columnValue = rs.getString(i);
			        System.out.println(i + "번 컬럼 (" + columnName + "): " + columnValue);
			    }
			    System.out.println("================================");
			
			// 확인차
			
			
		}catch(Exception e) {
			System.out.println("게시물 상세보기 중 예외 발생");
			e.printStackTrace();
		}
		return dto;
```

지금 실습이라서인지 rs.getString(번호)랑 rs.getString(컬럼명)이랑 섞여있는데

컬럼명이 안전하네 
>1️⃣ 옛날 JDBC 예제들의 “습관적인 관성”2️⃣ “인덱스로 접근하면 더 빠르다”는 오래된 인식3️⃣ “SELECT *”를 썼기 때문4️⃣ 프레임워크가 등장하면서 컬럼명 방식으로 전환됨

정 안되면 이런식으로 프린트해서 확인해서 고치면 맞는듯

결론

```java
dto.setVisitcount(rs.getString("visitcount"));
아니면
dto.setVisitcount(rs.getString(5)); 6에서 5로 수정
```
그렇게 고치면 된다

링크 파일 이름이라던가 그런것도 말이지(뭐였지? 아마 jsp파일 이름일껄?)

---


# 2. 글쓰기 with 로그인
# 2-1. 로그인 처리
login.jsp를 손 봐야 한다. 그전에 loginProcess.jsp도 만들고, MemberDAO랑 MemberDTO클래스도 만들어야 한다.
아. 그리고 이건 로그인 처리고

나는 loginProcess부터 만들려고 했는데, 순서 따라서 DAO랑 DTO부터 만들자

DAO
```java
package member;

import common.DBConnPool;

public class MemberDAO extends DBConnPool{
	public MemberDTO login(MemberDTO param) {
		MemberDTO dto = new MemberDTO();
		try {
			String query = "SELECT * FROM tourist_member "
						+ " WHERE email_id=? AND password=? ";
			psmt = con.prepareStatement(query);
			psmt.setString(1, param.getEmail_id());
			psmt.setString(2, param.getPassword());
			rs = psmt.executeQuery();
			if(rs.next()) {
				dto.setEmail_id(rs.getString("email_id"));
				dto.setEmail_address(rs.getString("email_address"));
				dto.setName(rs.getString("name"));
				dto.setPassword(rs.getString("password"));
				dto.setTel(rs.getString("tel"));
				dto.setGender(rs.getString("gender"));
				dto.setAgree(rs.getString("agree"));
				dto.setContent(rs.getString("content"));
				dto.setRegidate(rs.getDate("regidate"));
			}
		}catch(Exception e) {
			e.printStackTrace();
		}
		return dto;
	}
}

```



loginProcess.jsp
```
<!-- 3. loginProcess : 임포트하고   -->

<%@page import="utils.JSFunction"%>
<%@page import="utils.CookieManager"%>


<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%
	String id = request.getParameter("id");
	String pw = request.getParameter("pw");
	
	String save_check = request.getParameter("save_check");
	
	if(id.equals("must") && pw.equals("1234")){
		
// 4. loginProcess : 쿠기 관련 내용 추가, 체크박스의 equals의 기본은 "on"이라 그거 수정
		
		if(save_check != null && save_check.equals("on")){
			CookieManager.makeCookie(response, "loginId", id, 86400);
		}else{
			CookieManager.deleteCookie(response, "loginId");
		}
		
		
		
		session.setAttribute("user_id", id);
		response.sendRedirect("index.jsp");
	}else{
		request.getRequestDispatcher("login.jsp?loginErr=1")
			.forward(request, response);
	}
%>
```
원래 이렇게 하드코딩되어있는걸 고쳐야 한다. db조회식으로 바꿔야 한다.

```java
<%@page import="member.MemberDTO"%>
<%@page import="member.MemberDAO"%>
<%@page import="utils.CookieManager"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%
	// 로그인시 필요한 파리미터
	String id = request.getParameter("id");
	String pw = request.getParameter("pw");
	// 아이디 저장 기능 활성화 여부 확인 변수
	String save_check = request.getParameter("save_check");
	// DB연결
	MemberDAO dao = new MemberDAO();
	// 검색조건 설정
	MemberDTO param = new MemberDTO();
	param.setEmail_id(id);
	param.setPassword(pw);
	// 로그인 데이터 저장
	MemberDTO dto = dao.login(param);
	// id와 pw가 DB와 일치하는지 확인하는 if문
	if(dto.getEmail_id() != null && dto.getEmail_id().equals(id)){
		if(save_check != null && save_check.equals("on")){
			CookieManager.makeCookie(response, "loginId",id , 60*60*24*7);
		}else{
			CookieManager.deleteCookie(response, "loginId");
		}
		session.setAttribute("user_id", id);
		response.sendRedirect("index.jsp");
	}else{
		request.getRequestDispatcher("login.jsp?loginErr=1")
			.forward(request, response);
	}
%>
```

✔ 구조 : jsp의 id, pw -> loginprocess.jsp 의 request.getparameter(id,pw) -> DAO안에 getmemberDTO로 SQL조회 후 일치하면 그 값을 반환 -> 일치하면 세션에 id, 이름을 넣고 redirect

login.jsp는 수정할 필요가 없다 : value가 헷깔렸는데, 이건 자동완성(미리 적어놓는 값)이고, id, pw비교는 name으로 하니까.


---
# 2-2. 글쓰기
글쓰기하는것도 있다. notice_add.jsp, noticeAddProcess.jsp를 만들고, notice_list.jsp에 버튼을 추가해서 연결. (원래 BoardDAO안에 기능 넣어야되는데 이건 복붙해옴)

notice_add.jsp : 작성하는 페이지임

1. head안에 제목이나 내용을 다 입력하는거 확인하는 스크립트를 추가
```
<script type="text/javascript">
function validateForm(form){
	if(form.title.value == ""){
		alert("제목을 입력하세요.");
		form.title.focus();
		return false;
	}
	if(form.content.value == ""){
		alert("내용을 입력하세요.");
		form.content.focus();
		return false;
	}
}
</script>
```

2. 로그인한사람 안한사람 구분하는것 : 이건 실습에서는 link.jsp라고 만들었는데 여기서는 include header.jsp로 퉁치자

3. form 태그 안에 스크립트 실행을 추가 onsubmit="return validateForm(this);" . name은 잘 맞아있으니 패스

4. 더 없지? 아 있다 : <%@ include file="IsLoggedIn.jsp" %> 이걸 맨 처음에 넣어서 로그인되어있는지 확인

---

noticeAddProcess.jsp
1. 이건 사실 isLoggedin.jsp이라고, 로그인되어있는지 안되어있는지 확인하는 JSFunction이 들어있으니 이걸 만들어놓고(복사) import해옴
> 아 그럼 로그인 확인하는 <%@ include file="IsLoggedIn.jsp" %>은 쓰는 곳이랑 쓰는 프로세스 두군데 다 들어가있네요
> > 그 거 리다이렉트 페이지 이름도 수정해


notice_list.jsp
1.버튼 추가. 이건 사실 음 주변에 맞춰서 만들어야되는데
```
<table border="1" width="90%">
		<tr align="right">
			<td>
			<button type="button" onclick="location.href='notice_add.jsp';">글쓰기</button>
			</td>
		</tr>
	</table>
```

✔로그인은 되었는데 글이 안써져요ㅗ
> 기존에 쓰는 loginProcess.jsp의  session.setAttribute("user_id", id);랑, 새로 만든 isLoggedin.jsp의 if(session.getAttribute("UserId") == null){
> 의 변수가 달라서 그러하니, used_id로 통일해준다

이제 됐나?

✔noticeAddProcess.jsp에서 에러가 났어요
> 이 페이지에서도 dto.setId(session.getAttribute("UserId").toString());라고 세션을 이용하니까 user_id로 통일합니다.

이제야 된 것 같네요.......

# 4. 수정하기


