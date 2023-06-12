# [Dreamhack Wargame] sql injection bypass WAF Advanced
### [🚩sql injection bypass WAF advanced](https://dreamhack.io/wargame/challenges/416/)
  <img width="1073" alt="bypassWAF-advanced_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/61d8a907-3af8-4f47-9587-e513f217c733">

<br/><br/>

## 문제 파일 분석
### init.sql
```sql
CREATE DATABASE IF NOT EXISTS `users`; -- users 데이터베이스가 존재하지 않을 경우 CREATE DATABASE로 생성함
GRANT ALL PRIVILEGES ON users.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass'; -- 권한 설정

USE `users`;
CREATE TABLE user( -- CREATE TABLE로 user 테이블 생성 (열: idx, uid, upw)
  idx int auto_increment primary key, -- auto_increment (시작값은 1, 새로운 행이 추가될 때마다 1씩 자동 증가)
  uid varchar(128) not null,
  upw varchar(128) not null
);

-- user 테이블에 INSERT INTO 구문을 통해 adcde, admin, guest, test, dream 계정과 각 계정의 비밀번호를 삽입함
INSERT INTO user(uid, upw) values('abcde', '12345');
INSERT INTO user(uid, upw) values('admin', 'DH{**FLAG**}');
INSERT INTO user(uid, upw) values('guest', 'guest');
INSERT INTO user(uid, upw) values('test', 'test');
INSERT INTO user(uid, upw) values('dream', 'hack');
FLUSH PRIVILEGES;
```
* init.sql 실행 결과로 생성된 users 데이터베이스의 user xpdlqmf
    | idx(primary key) | uid | upw |
    |:---:|---|---|
    | 1 | 'abcde' | '1234' |
    | 2 | 'admin' | 플래그 |
    | 3 | 'guest' | 'guest' |
    | 4 | 'test' | 'test' |
    | 5 | 'dream' | 'hack' |

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
<pre>{result}</pre><hr/>
<form>
    <input tyupe='text' name='uid' placeholder='uid'>
    <input type='submit' value='submit'>
</form>
'''

keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', 
            '\n', '\r', '\t', '\x0b', '\x0c', '-', '+'] # 필터링할 키워드 목록
def check_WAF(data): # 필터링 함수 (목록에 포함된 키워드들이 존재하는지 검사함)
    for keyword in keywords:
        if keyword in data.lower(): # 필터링 키워드 검사 시 소문자로 모두 변환한 후 검사 (대소문자 혼용을 이용한 필터링 우회 방지)
            return True # 필터링 키워드가 존재하면 True를 반환함
    return False # 필터링 키워드가 존재하지 않으면 False를 반환함


# 인덱스 페이지
@app.route('/', methods=['POST', 'GET'])
def index():
    uid = request.args.get('uid') # 이용자가 전달한 uid 입력값을 가져옴
    if uid: # uid 인자값이 존재하는 경우 (/?uid= 뒤에 값이 존재함)
        # 필터링할 키워드가 포함되어 있는지 검사하여 존재할 경우 WAF에 의해 요청이 막혔다는 메시지를 출력함
        if check_WAF(uid):
            return 'your request has been blocked by WAF.'
        
        # 필터링 키워드가 요청에 포함되지 않을 경우 아래의 코드를 실행함
        cur = mysql.connection.cursor()
        cur.execute(f"SELECT * FROM user WHERE uid='{uid}';") # uid를 이용해 user 테이블을 조회하는 쿼리문을 생성하고 실행함
        result = cur.fetchone()
        
        if result: # 결과는 조건에 해당하는 행 전체를 반환 (행의 idx, uid, upw)
            return template.format(uid=uid, result=result[1]) # 반환되는 결과 중 두 번째에 해당하는 열의 값을 화면에 출력함
        else: # 결과가 존재하지 않을 경우
            return template.format(uid=uid, result='')

    else:
        return template # uid 인자값이 존재하지 않으면 template 변수의 내용을 그대로 출력함


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

<br/><br/>

## 문제 풀이
