sequenceDiagram
    participant Browser
    participant JSPServer

    Browser->>JSPServer: GET 요청
    JSPServer->>Browser: Set-Cookie 헤더 전송
    Browser->>Browser: 쿠키 저장
    Browser->>JSPServer: 다음 요청 시 Cookie 헤더 전송
    JSPServer->>JSPServer: request.getCookies()로 쿠키 읽음
