# Cross Site Request Forgery(CSRF)

<br/>

> ### Cookie?
> 쿠키 → 이용자의 신원 정보가 포함되어 있어 일종의 서명과 같은 역할을 함
>	* 쿠키는 클라이언트에서 보내진 요청이 이용자로부터 왔으며, 이용자가 해당 요청에 동의했음을 의미함 <br/> → *요청에 이용자의 권한이 부여돼야 함을 의미함*
>	* 임의 이용자의 쿠키를 사용할 수 있다면, 임의 이용자의 권한으로 웹 서비스의 기능을 사용할 수 있음

<br/>

**Cross Site Request Forgery(CSRF, 교차 사이트 요청 위조)**: 임의 이용자의 권한으로 임의 주소에 HTTP 요청을 보낼 수 있는 취약점 <br/>
&nbsp; &nbsp; &nbsp; &nbsp; = 피해자가 *인지하지 못하는 상태*에서 피해자의 브라우저가 *특정 사이트에 강제적으로 요청(Request)를 보내도록* 하는 기법
&nbsp; &nbsp; &nbsp; &nbsp; = 사이트에 방문하는 사용자가 정상적인 요청이 아닌 임의의 요청을 하도록 **위조**하는 것
* 공격자는 임의 이용자의 권한으로 서비스 기능을 사용해 이득을 취할 수 있음
  | No | 예시 상황 |
  |----|----------|
  | 01 | 송금 기능을 이용해 공격자의 계좌로 임의 금액을 송금하여 금전적인 이득을 취함 |
  | 02 | 비밀번호를 변경해 임의 이용자의 계정을 탈취함 |
  | 03 | 관리자 계정을 공격해 공지사항 작성 등으로 혼란을 야기할 수 있음 |
  
* 이용자가 HTTP 요청을 보내도록 하는 방법은 다양함

<br/><br/>

### CSRF 공격 성공 조건
**공격자가 작성한 악성 스크립트**(HTTP 요청을 보내는 코드)**를 이용자가 실행**하는 것
	- 공격자가 이용자에게 메일을 보내거나 게시판에 글을 작성해 이용자가 이를 조회하도록 유도하는 방법 등이 있음
* 공격 과정
	1. 공격자가 취약한 웹 서버에 크로스 사이트 요청 변조 코드를 삽입함
	2. 아무것도 모르는 사용자가 해당 사이트를 방문하면 악성 스크립트가 동작함
		- 악성 스크립트는 사용자로 하여금 개인정보 수정이나 회원 탈퇴 등 원치 않는 임의의 행동을 수행하게 만듦


<br/><br/>

### CSRF 공격 스크립트
HTML 또는 Javascript를 통해 작성할 수 있음

<br/>

#### HTML로 작성한 CSRF 공격 스크립트
이미지를 불러오는 ```<img>``` 태그를 사용하거나 웹 페이지에 입력된 양식을 전송하는 ```<form>``` 태그를 사용하는 방법이 있음
* 두 태그를 이용해 HTTP 요청을 보내면 HTTP 헤더인 **Cookie에 이용자의 인증 정보가 포함됨**
* ```<img>```태그를 사용한 스크립트의 예시
	```html
	<img src=‘https://test.csrf-script.com/sendmony?to=attacker&amount=10000' width=0px height=0px>
	```
	- 임의 금액을 송금할 수 있는 기능을 이용해 공격자(attacker)의 계좌로 10000원을 송금하여 공격자가 금전적인 이득을 취하고 있음
	- 이미지의 크기를 줄일 수 있는 속성(width, height)를 이용해 이용자에게 들키지 않고 임의 페이지에 요청을 보내고 있음

<br/>

#### Javascript로 작성한 CSRF 공격 스크립트
새로운 창을 띄우고, 현재 창의 주소를 옮기는 등의 행위가 가능함
* javascript를 사용한 스크립트의 예시
  ```javascript
  // 새로운 창 띄우기
  window.open(‘https://test.csrf-script.com/sendmony?to=attacker&amount=10000'');
  
  // 현재 창 주소 옮기기
  location.href = ‘https://test.csrf-script.com/sendmony?to=attacker&amount=10000'
  location.replace(‘https://test.csrf-script.com/sendmony?to=attacker&amount=10000');
  ```
  - 입력되 주소에 대해 새로운 창을 띄우거나 현재 창 주소를 옮기는 방법을 이용하여 공격자(attacker)의 계좌로 10000원을 송금하여 공격자가 금전적인 이득을 취하고 있음

<br/><br/>

> ### XSS와 CSRF의 차이
> XSS와 CSRF는 **스크립트를 웹 페이지에 작성해 공격한다는 점**에서 매우 유사함
> * 공통점
>	  - 모두 클라이언트를 대상으로 하는 공격이며, 이용자가 악성 스크립트가 포함된 페이지에 접속하도록 유도해야 함
> * 차이점 → *공격에 있어 서로 다른 목적을 가짐*
> 
>	  | 공격 | 차이점 |
>   |----|----------|
>	  | XSS | 인증 정보인 세션 및 쿠키 **탈취**를 목적으로 하는 공격 <br/> → 공격할 사이트의 오리진에서 스크립트를 실행시킴 |
>	  | CSRF | 이용자가 임의 페이지에 **HTTP 요청**을 보내는 것을 목적으로 하는 공격 <br/> → 공격자는 악성 스크립트가 포함된 페이지에 *접근한 이용자의 권한으로 웹 서비스의 임의 기능을 실행할 수 있음* |


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking] 📌 [ClientSide: CSRF](https://dreamhack.io/lecture/courses/172)
