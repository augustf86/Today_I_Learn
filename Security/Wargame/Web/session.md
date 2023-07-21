# [Dreamhack Wargame] session
### [🚩session](https://dreamhack.io/wargame/challenges/266/)
<img width="1068" alt="session_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/73a862ae-0d30-4175-a9d9-483b68e42e54">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest',
    'user': 'user1234',
    'admin': FLAG # admin 계정의 패스워드가 파일에서 읽어온 FLAG임
}

session_storage = {
}

# 인덱스(/) 페이지
@app.route('/')
def index():
    session_id = request.cookies.get('sessionid', None) # 이용자가 전송한 sessionid 값을 가져옴
    try:
        username = session_storage[session_id] # session_storage에 session_id가 존재하는지 확인하고, 존재한다면 username 값을 가져옴
    except KeyError:
        return render_template('index.html')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')


# login 페이지
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('login.html') # username과 password를 입력할 수 있는 로그인 페이지(login.html)를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청 시
        # 이용자가 전송한 username과 password 입력값을 가져옴
        username = request.form.get('username')
        password = request.form.get('password')
        
        try:
            # users에 이용자가 입력한 username이 존재하는지 확인하고, 존재한다면 해당 user의 패스워드를 가져옴
            pw = users[username]
        except:
            # 존재하지 않는 username인 경우 알림창을 출력함
            return '<script>alert("not found user");history.go(-1);</script>'
            
        if pw == password: # users에 존재하는 패스워드와 이용자가 전송한 password의 값을 비교 (동일한 경우에만 if문 블록 내 코드 실행)
            resp = make_response(redirect(url_for('index')) ) # index 페이지로 이동하는 응답을 생성함
            
            # session
            session_id = os.urandom(4).hex() # 4바이트의 랜덤한 16진수(문자열)을 생성하여 session_id에 대입함
            session_storage[session_id] = username # session_id를 key로, username을 value로 하여 session_storage에 이를 저장함
            
            resp.set_cookie('sessionid', session_id) # session_id로 쿠키를 설정함
            return resp
            
        return '<script>alert("wrong password");history.go(-1);</script>' # 패스워드가 일치하지 않을 경우 알림창을 출력함

if __name__ == '__main__':
    import os
    
    # 사이트에 접속했을 때 수행하는 과정
    session_storage[os.urandom(1).hex()] = 'admin' # 1바이트의 랜덤한 16진수(문자열)을 key로 하고, 'admin'을 value로 하여 이를 session_storage에 저장함
    print(session_storage)
    app.run(host='0.0.0.0', port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점 분석
제일 아래의 코드를 보면 admin의 Session ID를 1바이트의 hex 값으로 설정하고 있음 (```session_storage[os.urandom(1).hex(1)] = 'admin```)
* 1바이트의 값의 전체 가짓수는 $2^8 = 256$이므로 해당 값을 무작위 대입하여 admin 계정으로 로그인할 수 있음
    - 무작위 대입을 위해 **Burp Suite의 Intruder 기능**을 사용함 ← Intruder 기능 사용 방법은 Tools: Burp Suite [🔗](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Note/Tools%3A%20Burp%20Suite.md)를 참고

<br/><br/>

### 익스플로잇
