# Background: Web

## Web
### 웹(World Wide Web, W3, Web)의 정의
* 사전적 정의 - 인터넷에 연결된 컴퓨터들이 하이퍼텍스트 형식으로 표현된 다양한 정보를 효과적으로 이용할 수 있도록 구현한 전세계적인 시스템
  - 시간과 장소에 구애받지 않고 인터넷에 접속할 수만 있다면, 누구나 웹에서 정보를 구하거나 공유할 수 있음
* 기술적 정의 - 인터넷을 기반으로 구현된 서비스 중 HTTP를 이용하여 정보를 공유하는 서비스
  - HTTP: 웹 상에서 서로 통신을 하기 위해 정해둔 일종의 규칙(프로토콜)
  - 웹을 이용하는 컴퓨터들을 정보를 제공하는 주체인 **웹 서버**(Web Server)와 정보를 받는 이용자인 **웹 클라이언트**(Web Client)로 구분할 수 있음

<br/>

### 웹 보안의 중요성
* 웹의 과거와 현재 비교
  - 과거 - 저장된 문서의 내용을 출력해 이용자에게 제공하는 간단한 서비스 → 이용자가 요청하는 정보를 제공하기만 하는 수동적인 형태의 서비스
    + 단순한 텍스트와 링크 위주로 이루어진, 단순히 하이퍼링크(Hyperlink)로 여러 페이지를 묶어넣은 정도
  - 현재 - 다양한 분야에서 이용자에게 편의를 주는 복잡한 서비스로 진화함 → 이용자의 요청을 해석하고 가공하여 필요한 정보와 기능을 제공하는 능동적인 형태의 서비스에 가까움
    + 웹에서 처리하는 정보 자산들이 많아짐에 따라 이 자산들을 안전하게 보관하고 처리해야 할 필요성도 함께 증가하였음
    + 웹을 통한 정보 교환 과정에서 **민감한 정보들이 유출되거나 악용되지 않도록 보호**하는 웹 보안의 중요성이 대두하고 있음

<br/>

### 웹 서비스
* **프론트엔드(Front-end)**: 이용자의 요청을 받는 부분 (직접 보여지는 부분)
  - 웹 리소스(Web Resource)로 구성됨 → 페이지가 보여지고 있는 정보들은 모두 웹 리소스에 명시되어 있음
    + **공격자가 쉽게 변조할 수 있는 대상**이므로 좀 더 주의깊게 살펴봐야 함
* **백엔드(Back-end)**: 이용자의 요청을 처리하는 부분
  - 지금은 사용자가 입력한 값에 따라 다양한 결과를 화면에 보여주는 동적 기능도 제공하고 있음
  - 백엔드(서버)에서 제공하는 기능 - 서버 측 스크립트 언어, 웹 서버, 데이터베이스
    + 서버 측 스크립트 언어: 클라이언트가 요청한 데이터를 서버 측에서 처리하여 원하는 결과를 돌려주기 위해 사용함 *→ 소스 코드 분석 및 취약점 발견을 위해 스크립트 언어에 대한 이해가 필요*
    + 데이터베이스: 주로 많은 수의 데이터를 저장하거나 데이터 변경이 자주 일어나는 업무에 사용함

<br/>

### 웹 리소스
* **모든 웹 리소스는 고유의 Uniform Resource Indicator(URI)를 가지며, 이를 이용하여 식별함**
  - http://sample.com/index.html 주소 → sample.com에 존재하는 /index.html 경로의 리소스를 가져오라는 의미
* 종류
  - **Hyper Text Markup Language(HTML)** - 웹 문서의 기본적인 뼈대를 구성함
    + 태그와 속성을 통해 구조화된 문서 작업을 지원함
    + 현재 **HTML5**가 표존으로 확정되었고, 지속적으로 발전하고 있음
  - **Cascading Style Sheet(CSS)** - 웹 문서의 외관(디자인)을 담당함
    + 웹 리소스들의 시각화 방법을 기재한 스타일 시트
    + 브라우저는 CSS를 참고하여 웹 문서를 시각화함
  - **JavaScript(JS)** - 웹 문서의 동작을 구현함
    + 이용자의 브라우저에서 실행되는 코드 → JS를 Client-Side Script라고도 부름
    + 성능 문제로 인해 서버 측에서 처리하지 않는 부분을 클라이언트 측에서 처리할 때 주로 사용함
  - 문서, 이미지, 동영상, 폰트 등

<br/>

