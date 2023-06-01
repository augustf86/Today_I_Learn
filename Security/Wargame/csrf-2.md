# [Dreamhack Wargame] csrf-2
### [🚩csrf-2](https://dreamhack.io/wargame/challenges/269/)
  <img width="1070" alt="csrf-2_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6f1ea962-7e23-4b14-809f-e42105b56860">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for
from selenium import webdriver
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read() # flag.txt 파일로부터 FLAG 데이터를 가져옴
except:
    FLAG = "[**FLAG**]"


users = {
    'guest': 'guest', # guest 계정의 패스워드 = "guest"
    'admin': FLAG # admin 계정의 패스워드 = 파일에서 읽어온 FLAG
}

session_storage = {}



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
        driver.implicitly_wait(3) # 크롬 로딩타입을 위한 타임아웃 설정(3초)
        driver.set_page_load_timeout(3) # 페이지 오픈을 위한 타임아웃 설정(3초)
        driver.get("http://127.0.0.1:8000/") # 관리자가 csrf-2 문제 사이트에 접속함
        driver.add_cookie(cookie) # 관리자의 쿠키를 적용함
        driver.get(url) # 인자로 전달된 url에 접속함 (관리자 쿠키를 적용했으므로 관리자 권한으로 사이트 접속)
    except Exception as e:
        driver.quit() # 셀레늄 종료
        print(str(e))
        # return str(e)
        return False # URL 방문 실패 (접속 중 오류가 발생하여 비정상 종료 처리됨)
    driver.quit() # 셀레늄 종료
    return True # URL 방문 성공 (정상 종료 처리)


def check_csrf(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}" # 로컬 URL로 설정함
    return read_url(url, cookie) # read_url 함수를 호출하여 URL를 방문함



# 인덱스 페이지
@app.route("/")
def index():
    session_id = request.cookies.get('sessionid', None) # 이용자가 전송한 쿠키의 sessionid 값을 가져옴
    try:
        username = session_storage[session_id] # session_storage에 sessionid 값이 존재하는지 확인하고, sessionid 키에 해당하는 value(username)을 가져옴
    except KeyError:
        return render_template('index.html', text='please login') # session_storage에 존재하지 않는다면, 화면에 'please login'을 출력함

    # session_storage에 값이 존재하고 username이 "admin"인 경우 화면에 FLAG 값을 출력함
    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not an admin"}')



# vuln 페이지: 이용자가 입력한 값을 화면에 그대로 출력함 (XSS가 발생할 수 있는 키워드는 필터링함)
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower() # 이용자가 입력한 param 값을 가져와 소문자로 변경함
    
    xss_filter = ["frame", "script", "on"] # XSS가 발생할 수 있는 키워드 3가지
    for _ in xss_filter:
        param = param.replace(_, "*") # 이용자가 입력한 값 중에서 필터링 키워드가 있는 경우 '*'로 치환함
        
    return param # 이용자의 입력 값을 그대로 화면에 출력함



# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html") # 이용자에게 링크를 입력 받는 페이지(flag.html)를 출력함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param", "") # 이용자가 입력한 param 파라미터의 값을 가져온 후
        
        # session 사용
        session_id = os.urandom(16).hex() # 8비트의 랜덤한 16진수(문자열)를 생성하여 session_id에 대입함
        session_storage[session_id] = 'admin' # 키를 session_id로, 값을 username으로 하여 session_storage에 이를 저장함
        
        if not check_csrf(param, {"name":"sessionid", "value": session_id}): # 관리자에게 접속 요청을 함 (session을 사용함)
        
            return '<script>alert("wrong??");history.go(-1);</script>' # check_csrf 함수의 결과로 false를 반환할 경우(= URL 방문 실패 시) "wrong??" 알림창을 출력함

        return '<script>alert("good");history.go(-1);</script>' # check_csrf 함수의 결과로 true를 반환할 경우(= URL 방문 성공 시) "good" 알림창을 출력함



