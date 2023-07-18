# [Dreamhack Wargame] pathtraversal
### [🚩pathtraversal](https://dreamhack.io/wargame/challenges/12/)
<img width="1065" alt="pathtraversal_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/fcc55ad2-ac5d-4000-a630-8c854dc7b458">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, abort
from functools import wraps
import requests
import os, json

users = {
    '0': {
        'userid': 'guest',
        'level': 1,
        'password': 'guest'
    },
    '1': {
        'userid': 'admin',
        'level': 9999,
        'password': 'admin'
    }
}

def internal_api(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.remote_addr == '127.0.0.1':
            return func(*args, **kwargs)
        else:
            abort(401)
    return decorated_view

app = Flask(__name__)
app.secret_key = os.urandom(32)
API_HOST = 'http://127.0.0.1:8000'

try:
    FLAG = open('./flag.txt', 'r').read() # Flag is here!!
except:
    FLAG = '[**FLAG**]'

# 인덱스(/) 페이지
@app.route('/')
def index():
    return render_template('index.html')

# get_info 페이지: userid를 입력받아 userid에 해당하는 사용자의 users 데이터베이스 정보(userid, level, password)를 화면에 출력함
@app.route('/get_info', methods=['GET', 'POST'])
def get_info():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('get_info.html') # userid를 입력받는 페이지(get_info.html)를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청 시
        userid = request.form.get('userid', '') # 이용자가 입력한 userid 값을 가져옴
        info = requests.get(f'{API_HOST}/api/user/{userid}').text # /api/user/{userid}의 정보(info)를 가져옴 (text 형식)
        return render_template('get_info.html', info=info) # 가져온 info를 render_template 함수를 이용해 화면에 출력함

@app.route('/api')
@internal_api
def api():
    return '/user/<uid>, /flag' # /api의 하위 디렉터리로 /user/<uid>, /flag가 존재함

@app.route('/api/user/<uid>')
@internal_api
def get_flag(uid):
    try:
        info = users[uid] # 해당 uid가 존재하는 경우 users 데이터베이스에서 uid에 해당하는 데이터를 가져옴
    except:
        info = {} # 해당 uid가 존재하지 않을 경우 info에는 빈 값({})을 대입
    return json.dumps(info)

# /api/flag: 해당 위치에 플래그가 존재함
@app.route('/api/flag')
@internal_api
def flag():
    return FLAG # 이 경로로 접속하면 플래그를 획득할 수 있음

application = app # app.run(host='0.0.0.0', port=8000)
# Dockerfile
#     ENTRYPOINT ["uwsgi", "--socket", "0.0.0.0:8000", "--protocol=http", "--threads", "4", "--wsgi-file", "app.py"]
```

<br/><br/>

## 문제 풀이
