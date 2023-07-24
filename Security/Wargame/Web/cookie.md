# [Dreamhack Wargame] cookie
* 출처: 🚩 cookie [🔗](https://dreamhack.io/wargame/challenges/6/)
* Reference: 쿠키(Cookie)
* 문제 설명
  <br/><br/>
  <img width="1072" alt="cookie_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9f6291e6-c928-4f11-a094-390d2725804f">

<br/><br/>

## 문제 파일(app.py) 및 취약점 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read() # flag.txt 파일로부터 FLAG 데이터르 가져옴
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest', # guest 계정의 패스워드는 "guest"
    'admin': FLAG # admin 계정의 패스워드가 파일에서 읽어온 FLAG임
}



# 인덱스 페이지르 구성하는 코드: 이용자의 username을 출력하고 관리자 계정인지 확인함
@app.route('/')
def index():
    # 이용자가 전송한 쿠키의 username 값을 가져옴
    username = request.cookies.get('username', None)
    
    if username: # username의 값이 존재하는 경우 if문 내 코드 실행
        # username의 값이 "admin"인 경우 FLAG를, 그렇지 않으면 "you are not admin"을 출력함
        return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')
    return render_template('index.html')



# 로그인 페이지를 구성하는 코드: username, password를 입력 받고 로그인함
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET': # GET 메소드로 요청할 경우
        return render_template('login.html') # username과 password를 입력할 수 있는 로그인 페이지(login.html)를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청할 경우
        username = request.form.get('username') # 이용자가 전송한 username 입력값을 가져옴
        password = request.form.get('password') # 이용자가 전송한 password 입력값을 가져옴
        
        try:
            # users에 이용자가 전송한 username이 존재하는지 확인하고, 존재한다면 해당 user의 패스워드를 가져옴
            pw = users[username]
        except:
            # 존재하지 않는 username인 경우 경고를 출력함
            return '<script>alert("not found user");history.go(-1);</script>'
            
        if pw == password: # users에 존재하는 패스워드와 이용자가 전송한 password 값을 비교 (동일하다면 if문 내 코드 실행)
            resp = make_response(redirect(url_for('index')) ) # index 페이지로 이동하는 응답을 생성함
            resp.set_cookie('username', username) # username으로 쿠키를 설정
            return resp
        
        return '<script>alert("wrong password");history.go(-1);</script>' # password가 동일하지 않은 경우 경고를 출력함



app.run(host='0.0.0.0', port=8000)
```
* login 페이지에서 쿠키를 생성할 때 쿠키의 값으로 이용자가 입력한 ```username``` 값을 이용함 <br/> &nbsp;&nbsp; → 인덱스 페이지에서 ```username``` 변수의 값이 이용자가 전송한 요청에 포함된 쿠키에 의해서 결정됨
    - 클라이언트에 저장되고 클라이언트의 요청에 포함되는 쿠키의 특성 상 <U>이용자가 임의로 쿠키를 조작할 수 있음</U>
    - 서버는 쿠키에 대한 검증 과정 없이 이용자의 요청에 포함된 쿠키를 신뢰하고 이를 통해 이용자 인증 정보를 식별하고 있음 <br/> &nbsp;&nbsp; ⇒ ***쿠키의 ```username```의 값을 "admin" 문자열로 조작하면 FLAG를 획득할 수 있음***

<br/><br/>

## 문제 풀이 (익스플로잇)
1. 패스워드를 알고 있는 guest 계정으로 먼저 로그인을 진행함<br/>
    <img width="1157" alt="cookie_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/3bccf8a9-3694-4a03-8333-14f7cce6a7ca">
    <img width="1157" alt="cookie_1-1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e0227816-0e96-4364-b16f-63e212bcfeab">

2. Option + Command + I를 눌러 크롬 개발자 도구를 열고 Application 탭의 Cookies 목록에서 생성된 쿠키를 확인함<br/>
    <img width="1157" alt="cookie_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e099ba52-da53-4a15-83ea-970dbcc2c952">

3. 생성된 쿠키의 username의 Value를 “admin”으로 변경한 후 서버에 요청함(새로고침)<br/>
    <img width="1157" alt="cookie_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/bfbf01eb-67d8-4554-a79c-327fe6e55173">

4. index.html에서 FLAG를 획득할 수 있음<br/>
    <img width="1157" alt="cookie_4" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9a6d4f35-e17d-479d-b0a1-cbadb015e367">
