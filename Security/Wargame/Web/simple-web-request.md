# [Dreamhack Wargame] simple-web-request
* 출처: 🚩 simple-web-request [🔗](https://dreamhack.io/wargame/challenges/830/)
* Reference: X (코드 분석만으로 풀 수 있음)
* 문제 설명
  <br/><br/>
  <img width="1187" alt="simple-web-request_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/28b4868a-b0a8-4408-bdf2-adfd22267c60"><br/>

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
import os
from flask import Flask, request, render_template, redirect, url_for
import sys

app = Flask(__name__)

try: 
    # flag is here!
    FLAG = open("./flag.txt", "r").read()      
except:
    FLAG = "[**FLAG**]"


# 인덱스 페이지
@app.route("/")
def index():
    return render_template("index.html")


# step1 페이지
@app.route("/step1", methods=["GET", "POST"])
def step1():

    #### 풀이와 관계없는 치팅 방지 코드
    global step1_num
    step1_num = int.from_bytes(os.urandom(16), sys.byteorder)
    ####

    if request.method == "GET": # GET 메소드로 요청 시
        # 이용자가 인자로 입력한 param, param2의 값을 가져옴 → /step1?param=값&param2=값 형태
        prm1 = request.args.get("param", "")
        prm2 = request.args.get("param2", "")
        
        # 화면에 출력할 param, param2 텍스트를 생성함
        step1_text = "param : " + prm1 + "\nparam2 : " + prm2 + "\n"
        
        # param의 값이 getget이고, param2의 값이 rerequest임 → /step1?param=getget&param2=rerequest
        if prm1 == "getget" and prm2 == "rerequest":
            return redirect(url_for("step2", prev_step_num = step1_num)) # /step2 페이지로 이동함
        return render_template("step1.html", text = step1_text) # 조건을 만족하지 못하면 입력한 인자들을 화면에 출력함
    else:  # POST 메소드로 요청 시
        return render_template("step1.html", text = "Not POST")

# step2 페이지
@app.route("/step2", methods=["GET", "POST"])
def step2():
    if request.method == "GET": # GET 메소드로 요청 시

    #### 풀이와 관계없는 치팅 방지 코드
        if request.args.get("prev_step_num"):
            try:
                prev_step_num = request.args.get("prev_step_num")
                if prev_step_num == str(step1_num):
                    global step2_num
                    step2_num = int.from_bytes(os.urandom(16), sys.byteorder)
                    return render_template("step2.html", prev_step_num = step1_num, hidden_num = step2_num)
            except:
                return render_template("step2.html", text="Not yet")
        return render_template("step2.html", text="Not yet")
    ####

    else: 
        return render_template("step2.html", text="Not POST")

    
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET": # GET 메소드로 요청 시
        return render_template("flag.html", flag_txt="Not yet")
    else: # POST 메소드로 요청 시

        #### 풀이와 관계없는 치팅 방지 코드
        prev_step_num = request.form.get("check", "")
        try:
            # step2에서 계산한 랜덤한 값을 가지고 있을 경우(= step2를 거친 경우)
            if prev_step_num == str(step2_num):
        ####

                # 이용자의 입력값 param, parma2의 값을 가져옴
                prm1 = request.form.get("param", "")
                prm2 = request.form.get("param2", "")
                
                # param의 값이 poost이고, param2의 값이 requeeestf인 경우 FLAG 획득 가능
                if prm1 == "pooost" and prm2 == "requeeest":
                    return render_template("flag.html", flag_txt=FLAG)
                else: # 아니면 step2로 이동함
                    return redirect(url_for("step2", prev_step_num = str(step1_num)))
            
            # step2를 거치지 않은 경우
            return render_template("flag.html", flag_txt="Not yet")
        except:
            return render_template("flag.html", flag_txt="Not yet")
            

app.run(host="0.0.0.0", port=8000)
```

<br/><br/>

## 문제 풀이
