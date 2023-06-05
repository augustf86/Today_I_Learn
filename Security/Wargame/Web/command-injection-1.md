# [Dreamhack Wargame] command-injection-1
### [🚩command-injection-1](https://dreamhack.io/wargame/challenges/44/)
  <img width="1071" alt="command_injection_1_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/30b57c82-a990-4fa4-a946-eb6e5b39b46d">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/env python3
import subprocess

from flask import Flask, request, render_template, redirect

from flag import FLAG

APP = Flask(__name__)



# 인덱스 페이지:
@APP.route('/')
def index():
    return render_template('index.html') # 인덱스 페이지(index.html)를 화면에 출력함



# ping 페이지: 이용자에게 HOST를 입력 받아 해당 주소로 ping 명령을 실행하는 페이지
@APP.route('/ping', methods=['GET', 'POST'])
def ping():
    if request.method == 'POST': # POST 메소드로 요청할 경우
        host = request.form.get('host') # 이용자가 전달한 host를 가져옴
        cmd = f'ping -c 3 "{host}"' # 명령 생성(3개의 echo request를 host로 보냄)
        try:
            output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=5)
            return render_template('ping_result.html', data=output.decode('utf-8')) # 정상적인 ping 응답을 수신한 경우 ping 수행 결과를 화면에 출력함
        except subprocess.TimeoutExpired:
            return render_template('ping_result.html', data='Timeout !') # 타임아웃 발생
        except subprocess.CalledProcessError:
            return render_template('ping_result.html', data=f'an error occurred while executing the command. -> {cmd}') # 명령 수행 시 에러 발생

    # GET 메소드로 요청할 경우
    return render_template('ping.html') # 이용자에게 ping 명령을 수행하기 위해 필요한 Host를 입력 받는 페이지(ping.html)을 화면에 출력함


if __name__ == '__main__':
    APP.run(host='0.0.0.0', port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
ping 명령어의 IP 또는 도메인주소(```host```)를 이용자에게 입력받아 ping 명령을 생성함
```python
cmd = f'pint -c 3 "{host}"'
```
* command-injection-1 문제 사이트의 ping 페이지 내에서 메타 문자를 삽입해 ping을 시도하면 ```<input>``` 태그의 pattern 속성으로 인해 요청한 형식과 일치시키라는 경고가 뜸
  <img width="1155" alt="pattern 속성 확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9d246f05-9acc-411a-9fed-b9bcc8d59d45">
  
    - 크롬 개발자 도구의 Elements 탭에서 해당 부분의 코드를 (일시적으로) 수정하여 실행시킬 수 있음
    - 클라이언트에서 pattern 속성으로 이용자의 입력값을 제한하고 있지만, **서버에서는 이용자의 입력값을 검사하는 부분이 없기 때문**에 메타 문자를 삽입하여 Command Injection 공격을 수행할 수 있음


<br/><br/>

### 익스플로잇
1. 크롬 개발자 도구를 열고 ```<input>``` 태그의 pattern 속성을 지운 후 ```127.0.0.1; cat flag.py```를 입력한 후 Ping 버튼을 누름
  <img width="1155" alt="ci1_1번" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/228af2be-64f9-43ef-924c-71c67f4e55f5"><br/>
    - ```;``` 구분 문자(명령어 구분자)는 앞 명령어의 에러 유무와 관계 없이 뒤의 명령어를 실행하게 만듦
        + ```ping -c 3 127.0.0.1```이 실행된 후 ```cat flag.py```가 실행됨


2. 1번 결과로 에러 페이지가 출력됨
  <img width="1155" alt="ci1_2번" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8fb0b1ea-fc60-4f4c-8fe5-47f06335c5d1"><br/>
    - 결과를 보면 입력이 ```"```, ```"```로 감싸져 있는 것을 확인할 수 있음

3. ```<input>``` 태그의 pattern 속성을 다시 지우고  ```"``` 문자를 삽입하여 ```127.0.0.1"; cat "flag.py```를 입력한 후 Ping 버튼을 누르면 ping 명령어 수행 결과 바로 뒤에 flag가 출력됨
  <img width="1155" alt="ci1_3번" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f2085213-cd68-48fc-8077-c2a7ce40cd0f">
  <img width="1155" alt="ci1_result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/074afca9-72b7-49b3-9c45-932bc5f5386b">
