# [Dreamhack Wargame] web-deserialize-python
* 출처: 🚩 web-deserialize-python [🔗](https://dreamhack.io/wargame/challenges/40)
* Reference: Language specific Vulnerability (Python의 serialize/deserialize)
* 문제 설명
  <br/><br/>
  <img width="811" alt="web-deserialize-python_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/adffd2e7-8981-4c13-97c3-eb878eac2127">

<br/><br/>

## 문제 파일(app.py) 및 취약점 분석
```python
#!/usr/bin/env python3
from flask import Flask, request, render_template, redirect
import os, pickle, base64

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open('./flag.txt', 'r').read() # Flag is here!!
except:
    FLAG = '[**FLAG**]'

INFO = ['name', 'userid', 'password'] # INFO 구조

# index 페이지
@app.route('/')
def index():
    return render_template('index.html')

# create_session 페이지
@app.route('/create_session', methods=['GET', 'POST'])
def create_session():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('create_session.html') # Name, Userid, Password를 입력받는 페이지를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청 시
        # 사용자가 입력한 Name, Userid, Password를 가져와 파이썬 객체인 info를 생성함
        info = {}
        for _ in INFO:
            info[_] = request.form.get(_, '')
        
        # Python Object인 info를 직렬화하여 메모리에 저장함 (base64 사용)
        data = base64.b64encode(pickle.dumps(info)).decode('utf8')
        
        return render_template('create_session.html', data=data) # info를 직렬화한 값을 화면에 출력함

# check_session 페이지
@app.route('/check_session', methods=['GET', 'POST'])
def check_session():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('check_session.html') # session을 입력받는 페이지를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청 시
        session = request.form.get('session', '') # 사용자가 입력한 session을 가져옴
        info = pickle.loads(base64.b64decode(session)) # 직렬화되어 있는 바이트 객체인 session을 파이썬 객체로 역직렬화함
        return render_template('check_session.html', info=info) # session을 역직렬화한 결과를 화면에 출력함

app.run(host='0.0.0.0', port=8000)
```
* create_session 페이지에서 세션을 생성하는 과정에서 직렬화(```pickle.dumps(obj)```)가 사용되고, check_session 페이지에서 입력 받은 세션을 복호화하는 과정에서 역직렬화(```pickle.loads(data)```)가 사용되고 있음
    | 상황 | 설명 |
    |:---:|------|
    | 세션 생성 | ```data = base64.b64encoded(pickle.dumps(info)).decode('utf8')``` |
    | 세션 복호화 | ```info = pickle.loads(base64.b64encode(session))``` |

<br/>

* 서버가 세션을 역직렬화할 때 사용하는 Python의 pickle 모듈에는 역직렬화(deserialize) 취약점이 존재함
    - pickle 모듈이 지원하는 ```object.__reduce__``` 메소드의 반환 값
      <br/><br/>
      <img width="1512" alt="reduce 메소드 docs" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a9abede8-b242-48ef-b269-bf0e9b234322"><br/>
      
        + 호출할 수 있는 객체
        + 호출할 수 있는 객체에 대한 인수 tuple (인수를 허용하지 않을 경우 빈 tuple을 제공) <br/> &nbsp;&nbsp; → tuple은 함수를 반환할 수 있으므로 함수 실행이 가능함
    - ⚠️ 역직렬화된 객체에 접근하는 것이 가능하다면 명령어를 실행할 수 있는 클래스를 임의로 지정하여 RCE(Remote Code Execution) 공격을 수행할 수 있음

<br/><br/>

## 문제 풀이(익스플로잇)