### 웹 클라이언트와 서버의 통신 과정
1. (클라이언트) 이용자가 브라우저를 이용해 웹 서버에 전송함
2. (클라이언트) 브라우저는 이용자의 요청을 해석하여 HTTP 형식으로 웹 서버에 리소르를 요청함 (HTTP Request)
3. (서버) HTTP로 전달한 이용자의 요청을 해석함
4. (서버) 해석한 이용자의 요청에 따라 적절한 동작을 수행함 → 리소스 요청 시 이를 탐색하고, 복잡한 동작을 요구하는 경우 내부적으로 필요한 연산을 수행하기도 함
6. (서버) 이용자에게 전달할 리소스를 HTTP 형식으로 이용자에게 전달함 (HTTP Response)
7. (클라이언트) 브라우저는 서버에게 응답받은 웹 리소스를 시각화하여 이용자에게 보여줌

<br/><br/>

## HTTP/HTTPS
### 통신 프로토콜
프로토콜(Protocol): 컴퓨터 간에 정보를 원활하게 교환하기 위해 정한 여러 가지 통신 규칙과 방법에 대한 약속 또는 규약
* 인터넷에서의 프로토콜: 두 컴퓨터 간의 서로 다른 방식을 연결하여 통신할 수 있도록 정한 공통의 규약
* 프로토콜의 3가지 구성 요소
	- 구문(Syntax): 데이터의 형식이나 신호 → 부호나 방법을 정의함
	- 의미(Semantics): 정확한 정보 전송을 위한 전송 제어와 오류 제어 방법을 정의함
	- 순서(Timing): 송신자와 수신자 간 혹은 양단(End-to-End)의 통신 시스템, 망 사이의 통신 속도 및 순서를 정의함
* 대표적인 예시 - 네트워크 통신의 기초가 되며 가장 많이 사용하는 **TCP/IP**, 웹 애플리케이션이 사용하는 **HTTP**, 파일을 주고 받을 때 사용하는 **FTP** 등
* 프로토콜에 대한 상세한 설명은 **RFC(Request for Comments)** 문서를 통해 공개됨
	- RFC: 인터넷에서 기술을 구현하는 데 필요한 상세 절차와 기본 틀을 제공하는 기술 관련 문서

<br/>

### HTTP (Hyper Text Transfer Protocol)
HTTP: 서버와 클라이언트의 데이터 교환을 *요청(Request)*과 *응답(Response)* 형식으로 정의한 프로토콜
→ 문서 간의 상호 연결을 통해 다양한 텍스트, 그래픽, 애니메이션을 화면에 보여주고 사운드를 재생해주는 역할을 함.
* 기본 메커니즘 - **”클라이언트가 서버에 요청 → 서버가 이에 응답”**
* 기본 연결 (index.html를 요청하는 경우)
	1. **Connect** [Client → Server]: 클라이언트가 웹 브라우저를 이용하여 서버에 연결을 요청하면 서버는 요청한 클라이언트에 대해 서비스를 준비함
	2. **Request(index.html)** [Client → Server]: 클라이언트가 읽고자 하는 문서를 서버에 요청함
	3. **Response(index.html)** [Server → Client]: 서버는 그 문서를 클라이언트로 전달
	4. **Close**: 연결을 종료함

<br/>

HTTP 메시지
* 클라이언트가 전송하는 **HTTP 요청**(Request)와 서버가 반환하는 **HTTP 응답**(Response)가 있음
* HTTP 메시지의 구성
	- HTTP 헤드(Head)
		+ 각 줄은 CRLF로 구분
		+ 첫 줄 = 시작줄(Start-line) → 요청과 응답에서 하는 역할에 큰 차이가 있음
		+ 나머지 줄 = 헤더(header) → 필드와 값으로 구성되며 HTTP 메지시 혹은 바디의 속성을 나타냄
		+ 헤드의 끝은 CRLF 한 줄로 나타냄
	- HTTP 바디(Body)
		+ 헤드의 끝을 나타내는 CRLF 뒤의 모든 줄을 의미
		+ 클라이언트나 서버에게 전송하려는 데이터가 바디에 담김

<br/>

**HTTP 요청(Request)**: 서버에게 특정 동작을 요구하는 메시지
<img width="1097" alt="HTTP Request Example(naver com)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/373ccedf-23c2-4318-96f9-072d76257c2e">
* 시작 줄 - **메소드(Method), 요청 URI(Request-URI), HTTP 버전**으로 구성 (각각은 띄어쓰기로 구분됨)
	- 메소드: URI가 가리키는 리소스를 대상으로 서버가 수행하길 바라는 동작(= HTTP 전송 방법)을 나타냄
		+ *8개의 메소드 중 GET, POST 메소드가 비교적 자주 사용됨*
		+ 참고: [HTTP Method](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Supplement/HTTP%20Method%20%26%20Status%20Code.md)
	- 요청 URI: 메소드의 대상
	- HTTP 버전: 클라이언트가 사용하는 HTTP 프로토콜의 버전
