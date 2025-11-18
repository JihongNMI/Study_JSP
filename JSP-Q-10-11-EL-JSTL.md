# 1. 고쳤는데 안 돼

자 일단 상황을 정리하고 다시 접근해보자
원 코드가 이것이고
```
				<tbody>
					<%if(boardLists.isEmpty()){ %>
						<tr><td>게시글이 없습니다.</td></tr>
					<%}else{ 
						for(BoardDTO dto : boardLists){
					%>
					<tr>
						<td><%=dto.getNum() %></td>
						<td class="tit_notice"><a href="notice_view.jsp?num=<%=dto.getNum()%>"><%=dto.getTitle() %></a> </td>
						<td><%=dto.getVisitcount() %></td>
						<td><%=dto.getPostdate() %></td>
					</tr>
					<%}} %>
				</tbody>
			</table>
```
수정 코드는 이것이지
```
<tbody>
				    <c:choose>
				        <c:when test="${empty boardLists}">
				            <tr>
				                <td colspan="4">게시글이 없습니다.</td>
				            </tr>
				        </c:when>
				
				        <c:otherwise>
				            <c:forEach var="dto" items="${boardLists}">
				                <tr>
				                    <td>${dto.num}</td>
				                    <td class="tit_notice">
				                        <a href="notice_view.jsp?num=${dto.num}">${dto.title}</a>
				                    </td>
				                    <td>${dto.visitcount}</td>
				                    <td>${dto.postdate}</td>
				                </tr>
				            </c:forEach>
				        </c:otherwise>
				        
				    </c:choose>
				</tbody>
			</table> 
```
그리고 지금 문제는 수정코드는 db데이터가 출력이 안된다 이거지

아, 이제 상황이 명확하네요. 😄
즉, **스크립틀릿 버전은 정상적으로 DB 데이터가 나오지만 JSTL/EL 버전은 출력이 안 된다**는 거죠.

이 경우 **원인은 거의 확실하게 “JSP에 EL이 접근할 수 있는 속성이 전달되지 않았기 때문”**입니다.

---

### 1️⃣ 핵심 포인트

1. 스크립틀릿 버전에서는 이렇게 쓰고 있었어요:

```jsp
for(BoardDTO dto : boardLists){
    <%=dto.getNum() %>
    ...
}
```

* 이때 `boardLists`는 **JSP 스크립틀릿 변수**, EL에서 접근 불가

2. JSTL/EL 버전에서는 이렇게 쓰고 있어요:

```jsp
<c:forEach var="dto" items="${boardLists}">
```

* EL은 **request, session, page, application 속성**만 접근 가능
* JSP 안에서 스크립틀릿으로 만든 `boardLists`는 **속성이 아니므로 EL에서 null**
* 따라서 반복문이 실행되지 않아 DB 데이터가 안 나오는 것

---

### 2️⃣ 해결 방법

#### (1) 스크립틀릿으로 만든 list를 **request attribute로 전달**

```jsp
<%
BoardDAO dao = new BoardDAO();
String searchWord = request.getParameter("searchWord");

// 현재 페이지 데이터
int pageNum = 1;
if(request.getParameter("pageNum") != null && !request.getParameter("pageNum").equals("")){
    pageNum = Integer.parseInt(request.getParameter("pageNum"));
}

// 한페이지 출력할 데이터의 개수
int pageSize = 10;

// 검색 및 페이징에 사용할 데이터를 맵에 저장
Map<String, Object> param = new HashMap<>();
param.put("searchWord", searchWord);
param.put("pageNum", pageNum);
param.put("pageSize", pageSize);

List<BoardDTO> boardLists = dao.selectListPage(param);

// **중요**: EL에서 접근 가능하도록 request attribute로 저장
request.setAttribute("boardLists", boardLists);

dao.close();
%>
```

* 이렇게 해야 EL `${boardLists}`가 null이 아니고 반복문이 정상 동작함

#### (2) `<c:forEach>`와 EL은 그대로 사용 가능

```jsp
<c:forEach var="dto" items="${boardLists}">
    <tr>
        <td>${dto.num}</td>
        <td class="tit_notice">
            <a href="notice_view.jsp?num=${dto.num}">${dto.title}</a>
        </td>
        <td>${dto.visitcount}</td>
        <td>${dto.postdate}</td>
    </tr>
</c:forEach>
```

---

### ✅ 핵심 요약

* EL은 **스크립틀릿 변수 접근 불가**
* 반드시 `request.setAttribute()` 등으로 JSP에 전달해야 함
* `<c:forEach>`와 `<c:choose>`는 EL로 접근 가능한 속성이 있어야 작동함

---

지금 윗부분에 마지막 request만 추가한건가?

맞아요, 정확히 말하면 **추가한 부분은 딱 한 줄**이에요:

```jsp
request.setAttribute("boardLists", boardLists);
```

* 이 한 줄 덕분에 EL `${boardLists}`가 JSP에서 접근 가능해집니다.
* 나머지 스크립틀릿 코드(DAO 생성, 리스트 가져오기, 페이징 계산 등)는 그대로 두면 됨.

즉, **EL에서 DB 데이터를 읽게 하려면 “JSP 속성으로 올려주기”가 핵심**이에요.
나머지는 `<c:forEach>` / `<c:choose>` 구조로 이미 바꾼 수정 코드 그대로 사용하면 됩니다.

원하면 제가 **스크립틀릿 최소화 + JSTL/EL 완전 변환 버전**을 한눈에 보여줄 수도 있어요.

