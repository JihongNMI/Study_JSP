# 2025-11-25 ~ 27 3일간


2025-11-25 
# 1. 9~11시 : 맨 처음 무료 템플릿을 찾기가 어렵다 
무료 템플릿 자체는 많이 있는데 일회성이 아니고 지속적으로 개인적으로도 사용하고 싶어서
> 맨 처음엔 웹디자인기능사 공개문제를 활용하려고 했으나 저작권 문제를 알 수가 없어서 기각했다

처음엔 코드가 작고 가벼운 옛날 템플릿을 사용하려고 했다. Frozen age 
> https://all-free-download.com/free-website-templates/download/frozen_age_6889120.html

하지만 역시 반응성이 있어야 한다고 생각했다.
> 반응성 웹이 기억속에서 사라져있었다...

Responsive free html 이런걸로 열심히 검색했다
> 로그인폼도 같이 있는데가 좋은데 잘 안나오더라
>> 그럼 뭐 따로 만들어야지...

kml이나 카노나 이런거 쓸까하다가 음~

결국 예전에 기억을 더듬어서 하나 찾았음. 여기는 유명한데고 You are free to modify, save, share, and use them in all your projects.라고 적혀있어서 제약 없이 가능할 것으로 보였음
> https://www.w3schools.com/css/css_rwd_templates.asp
 
# 2. 11시~12시 : 구조파악 실시, 로그인폼 만들기
head, 윗단에 nav, 왼쪽 가운데 오른쪽, 푸터, 스크립트 순이였다
로그인폼은 일단 구색만 갖추는 수준으로

# 3. 12시~ include랑 라이브러리
아~ 각 부분(nav, left column, right column, footer, 그리고 javascript까지도)을 분리해서 include하는 게 있었다.
> 기억 속에서 사라져 있었다
 
그래서 이것들 작업함.

ojdbc11은 db용이고, jakarta.servlet.jsp.jstl-api-3.0.0랑 jakarta.servlet.jsp.jstl-3.0.1는 <c: 이런거(JSTL)용인데 왜 파일이 2개였지...
> 3.0.0은 명세만 정의[API 파일], 3.0.1은 실제 java코드[구현체 파일]

정리하니까 깔끔해졌다

# 4. 14시~ : xml설정, 절대경로 설정
web.xml을 설정, 설정 내용은 로그인 세션 시간, UTF-8
> 이것도 까먹고 있었는데 오라클 로그인 어떻게 하지?? -> server.xml에서 한다

/WEB-INF/common/navbar.jsp 이런식으로 구성요소들을 모아서 절대경로로 설정

# 5. 15시~ : dto dao 등 정리

이번에 패키지를 다 정리해서 model2식으로 만들려고 하니 어떻게 패키지를 구성해야할지 몰라서 좀 고생

각종 임포트 이름들도 고치고

2025-11-26
# 1. 9~10시 : 예외처리

## 1. joinProcess.jsp의 회원가입 부분을 가지고 왔더니 -> Unhandled exception type SQLException 에러가 나와서 보아하니 -> 예외처리를 추가해야한다고 하더라.
## 2. 성공했는지 실패했는지? -> int result = db에 저장(cp.psmt.executeUpdate();) 하면 db저장도 해주고 result에 성공 1값을 돌려준다고 해서 좋다고 하더라.

# 2. 10시~ : form 수정, BoardDAO BOardDTO

form의 각종 속성(form의 post, button의 submit


2025-11-27
# 1. 9시~ 빼먹은거 처리, 안되는 문제 해결(ㅜㅜ), 중복된 코드 처리, ppt작성

