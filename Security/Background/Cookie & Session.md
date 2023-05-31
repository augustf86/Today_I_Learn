# Cookie & Session
웹 서버가 수많은 클라이언트를 구별하고 클라이언트 별로 서로 다른 결과를 반환하기 위해서 헤더에 클라이언트의 정보와 요청의 내용을 구체화하는 등의 데이터를 포함하는데, 이 헤더에 클라이언트의 인증 정보 또한 포함될 수 있음
* 클라이언트의 인증 정보 → **Cookie**(쿠키)와 **Session**(세션)에 포함됨

<br/>

### Cookie와 Session가 필요한 이유
* 클라이언트의 IP 주소와 User-Agent → 매번 변경될 수 있는 고유하지 않은 정보
* HTTP 프로토콜의 Connectionless와 Stateless 특징 → **웹 서버는 클라이언트를 기억할 수 없음**
	- Connectionless: 하나의 요청에 하나의 응답을 한 후 연결을 종료하는 것
		+ 새 요청이 있을 때마다 항상 새로운 연결을 맺는다는 것을 의미함
	- Stateless: 통신이 끝난 후 상태 정보를 저장하지 않는 것
		+ 이전 통신에서 사용한 데이터를 다른 통신에서 요구할 수 없음을 의미함

<br/>

## Cookie
* Key와 Value로 이뤄진 일종의 단위(사용자가 인터넷 웹 사이트를 방문할 때 생성되는 4KB 이하의 파일)
	- 쿠키를 통해 상태를 유지하는 방법
		1. 서버가 클라이언트에게 쿠키를 발급
		2. 클라이언트는 서버에 요청을 보낼 때마다 쿠키를 같이 전송
		3. 서버는 클라이언트의 요청에 포함된 쿠키를 확인해 클라이언트를 구분할 수 있음(클라이언트 신분 확인 가능)
* 쿠키의 용도 - 클라이언트의 정보 기록과 상태 정보를 표현하는 용도로 사용함
	- 정보 기록
		+ 팝업 창 표시 여부를 쿠키를 통해 판단함
		+ 전자상거래의 장바구니 시스템
		+ 웹 사이트 이용 방식 추적: 쿠키를 사용해 사용자의 방문 정보를 추적함
		+ 타깃 마케팅 - 쿠키를 이용해 광고 시청 수, 선호 광고 등을 파악할 수 있음
	- 상태 정보
		+ 클라이언트를 식별할 수 있는 값을 쿠키에 저장함 → 사용자의 ‘성향’까지 파악하여 사용자별 성향을 사이트에 그대로 반영할 수 있음
* 쿠키의 특징
	- 클라이언트에 저장됨 → 클라이언트는 저장된 쿠키를 **조회﹒수정﹒추가**할 수 있음
	- 클라이언트의 요청 헤더에 넣어 전송함 → 이용자가 요청을 보낼 때 **쿠키 헤더를 변조**할 수 있음
	- 쿠키 설정 시 **만료 시간**을 지정할 수 있음
		+ 만료 시간 이후에는 클라이언트에서 쿠키가 삭제됨
		+ 쿠키의 만료는 클라이언트(브라우저)에서 관리됨
	- 서버와 클라이언트 둘 다 쿠키를 설정할 수 있음

<br/>

### 쿠키 변조
쿠키 = 클라이언트의 브라우저에 저장되고 요청에 포함되는 정보 → 악의적인 클라이언트(공격자)는 쿠키 정보를 변조해 서버에 요청을 보낼 수 있음
* 서버가 **별다른 검증 없이** 쿠키를 통해 이용자의 정보를 식별하는 경우 공격자가 **타 이용자를 사칭**해 정보를 탈취할 수 있음

<br/>

### 쿠키를 웹 페이지에 적용할 수 있는 방법
* 서버 *(일반적으로 서버가 쿠키를 설정함)*
	- HTTP 응답 중 헤더에 **쿠키 설정 해더**(Set-Cookie)를 추가하면 클라이언트의 브라우저가 쿠키를 설정함
	- 서버가 설정한 쿠키의 사용 단계
		1. **사용자 컴퓨터에 쿠키를 생성함** (사용자가 사이트를 방문하면 사용자 컴퓨터에 쿠키를 만듦)
			+ 윈도우 11 버전 이후부터는 Internet Cookie API로 쿠키에 접근하기 때문에 사용자 컴퓨터에서 쿠키 파일을 저장하지 않음
			+ 쿠키를 만든 사이트의 도메인 이름, 그 사이트를 구분하는 숫자, 쿠키 만기일 등의 정보기 있음
		2. **사용자 컴퓨터의 쿠키를 웹 서버에 전송** (웹 서버에 접속할 때 사용자 컴퓨터에 있는 쿠키를 웹 서버로 전송함)
			+ 이전에 방문했던 사이트에 재접속하면 사용자 컴퓨터에 저장된 쿠키를 통해 방문자의 개인정보를 알 수 있음 
* 클라이언트
	- 자바스크립트를 사용해 쿠키를 설정함
		```javascript
		document.cookie = “name=alice;”
		document.cookie = “age=25; Expires=Fri, 30 Sep 2022 14:51:50 GMT;”
		```

<br/>

### 클라이언트에서 쿠키를 다루는 방법 (macOS, Chrome 기준)
* 크롬 Console을 활용하는 방법 - 크롬 개발자 도구의 Console 탭에서 **document.cookie**를 입력하면 쿠키 정보를 확인할 수 있음<br/>
	<img width="771" alt="Cookie - Console" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9eb20c0e-6515-4ef2-bbb5-3fbda6dcae78">
 
* 크롬 Application을 활용하는 방법: 크롬 개발자 도구의 Application 탭의 왼쪽 목록에서 Cookies를 펼치면 Origin 별로 설정된 쿠키 정보를 확인/수정할 수 있음<br/>
	<img width="779" alt="Cookie - Application" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f87bf7df-f4ce-4f7c-afc2-2127b3166b00">

<br/><br/>

## Session
* 인증 정보를 서버에 저장하고 해당 데이터에 접근할 수 있는 키(유추할 수 없는 랜덤한 문자열, **Session ID**)를 만들어 클라이언트에 전달하는 방식으로 작동함
	- 쿠키에 인증 상태를 저장하지만 클라이언트가 인증 정보를 변조할 수 없게 하기 위해 사용함
	- 세션 사용 방식
		+ 브라우저는 Session ID를 쿠키에 저장하고 이후에 **HTTP 요청을 보낼 때 Cookie 헤더에 이 값을 넣음**
		+ 서버는 요청에 포함된 Session ID에 해당하는 데이터를 가져와 인증 상태를 확인함

<br/>

* 쿠키와 세션의 차이점
	| 종류 | 차이점 |
	|-----------|---------------|
	| 쿠키 | 데이터 자체를 이용자가 저장하고, 요청의 Cookie 헤더에 해당 값을 넣어서 전달함 |
	| 세션 | 데이터를 서버가 저장하고, 요청에 Session 값을 넣고 요청을 전송하면 서버의 <br/>Session Storage(DB)에서 해당 키를 가지고 있는 데이터를 가져옴 |
