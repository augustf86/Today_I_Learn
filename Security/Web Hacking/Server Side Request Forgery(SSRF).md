# Server Side Request Forgery(SSRF)

웹 서비스의 요청을 변조하는 취약점
* CSRF와 SSRF의 비교
    | CSRF | SSRF |
    |---|---|
    | 브라우저가 변조된 요청을 보냄 | 웹 서비스의 권한으로 변조된 요청을 보냄 |
* 웹 애플리케이션의 요청을 변조할 수 있기 때문에 상황에 따라 매우 치명적인 취약점이 될 수 있음

<br/>

## SSRF 발생 배경
단일 서비스로 구현할 수 있었던 과거의 웹 서비스와 달리, 최근의 웹 서비스는 지원하는 기능이 증가함에 따라 구성요소가 증가했음
* 관리 및 코드 복잡도를 낮추기 위해 **마이크로서비스**들로 웹 서비스를 구현하는 추세임
    - 마이크로서비스: 소프트웨어가 잘 정의된 API를 통해 통신하는, 소규모의 독립적인 서비스로 구성되어 있는 경우의 소프트웨어 개발을 위한 아키텍처 및 조직적 접근 방식
        + 주로 HTTP, GRPC 등을 사용해 API 통신을 함
* 서비스 간 HTTP 통신이 이루어질 때 요청 내에 이용자의 입력값이 포함될 수 있음
    - 이용자의 입력값이 포함될 경우 개발자가 의도하지 않은 요청이 전송될 수 있음
* 대다수의 서비스들이 마이크로서비스들로 구조를 많이 바꾸고 새롭게 개발하는 추세임 → SSRF 취약점이 파급력이 더욱 높아지고 있음

<br/>

## SSRF 발생 상황
* 웹 서비스는 외부에서 접근할 수 없는 내부망의 기능을 사용할 때가 있음
    - 대표적인 내부망의 기능 = 백오피스 서비스
        + 백오피스 서비스(관리자 페이지): 관리자만이 수행할 수 있는 모든 기능을 구현한 서비스
            - 관리자만 이용할 수 있어야 하기 때문에 외부에서 접근 불가능한 내부망에 위치함
        + 웹 서비스는 의심스러운 행위를 탐지하고 실시간으로 대응하기 위해 백오피스의 기능을 실행할 수 있음
            - 외부에서 직접 접근할 수 없는 내부망 서비스와 통신할 수 있다는 의미
            - SSRF 취약점을 통해 웹 서비스의 권한으로 요청을 보낼 수 있다면 외부에서 간접적으로 내부망 서비스를 이용할 수 있음
* 웹 서비스가 보내는 요청을 변조하기 위해서는 **요청 내에 이용자의 입력값이 포함**되어야 함
    | No. | SSRF 취약점이 발생하는 경우 |
    |--|-----|
    | 1 | 웹 서비스가 이용자가 입력한 URL에 요청을 보내는 경우 |
    | 2 | 웹 서비스의 요청 URL에 이용자의 입력값이 포함되는 경우 |
    | 3 | 웹 서비스의 요청 Body에 이용자의 입력값이 포함되는 경우 |

<br/>

### CASE 1. 이용자가 입력한 URL에 요청을 보내는 경우
웹 서비스에서 사용하는 마이크로서비스의 API 주소를 알아내어 이를 이용자의 입력값으로 전달하면, 외부에서 직접 접근할 수 없는 마이크로서비스의 기능을 임의로 사용할 수 있음

#### CASE 1 예제 코드
```python
# pip3 install flask requests # 파이썬 flask, requests 라이브러리를 설치하는 명령입니다.
# python3 main.py # 파이썬 코드를 실행하는 명령입니다.

from flask import Flask, request
import requests

app = Flask(__name__)


# image_downloader: 이용자가 입력한 URL에 HTTP 요청을 보내고 응답을 반환함
@app.route("/image_downloader")
def image_downloader():
	image_url = request.args.get("image_url", "") # 이용자가 입력한 image_url를 가져옴
	response = requests.get(image_url) # requests 라이브러리를 사용해서 image_url에 HTTP GET 메소드 요청을 보내고 결과를 response에 저장함

	return ( # 아래의 3가지 정보를 반환함
		response.content, # HTTP 응답으로 온 데이터
		200, # HTTP 응답 코드
		{"Content-Type": response.headers.get("Content-Type", "")}, # HTTP 응답으로 온 헤더 중 Content-Type(응답의 타입)
	)


# request_info: 웹 페이지에 접속한 브라우저의 정보(User-Agent)를 반환함
@app.route("/request_info")
def request_info():
	return request.user_agent.string # '접속하는데'에 사용한 브라우저의 정보가 출력됨

app.run(host="127.0.0.1", port=8000)
```
* image_downloader 페이지에서 ```image_url```로 request_info 경로를 입력하면 request_info에 HTTP 요청을 보내고 응답을 반환함
    ```
    http://127.0.0.1:8000/image_downloader?image_url=http://127.0.0.1:8000/request_info
    ```
    - 웹 서버에 위치한 image_downloader(웹 서비스)에서 request_info에 HTTP 요청을 보냈기 때문에 브라우저 정보로 ```python-requests/...```이 출력됨

<br/>

