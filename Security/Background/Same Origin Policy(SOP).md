# Same Origin Policy(SOP)

## 동일 출처 정책(Same Origin Policy, SOP) 보안 메커니즘
### 탄생 배경
* 인증 상태를 나타내는 민감한 정보가 들어있는 쿠키가 브라우저 내부에 저장됨 → 브라우저가 웹 서비스에 접속할 때 자동으로 쿠키를 Cookie 헤더에 포함시켜 요청을 전송함
* 악의적인 페이지가 클라이언트의 권한을 이용해 대상 사이트에 HTTP 요청을 보내고, HTTP 응답 정보를 획득하는 코드를 실행할 수 있음
  - *클라이언트 입장에서는 가져온 데이터를 악의적인 페이지에서 읽을 수 없도록 해야 함* → **SOP 보안 메커니즘**

<br/>

### SOP의 Origin 구분 방법
* Origin: 브라우저가 가져온 정보의 출처<br/><br/>
  <img width="549" alt="Origin" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/4e6a85b4-81a2-4883-8717-5fd0a7f03898"><br/>
  - 프로토콜(Protocol, Scheme), 포트(Port), 호스트(Host)로 구성됨
  - **구성 요소가 모두 일치**해야 동일한 출처(Same Origin)로 판단함

* SOP - Cross Origin이 아니라 *Same Origin일 때만 정보를 읽을 수 있도록 해줌*
	| 상황 | 결과 |
	|----------------|-----|
	| 동일 출처(Same Origin)에서 데이터를 읽는 것을 시도 | 성공 |
	| 동일 출처(Same Origin)에서 데이터를 쓰는 것을 시도 | 성공 |
	| 외부 출처(Cross Origin)에서 데이터를 읽는 것을 시도 | 오류 발생(실패) |
	| 외부 출처(Cross Origin)에서 데이터를 쓰는 것을 시도 | 성공 |

<br/><br/>

## Cross Origin Resource Sharing (CORS)

### CORS 탄생 배경
* 브라우저가 SOP에 구애받지 않고 외부 출처에 대한 접근을 허용해주는 경우가 존재함
	- 리소스를 불러오는 <img>, <style>, <script> 등의 태그들은 SOP의 영향을 받지 않음
* 이 외에도 *웹 서비스에서 SOP를 완화하여 다른 출처의 데이터를 처리해야 하는 경우*도 존재함
	- Cross Origin에서 자원을 공유하기 위해 사용할 수 있는 방법
		+ **교차 출처 리소스 공유(Cross Origin Resource Sharing, CORS)**: CORS와 관련된 HTTP 헤더를 추가하여 전송하는 방식
		+ **JSON with Padding(JSONP) 방식**

<br/>

### 교차 출처 리소스 공유 (CORS)
HTTP 헤더에 기반하여 Cross Origin 간에 리소스를 공유하는 방법
	- 발신 측에서 CORS 헤더를 설정해 요청 → 수신 측에서 헤더를 구분해 정해진 규칙에 맞게 데이터를 가져갈 수 있도록 설정함

#### CORS preflight - **수신 측에 웹 리소스를 요청해도 되는지** 질의하는 과정
* 발신 측에서 HTTP 요청을 보낼 때, OPTIONS 메소드를 가진 HTTP 요청을 먼저 전달함
	- 요청에 포함된 “Aceess-Control-Request”로 시작하는 헤더
		| 헤더 | 설명 |
		|------------|------------------|
		| Access-Control-Request-Method | 값으로 준 메소드를 추가적으로 사용할 수 있는지 질의함 |
		| Access-Control-Request-Headers | 값으로 준 헤더를 추가적으로 사용할 수 있는지 질의함 |		
	- 질의에 대한 서버의 응답에 포함된 헤더
		| 헤더 | 설명 |
		|------------|------------------|
		| Access-Control-Allow-Origin | 헤더 값에 해당하는 Origin에서 들어오는 요청만 처리함 |
		| Access-Control-Allow-Methods | 헤더 값에 해당하는 메소드의 요청만 처리함 |
		| Access-Control-Allow-Credentials | 쿠키 사용 여부를 판단함 (true/false) |
		| Access-Control-Allow-Header | 헤더 값에 해당하는 헤더의 사용 가능 여부를 나타냄 |
* 해당 과정을 마치면 수신 측의 응답이 발신 측의 요청과 상응하는지 판단한 후 상응한다면 수신 측의 웹 리소스를 요청하는 HTTP 요청을 보냄

<br/>

### JSON with Padding(JSONP)
* *이미지, 자바스크립트, CSS 등의 리소스는 SOP에 구애받지 않고 외부 출처(Cross Origin)에 대해 접근을 허용한다는 점*을 이용해 **<script> 태그**로 Cross Origin의 데이터를 불러옴
	- 요청 시 callback 파라미터에 어떤 함수로 받아오는 데이터를 핸들링할 지 넘겨줌 → 대상 서버는 전달된 Callback으로 데이터를 감싸 응답함
		+ <script> 태그 내의 데이터는 자바스크립트 코드로 인식되기 때문에 CallBack 함수를 활용함
	- CORS가 생기기 전에 사용하던 방법으로 현재는 거의 사용하지 않는 추세임 → CORS 사용을 권장함
* JSONP 예시 코드
	- 웹 리소스 요청 코드
		```html
		<script>
			function CallbackFunc(data) {
				console.log(data.id)
			}
		</script>
	
		<script src=‘https://sample.com/whoami?callback=CallbackFunc'></script>
		```
		+ ```https://sample.com``` 스크립트를 로드하는 HTML 코드 (callback 파라미터를 CallbackFunc으로 지정함으로써 **수신 측에게 CallbackFunc 함수를 사용해 수신 받겠다고 알림**)
		+ *반환된 코드는 요청 측에서 실행*되기 때문에 CallbackFunc 함수가 Cross Origind에서 전달된 데이터를 읽을 수 있음
	- 웹 리소스 요청에 따른 응답 코드
		```javascript
		CallbackFunc({‘id’: ‘augustf’});
		```
		+ 수신 측은 CallbackFunc 함수를 통해 요청 측에 데이터를 전달함

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking] 📌 [Mitigation: Same Origin Policy](https://dreamhack.io/lecture/courses/186)
