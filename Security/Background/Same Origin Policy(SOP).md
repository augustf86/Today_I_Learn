# Same Origin Policy(SOP)

## 동일 출처 정책(Same Origin Policy, SOP) 보안 메커니즘
### 탄생 배경
* 인증 상태를 나타내는 민감한 정보가 들어있는 쿠키가 브라우저 내부에 저장됨 → 브라우저가 웹 서비스에 접속할 때 자동으로 쿠키를 Cookie 헤더에 포함시켜 요청을 전송함
* 악의적인 페이지가 클라이언트의 권한을 이용해 대상 사이트에 HTTP 요청을 보내고, HTTP 응답 정보를 획득하는 코드를 실행할 수 있음
  - *클라이언트 입장에서는 가져온 데이터를 악의적인 페이지에서 읽을 수 없도록 해야 함* → SOP 보안 메커니즘

<br/>

### SOP의 Origin 구분 방법
* Origin: 브라우저가 가져온 정보의 출처
  - 프로토콜(Protocol, Scheme), 포트(Port), 호스트(Host)로 구성됨
  - **구성 요소가 모두 일치**해야 동일한 출처(Same Origin)으로 판단함

* SOP - Cross Origin이 아니라 Same Origin일 때만 정보를 읽을 수 있도록 해줌

<br/><br/>

## Cross Origin Resource Sharing (CORS)

### CORS 탄생 배경
* 브라우저가 SOP에 구애받지 않고 외부 출처에 대한 접근을 허용해주는 경우가 존재함
	- 리소스를 불러오는 <img>, <style>, <script> 등의 태그들은 SOP의 영향을 받지 않음
* 이 외에도 *웹 서비스에서 SOP를 완화하여 다른 출처의 데이터를 처리해야 하는 경우*도 존재함
	- Cross Origin에서 자원을 공유하기 위해 사용할 수 있는 방법
		+ **교차 출처 리소스 공유(Cross Origin Resource Sharing, CORS)**: CORS와 관련된 HTTP 헤더를 추가하여 전송하는 방식
		+ **JSON with Padding(JSONP) 방식**