### CASE 2. 웹 서비스의 요청 URL에 이용자의 입력값이 포함되는 경우
이용자의 입력값 중 URL의 구성 요소 문자를 삽입하면 API 경로를 조작할 수 있음
| 구성 요소 문자 | 설명 |
|---|------|
| ```..``` | 상위 경로로 이동하기 위한 구분자 <br/>&nbsp;&nbsp; → Path Traversal(경로 변조) 취약점에 사용될 수 있음 |
| ```#``` | Fragment Idenifier 구분자, 뒤에 붙은 문자열은 API 경로에서 생략됨 |

#### CASE 2 예제 코드
```python
INTERNAL_API = "http://api.internal/"
# INTERNAL_API = "http://127.17.0.3/"


@app.route("/v1/api/user/information")
def user_info():
	user_idx = request.args.get("user_idx", "") # 이용자가 전달한 user_idx를 가져옴
	response = requests.get(f"{INTERNAL_API}/user/{user_idx}") # user_idx(이용자의 입력값)를 내부 API의 URL 경로로 사용함


@app.route("/v1/api/user/search")
def user_search():
	user_name = request.args.get("user_name", "") # 이용자가 전달한 user_name을 가져옴
	user_type = "public"
	response = requests.get(f"{INTERNAL_API}/user/search?user_name={user_name}&user_type={user_type}") # user_name(이용자의 입력값)을 내부 API의 쿼리로 사용함
```
* user_info 함수에서 user_idx에 ```..``` 문자가 포함된 ```../search```를 입력했을 경우
    - /v1/api/user/search로 접근하여 해당 위치로 URL 요청을 보낼 수 있음
* user_search 함수에서 user_name에 ```#``` 문자가 포함된 ```admin&user_type=private#```를 입력한 경우
    - 뒤의 ```&user_type=public```이 생략되어 ```/search?user_name=admin&user_type=private```이 전송됨

<br/>

### CASE 3. 웹 서비스의 요청 Body에 이용자의 입력값이 포함되는 경우
데이터를 구성할 때 이용자의 입력값을 파라미터의 형식으로 설정하는데, 이때 이용자가 URL에서 파라미터를 구분하기 위해 사용하는 ```&``` 구분 문자를 포함하면 설정되는 데이터의 값을 변조할 수 있음
* 내부 API에서는 전달받은 값을 파싱할 때 파라미터의 값을 차례대로 가져와 사용하기 때문에 가능함

#### 구분 문자(Delimiter): 일반 텍스트 또는 데이터 스트림에서 별도의 독립적 영역 사이의 경계를 지정하는 데 사용하는 하나의 문자 혹은 문자들의 배열
* URL에서 구분 문자는 ```/```(Path identifier), ```?```(Query identifier) 등이 있음
* 구분 문자에 따라 URL의 해석이 달라질 수 있음

#### CASE 3 예제 코드
```python
# pip3 install flask
# python main.py

from flask import Flask, request, session
import requests
from os import urandom

app = Flask(__name__)
app.secret_key = urandom(32)

INTERNAL_API = "http://127.0.0.1:8000/"
header = {"Content-Type": "application/x-www-form-urlencoded"}


# board_write
@app.route("/v1/api/board/write", methods=["POST"])
def board_write():
	session["idx"] = "guest" # session idx를 guest로 설정함

    # form 데이터(이용자의 입력값)에서 title, body의 값을 가져옴
	title = request.form.get("title", "")
	body = request.form.get("body", "")

    # 전송할 데이터를 구성
	data = f"title={title}&body={body}&user={session['idx']}" # 전송할 데이터(HTTP body 데이터)를 구성합니다.

    # INTERNAL API에 이용자가 입력한 값을 HTTP body 데이터로 사용해서 요청함
	response = requests.post(f"{INTERNAL_API}/board/write", headers=header, data=data)
	return response.content # INTERNAL API의 응답 결과를 반환함


# internal_board_write: board_write 함수에서 요청하는 내부 API를 구성하는 기능
@app.route("/board/write", methods=["POST"])
def internal_board_write():
	# form 데이터로 입력받은 값(전달된 title, body, 계정 이름)을 JSON 형식으로 반환함
	title = request.form.get("title", "")
	body = request.form.get("body", "")
	user = request.form.get("user", "")
	info = {
		"title": title,
		"body": body,
		"user": user
	}
	return info


# index: board_write 기능을 호출하기 위한 인덱스 페이지
@app.route("/")
def index():
	return """
		<form action="/v1/api/board/write" method="POST">
			<input type="text" placeholder="title" name="tite"/><br/>
			<input type="text" placeholder="body" name="body"/><br/>
			<input type="submit"/>
		</form>
	"""

app.run(host="127.0.0.1", port=8000, debug=True)
```
* 이용자의 입력값인 title에서 ```&``` 구분 문자를 포함하는 ```title&user=admin```을 입력하면 data의 값을 변조할 수 있음
    ```
    data = f"title={title}&body={body}&user={session['idx']}
    // Result: title=title&user=admin&body=body&user=guest
    ```
    - 내부 API에서는 전달받은 값을 파싱할 때 앞에 존재하는 파라미터의 값을 가져와 사용함
        + ```user=admin```이 ```user=guest```보다 앞에 위치하므로 user의 값을 변조할 수 있음
