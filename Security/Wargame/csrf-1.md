# [Dreamhack Wargame] csrf-1
### [🚩csrf-1](https://dreamhack.io/wargame/challenges/26/)
  <img width="1071" alt="csrf-1_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/2c53676d-0c28-4074-95a2-6abacc85133d">

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
    cookie.update({"domain": "127.0.0.1"}) # 관리자의 쿠키가 적용되는 범위를 로컬 호스트(127.0.0.1)로 제한되도록 설정
    try:
        options = webdriver.ChromeOptions() # 크롬 옵션을 사용하도록 설정
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_) # 크롬 브라우저 옵션 설정
        # 셀리늄에서 크롬 브라우저를 사용함
        driver = webdriver.Chrome("/chromedriver", options=options)
        driver.implicitly_wait(3) # 크롬 로딩타임을 위한 타임아웃 설정(3초)
        driver.set_page_load_timeout(3) # 페이지 오픈을 위한 타임아웃 설정(3초)
        driver.get("http://127.0.0.1:8000/") # 관리자가 csrf-1 문제 사이트에 접속함
        driver.add_cookie(cookie) # 관리자의 쿠키를 적용함
        driver.get(url) # 인자로 전달된 url에 접속함 (관리자 쿠키를 적용했으므로 관리자 권한으로 접속한 것이 됨)
    except Exception as e:
        driver.quit() # 샐래늄 종료
        print(str(e))
        # return str(e)
        return False # 접속 중 오류가 발생하여 비정상 종료 처리됨 (false = URL 방문 실패)
    driver.quit() # 셀레늄 종료
    return True # 정상 종료 처리 (true = URL 방문 성공)


def check_csrf(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}" # 로컬 URL로 설정
    return read_url(url, cookie) # read_url 함수를 호출하여 URL를 방문함



# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html") # index.html를 화면에 출력



# vuln 페이지: 이용자가 입력한 값을 출력 (XSS가 발생할 수 있는 키워드는 필터링)
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower() # 이용자가 입력한 param 값을 가져와 소문자로 변경함
    
    xss_filter = ["frame", "script", "on"] # XSS가 발생할 수 있는 키워드 3가지
    for _ in xss_filter:
        param = param.replace(_, "*") # 이용자가 입력한 값 중에 필터링 키워드가 있는 경우 '*'로 치환함
    
    return param # 이용자의 입력 값을 화면에 출력함



# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html") # 이용자에게 링크를 입력 받는 페이지(flag.html)를 출력함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param", "") # 이용자가 입력한 param 파라미터를 가져온 후
        if not check_csrf(param): # 관리자에게 접속을 요청 함
            return '<script>alert("wrong??");history.go(-1);</script>' # check_csrf 함수의 결과로 false를 반환할 경우(= URL 방문 실패 사) "wrong??" 알림창을 출력

        return '<script>alert("good");history.go(-1);</script>' # check_csrf 함수의 결과로 true를 반환할 경우(= URL 방문 성공 시) "good" 알림창을 출력



memo_text = ""

# memo 페이지: 이용자가 메모를 남길 수 있으며, 작성한 메모를 출력함
@app.route("/memo")
def memo():
    global memo_text # memo_text를 전역 변수로 참조
    text = request.args.get("memo", None) # 이용자가 전송한 memo의 입력값을 가져옴
    if text:
        memo_text += text # memo_text에 가져온 입력값을 추가함
    return render_template("memo.html", memo=memo_text) # 사이트에 기록된 memo_text를 render_template 함수를 이용해 화면에 출력함



# /admin/notice 페이지: 메모에 FLAG를 작성하는 기능
@app.route("/admin/notice_flag")
def admin_notice_flag():
    global memo_text # memo_text를 전역 변수로 참조
    
    # 이용자의 IP가 로컬 호스트(네트워크에서 사용하는 호스트명, 자기 자신의 컴퓨터를 의미함)가 아닌 경우 접근 제한
    if request.remote_addr != "127.0.0.1":
        return "Access Denied"
    # userid 파라미터의 값이 "admin"이 아닌 경우 접근 제한
    if request.args.get("userid", "") != "admin":
        return "Access Denied 2"
    
    # 로컬 호스트에서 접속하고 사이트의 관리자인 경우에만 메모에 FLAG를 기록함
    # 페이지 자체는 모두가 접근할 수 있고 userid에 "admin"을 넣는 것도 가능하지만, 일반 유저가 해당 페이지에 접근할 때의 IP 주소는 조작할 수 없음
    #   → 페이지에 단순히 접근하는 것만으로는 플래그를 획득할 수 없음
    memo_text += f"[Notice] flag is {FLAG}\n"
    return "Ok" # "Ok"를 반환함


app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
이용자의 입력값을 ```render_template``` 함수를 이용해 HTML 엔티티 코드로 변환해 저장하지 않고 그대로 출력하는 /vuln 페이지에 취약점이 존재함
* 입력값에서 ```frame```, ```script```, ```on``` 키워드를 필터링하고 있지만 필터링되는 키워드 외의 다른 키워드와 태그를 사용(우회)하여 CSRF 공격을 수행할 수 있음
	- 세 가지 키워드 외의 태그의 꺽쇠(```<```, ```>```)와 다른 키워드는 필터링하고 있지 않음
* 악성 스크립트가 삽입된 /vuln 페이지를 다른 이용자가 방문할 경우 의도하지 않은 페이지로 요청을 전송하는 익스플로잇 코드를 작성해야 함

<br/><br/>

### 익스플로잇
/vuln 페이지를 방문하는 로컬호스트 이용자가 /admin/notice_flag 페이지로 요청을 전송하는 코드를 작성해야 함

