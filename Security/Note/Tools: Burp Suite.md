# Tools: Burp Suite

## Burp Suite 다운로드
1. JRE 다운로드 및 설치
    - Burp Suite은 기본적으로 Java를 기반으로 만들어졌기 때문에 Java 실행 환경이 구축되어 있어야 실행할 수 있음
        + Java 실행 환경 구축을 위해선 오라클 홈페이지에서 JRE를 다운로드하여 설치함

2. Burp Suite Community Edition을 검색하여 https://portswigger.net/burp/communitydownload 사이트에 접속함
    - DOWNLOAD 버튼 대신에 [Go straight to downloads] 버튼을 클릭하여 자신의 운영체제에 맞는 것을 다운로드함
      <img width="2560" alt="Burp Suite Download" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b793e190-933b-4cfd-9735-27a31fad770d">

3. 설치된 Burp Suite Community Edition을 실행함
    - 실행 시 Project와 Configuration은 변경하지 않고 Next와 Start Burp 버튼을 차례대로 클릭함
      <img width="2560" alt="Burp Suite 실행" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a3165c71-f97c-4131-99de-74dc30072a9f">
    - 실행 후 첫 화면은 다음과 같음
          <img width="2560" alt="Burp Suite 실행 후 화면" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f0444375-a0ce-4351-9e73-3ba71ebaab52">

<br/><br/>

## Burp Suite: Proxy 탭
* 웹 브라우저와 웹 서버 간의 HTTP 패킷(Request, Response)를 확인할 수 있음
    - [Intercept]에서 [Open Browser] 버튼을 누르고 임의의 웹 사이트에 접속하면 클라이언트의 Request 패킷과 서버의 Response 패킷을 볼 수 있음
    - 웹 프록시는 웹 해킹의 가장 기본이며, Burp Suite과 같은 웹 프록시 툴을 이용해 여러 웹 사이트를 살펴보면 HTTP 패킷에 어떤 내용이 있는지 알 수 있음

<br/>

### Proxy의 Intercept 기능 사용법
1. [Proxy]의 [Intercept]에서 \<Intercept is on\>으로 되어 있는지 확인하고 [⚙️ Proxy Settings]에서 다음의 두 항목이 체크되어 있는지 확인함
   <img width="2560" alt="Proxy 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/4fecc795-439d-4c46-9bf6-2f5d7bd68147">
    - \<Intercept is off\>로 되어 있는 경우 클릭하여 \<Intercept is on\>으로 변경함
    - Request/Response Interception rules에서 'Intercept requests/responses based on the following rules:' 항목에 체크함

2. 본인 컴퓨터의 시스템 설정에서 현재 접속 중인 네트워크의 프록시 설정함
    - 프록시를 통해 패킷을 전달할 수 있도록 하기 위해 설정함
    - macOS 기준
      <img width="2560" alt="Proxy 네트워크 설정" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/c94d9a3c-3e51-4694-a7d2-f9c9f046121e">

3. [Proxy]의 [Intercept]에서 [Open Browser] 버튼을 클릭한 후 임의의 웹 사이트(```www.hanb.co.kr```)에 접속함
    - [Proxy]의 [Intercept] 화면을 보면 클라이언트의 Request 패킷을 볼 수 있음
      <img width="2560" alt="Proxy Request" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/94248dbc-6398-4162-aa09-a28ed5e776d5">

<br/><br/>

## Burp Suite: Target 탭
* Burp Suite로 탐색할 사이트에 대한 정보를 확인할 수 있음 (웹 사이트 구조 분석 시 매우 유용함)
    - Burp Suite 실행 후 프록시를 설정한 다음 웹 사이트를 탐색하면 [Target] 탭 아래에 해당 사이트의 구조가 나타남

<br/>

### Target 기능 사용법
1. Burp Suite를 실행한 후 Burp Suite의 [Proxy] 탭에서 Burp Suite의 프록시 기능을 비활성화함(Intercept if off)
   <img width="2560" alt="Target 기본" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/4b64e73e-9444-47ff-a09f-2e3a6e2e5466">
    - 프록시 기능이 기본적으로 활성화돠어 있는데, 이 때문에 Target 탭의 [Open Browser]로 분석하고자 하는 웹 사이트에 접속할 때 페이지가 넘어가지 않고 멈춰있게 됨

<br/>

2. Target 탭에서 [Open Browser]로 브라우저를 열고 분석하고자 하는 웹 사이트에 접속함

<br/>

3. 웹 사이트에 접속한 후 메뉴나 게시글 등 다양한 부분을 모두 클릭하고 가능하다면 로그인도 시도해본 다음 [Target] 탭을 보면 아래와 같이 많은 수의 요청 목록을 볼 수 있음
   <img width="2560" alt="Target 웹 사이트 접속" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/61ac9b60-4219-4ec3-a6f7-c8e4de071427">
    - 웹 사이트의 구조를 분석할 때 해당 페이지에 포함된 링크 사이트까지 조사함 → 직접 방문한 페이지 외에도 링크되어 있는 페이지와 웹 사이트까지 분석할 수 있음

<br/>

4. 분석을 원하는 항목을 클릭하여 해당 웹 사이트의 구조를 확인함
   <img width="2560" alt="Target 웹 사이트 분석" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6fdecaac-3c6f-4b65-b38d-b0a2da3eb78e">
    - 웹 사이트의 디렉터리 구조 중 흐릿한 글씨(회색 글씨)로 되어 있는 것과 선명한 글씨(검정색 글씨)로 되어 있는 것이 있음
        | 색상 | 설명 |
        |---|-----|
        | 검정색(선명한 글씨) | 해당 페이지를 직접 방문한 경우 |
        | 회색(흐릿한 글씨) | 방문한 페이지에 연결되어 있는 페이지 또는 디렉터리를 나타냄 |
    - 특정 페이지를 클릭하면 오른쪽 화면에서 해당 페이지에 대한 HTTP 헤더와 Request, Response 정보를 상세히 확인할 수 있음


<br/><br/>

## Burp Suite: Intruder 기능

