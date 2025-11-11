#  수정방안
그러니까... 게시판을 만드는 기능임
1. 목록 보기
2. 글쓰기
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
   - selectCount
   - selectList
   - insertWrite
   - selectView
   - updateVisitCount
   - updateEdit
   - deletePost
  
   가 있다.(많이도 있다...)

   원래 SQL쿼리문에 FROM board를 여기서 쓰는 보드DB 이름으로 바꿔야 한다.(from tourist_board)

3. notice_list.jsp : 해왔던거
   - <jsp:include page="header.jsp" />,랑 <jsp:include page="footer.jsp" /> 를 대체해서 넣는다

# 1. 목록 보기
notice_list.jsp
   - 도데체 표를 어떻게 생성했는지 궁금했는데, 별거 아니고 table 안에, thead는 먼저 만들고, tbody를 반복문으로 만드는 것이였다. 이 부분을 수정하면 된다.
     - for(BoardDTO dto : boardLists){ 로 해서 <%=dto.getNum()%>"> <%=dto.getTitle() %> 이런 식으로 만든다. 자세한건 코드 참고...
   - 그리고 맨 위에 import하고 선언하는거 있는데 이것도 빼먹지 말고...사실 이거 먼저 만들어야 겠지

여기까지가 1. 목록 보기 기능이다. 

# 2. 
