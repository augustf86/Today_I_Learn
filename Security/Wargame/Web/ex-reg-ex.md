# [Dreamhack Wargame] ex-reg-ex
* 출처: 🚩 ex-reg-ex [🔗](https://dreamhack.io/wargame/challenges/834/)
* Reference: 정규표현식
* 문제 설명
  <br/><br/>
  <img width="1184" alt="ex-reg-ex_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/50007b1c-f472-442e-a14a-6dffd4ab1ab5">

<br/><br/>

## 문제 파일(app.py) 및 취약점 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template
import re

app = Flask(__name__)

try:
    FLAG = open("./flag.txt", "r").read()       # flag is here!
except:
    FLAG = "[**FLAG**]"

# 인덱스 페이지
@app.route("/", methods = ["GET", "POST"])
def index():
    input_val = ""
    if request.method == "POST": # POST 메소드로 요청 시
        input_val = request.form.get("input_val", "") # 이용자가 입력한 input_val 값을 가져옴
        m = re.match(r'dr\w{5,7}e\d+am@[a-z]{3,7}\.\w+', input_val) # 이용자의 입력 값이 정규표현식을 만족하는지 확인함
        if m: # 정규표현식을 만족하는 경우
            return render_template("index.html", pre_txt=input_val, flag=FLAG) # 화면에 이용자의 입력값과 FLAG를 출력함
    
    # GET 메소드로 요청 시
    return render_template("index.html", pre_txt=input_val, flag='?') # 화면에 이용자의 입력값과 Flag: ?를 출력함

app.run(host="0.0.0.0", port=8000)
```
* 정규표현식 ```dr\w{5,7}e\d+am@[a-z]{3,7}\.\w``` 분석
    | | 설명 | 만족하는 문자 |
    |:---:|------|----|
    | dr | 문자 dr과 매칭됨 | ```dr``` |
    | \w{5, 7} | ***\w***: alphanumeric&underscore 범위의 모든 문자와 매칭됨 <br/> ***{5, 7}***: \w의 매칭 범위가 5~7 사이임을 의미함 → 문자의 길이는 5~7이어야 함 | ```abcde```, ```aaaae```, ```abcdef``` 등 |
    | e | 문자 e와 매칭됨 | ```e``` |
    | \d+ | ***\d***: 0~9 사이의 숫자를 의미함 <br/> ***+***: \d의 매칭 범위가 1 또는 그 이상임을 의미함 → 숫자의 길이는 1 이상이면 됨 | ```1```, ```11``` 등 |
    | am@ | 문자 am@와 매칭됨 | ```am@```
    | [a-z]{3, 7} | ***[a-z]***: a-z 범위 내의 문자와 매칭됨 <br/> ***{3, 7}***: [a-z]의 매칭 범위가 3~7 사이임을 의미함 → 길이는 3~7이어야 함 | ```awer```, ```awerds``` 등 |
    | \\. | 문자 .과 매칭됨 | ```.``` |
    | \w | alphanumeric&underscore 범위의 모든 문자와 매칭됨 (매칭 문자 길이는 1) | ```a```, ```w``` 등 |
    - 정규표현식을 만족하는 전체 입력값 예시: ```drabcdee1am@awer.w```, ```dralicee12am@dream.d``` 등

<br/><br/>

## 문제 풀이(익스플로잇)