# 로그인 페이지:
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET': # GET 메소드로 요청할 경우
        return render_template('login.html') # username과 password를 입력할 수 있는 로그인 페이지(login.html)를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청할 경우
        username = request.form.get('username') # 이용자가 전송한 username 값을 가져옴
        password = request.form.get('password') # 이용자가 전송한 password 값을 가져옴
        
        try:
            # users에 이용자가 전송한 username이 존재한다면 해당 username의 password를 가져옴
            pw = users[username]
        except:
            # users에 이용자가 전송한 username이 존재하지 않는다면 해당 유저를 찾지 못했다는 알림창을 발생시킴
            return '<script>alert("not found user");history.go(-1);</script>'
            
        if pw == password: # users에 존재하는 패스워드와 이용자가 전송한 password의 값을 비교하여 동일하다면 if문 내의 실행문을 실행함
            resp = make_response(redirect(url_for('index')) ) # index 페이지로 이동하는 응답을 생성함
            
            # session을 사용
            session_id = os.urandom(8).hex() # 8비트의 랜덤한 16진수(문자열)를 생성하여 session_id에 대입함
            session_storage[session_id] = username # 키를 session_id로, 값을 username으로 하여 session_storage에 이를 저장함
            
            resp.set_cookie('sessionid', session_id) # session_id로 쿠키를 설정
            return resp
            
        return '<script>alert("wrong password");history.go(-1);</script>' # 패스워드가 일치하지 않는 경우 잘못된 패스워드라는 경고창을 출력함



# 비밀번호 변경 페이지
@app.route("/change_password")
def change_password():
    pw = request.args.get("pw", "") # 이용자가 전송한 pw 값을 가져옴
    session_id = request.cookies.get('sessionid', None) # 이용자가 전송한 sessionid 값을 가져옴
    
    try:
        username = session_storage[session_id] # session_storage에 session_id 값이 존재하면 sessionid 키에 해당하는 value(username)을 가져옴
    except KeyError:
        return render_template('index.html', text='please login') # session_storage에 존재하지 않는다면, 화면에 'please login'을 출력함

    users[username] = pw # users에서 username에 해당하는 계정의 비밀번호를 pw로 변경함
    return 'Done' # 'Done'을 반환함



app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
이용자의 입력값을 ```render_template``` 함수를 이용해 HTML 엔티티 코드로 변환해 저장하지 않고 그대로 출력하는 /vuln 페이지를 이용해 CSRF 공격을 수행할 수 있음
* 입력값에서 ```frame```, ```script```, ```on``` 키워드를 필터링하고 있지만 필터링되는 키워드 외의 다른 키워드와 태그를 사용(우회)하여 CSRF 공격을 수행할 수 있음
* 악성 스크립트가 삽입된 /vuln 페이지를 다른 이용자가 방문할 경우 의도하지 않은 페이지로 요청을 전송하는 익스플로잇 코드를 작성해야 함


<br/><br/>

### 익스플로잇
/vuln 페이지를 방문하는 관리자(admin 계정)가 /change_password 페이지로 패스워드를 변경하는 요청을 전송하는 코드를 작성해야 함
* CSRF 취약점 테스트
	- ```<img>``` 태그를 사용해 외부 웹 서버로 요청을 보내는 공격 코드를 작성한 뒤, 취약점이 발생하는 /vuln 페이지에 해당 코드를 삽입하여 CSRF 취약점이 존재하는지 테스트함
		```html
		<img src=“http://RANDOMHOST.request.dreamhack.games">
		```
    <img width="1505" alt="취약점 체크_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/166fec00-60f6-4afd-aab3-625c611d6061">
    <img width="1034" alt="취약점 체크_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e61f7bc5-8518-4d47-88ec-dfaa78adbd32">
    <img width="1505" alt="취약점 체크_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/74048704-ab2e-4eb8-9820-1cdf2f26cf1e">

<br/><br/>

#### 익스플로잇 과정
1. flag 엔드포인트에서 ```<img src=“/change_password?pw=admin”>```를 입력하고 제출 버튼을 누름
	- 변경하려는 pw의 값을 이용자의 요청을 받으므로 ```pw=“admin”```이 포함되어야 함
	- 성공적으로 접속했다면 로컬 호스트에서 ```http://127.0.0.1:8000/vuln?param=<img src=“/change_password?pw=admin”>```에 접속하게 됨
	<img width="1034" alt="익스플로잇_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b1ca2ebc-829f-42e3-95f3-402d97843e14">

2. “good” 알림창이 출력되고 나면 /vuln 페이지를 방문한 계정의 비밀번호가 “admin”으로 바뀌게 됨

3. login 페이지로 이동하여 username에 “admin”, password에 “admin”을 입력하고 [Login] 버튼을 누르면 로그인에 성공하여 index 페이지에 FLAG가 출력됨
  <img width="1034" alt="익스플로잇_3-1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/de6ef0b0-a39d-4e77-b064-1ff3be4a06fa">
  <img width="1034" alt="익스플로잇_3-2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f048ede6-8759-4658-b4e8-c2706a6e7b62">
