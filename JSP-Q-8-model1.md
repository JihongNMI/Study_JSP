#  수정방안
그러니까... 게시판을 만드는 기능임
1. 목록 보기
2. 글쓰기
3. 상세 보기
4. 수정하기
5. 삭제하기

# 0. 밑준비
java 밑에 model1 패키지 만들고
1. BoardDTO
2. BoardDAO
   를 복붙

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
   - 음..........