* 헤더와 바디
	- 웹 해킹과 관련된 Request 헤더의 주요 속성
		+ Cookie 헤더: 서버가 클라이언트에 전송한 인자값에 추가 정보를 보낼 때 사용함
		+ Host 헤더: URL 주소에 나타난 호스트명을 자세하게 나타내기 위해 사용됨
		+ User-Agent 헤더: 브라우저나 기타 클라이언트의 소프트웨어 정보를 보여줌

<br/>

**HTTP 응답(Response)**: HTTP 요청에 대한 결과를 반환하는 메시지
<img width="1030" alt="HTTP Response Example(naver)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f7c35151-c730-45d5-a506-79358853e32a">
* 시작 줄: **HTTP 버전**, **상태코드(Status Code)**, **처리 사유**(Reason Phrase)로 구성 (각각은 띄어쓰기로 구분됨)
	- HTTP 버전: 서버에서 사용하는 HTTP 프로토콜의 버전
	- 상태 코드: 요청에 대한 처리 결과를 세 자릿수로 나타냄
		+ 참고: [HTTP Status Code](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Supplement/HTTP%20Method%20%26%20Status%20Code.md)
	- 처리 사유: 상태 코드가 발생한 이유를 짧게 기술한 것
* 헤더와 바디



### HTTPS (HTTP over Secure socket layer)
* HTTP의 문제점 - **응답과 요청을 평문으로 전달**하는 점
	- 데이터가 네트워크 장비를 통과할 때 **암호화되지 않은 단순한 TCP를 사용**함 → 네트워크에 잠임해 있는 *공격자가 중간에 정보를 가로챌 수 있음*
* HTTPS - **TLS(Transport Layer Security) 프로토콜**을 도입하여 HTTP의 문제점을 해결함
	- TLS - 서버와 클라이언트 사이에 오가는 모든 HTTP 메시지를 암호화함
		+ 공격자가 중간에 메시지를 탈취하더라도 이를 해석하는 것이 불가능한 → 결과적으로 HTTP 통신이 도청과 변조로부터 보호됨
	-공격자가 **중간에서 스니핑을 하기는 어렵게 만들지만**, 웹 애플리케이션 해킹의 주요 원인인 **사용자 입력값에 대한 검증은 하지 못함**
	- 현재 개발되는 서비스들은 규모에 상관없이 HTTPS를 사용하는 추세
* HTTP와 HTTPS의 구분
	- 웹 서버의 URL이 " http:// "로 시작 → HTTP 프로토콜을 사용함
	- 웹 서버의 URL이 " https:// "로 시작 → HTTPS 프로토콜을 사용

<br/><br/>

## Web Browser
**웹 브라우저(Web Browser)**: 서버와 HTTP 통신을 대신해주고, 수신한 리소스를 시각화하여 일반 이용자가 편리하게 인터넷을 이용할 수 있도록 하는 소프트웨어
* 이용자가 주소창에 google.com을 입력했을 때 웹 브라우저의 동작
	1. 웹 브라우저의 주소창에 입력된 주소(google.com)을 해석 (**URL 분석**)
	2. google.com에 해당하는 주소 탐색 (**DNS 요청**)
	3. HTTP를 통해 google.com에 요청
	4. 리소스 다운로드 및 웹 랜더링
* 브라우저는 보안과 개발에 필요한 다양한 도구들도 제공하고 있음

<br/>

### URL(Uniform Resource Locator)
URL은 웹에 있는 리소스의 위치를 표현하는 문자열로, 브라우저로 특정 웹 리소스를 서버에 요청할 때 사용함
<img width="647" alt="URL" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/14b8ee3e-4e07-4cee-a7ec-90c5846fefdf">
* Scheme, Authority(Userinfo, Host, Port), Path, Query, Fragment 등으로 구성됨
	| 구성 요소 | 설명 |
	|-------|----------------------------|
	| Scheme | 웹 서버와 어떤 프로토콜로 통신할 지 나타냄 |
	| Host | <Authority의 일부> 접속할 웹 서버의 주소에 대한 정보를 가지고 있음 |
	| Port | <Authority의 일부> 접속할 웹 서버의 포트에 대한 정보를 가지고 있음 |
	| Path | 접근할 웹 서버의 리소스 경로를 ‘/‘로 구분함 |
	| Query | 웹 서버에 전달하는 파라미터, URL에서 ‘?’ 뒤에 위치함 |
	| Fragment | 메인 리소스에 존재하는 서브 리소스에 접근할 때 이를 식별하기 위한 정보를 담고 있음, ‘#’ 문자 뒤에 위치함 |

