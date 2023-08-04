# [Dreamhack Wargame] simple-ssti
* 출처: 🚩 simple-ssti [🔗](https://dreamhack.io/wargame/challenges/39)
* Reference: Server Side Template Injection (SSTI)
* 문제 설명
  <br/><br/>
  <img width="819" alt="simple-ssti_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f1d71951-e5fe-4353-82dd-94dde5e09ab3">

<br/><br/>

## 문제 파일(app.py) 및 취약점 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, render_template_string, make_response, redirect, url_for
import socket

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

app.secret_key = FLAG # Flask의 비밀 키(secret key)를 플래그로 설정함 → config.items()를 통해 딕셔너리 형태로 정의되어 있는 config를 확인하여 비밀 키를 확인할 수 있음

# 인덱스 페이지
@app.route('/')
def index():
    return render_template('index.html') # 인덱스 페이지를 render_template 함수로 화면에 출력함

# 404 Not Found 에러 발생 시 Error404() 함수에서 사용자 정의 오류 페이지(미리 정의한 Template에 동적인 값을 넣은 것)을 화면에 출력함
@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path) # 사용자가 입력한 request.path 값에 대한 별다른 검증 과정이 없음 (SSTI 발생!)

    # render_template_string 함수로 미리 정의한 템플릿을 화면에 출력함
    return render_template_string(template), 404

app.run(host='0.0.0.0', port=8000)
```
* 404 에러 발생 시 이를 처리하는 ```Error404()``` 함수(```@app.errorhandler```)에서 request.path에 대한 별다른 검증 과정을 수행하지 않아 템플릿 구문을 삽입하여 서버 정보를 탈취할 수 있음
    - Template Engine마다 작동하는 문법이 다름 → 웹 어플리케이션이 사용 중인 템플릿 엔진을 특정해야 함
      <br/><br/>
      <img width="1512" alt="simple-ssti_템플릿 엔진 확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/872e7b4d-7fd6-4cd6-95f9-838ac04d9fa4"><br/>
      
    - FLAG가 ```app.secret_key```로 설정되어 있으므로 **```config``` 딕셔너리의 ```SECRET_KEY```를 확인하면 플래그를 획득할 수 있음**

<br/><br/>

## 문제 풀이 (익스플로잇)
문제 사이트에 접속한 후 주소 뒤에 ```{{config.items()}}```를 덧붙인 후 이를 요청하면 404 에러가 발생하여 SSTI 취약점으로 인해 config 딕셔너리의 key-value 쌍들이 화면에 출력되어 여기서 비밀키로 설정된 플래그를 확인할 수 있음
<br/><br/>
<img width="1512" alt="simple-ssti_문제 풀이" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/1ecbaf4d-9224-4b08-8639-83440a96d511">
