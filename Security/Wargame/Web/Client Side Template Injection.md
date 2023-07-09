# [Dreamhack Wargame] Client Side Template Injection
### [🚩Client Side Template Injection](https://dreamhack.io/wargame/challenges/437/)

* **XSS 취약점은 발생하지만 실질적인 공격으로 연계하기 까다로울 때** Client Side Template Injection을 활용하여 공격함

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
    # 전달받은 param 값을 param 파라미터의 값으로 하여 vuln 페이지에 접속하는 URL 주소를 반환함
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    
    # read_url 함수에 생성한 주소(url)과 쿠키 값을 인자로 넘겨주어 vuln 페이지에 접속함
    return read_url(url, cookie)

@app.after_request
def add_header(response): # HTTP 응답에 CSP 헤더를 추가함
    global nonce
    
    # 유심히 살펴봐야 할 부분: CSP 정책 정의 (아래 표 참고)
    response.headers['Content-Security-Policy'] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'nonce-{nonce}' 'unsafe-eval' https://ajax.googleapis.com; object-src 'none'"
    nonce = os.urandom(16).hex() # 공격자는 nonce 값을 알 수 없음 → XSS 취약점이 존재하도 일반적인 방법으로는 공격 수행 불가
    return response


# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html", nonce=nonce)


# vuln 페이지: 이용자가 입력한 값을 출력함
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 전달한 param 파라미터의 값을 가져옴
    return param # param 파라미터의 값을 return을 이용해 '그대로' 출력함


# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함 (해당 이용자의 쿠키에는 FLAG가 존재함)
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html", nonce=nonce) # 이용자에게 URL를 입력 받는 페이지(flag.html)을 제공함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param") # 이용자가 전달한 param 값을 가져옴
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}): # param 값과 쿠키에 FLAG를 포함하여 check_xss 함수를 호출함
            return f'<script nonce={nonce}>alert("wrong??");history.go(-1);</script>' # check_xss의 리턴값이 false인 경우

        return f'<script nonce={nonce}>alert("good");history.go(-1);</script>' # check_xss의 리턴값이 true인 경우


memo_text = ""

# memo 페이지: 이용자가 메모를 남길 수 있으며, 작성한 메모를 출력함
@app.route("/memo")
def memo():
    global memo_text # memo_text 변수를 전역으로 설정
    text = request.args.get("memo", "") # 이용자가 전달한 memo 파라미터의 값을 가져옴
    memo_text += text + "\n" # 전역변수 memo_text 뒤에 이용자가 전달한 text와 개행 문자(\n)을 추가함
    return render_template("memo.html", memo=memo_text, nonce=nonce) # render_template 함수를 통해 memo_text를 화면에 출력함


app.run(host="0.0.0.0", port=8000)
```
* ```add_header``` 함수의 ```Content-Security-Policy``` HTTP 헤더에 정의된 정책 지시문
    | directive | value | 설명 |
    |---|---|------|
    | ```default-src``` | ```'self'``` | 모든 리소스의 출처를 현재 페이지와 같은 출처로 제한함 |
    | ```img-src``` | ```https://dreamhack.io``` | 이미지의 출처를 ```https://dreamhack.io```로 제한함 |
    | ```style-src``` | ```'self' 'unsafe-inline'``` | 스타일 시트의 권한과 출처를 현재 페이지와 같은 출처로 제한하고, <br/> 스타일 시트 내 인라인 코드의 사용을 허용함 |
    | ```script-src``` | ```'nonce-{nonce}' 'unsafe-eval' ``` <br/>```https://ajax.googleapis.com``` | 스크립트 태그의 출처는 ```https://ajax.googleapis.com```만 허용하고, <br/> ```nonce``` 속성을 필요로 하며 스크립트 내에서 ```eval```를 통핸 코드 실행이 가능함 |
    | ```object-src``` | ```'none``` | ```<object>```, ```<embeded>``` 태그의 출처로 어느 것도 허용하지 않음 | 
    - 공격자는 ```nonce``` 값을 알지 못하기 때문에 XSS 취약점이 존재해도 일반적인 방법으로는 공격을 수행할 수 없음

<br/><br/>

## 문제 풀이
### 취약점 분석
vuln 페이지에서 전달한 템플릿 변수를 기록할 때 HTML 엔티티 코드로 변환해 XSS 취약점이 발생하지 않는 ```render_template``` 함수를 이용하지 않고 ```return```을 이용하여 이용자의 입력값을 페이지에 그대로 출력하기 때문에 XSS 취약점이 발생함
* CSP 정책으로 인해 곧바로 스크립트를 수행하는 것은 불가능함 → 📌 스크립트를 실행하기 위해서는 ```script-src``` 지시문의 출처를 확인해봐야 함
    - CSP 정책에서 허용하고 있는 스크립트 태그의 출처는 ```httsp://ajax.googleapis.com```
        + 해당 출처 내에 JSONP callback이 존재하거나 활용할만한 스크립트가 존재한다면 이를 공격에 이용할 수 있음
        + ```https://ajax.googleapis.com``` 출처 = AngularJS 파일을 서빙할 때도 사용되는 서버 <br/> &nbsp;&nbsp; → ⚠️ AngularJS 파일을 script 태그의 src로 사용한 후 Client Side Injection을 수행할 수 있음
* 공격에 사용할 수 있는 속성
    | 속성 | 설명 |
    |---|------|
    | location.href | 전체 URL를 반환하거나, URL을 업데이트할 수 있는 속성값 |
    | document.cookie | 해당 페이지에서 사용하는 쿠키를 읽고 쓰는 속성값 |

<br/><br/>

### 익스플로잇