<br/>

### Domain Name
URL의 구성 요소 중 Host - **Domain Name**, **IP Address**의 값을 가질 수 있음
* IP Address(IP 주소): 네트워크 상에서 통신이 이루어질 때 장치를 식별하기 위해 사용되는 주소
	- IPv4의 경우 32비트의 값을, IPv6의 경우 128비트의 값을 가지며 불규칙한 숫자로 이루어져 있어 사람이 외우기 어려움 → *일반적으로 Domain Name을 정의하여 IP 주소 대신에 사용함*
* Domain Name: 도메인의 특성을 담은 이름
	- Domain Name을 Host 값으로 이용할 때, *브라우저는 Domain Name Server(DNS)에 Domain Name을 질의하고 DNS가 응답한 IP Address를 사용함*
	- Domain Name 정보를 확인하는 방법
		+ MacOS/Linux/Windows에서 **nslookup 명령어**를 사용해 확인할 수 있음 <br/>
			<img width="422" alt="nslookup" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/1676689e-f922-42b2-92a8-c54ccfbdab58">

<br/>

### Web Rendering
웹 랜더링: 서버로부터 받은 리소스를 이용자에게 **시각화**하는 행위
* 서버의 응답을 받은 웹 브라우저는 리소스의 타입을 확인하고, 적절한 방식으로 이용자에게 전달함
* 웹 랜더링 엔진에 의해서 웹 랜더링 과정이 이루어짐
	- 브라우저별로 서로 다른 엔진을 사용함
		| 브라우저 | 웹 렌더링 엔진 |
		|------|-----------------|
		| 사파리 | 웹킷(Webkit) |
		| 크롬 | 블링크(Blink) |
		| 파이어폭스 | 개코(Gecko) |
	- 각각의 엔진에 따라 렌더링 과정과 순서, 속도의 차이는 있지만, **HTML을 파싱하고 시각화하여 이용자에게 보여주는 것**은 동일함

<br/>

### Browser: Devtools
* 브라우저 개발자 도구(Devtools)의 기능
	- HTML, CSS 코드를 브라우저에서 수정하고 결과를 바로 확인할 수 있음
	- 자바스크립트 코드를 대상으로 디버거도 제공함
	- 서버와 오가는 HTTP 패킷도 상세히 보여줌 → 프로토콜 상에서 발생하는 문제도 쉽게 발견할 수 있음
* 크롬, 사파리, 엣지, 파이어폭스 모두 개발자 도구를 포함하고 있으며, 각각의 인터페이스는 조금씩 다르지만 기능은 흡사함
	- macOS 환경의 크롬 브라우저 사용 시 개발자 도구 실행 방법
		+ 브라우저를 열고 *option + command + I* 키를 누르거나 '보기 - 개발자 정보 - 개발자 도구'를 클릭함
		+ 개발자 도구의 주요 패널
		  <img width="777" alt="크롬 개발자 도구" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/841f159b-1d98-42d8-b82d-000cd8f8ec84">
			| 패널 | 설명 |
			|--------|-------------------------|
			| Elements | 페이지를 구성하는 HTML 검사 |
			| Console | 자바스크립트를 실행하고 결과를 확인할 수 있음 |
			| Sources | HTML, CSS, JS 등 페이지를 구성하는 리소스를 확인하고 디버깅할 수 있음 |
			| Network | 서버와 오가는 데이터를 확인할 수 있음 |
			| Application | 쿠키를 포함하여 웹 애플리케이션과 관련된 데이터를 확인할 수 있음 |

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking] 📌 [Background: Web](https://dreamhack.io/lecture/courses/170)
* [드림핵 Web Hacking] 📌 [Background: HTTP/HTTPS](https://dreamhack.io/lecture/courses/199)
* [드림핵 Web Hacking] 📌 [Background: Web Browser](https://dreamhack.io/lecture/courses/171)
* [드림핵 Web Hacking] 📌 [Tools: Browser DevTools](https://dreamhack.io/lecture/courses/198)
