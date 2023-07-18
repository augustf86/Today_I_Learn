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
### 취약점 분석
get_info 페이지에서 이용자가 입력한 userid 값에 대해 어떠한 검사도 수행하지 않음 → 입력값에 메타 문자 ```../```를 삽입하여 상위 디렉터리나 다른 디렉터리로 이동할 수 있음
* app.py 파일 내 ```api``` 함수를 보면 /api의 하위 디렉터리로 ```/user/<uid>```와 ```/flag```가 존재함을 알 수 있음
    ```python
    @app.route('/api')
    @internal_api
    def api():
        return '/user/<uid>, /flag'
    ```
    - get_info 페이지에서 userid(이용자의 입력값)을 이용해 ```/user/<uid>```에 접속함 → userid 값으로 ```../flag```를 입력하면 플래그를 획득할 수 있음
* get_info 페이지의 userid를 입력하는 창에 ```../flag```를 입력한 후 [View] 버튼을 누르면 플래그 대신에 ```{}```이 출력되는 것을 확인할 수 있음
  <br/><br/>
  <img width="2560" alt="취약점 분석1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f6e40a7c-fd0a-492c-83d6-c39b8e207189"><br/><br/>
    - 해당 POST 요청을 확인해본 결과 userid 값에 ```undefined```이 입력되어 있는 것을 알 수 있음
    - 정상적인 값(```guest```)를 입력한 후 [View] 버튼을 눌러 POST 요청을 burp suite로 확인해보면 ```userid=0```을 확인할 수 있음
      <br/><br/>
      <img width="2560" alt="취약점 분석2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/32e4e993-2bac-42a6-8853-d18b12901fee"><br/><br/>
        + userid가 아니라 해당 userid의 key값을 이용해 ```/user/<uid>```로 접속함
        + 📌 해결책: burp suite에서 Target 탭의 해당 패킷을 Repeater 탭으로 보낸 후 직접 userid의 값을 ```../flag```로 변경함

<br/><br/>

### 익스플로잇
burp suite의 repeater 탭에서 Request의 userid 값을 ```../flag```로 변경한 후 [Send] 버튼을 클릭함 → 해당 Request에 대한 Response에서 플래그를 획득할 수 있음
<br/><br/>
<img width="2560" alt="익스플로잇" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/91924eb1-312e-4639-a82d-c61cfba81825">
