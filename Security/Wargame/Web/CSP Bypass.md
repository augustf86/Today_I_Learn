# [Dreamhack Wargame] CSP Bypass
### [🚩CSP Bypass](https://dreamhack.io/wargame/challenges/435/)
  <img width="1070" alt="CSP Bypass_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/1c3b77c5-a133-45dc-b26a-8a099cb17224">

* CSP를 이용해 XSS 취약점이 발생해도 공격할 수 없도록 설정하였으나 잘못 설정된 부분을 찾아 우회하는 실습

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
    
    # read_url 함수에 생성한 주소와 쿠키 값을 인자로 넘겨주어 vuln 페이지에 접속함
    return read_url(url, cookie)


@app.after_request
def add_header(response): # HTTP 응답에 CSP 헤더를 추가하는 함수
    global nonce
    response.headers[ # CSP 정책 지시문 내용 (아래의 표를 참고)
        "Content-Security-Policy"
    ] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'self' 'nonce-{nonce}'"
    nonce = os.urandom(16).hex()
    return response


# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html", nonce=nonce)


# vuln 페이지: 이용자가 입력한 값을 그대로 출력함
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 전달한 param 파라미터의 값을 가져옴
    return param # 이용자의 입력값을 return 문을 이용해 그대로 출력함


# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함 (해당 이용자의 쿠키에는 FLAG가 존재함)
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html", nonce=nonce) # 이용자에게 URL를 입력받는 페이지(flag.html)을 제공함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param") # 이용자가 입력한 param의 값을 가져옴
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}): # param 파라미터의 값과 쿠키에 FLAG를 포함하여 check_xss 함수를 호출함
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
    | ```img-src``` | ```https://dreamhack.io```| 이미지의 출처를 ```https://dreamhack.io```로 제한함 |
    | ```style-src``` | ```'self' 'unsafe-inline'``` | 스타일시트의 권한과 출처를 현재 페이지와 같은 출처로 제한하고, <br/> 스타일시트 내 인라인 코드의 사용을 허용함 |
    | ```script-src``` | ```'self' 'nonce-{nonce}'``` | 스크립트 태그의 출처는 현재 페이지와 같은 출처로 제한하고, <br/> nonce 속성에 {nonce} 값이 존재하지 않으면 스크립트 로드에 실패함 |
    - 📌 공격자는 ```nonce``` 값을 알지 못하기 때문에 XSS 취약점이 존재해도 일반적인 방법으로는 공격을 수행할 수 없음

<br/><br/>

## 문제 풀이
