# [Dreamhack Wargame] cookie
### [🚩 cookie](https://dreamhack.io/wargame/challenges/6/)

<br/><br/>

## 문제 파일(app.py)
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

<br/><br/>

## 문제 풀이
