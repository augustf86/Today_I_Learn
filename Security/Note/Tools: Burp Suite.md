# Tools: Burp Suite

## Burp Suite 설치

<br/><br/>

## Burp Suite: Target 탭
<img width="1187" alt="burp suite_target" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9d8ed2a8-944b-4175-9b72-026801e08dda">

* Burp Suite로 탐색할 사이트에 대한 정보를 확인할 수 있음 (웹 사이트 구조 분석 시 매우 유용함)
    - Burp Suite 실행 후 프록시를 설정한 다음 웹 사이트를 탐색하면 [Target] 탭 아래에 해당 사이트의 구조가 나타남

<br/>

### Target 기능 사용법
1. Burp Suite를 실행한 후 Burp Suite의 [Proxy] 탭에서 Burp Suite의 프록시 기능을 비활성화함(Intercept if off)
    <img width="1143" alt="프록시 기능 비활성화(traget)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/27b8ad18-230f-4e5e-a7f3-5938e17a4b1a">
    - 프록시 기능이 기본적으로 활성화돠어 있는데, 이 때문에 Target 탭의 [Open Browser]로 분석하고자 하는 웹 사이트에 접속할 때 페이지가 넘어가지 않고 멈춰있게 됨

<br/>

2. Target 탭에서 [Open Browser]로 브라우저를 열고 분석하고자 하는 웹 사이트에 접속함

<br/>

4. 웹 사이트에 접속한 후 메뉴나 게시글 등 다양한 부분을 모두 클릭하고 가능하다면 로그인도 시도해본 다음 [Target] 탭을 보면 아래와 같이 많은 수의 요청 목록을 볼 수 있음
    <img width="2560" alt="탐색 후 traget" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/111d4599-d354-475c-a969-e22da7346faf">
    - 웹 사이트의 구조를 분석할 때 해당 페이지에 포함된 링크 사이트까지 조사함 → 직접 방문한 페이지 외에도 링크되어 있는 페이지와 웹 사이트까지 분석할 수 있음

<br/>

5. 분석을 원하는 항목을 클릭하여 해당 웹 사이트의 구조를 확인함
    <img width="2560" alt="target 탭 분석" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/113d7826-787d-40b7-9dd3-20b01b2f050a">
    - 웹 사이트의 디렉터리 구조 중 흐릿한 글씨(회색 글씨)로 되어 있는 것과 선명한 글씨(검정색 글씨)로 되어 있는 것이 있음
        | 색상 | 설명 |
        |---|-----|
        | 검정색(선명한 글씨) | 해당 페이지를 직접 방문한 경우 |
        | 회색(흐릿한 글씨) | 방문한 페이지에 연결되어 있는 페이지 또는 디렉터리를 나타냄 |
    - 특정 페이지를 클릭하면 오른쪽 화면에서 해당 페이지에 대한 HTTP 헤더와 Request, Response 정보를 상세히 확인할 수 있음

<br/><br/>

## Burp Suite: Intruder 기능

