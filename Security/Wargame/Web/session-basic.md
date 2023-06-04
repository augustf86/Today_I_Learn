# [Dreamhack Wargame] session-basic
### [🚩session-basic](https://dreamhack.io/wargame/challenges/409/)
<img width="1071" alt="session-basic_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e4c24a1c-cb52-4677-a79b-c4c2726d4a63">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read() # flag.txt 파일로부터 FLAG 데이터를 가져옴
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest', # guest 계정의 패스워드 = "guest"
    'user': 'user1234', # user 계정의 패스워드 = "user1234"
    'admin': FLAG # admin 계정의 패스워드 = 파일에서 읽어온 FLAG
}


# this is our session storage
session_storage = {
}



# 인덱스 페이지를 구성하는 코드: 이용자의 username을 출력하고 관리자 계정인지 확인함
@app.route('/')
def index():
    # 이용자가 전송한 쿠키의 session_id 값을 가져옴
    session_id = request.cookies.get('sessionid', None)
    try:
        # get username from session_storage
        username = session_storage[session_id] # session_storage에 session_id 값이 존재하는지 확인하고, 존재한다면 username 값을 가져옴
    except KeyError:
        return render_template('index.html')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')



# 로그인 페이지를 구성하는 코드
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
            # 존재하지 않는 username일 경우 경고를 출력함
            return '<script>alert("not found user");history.go(-1);</script>'
            
        if pw == password: # users에 존재하는 패스워드와 이용자가 전송한 password 값을 비교 (동일하다면 if문 내의 코드 실행)
            resp = make_response(redirect(url_for('index')) ) # index 페이지로 이동하는 응답을 생성함
            
            # session 사용
            session_id = os.urandom(32).hex() # 32바이트의 랜덤한 16진수(문자열)를 생성하여 session_id에 대입함
            session_storage[session_id] = username # 키를 session_id로, 값을 username으로 하여 session_storage에 이를 저장함
            
            resp.set_cookie('sessionid', session_id) # session_id로 쿠키를 설정
            return resp
        
        return '<script>alert("wrong password");history.go(-1);</script>' # 패스워드가 일치하지 않은 경우 경고를 출력함



# 관리자 페이지를 구성하는 코드
@app.route('/admin')
def admin():
    # developer's note: review below commented code and uncomment it (TODO)

    #session_id = request.cookies.get('sessionid', None)
    #username = session_storage[session_id]
    #if username != 'admin':
    #    return render_template('index.html')

    return session_storage # session_storage의 session_id와 username 목록을 화면에 출력함



if __name__ == '__main__':
    import os
    # create admin sessionid and save it to our storage
    # and also you cannot reveal admin's sesseionid by brute forcing!!! haha
    
    # 사이트를 접속했을 때 수행하는 과정
    session_storage[os.urandom(32).hex()] = 'admin' # 32바이트의 랜던함 16진수(문자열)을 키로 하고, 'admin'을 값으로 하여 이를 session_storage에 저장함
    print(session_storage)
    app.run(host='0.0.0.0', port=8000)
```


<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
* Cookie 문제와 session-basic 문제의 비교
	- Cookie 문제에서는 인덱스 페이지에서 이용자가 전송한 쿠키의 ```username``` 변수가 요청에 포함된 쿠키에 의해서 결정되었음
	- session-basic 문제에서는 쿠키에 ```username``` 대신에 랜덤으로 생성된 ```session_id```가 사용됨
		+ guest 계정으로 로그인한 후 크롬 개발자 도구의 Application 탭에서 Cookies 목록을 확인하면 session_id가 생성된 것을 확인할 수 있음
* 문제 파일을 살펴보면 **사이트에 처음 접속할 때 admin의 session_id를 session_storage에 삽입하고, admin 페이지에 접속하면 session_storage를 출력한다는 것을 확인할 수 있음**
	- admin 페이지로 이동하면 admin 계정의 session_id를 획득할 수 있으므로 이 부분을 통해 공격할 수 있음

<br/>

### 익스플로잇
1. guest 계정으로 로그인을 수행하여, 크롬 개발자 도구의 Application 탭의 Cookies 항목에 session_id를 생성함
    <img width="1033" alt="session-basic_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/2d53bda3-b6b1-44c7-a229-ff66e179fb72">

2. 다음과 같이 주소를 입력하여 /admin 페이지로 이동함
	- 화면에서 “admin”을 값으로 갖는 session_id 정보를 획득할 수 있음 → 이 값을 복사하여 인덱스 페이지로 이동함
    <img width="1077" alt="session-basic_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/db6841de-83d5-4452-a86a-f2c5ceffd960">

3. 인덱스 페이지의 Application 탭에서 session_id의 Value를 복사한 값으로 바꾼 후 페이지를 새로고침함
    <img width="1077" alt="session-basic_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/3a820a4d-2016-40f4-9462-d4b56c6a72b0">
4. admin으로 로그인에 성공하여 flag를 획득할 수 있음
    <img width="1077" alt="session-basic_4" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6b90df63-ebed-4f55-8813-00a7efb99869">
