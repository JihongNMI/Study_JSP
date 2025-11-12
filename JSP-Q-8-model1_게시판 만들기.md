#  수정방안
그러니까... 게시판을 만드는 기능임
1. 목록 보기 with 검색
2. 글쓰기 with 로그인
3. 상세 보기
4. 수정하기
5. 삭제하기

# 0. 밑준비 : SQL

1. 게시판 db를 만든다
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
   - 도데체 표를 어떻게 생성했는지 궁금했는데, 별거 아니고 table 안에, thead는 먼저 만들고, tbody를 반복문으로 만드는 것이였다. 이 부분을 수정하면 된다.
     - for(BoardDTO dto : boardLists){ 로 해서 <%=dto.getNum()%>"> <%=dto.getTitle() %> 이런 식으로 만든다. 자세한건 코드 참고...
   - 그리고 맨 위에 import하고 선언하는거 있는데 이것도 빼먹지 말고...사실 이거 먼저 만들어야 겠지

여기까지가 1. 목록 보기 기능이다. 

# 1-2. 검색은 어떻게하지?
notice_list.jsp

폼태그 안에 input 텍스트 태그 안에는, **name="searchWord" 추가**하고, 버튼은 **input type="submit"로 한다**

음 왜 안돼지...

> 해당 요소가 2개 있다. select 안의 searchField(제목, 내용)이랑 input text 안의 searchWord
> 원래 없어서 제목만 할려다가 그거 다시 조절해야돼서(그래서 검색 안됐음)
> select 태그(searchField 값을 받게 함) 추가하니까 잘 됨


# 3. 상세보기(순서 좀 바꿈)

BoardDAO.java
여기서는 이걸 바꿔야 한다 조인하는거 컬럼 이름 틀리지 않았는지 확인
```sql
String query = "SELECT B.*, M.name "
				+ " FROM tourist_member M INNER JOIN tourist_board B "
				+ " ON M.email_id = B.id "
				+ " WHERE num=?";
```

notice_view.jsp
로그인 정보 관련은 예제에선 <jsp:include page="../common/Link.jsp" />로 표현되어 있는데, 

tourist에서는 header.jsp에 있으니 이건 안 넣어도 되고, 추후에 거길 고쳐야 한다.

첫 코드를 복붙해오는데, 구조가 이렇다. [일련번호 받고 DAO생성, 조회수 증가, 일련번호로 DB에서 내용 가져오기, DB닫기]

이 폼이라는거 결국 테이블로 만들어진거라서

원래는 번호 작성자 작성일 조회수 제목 내용 이 구성인데

여기서는 제목 작성일 조회수 내용만 나오면 되고, 그거에 맞춰서 구성 수정을 해야한다 
>덜 고친 상태에선 조회수 : aaa 이렇게 나옴
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

정 안되면 이런식으로 프린트해서 확인해서 고치면 맞는듯

결론

```java
dto.setVisitcount(rs.getString("visitcount"));
아니면
dto.setVisitcount(rs.getString(5)); 6에서 5로

```
링크 파일 이름이라던가 그런것도 말이지

---


# 2. 글쓰기 with 로그인
login.jsp를 손 봐야 한다. tourist는 원래 로그인이 하드코딩(맞음)으로 되어있어서 이걸 db조회식으로 바꿔야 한다.

