# [Dreamhack Wargame] proxy-1
### [🚩proxy-1](https://dreamhack.io/wargame/challenges/13/)
<img width="1068" alt="proxy-1_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/80cfd06d-55d5-434f-b4a4-bb087de63238">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for
import socket

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

# index(/) 페이지
@app.route('/')
def index():
    return render_template('index.html')

# socket 페이지: host, port, data를 입력 받아 해당 host에 port에 해당하는 포트로 연결을 생성하고 data를 전송하고 그에 대한 응답을 화면에 출력함
@app.route('/socket', methods=['GET', 'POST'])
def login():
    if request.method == 'GET': # GET 메소드로 요청 시
        return render_template('socket.html') # 이용자에게 host, port, data를 입력 받는 페이지(socket.html)를 화면에 출력함
    elif request.method == 'POST': # POST 메소드로 요청 시
    
        # 이용자가 입력한 host, port, data의 입력값을 가져옴
        host = request.form.get('host')
        port = request.form.get('port', type=int) # port 입력값은 int 타입(정수)으로 가져옴
        data = request.form.get('data')

        retData = ""
        try:
            with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
                s.settimeout(3)
                s.connect((host, port)) # host, port 정보로 연결 생성
                s.sendall(data.encode()) # 연결이 생성되면 data를 인코딩하여 전송함
                while True:
                    tmpData = s.recv(1024)
                    retData += tmpData.decode()
                    if not tmpData: break
            
        except Exception as e:
            return render_template('socket_result.html', data=e) # 에러 발생 시 해당 에러를 화면에 출력함
        
        return render_template('socket_result.html', data=retData) # 요청에 대한 응답 내용을 화면에 출력함


# admin 페이지 (POST 메소드만 허용됨)
@app.route('/admin', methods=['POST'])
def admin():
    # 플래그 획득을 위해 만족해야 하는 조건
    if request.remote_addr != '127.0.0.1': # 1) 클라이언트 ip가 127.0.0.1인지 확인함 (Host: 127.0.0.1)
        return 'Only localhost'

    if request.headers.get('User-Agent') != 'Admin Browser': # 2) 요청의 User-Agent 헤더 값이 Admin Browser인지 확인함 (User-Agent: Admin Browser)
        return 'Only Admin Browser'

    if request.headers.get('DreamhackUser') != 'admin': # 3) 요청의 DreamhackUser 헤더 값이 admin인지 확인함 (DreamhackUser: admin)
        return 'Only Admin'

    if request.cookies.get('admin') != 'true': # 4) 요청의 admin 쿠키의 값이 true인지 확인함 (Cookie: admin=true)
        return 'Admin Cookie'

    if request.form.get('userid') != 'admin': # 5) POST 요청의 Body에 userid 파라미터 값으로 admin을 주고 있는지 확인함 (userid=admin)
        return 'Admin id'

    return FLAG # 위의 5개의 조건을 모두 만족하면 플래그 획득

app.run(host='0.0.0.0', port=8000) # 플라스크에서 8000번 포트로 실행 → 사이트의 port에 8000을 입력함
```

<br/><br/>

## 문제 풀이
