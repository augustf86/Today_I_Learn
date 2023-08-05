# Web Hacking: Server-side Basic
### 🔖 출처
* [Dreamhack Lecture] Server-side Basic [🔗](https://dreamhack.io/lecture/courses/15)
* [Dreamhack Web Hacking] Server Side: SQL Injection [🔗](https://dreamhack.io/lecture/courses/191)
* [Dreamhack Web Hacking] Server Side: Command Injection [🔗](https://dreamhack.io/lecture/courses/187)
* [Dreamhack Web Hacking] Server Side: SSRF [🔗](https://dreamhack.io/lecture/courses/190)
* [Dreamhack Web Hacking] Server Side: File Vulnerability [🔗](https://dreamhack.io/lecture/courses/202)

<br/><br/>

## Server-side(서버 사이드) 취약점
| | 설명 |
|:---:|------|
| 정의 | 서버에서 사용자가 요청한 데이터를 해석하고 처리한 후 사용자에게 응답하는 과정에서 **웹 어플리케이션이나 데이터베이스와 <br/>같은 서버의 자원을 사용해 처리할 때** 사용자의 요청 데이터에 의해서 발생하는 취약점 |
| 공격 대상 | 웹 서비스를 제공하는 <U>서버</U> |
| 공격 목적 | 서버 내에 존재하는 사용자들의 정보를 탈취하거나, 서버의 권한을 장악함 |
| 발생 원인 | - HTTP 요청 시 악의적인 사용자가 메소드나 요청 헤더 등을 포함한 모든 데이터를 조작하여 전송할 수 있기 때문에 발생함 <br/> - **개발과 운영 상의 실수**로 사용자의 데이터에 대한 검증 과정의 부재 또는 올바르지 않은 검증으로 인해 발생함 |
| 기본 방지 대책 | 📌 ***서버는 사용자로부터 받는 모든 입력을 신뢰하지 않도록 해야 함*** |

<br/>

* Server-side 취약점의 분류
    | 분류 | 설명 |
    |:---:|------|
    | ***Injection*** <br/> (인젝션) | 서버의 처리 과정 중 사용자가 입력한 데이터가 처리 과정의 구조나 문법적으로 사용되어 <br/>발생하는 취약점 |
    | ***File Vulnerability*** | 서버의 파일 시스템에 사용자가 원하는 행위를 할 수 있을 때 발생하는 취약점 |
    | ***Business Logic Vulnerability*** <br/> (비즈니스 로직 취약점) | 인젝션, 파일 관련 취약점들과는 다르게 정상적인 흐름을 악용하는 것 |
    | ***Language specific Vulnerability*** <br/> (PHP, Python, NodeJS) | 웹 어플리케이션에서 사용하는 언어의 특성으로 인해 발생하는 취약점 |
    | ***Misconfiguration*** | 잘못된 설정으로 인해 발생하는 취약점 |

<br/><br/><br/>

## Injection
사용자의 입력 값이 어플리케이션의 처리 과정에서 구조나 문법적인 데이터로 해석되어 발생하는 취약점 <br/> &nbsp;&nbsp; → 실행 흐름 중간에 들어갈 수 있는 사용자의 입력에 **변조된 입력을 주입해 의도하지 않은 행위를 발생**시키는 것을 의미함
* Injection의 종류
    | 종류 | 설명 |
    |:---:|------| 
    | ***SQL Injection*** | SQL를 사용할 때 공격자의 입력값이 정상적인 요청에 영향을 주는 취약점 |
    | ***Command Injection*** | OS Command를 사용 시 사용자의 입력 데이터에 의해 실행되는 Command를 변조할 수 있는 취약점 |
    | ***Server Side Template Injection*** <br/> (SSTI) | 템플릿 변환 도중 사용자의 입력 데이터가 템플릿으로 사용되어 발생하는 취약점 |
    | ***Path Traversal*** | URL/File Path를 사용 시 사용자의 입력 데이터에 의해 임의의 경로에 접근하는 취약점 |
    | ***Server Side Request Forgery*** <br/> (SSRF) | 공격자가 서버에서 변조된 요청을 보낼 수 있는 취약점 |

<br/><br/>

### SQL Injection
| | 설명 |
|:---:|------|
| 정의 | SQL 쿼리에 사용자의 입력값이 삽입되어 사용자가 원하는 쿼리를 실행할 수 있는 취약점 |
| 관련 배경 지식 | SQL (Structured Query Language) [🔗]() <br/> RDBMS (Relational DBMS) [🔗]() |
| 취약점 발생 상황 | 로그인/검색과 같이 사용자의 입력 데이터를 기반으로 DBMS에 저장된 정보를 조회하는 기능을 구현하기 위해 SQL 쿼리에 사용자의 <br/>입력 데이터를 추가하여 DBMS에 요청함 (**📌 웹 애플리케이션과 데이터베이스가 연동되는 부분에서 발생함**) <br/> ***→ 사용자의 입력이 SQL 쿼리에 삽입되어 SQL 구문으로 해석되거나 문법적으로 조작하게 되면 개발자가 의도한 정상적인 SQL <br/>&nbsp;&nbsp;&nbsp;&nbsp;쿼리가 아닌 임의의 쿼리가 실행됨*** |
| 공격 결과 | 현재 쿼리를 실행하는 DBMS 계정의 권한으로 공격이 가능함 <br/> &nbsp;&nbsp; - 일반적으로 조작된 쿼리를 이용하여 인증을 우회하거나 데이터베이스의 내용을 추출﹒변조﹒삭제하는 등의 행위가 가능함 |
| 특징 | ***DBMS의 종류에 따라 조금씩 다른 기법이 필요함*** (각 DBMS마다 SQL Injection 공격이 상이함) <br/> &nbsp;&nbsp; - 기업이 웹 어플리케이션을 구축할 때 가장 많이 사용하는 RDBMS는 Microsoft SQL Server, 오라클 등임 |

<br/>

* SQL Injection 예시: 로그인 기능에서의 인증 우회
    | | 설명 |
    |:---:|------|
    | 로그인 과정 | 시용지기 아이디와 패스워드를 입력해 서버에 전송함 → 서버는 해당 데이터가 데이터베이스에 존재하는지 확인하고 로그인 <br/>성공 여부를 판단함 |
    | SQL Injection <br/> 수행 방법 | - 로그인 과정에서 생성된 SQL 쿼리문에서 문자열 구분자를 확인함 <br/> &nbsp;&nbsp; → **입력에 문자열 구분자를 삽입해 문자열을 탈출하고 뒷 부분에 새로운 쿼리를 작성**하여 전달하면 DBMS에서 사용자가 직접 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 작성한 쿼리를 실행할 수 있음 <br/> - admin으로 로그인한다고 가정했을 때 upw를 모르는 상황이라면 SQL Injection을 통해 **논리적으로 참이 되는 구조를 만들어** <br/> &nbsp;&nbsp; admin으로 로그인하거나 upw를 알아낼 수 있음 |
    - 예시
        ```sql
        # 아이디(uid)와 비밀번호(upw)를 입력받고 DBMS의 user_table에 조회하기 위한 쿼리 (가장 간단한 형태)
        # → {uid}, {upw} 부분에 사용자가 입력한 문자열을 삽입하고 DBMS를 전달해 실행함
        SELECT uid FROM user_tale WHERE uid='{uid}' AND upw='{upw}'
        ```
        - 사용자의 입력과 웹 어플리케이션이 작성한 SQL 쿼리를 해석할 때 문자열 구분자로 ```'``` 문자를 사용하고 있음 <br/> &nbsp;&nbsp; ***→ 사용자의 입력에 ```'``` 문자를 입력하여 문자열을 탈출하고 뒷 부분이 SQL 구문으로 해석되게 만듦***
        - SQL Injection 수행 예시
            - 방법 1: **쿼리 질의를 통해 admin 결과를 반환하는 방법**
                ```sql
                SELECT uid FROM user_table WHERE uid='admin' or '1' AND upw=''
                ```
                | 항목 | 입력 |
                |:---:|------|
                | **uid** | ```admin' or '1``` <br/> &nbsp;&nbsp; - ```or``` 연산: ```A or B```일 때 A와 B 둘 중 하나라도 참(True)인 경우 결과가 참이 됨 |
                | **upw** | 아무 문자열이나 입력 가능 (여기서는 아무것도 입력하지 않음) |
            - 방법 2: **쿼리 질의를 통해 admin의 upw를 알아내는 방법**
                ```sql
                SELECT uid FROM user_table WHERE uid='' AND upw='' UNION SELECT upw FROM user_table WHERE uid='admin'
                ```
                | 항목 | 입력 |
                |:---:|------|
                | **uid** | 아무것도 입력하지 않음 |
                | **upw** | ```' UNION SELECT upw FROM user_table WHERE uid='admin``` <br/> &nbsp;&nbsp; - ```UNION``` 문으로 2개 이상의 ```SELECT``` 문(쿼리)를 결합하고 있음 |
            - 방법 3: **쿼리 질의를 통해 admin 계정의 upw를 변경하는 방법**
                ```sql
                SELECT uid FROM user_table WHERE uid='admin' AND upw=''; UPDATE user_table SET upw='12345' WHERE uid='admin'
                ```
                | 항목 | 입력 |
                |:---:|------|
                | **uid** | ```admin```
                | **upw** | ```'; UPDATE user_table SET upw='12345' WHERE uid='admin``` <br/> &nbsp;&nbsp; → 데이터베이스 쿼리문을 연달아 실행할 수 있게 하는 ```;``` 구분자를 사용함 |

<br/>

* SQL Injection 방지 방법
    - 📌 ***사용자의 입력 데이터가 SQL 쿼리로 해석되지 않아야 함***
    - SQL Injection 발생 이유: **⚠️ 사용자의 입력값에 문자열 구분자(```'```, ```"``` 등)가 삽입되어 본래의 쿼리를 벗어날 수 있음**
        | | 해결 방안 |
        |:---:|------|
        | 과거 | 문자열 구분자 앞에 백슬래시(```\```) 문자를 붙여 **사용자의 입력을 escape**해 사용하는 방식을 자주 이용함 <br/><br/> ⚠️ 현재는 이 방식을 권장하지 않음 <br/> &nbsp;&nbsp; - 사용자로부터 입력을 받는 타입이 문자일일 수도 있지만 숫자 타입일 수도 있기 때문 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 문자열 구분자를 escape해도 숫자 뒤에 공백 문자(```' '```, ```\n``` 등)를 넣는 것만으로도 쿼리에 사용자의 입력을 삽입할 수 있음 | 
        | 현재 | ORM과 같이 검증된 SQL 라이브러리를 사용하는 것을 권장함 <br/> &nbsp;&nbsp; - 개발자가 직접 쿼리르 작성하는 Raw 쿼리를 사용하지 않아도 기능 구현이 가능함 <br/> &nbsp;&nbsp; - SQL Injection으로부터 상대적으로 안전함 <br/><br/> ⚠️ ***ORM을 사용하더라도 입력 데이터의 타입 검증이 없으면 잠재적인 위협이 있을 수 있음 → 반드시 입력 데이터의 타입 검증이 필요함***<br/>💡 **ORM**(Object Relational Mapper): SQL의 쿼리 작성을 돕기 위한 라이브러리 <br/> &nbsp;&nbsp;&nbsp;&nbsp; - 생산성과 SQL 쿼리의 안전한 실행을 위해서 사용함 <br/> &nbsp;&nbsp;&nbsp;&nbsp; - *사용자의 입력값을 라이브러리 단에서 스스로 escape하고 쿼리애 매핑시킴* |
        + ORM 예시: python SQLAlchemy
            ```python
            from flask_sqlalchemy import SQLAlchemy

            ...
            db = SQLAlchemy(...)
            ...

            class User(db.Model):
                id = db.Column(db.Integer, primary_key=True)
                uid = db.Column(...)
                upw = db.Columb(...)
                ...
            
            User.query.filter(User.uid == uid, User.upw == upw).all()
            ```

<br/>

* SQL Injection 응용: ***Blind SQL Injection***
    | | 설명 |
    |:---:|------|
    | 정의 | 참/거짓 반환 결과 또는 정상적인 쿼리문을 보냈을 때와 삽입 공격이 성공했을 때의 차이(시간 차이 등)으로 데이터를 획득하거나 취약성 <br/>여부를 판단하는 공격 기법 |
    | 사용 배경 | 질의 결과(SQL Injection 공격의 성공 여부)를 공격자가 화면에서 직접 확인하지 못할 때 사용함 |
    | 특징 | 한 바이트씩 비교해서 공격하는 방법이므로, 다른 공격에 비해 시간이 많이 소요됨 <br/> &nbsp;&nbsp; → **공격을 자동화하는 스크립트**를 작성하여 시간을 단츅시킬 수 있음 <br/> 공격하기 전에 이용자가 입력할 수 있는 모든 문자의 범위를 지정해야 함 |
    - Background: 공격 시 사용하는 함수
        | 함수 | 설명 |
        |:---:|------|
        | ```ascii(char)``` | 전달된 문자를 아스키 형태로 변환하는 함수 <br/> &nbsp;&nbsp; - ```ascii(a)```의 실행 결과 'a' 문자의 아스키 값인 97이 반환됨 |
        | ```substr(string, position, length)``` | 문자열에서 지정한 위치(position)부터 길이(length)까지의 값을 가져옴 <br/> &nbsp;&nbsp; - ```substr('ABCD', 2, 2)```의 실행 결과 'BC'가 반환됨 |
    - Blind SQL Injection 예시: 로그인 기능에서 비밀번호를 획득하는 방법
        ```sql
        # 아이디(uid)와 비밀번호(upw)를 입력받고 DBMS의 user_table에 조회하기 위한 쿼리 (가장 간단한 형태)
        # → {uid}, {upw} 부분에 사용자가 입력한 문자열을 삽입하고 DBMS를 전달해 실행함
        SELECT uid FROM user_table WHERE uid='{uid}' AND upw='{upw}'
        ```
        + 사용자의 입력과 웹 어플리케이션이 작성한 SQL 쿼리를 해석할 때 문자열 구분자로 ```'``` 문자를 사용하고 있음 <br/> &nbsp;&nbsp; ***→ 사용자의 입력에 ```'``` 문자를 입력하여 문자열을 탈출하고 뒷 부분이 SQL 구문으로 해석되게 만듦***
        + 쿼리 질의를 통해 admin의 upw를 한 자리씩 알아내는 방법
            ```sql
            # admin의 upw가 strawberry인 경우

            # 첫 번째 문자(substr(upw, 1, 1)) 구하기
             SELECT * FROM user_table WHERE uid=‘admin’ AND ascii(substr(upw, 1, 1))=114 --' AND upw=‘’ # False (114 = 'r')
            SELECT * FROM user_table WHERE uid=‘admin’ AND ascii(substr(upw, 1, 1))=115 --' AND upw=‘’ # True (115 = 's')

            # 두 번째 문자(substr(upw, 2, 1)) 구하기
            SELECT * FROM user_table WHERE uid=‘admin’ AND ascii(substr(upw, 2, 1))=115 --‘ AND upw=‘’ # False (115 = 's')
            SELECT * FROM user_table WHERE uid=‘admin’ AND ascii(substr(upw, 2, 1))=116 --' AND upw=‘’ # True (116 = 't')
            ```
            | 항목 | 입력 |
            |:---:|------|
            | **uid** | ```admin' AND ascii(substr(upw, 시작위치, 1))=아스키코드 --``` <br/> &nbsp;&nbsp; - ```시작위치```: 1에서부터 하나씩 늘려가며 검사함 <br/> &nbsp;&nbsp; - ```아스키 코드```: 비밀번호의 경우 일반적으로 알파벳 대소문자, 숫자, 특수문자 범위로 지정함 <br/> &nbsp;&nbsp; - ```--``` 문자: 해당 문자 뒤에 오는 문장을 주석 처리하여 SQL 쿼리로 실행되지 않도록 만듦 |
            | **upw** | 아무 문자열이나 입력 가능 (야기서는 아무것도 입력하지 않음) |
        + 공격 자동화 스크립트 작성 방법
            ```python
            #!/usr/bin/python3

            import requests
            import string

            url = 'https://example.com/login' # 로그인 기능의 URL 입력
            params = {
                'uid': '',
                'upw': ''
            }

            # 비밀번호가 포함될 수 있는 문자(알파벳, 숫자, 특수문자)를 string 모듈을 사용해 생성함
            tc = string.ascii_letters + string.digits + string.punctuation
            # abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~

            # uid 입력값에 해당하는 변수 query
            query = '''
            admin' and ascii(substr(upw, {idx}, 1)) = {val}--
            '''

            # password를 저장할 변수
            password = ''

            # 길이를 먼저 구하고 길이만큼 반복문을 돌릴 수도 있음
            for idx in range(0, 20): # 한 바이트씩 모든 문자를 비교하는 반복문
                for ch in tc:
                    params['uid'] = query.format(idx=idx, val=ord(ch)).strip("\n")
                    c = requests.get(url, params=params)
                    print(c.request.url)
                    if c.text.find("Login success") != -1: # 반환 결과가 참일 경우 페이지에 표시되는 "Login success" 문자열을 찾음
                        password += chr(ch) # 해당 결과를 반환하는 문자를 password 변수에 추가함
                        break
            
            # 반복문을 마치고 나면 admin 계정의 비밀번호를 획득할 수 있음
            print(f"Password is {password}")
            ```

<br/><br/>

### Command Injection
| | 설명 |
|:---:|------|
| 정의 | 이용자의 입력을 시스템 명령어로 실행하게 하는 취약점 |
| 발생 상황 | 명령어를 실행하는 함수에 이용자가 임의의 인자를 전달할 수 있을 때 발생함 <br/> → ***OS Command를 사용할 때 사용자의 입력값을 검증하지 않고 함수의 인자로 전달하는 경우 특수 문자를 이용해 사용자가 원하는 명령어를 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 함께 실행할 수 있음*** |
| 공격 결과 | 공격에 사용되면 웹 애플리케이션에서 임의 명령어를 실행할 수 있어 **공격 파급력이 높음** <br/> &nbsp;&nbsp; - 현재 Command를 실행하는 웹 어플리케이션의 권한으로 임의 명령어를 실행할 수 있음 |

<br/>

* Background
    - 시스템 함수(System Function)
        + 웹 어플리케이션에서 전달된 인자를 셸 프로그램에 전달해 OS Command(시스템 명령어)를 실행하기 위한 함수
            | 언어 | 시스템 함수 |
            |:---:|------|
            | **PHP** | ```system``` 함수 |
            | **NodeJS** | ```child_process``` 함수 |
            | **Python** | ```os.system``` 함수 |
    - OS Command(시스템 명령어)
        | | 설명 |
        |:---:|------|
        | 정의 | linux에서 사용하는 ```ls```, ```pwd```, ```ping```, ```zip``` 등과 Windows에서 사용하는 ```dir```, ```pwd```, ```ping``` 등과 같이 **OS에서 <br/>사용하는 명령어** |
        | 사용하는 이유 | 이미 기능을 구현한 OS 실행 파일이 존재할 때 코드 상에서 다시 구현하지 않고 이를 실행하면 더 편리하기 때문 |
        | 내부 수행 과정 | 내부적으로 셸(Shell)을 이용해 실행함 <br/> &nbsp;&nbsp; - 셸은 사용자 편의를 위해 **한 줄에 여러 명령어를 실행하는 특수 문자**가 여럿 존재함 |
        | 취약점 발생 | OS Command를 사용할 때 **사용자의 입력값을 검증하지 않고** 함수의 인자로 전달함 <br/> &nbsp;&nbsp; → ```&&```, ```;```, ```\|``` 등과 같이 명령어를 연속으로 실행시키는 메타 문자를 사용해 임의 명령어를 함께 실행시킬 수 있음 |
        + 셸 프로그램이 지원하는 메타 문자 <br/><br/>
          <img width="704" alt="메타 문자" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/d5a8386b-7388-4797-83af-b541bf45fe7e"><br/>

<br/>

* Commnad Injection 예시: ping 명령어 실행
    ```python
    @app.route('/ping')
    def ping():
        ip = request.args.get('ip') # 사용자가 입력한 ip 값(주소)를 가져옴
        return os.system(f'ping -c 3 {ip}') # 사용자가 입력한 ip 주소로 ping 명령어를 실행함
    ```
    - 정상적인 사용자와 악의적인 공격자의 차이
        | | 설명 |
        |:---:|------|
        | 정상적인 사용자 | **일반적인 IP나 도메인 주소를 ip의 값으로 전달함** → 해당 주소로 ping 명령어를 수행함 |
        | 악의적인 공격자 | **메타 문자를 입력**하여 ping 명령어 외에도 다른 OS Command를 실행하도록 할 수 있음 <br/> - ```127.0.0.1 && id```, ```127.0.0.1; id```, ```127.0.0.1 \| id```와 같이 명령어 구분자, 명령어 연속 실행, 또는 파이프 <br/> &nbsp;&nbsp; 메타 문자를 이용해 임의 명령어(id)를 실행할 수 있음 | 

<br/>

* Commnad Injection 방지 방법 → ***📌 사용자의 입력 데이터가 Command 인자가 아닌 다른 값으로 해석되는 것을 방지해야 함***
    - 가장 좋은 방법 → 웹 어플리케이션에서 **OS Command를 사용하지 않는** 것
        + ⚠️ *OS Command를 사용할 경우 해당 Command 내부에서 다른 취약점이 발생하는 등 잠재적인 위협이 될 수 있음*
        + 웹 어플리케이션에서 OS Command가 필요한 경우 이를 대체하는 방법
            - 필요한 OS Command가 **라이브러리의 형태로 구현**되어 있는 경우 해당 라이브러리를 사용함
                + 예시
                    ```python
                    #! pip install ping3
                    # https://github.com/kyan001/ping3/blob/master/ping3.py

                    import ping3 # ping3: 소켓 프로그래밍을 통해 ping 기능을 구현한 라이브러리 (OS command인 ping을 대체함)

                    ping3.ping(ip)
                    ```
                + ⚠️ *라이브러리의 보안성 및 안전성 등을 검토하고 사용해야 함!*
            - 필요한 OS Command가 **라이브러리 형태로 구현되어 있지 않은** 경우 직접 프로그램 코드로 포팅해 사용함
                + execve args 인자로 사용하는 방법
                    ```python
                    subprocess.Popen(['ping', '-c', '3', ip]) # shell meta 문자로 해석되지 입력값을 넣음
                    ```
    - OS Command에 사용자의 입력 데이터를 사용해야 하는 경우 **필터링을 사용함** → Allowlist/Blocklist 필터링 방식
        + **Allowlist 필터링 방식**: 필요한 특정 입력값만 받아들이고 그 외의 모든 값들은 필터링하는 방식
            - 정규식을 통한 Allowlist 방식의 필터링
                ```python
                # ping을 보내는 페이지의 경우 사용자가 입력한 ip가 정상적인 ip 형식인지 정규식으로 검증한 후 사용할 수 있음
                
                import re, os, ...
                ...
                chk_ip = re.compile('^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$')

                if bool(chk_ip.match(ip)): # 정규식을 만족하는 경우 (IP 형식의 입력인 경우) ping 명령어를 실행함
                    return run_system(f'ping -c 3 {ip}')
                else:
                    return 'ip format error'
                ```
        + **Blocklist 필터링 방식**: 알려진 악성 패턴에 한해서만 필터링하고 그 외의 모든 값들은 받아들이는 방식
            - OS Command에서 메타 문자로 사용되는 값을 필터링하고 따옴표로 감싸는 방법
                ```python
                # ping을 보내는 페이지의 경우 사용자가 입력한 ip를 따옴표(Single Quotes)로 감싸서 사용할 수 있음
                if '\' in ip:
                    return 'not allow character'
                return run_system(f'ping -c 3 \'{ip}\'')
                ```

<br/><br/>

### Server Side Template Injection (SSTI)
| | 설명 |
|:---:|------|
| 정의 | Template Engine을 이용하여 웹 어플리케이션을 구동할 때 사용자의 입력값이 적절하게 필터링되지 않아 템플릿 구문을 삽입할 수 있는 취약점 |
| 발생 상황 | Template 내부에서 사용되는 context가 아닌 **Template source에 사용자 입력이 들어가는 경우**에 발생함 <br/> &nbsp;&nbsp; - 사용자의 입력 데이터가 **Template에 직접 사용**되면 Template Engine이 실행하는 문법을 사용할 수 있기 때문에 발생함 |
| 공격 결과 | 개발자가 의도히지 않은 임의의 Template 기능을 실행할 수 있음 <br/> &nbsp;&nbsp; - ⚠️ Server side에서 발생하기 때문에 서버가 가지고 있는 **민감한 정보를 탈취**할 수 있음 |
| 방지 방법 | - 사용자의 입력 데이터를 Template Source에 삽입되지 않도록 해야 함 <br/> - 사용자의 입력 데이터를 Template에서 출력할 때는 Template Source에 값을 넣어 출력해야 함 |

<br/>

* Background: Template Engine
    - Template Engine을 사용하는 이유
        + 웹 어플리케이션에서 **동적인 내용을 HTML로 출력할 때** 미리 정의한 Template에 동적인 값을 넣어 출력하기 위해 사용함
        + 예시: 사용자의 정보를 출룍하는 페이지 (Template Engine으로 jinja2를 사용해 Render함)
            ```python
            from flask import ...
            ...

            @app.route('/user_info')
            def user_info():
                ...
                # Template을 생성함 ({{}}위치에 변수를 넣고 렌더할 때 해당 변수에 들어있는 값을 화면에 출력함)
                template = '''
                <html>
                    <body>
                        <h3> 유저 아이디: {{user.uid}}</h3> 
                        <h3> 유제 레벨: {{user.level}}</h3>
                    </body>
                </html>'''
            
                # 생성한 Template를 Render할 때 guest 유저가 접근한 경우 guest 유저의 아이디와 레벨이 화면에 표시됨
                return render_template_string(template, user=user)
            ```
    - 각 언어별 사용되는 Template Engine
        | 언어 | Template Engine |
        |:---:|------|
        | **Python** | Jinja2, Mako, Tornaod, ... |
        | **PHP** | Smarty, Twig, ... |
        | **Javascript** | Pug, Marko, EJS, ... |
        + 대부분의 Template 엔진에서 ```{{2*3}}```, ```${2*3}```과 같은 문법을 지원함

<br/>

* SSTI 취약점 예시: Title과 Content를 입력하는 게시판 기능
    ```python
    ...
    
    class Secret(object):
        def __init__(self, username, password):
            self.username = username
            self.password = password
    
    secret = Secret('admin', secret_password) # 📌 SSTI 취약점을 이용해 secret 변수의 password를 탈취

    @app.route('/board')
    def board():
        title = request.form['title']
        content = request.form['content']
        template = '''
        <html>
            <body>
                <h3 id="title">{{title}}</h3>
                <h3 id="content">%s</h3>
            </body>
        </html>
        ''' % content # 사용자가 입력한 content 값에 대한 별다른 검증 없이 사용하고 있음 → ⚠️ jinja2 구문을 삽입하여 서버 정보 탈취 가능
    
        # render_template_string 함수 실행 시 template으로 사용되는 데이터가 사용자의 입력 데이터에 의해 변조될 수 있음 (⚠️ SSTI 발생)
        return render_template_string(template, title=title, secret=secret)
    ```
    - SSTI 취약점 공격 과정
        | 순서 | 설명 |
        |:---:|------|
        | 01 | 템플릿 엔진마다 작동하는 문법이 다르기 때문에 웹 어플리케이션이 사용 중인 템플릿 엔진을 특정해줘야 함 <br/> &nbsp;&nbsp; - 예시에서는 Flask의 jinja2 템플릿 엔진을 사용하고 있음 |
        | 02 | content에 ```{{secret.password}}```를 입력하면 template Engine이 이를 해석하여 secret의 password를 화면에 출력함 <br/> &nbsp;&nbsp; - ```{{...}}``` 구문: 중괄호 안에 들어가는 내용을 동적으로 보여줌 → 이를 이용해 공격함 |

<br/><br/>

### Path Traversal
| | 설명 |
|:---:|------|
| 정의 | URL/File Path를 사용할 때 사용자의 입력값에 메타문자를 삽입하여 임의의 경로에 접근 가능한 취약점 <br/> &nbsp;&nbsp; - *📌 File Patth에서 발생하는 Path Traversal은 File Vulnerability를 참고* <br/> &nbsp;&nbsp; - 여기서는 URL에서 발생하는 Path Traversal만 다룸 |
| 발생 상황 | 사용자의 입력 데이터가 적절한 검증 없이 URL/File Path에 직접적으로 사용될 경우에 발생함 <br/> → ⚠️ 사용자의 입력 데이터가 URL Path에 사용될 경우 **URL 구분 문자를 사용하지 못하도록 하는 필터링/인코딩이 부재한 경우**에 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 발생함|
| 공격 효과 | 설계 및 개발 당시에 **의도하지 않은 임의의 경로에 접근**할 수 있음 <br/> &nbsp;&nbsp; - URL에 구분 문자를 삽입하여 의도한 경로가 아닌 상위 경로에 접근해 다른 API를 호출할 수 있음 |
| 방지 방법 | URL Encoding과 같은 인코딩을 사용하여 사용자의 입력 데이터에 포함된 구분문자를 인식하지 않도록 만듦 |

<br/>

* Background: 구분 문자(Delimeter)
    - 일반 텍스트 또는 데이터 스트림에서 별도의 독립적 영역 사이의 경계를 지정하는 데 사용하는 하나의 문자 혹은 문자들의 배열
    - **URL/File Path**에 사용되는 구분 문자
        | 문자 | 의미 |
        |:---:|------|
        | ```/``` | Path identifier (경로 구분자) |
        | ```..``` | Parent directory → 상위 디렉터리를 의미함 <br/> - EX: ```/tmp/test/..a``` 경로의 해석 결과 ```/tmp/a```를 나타냄 |
        | ```?``` | Qeury identifier → ```?``` 뒤는 query로 해석함 |
        | ```#``` | Fragment Idneitifer → ```#``` 뒤의 값은 Server로 전달되지 않음 |
        | ```&``` | Parameter separator → ```key1=value1&key2=value2``` 형식으로 사용됨 |
        + 💡 구분 문자에 따라 URL/File Path의 해석이 달라질 수 있음

<br/>

* Path Traversal 예시: URL Path Traversal
    - 외부에서 접근이 가능한 External Server와 내부망에서만 접근이 가능한 Internal Server로 나뉘어 작동하는 페이지의 코드
        + 외부에서 접근이 가능한 External Server의 코드
            ```python
            # https://external.deramhack.io/
            ...

            def api_get(url):
                return get('https://internal.dreamhack.io/' + url).text
            
            @app.route('/get_info', methods=['POST', 'GET'])
            def get_into():
                result = ''
                if request.method == 'POST': # POST 메소드로 요청하는 경우
                    # 사용자가 입력한 username 값을 가져와 api_get 함수에 '/api/user/username' 값을 전달함
                    username = requests.form['username']
                    result = api_get('api/user/' + username) # api_get 함수 수행 결과 https://internal.dreamhack.io/api/user/username에 요청을 전달하고 그 응답을 result에 저장함
                return render_template('get_info.html', result=result)
            
            ...
            ```
            - ⚠️ 사용자의 입력 데이터(```username```)에 대한 필터링이나 인코딩이 부재함 ***→ Path Traversal 공격이 가능함***
        + 내부에서만 접근이 가능한 Intenral Server의 코드
            ```python
            # https://internal.dreamhack.io/
            ...

            # index page란 문자열을 화면에 출력함
            @app.route('/')
            def index():
                return 'index page'

            # Route Info(Internal 페이지의 경로 정보)를 화면에 출력함
            @app.route('/api')
            def api():
                return route_info()

            # 입력한 username에 해당하는 계정의 레벨을 admin으로 변경하고 변경 사실을 화면에 출력함
            @app.route("/api/admin/make_admin/<str:username>")
            def make_admin(username):
                users[username].level = 'admin'
                return username + '의 레벨을 관리자로 설정'
            
            # 정상적인 사용자의 입력 데이터는 해당 경로로 접속하여 username에 해당하는 정보를 화면에 출력함
            @app.route("/api/user/<str:username>")
            def user(username):
                return json.dumps(users[username])
            ```
    - Path Traversal 공격을 수행하기 위해 External Server 페이지의 ```username```에 입력해야 하는 값
        | Internal Server 페이지 | Path Traversal를 통한 접근 |
        |:---:|------|
        | **api 페이지** | ```../```를 입력하면 api 페이지에 접근하여 화면에 Route Info를 출력함 <br/> &nbsp;&nbsp; - ```../```에 의해 상위 디렉터리인 ***/api***로 이동함 |
        | **인덱스 페이지** | ```../../```를 입력하면 인덱스 페이지에 접근할 수 있음 <br/> &nbsp;&nbsp; - 첫 번째 ```../```에 의해 ***/api***로 이동하고, 두 번째 ```../```에 의해 ***/***로 이동함 |
        | **make_admin/username <br/> 페이지** | ```../admin/make_admin/guest```를 입력하면 make_admin 페이지에 접근하여 guest 계정의 level를 관리자로 변경할 수 있음 ***→ ⚠️ 이 경로로 접근하면 모든 계정의 레벨을 admin으로 변경할 수 있음!*** |

<br/><br/>

### Server Side Request Forgery (SSRF)
| | 설명 |
|:---:|------|
| 정의 | 웹 서비스(웹 어플리케이션)의 요청을 변조하는 취약점 |
| 취약점 발생 상황 | 웹 서비스가 보내는 요청에 **이용자의 입력값**을 포함하고 있고 이를 적절히 필터링하지 않는 경우에 발생함 |
| 공격 결과 | Server-side에서 변조된 요청/의도하지 않은 웹 서버로 요청을 전송함 <br/> &nbsp;&nbsp; ***→ ⚠️ 웹 서비스의 권한으로 변조된 요청을 전송하기 때문에 상황에 따라 매우 치명적인 취약점이 될 수 있음*** |

<br/>

* Background
    - SSRF 발생 배경
        + 단일 서비스로 구현할 수 있었던 과거의 웹 서비스와는 달리, 최근의 웹 서비스는 지원하는 기능이 증가함에 따라 구성요소가 증가했음 <br/> &nbsp;&nbsp; → 관리 및 코드 복잡도를 낮추기 위해 마이크로서비스들로 웹 서비스를 구현하는 추세임
        + 웹 어플리케이션에서 사용자가 입력한 URL에 요청을 보내는 기능이 구현되어야 하는 경우도 존재함 <br/> &nbsp;&nbsp; → 사용자가 입력한 URL을 웹 어플리케이션에서 접근해야 하는 상황이 발생함
    - 마이크로서비스
        | | 설명 |
        |:---:|------|
        | 정의 | 소프트웨어가 잘 정의된 API를 통해서 통신하는, **소규모의 독립적인 서비스로 구성되어 있는 경우**의 소프트웨어 개발을 위한 아키텍처<br/> 및 조직적 접근 방식 |
        | 통신 방법 | 주로 HTTP, GRPC 등을 사용해 API 통신을 함 <br/> &nbsp;&nbsp; - 서비스 간 HTTP 통신이 이루어질 때 요청 내에 **이용자의 입력값**이 포함될 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *→ ⚠️ 이용자의 입력값이 포함될 경우 개발자가 의도하지 않은 요청이 전송될 수 있음* |
        | 추세 | 대다수의 서비스들이 마이크로서비스들로 구조를 많이 바꾸고 새롭게 개발하는 추세임 <br/> &nbsp;&nbsp; ***⇒ SSRF 취약점의 파급력이 더욱 높아지고 있음*** |
    - CSRF(Cross-site Request Forgery)와 SSRF(Server Side Request Forgery)의 차이점 → **변조된 요청을 보내는 대상의 차이**
        | 공격 | 차이점 |
        |:---:|------|
        | ***CSRF*** | 변조된 요청을 웹 클라이언트(브라우저)가 보냄 |
        | ***SSRF*** | 변조된 요청을 웹 어플리케이션(서버)이 보냄 *→ 웹 서비스의 권한으로 변조된 요청을 보냄* <br/> &nbsp;&nbsp; - 웹 어플리케이션에서 작동하고 있는 서버 내부의 포트, 서버와 연결된 내부망에 요청을 보낼 수 있음 |

<br/>

* SSRF 발생 상황
    - **CASE 1**: 웹 서비스가 이용자가 입력한 URL에 요청을 보내는 경우
        + 웹 서비스에서 사용하는 **마이크로서비스의 API 주소**를 알아내어 이를 이용자의 입력값으로 전달함 <br/> &nbsp;&nbsp; → **외부에서 직접 접근할 수 없는 마이크로서비스**의 기능을 사용할 수 있음
        + 예시 코드
            ```python
            # pip install flask requests # 파이썬의 flask, requests 라이브러리를 설치하는 명령
            # python3 main.py # 터미널에서 python 코드를 실행하는 명령어

            from flask import Flask, request
            import requests

            app = Flaks(__name__)

            # image_downloader: 이용자가 입력한 URL에 HTTP 요청을 보내고 응답을 반환함
            @app.route("/image_downloader")
            def image_downloader():
                image_url = request.args.get("image_url", "") # 이용자가 입력한 image_url를 가져옴
                response = requests.get(image_url) # requests 라이브러리를 사용해서 image_url에 HTTP GET 메소드 요청을 보내고 결과를 response에 저장함

                return ( # 아래의 3가지 정보를 반환함
                    response.content, # HTTP 응답으로 온 데이터
                    200, # HTTP 응답 코드
                    { "Content-Type": response.headers.get("Content-Type", "")}, # HTTP 응답으로 온 헤더 중 Content-Type(응답의 타입)
                )
            
            # request_info: 웹 페이지에 접속한 브라우저의 정보(User-Agent)를 반환함
            @app.route("/request_info")
            def request_info():
                return request.user_agent.string # '접속할 때' 사용한 브라우저의 정보가 출력됨
            
            app.run(host="127.0.0.1", port=8000)
            ```
            - image_downloader 페이지에서 ```image_url```로 아래와 같이 **request_info** 경로를 입력 (```https://127.0.0.1:8000/request_info```)하면 request_info에 HTTP 요청을 보내고 응답을 반환함
            - 웹 서버에 위치한 image_downloader(웹 서비스)에서 request_info에 HTTP Reqeust를 전송함 <br/> &nbsp;&nbsp; → **접속할 때** 사용한 브라우저의 정보(```user-agent```)로 ```python-requests/...```가 출력됨
    - **CASE 2**: 웹 서비스의 요청 URL에 이용자의 입력값이 포함되는 경우
        + 이용자의 입력값 중 URL의 구성 요소 문자를 삽입함 → API의 경로를 조작할 수 있음 (📌 **Path Traversal**)
        + 예시
            ```python
            INTERNAL_API = "http://api.internal" # "http;://127.17.0.3/"

            @app.route("/v1/api/user/information")
            def user_info():
                user_idx = request.args.get("user_idx", "") # 이용자가 전달한 user_idx를 가져옴
                response = requests.get(f"{INTERNAL_API}/user/{user_idx}") # user_idx(이용자의 입력값)을 내부 API의 경로로 사용함

            @app.route("/v1/api/user/search")
            def user_search():
                user_name = request.args.get("user_name", "") # 이용자가 전달한 user_name을 가져옴
                user_type = "public"
                response = requests.get(f"{INTERNAL_API}/user/search?user_name={user_name}&user_type={user_type}") # user_name(이용자의 입력값)을 내부 API의 쿼리로 사용함
            ```
            - 이용자의 입력값에 따른 결과
                | 이용자의 입력값 | 설명 |
                |:---:|------|
                | **user_info** 페이지에서 user_idx에  <br/> ```../search```를 입력 | **/v1/api/user/search**에 접근하여 해당 위치로 URL 요청을 보낼 수 있음 <br/> &nbsp;&nbsp; - ```..``` 문자: 상위 디렉터리로 이동하기 위한 구분자 |
                | **user_search** 페이지에서 user_name에 <br/> ```admin&user_type=private#```를 입력 | 생성된 URL에서 ```#``` 문자 뒤의 ```&user_type=public```이 생략되어 <br/> ```/search?user_name=admin&user_type=private```로 URL 요청을 전송함 <br/> &nbsp;&nbsp; - ```#``` 문자: Fragment Identifier, ```#``` 뒤에 붙은 문자열은 API 경로에서 생략됨 |
    - **CASE 3**: 웹 서비스의 요청 Body에 이용자의 입력값이 포함되는 경우
        + 데이터를 구성할 때 이용자의 입력값을 파라미터의 형식으로 설정할 때 **URL에서 파라미터 구분 문자인 ```&```를 포함시킴** <br/> &nbsp;&nbsp; → 설정되는 데이터의 값을 변조할 수 있음
            + 내부 API에서는 전달받은 값을 파싱할 때 **파라미터의 값을 차례대로 가져와 사용함** (동일한 파라미터가 존재하는 경우 **앞에 위치한 값**을 가져와 사용함) <br/> &nbsp;&nbsp; → ⚠️ 이 점을 이용해 변조해야 할 값을 앞에 위치하도록 만들어 파라미터의 값을 변조할 수 있음
        + 예시
            ```python
            # pip3 install Flask
            # python main.py # 터미널에서 python 코드 실행 시 사용하는 명령어

            from flask import Flask, request, session
            import requests
            from os import urandom

            app = Flask(__name__)
            app.secret_key = urandom(32)

            INTERNAL_API = "http://127.0.0.1:8000/"
            header = {"Content-Type": "application/x-www-form-urlencoded"}

            # board_write: 사용자에게 title, body를 입력받고 이를 이용해 POST 메소드로 내부 API에 요청을 보내고 그 응답을 반환함
            @app.route("/v1/api/board/write", methods=["POST"])
            def board_write():
                session["idx"] = "guest" # session idx를 guest로 설정함

                # form 데이터(이용자의 입력값)에서 title, body 값을 가져옴
                title = request.form.get("title", "")
                body = request.form.get("body", "")

                # 전송할 데이터(HTTP body 데이터)를 구성 → 형식: key1=value1&key2=value2&key3=value3 형식
                data = f"title={title}&body={body}&user={session['idx']}"

                # INTERNAL_API에 이용자가 입력한 값을 HTTP body 데이터로 사용해서 요청함
                response = requests.post(f"{INTERNAL_API}/board/write", headers=header, data=data)

                # INTERNAL_API에 전송한 요청에 대한 응답 결과를 화면에 출력함
                return response.content
            
            # internal_board_write: board_write 함수에서 요청하는 내부 API를 구현하는 기능
            @app.route("/board/write", methods=["POST"])
            def internal_board_write():
                # form 데이터로 입력받은 값(전달된 title, body, 계정 이름)을 JSON 형식으로 변환함
                title = request.form.get("title", "")
                body = request.form.get("body", "")
                user = request.form.get("user", "")
                info = {
                    "title": title,
                    "body": body,
                    "user": user
                }
                return info

            # index: board_write 기능을 호출하기 위한 인덱스 페이지
            @app.route("/")
            def index():
                return """
                    <form action="/v1/api/board/write" method="POST">
                        <input type="text" placeholder="title" name="title"/><br/>
                        <input type="text" placeholder="body" name="body"/><br/>
                        <input type="submit"/>
                    </form>
                """
            
            app.run(host="127.0.0.1", port=8000, debug=True)
            ```
            - 이용자의 입력값인 title에서 ```&``` 구분 문자를 포함하는 ```title&user=admin```을 입력하면 data을 변조할 수 있음
                ```python
                data = f"title=title&user=admin&body=body&user=guest" # 변조된 데이터
                ```
                + 내부 API에서는 전달받은 값을 파싱할 때 **앞에 존재하는 파라미터의 값**을 가져와 사용함 <br/> &nbsp;&nbsp; → ```user=admin```이 ```user=guest```보다 앞에 위치하므로 user를 변조할 수 있음
<br/>

* SSRF 방지 방법
    - 사용자가 입력한 URL의 Host를 **Allowlist 방식**으로 검증하는 방법
        + 미리 신뢰할 수 있는 Domain Name, IP Address를 Allowlist에 등록함 → 사용자가 입력한 URL에서 Host 부분을 파싱해 Allowlist에 존재하는지 확인함
        + 예시
            ```python
            from urllib.parse import ulrparse

            ALLOWLIST_URL = [ # 미리 신뢰할 수 있는 Domain Name, IP Address를 등록해둔 Allowlist
                'i.imgur.com',
                'img.dreamhack.io',
                ...
            ]
            SCHEME = ['http', 'https'] # 허용하는 프로토콜의 목록

            def is_safehost(url):
                urlp = urlparse(url) # 사용자가 입력한 url를 파싱하여 scheme, host를 얻음
                if not urlp.scheme in SCHEME: # http, https 외의 프로토콜을 사용하여 접근 시 이를 차단함
                    return False
                hostname = urlp.hostname.lower() # hostname을 모두 소문자로 변환함 (대소문자 혼용으로 필터링을 우회하는 것을 방지)
                if hostname in ALLOWLIST_URL:
                    return True # hostname이 Allowlist에 등록되어 있는 경우 True를 반환함 (그 외의 경우는 False를 반환함)
                return False

            print(is_safehost('https://127.0.0.1/')) # False
            print(is_safehost('https://i.imgur.com/image.png')) # True
            ```
            - ⚠️ Blocklist 방식으로 사용자가 입력한 URL의 Host가 내부망/루프백 주소인지 검증할 경우 발생하는 문제점
                + ```http://127.0.0.4/```, ```https://0x7f000001```와 같은 다양한 루프백 주소를 사용하여 우회 가능
                + Host에 Domain Name을 넣어 DNS Rebinding 공격 등으로 우회 가능
    - 사용자의 URL을 처리하는 서버를 **독립적으로 망 분리**를 하는 방법
        + SSRF 취약점이 발생하더라도 다른 취약점과 연계를 하지 못하도록 방지함

<br/><br/><br/>

## File Vulnerability
파일을 업로드하거나 다운로드할 때 발생하는 취약점
* File Vulnerability의 분류
    | 종류 | 설명 |
    |:---:|------|
    | ***File Upload Vulnerabiltiy*** <br/> (파일 업로드 취약점) | 서버의 파일 시스템에 사용자가 원하는 경로 또는 파일명 등으로 업로드가 가능하여 악영향을 미칠 수 있는 파일이 업로드되는 취약점 |
    | ***File Download Vulnerability*** <br/> (파일 다운로드 취약점) | 서버의 기능 구현 상 의도하지 않은 파일을 다운로드할 수 있는 취약점 |
    - ⚠️ **File Vulnerability가 발생하는 원리는 단순하지만, 공격에 사용되면 <U>웹 애플리케이션에 임의 명령어를 실행하거나 중요 파일을 탈취할 수 있는 만큼</U> 매우 치명적임**

<br/><br/>

### File Upload Vulnerability
| | 설명 |
|:---:|------|
| 정의 | 공격자의 파일을 웹 서비스의 파일 시스템에 업로드하는 과정에서 발생하는 보안 취약점 |
| 취약점 발생 상황 | 파일 시스템 상 임의 경로에 원하는 파일을 업로드하거나 악성 확장자를 갖는 파일을 업로드할 수 있을 때 발생함 <br/> &nbsp;&nbsp; → ***이용자가 원하는 파일의 이름을 임의로 정할 수 있을 때 발생함*** |
| 분류 | 크게 <U>Injection의 일종인 Path Traversal를 이용한 공격</U>과 <U>악성 파일 업로드</U>로 구분됨 |

<br/>

* Background
    - **File Upload**(파일 업로드): 웹 서비스를 통해 파일을 서버에 업로드하는 기능
        + 사용자의 사진, 문서와 같은 파일을 서버에 업로드하여 <u>다른 사용자와 공유하기 위한 목적</U>으로 사용됨
        + ⚠️ 파일 업로드 기능은 사용자의 파일이 서버의 파일 시스템에 저장되어 처리됨 → ***File Upload Vulnerability 취약점 발생***
    - **CGI**(Common Gateway Interface): 사용자의 요청을 받은 서버가 동적인 페이지를 구성하기 위해 엔진에 요청을 보내고 엔진이 처리한 결과를 서버에 반환하는 기능
        + php, jsp, asp 등과 같이 CGI를 통해 서비스를 하는 형태에서는 <U>**확장자를 통해** 웹 어플리케이션 엔진에 요청 여부를 판단함</U>
        + 예시: Apache 설정 파일
            ```Apache
            <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
                SetHandler application/x-httpd-php
            </FilesMatch>
            ```
            - 정규표현식을 만족하는 .php, .php3/4/5/6, .phtml 등의 확장자를 가진 파일들은 php에서 처리되도록(x-httpd-php) 핸들링되어 있음
                | | 설명 |
                |:---:|-------|
                | 정규표현식 | 문자열의 끝이 .php, .php3, .php4, .php5, .php7, .pht, phtml인 경우 <br/>해당 정규표현식을 만족한다는 의미 |
                | ```x-httpd-php``` | PHP 엔진으로 요청한 파일을 실행하고 그 결과를 반환함 | 
            - Apache는 사용자가 서버로 보낸 요청을 해석함 → 사용자가 요청하는 리소스의 확장자가 php 엔진을 사용하도록 설정되어 있다면 **mod_php(CGI)를 통해 사용자의 요청을 php 엔진이 처리 및 실행하도록 요청함**
        + 💡 Python의 Flask, django와 NodeJS의 express 등과 같은 프레임워크
            - 서버, CGI, 웹 어플리케이션 등이 통합되어 있음
            - <U>미리 라우팅된 경로에만 접근이 가능하도록 설정되어 서비스됨</U>

<br/>

* File Upload Vulnerability 1: <U>File Path에서 발생하는 Path Traversal를 이용한 공격</U>
    | | 설명 |
    |:---:|------|
    | 정의 | 파일 업로드를 허용하는 대개의 서비스가 **특정 디렉터리에만 업로드를 허용하는 제약을 우회**하여, 임의 디렉터리에 <br/>파일을 업로드할 수 있는 취약점 |
    | 취약점 <br/> 발생 원인 | 이용자가 입력한 파일 이름을 별다른 검증 없이 그대로 사용할 때 발생함 <br/> (*Path Traversal은 Injection의 일종*) |
    | 공격 방법 | 상위 디렉터리로 이동하는 ```..``` 메타 문자 등을 사용하여 다른 디렉터리에 파일을 업로드할 수 있음 |
    - 예시: File Upload Code
        ```python
        from flask import Flask, request
        app = Flask(__name__)

        @app.route('/fileUpload', methods=['GET', 'POST'])
        def upload_file():
            if request.method == 'POST': # POST 메소드로 요청하는 경우
                f = request.files['file'] # 사용자가 업로드한 file을 가져옴
                f.save("./uploads/" + f.filename) # 사용자가 입력한 파일명으로 /uploads 디렉터리에 업로드함
                return 'Upload Success'
            else: # GET 메소드로 요청하는 경우
                return """
                <form action="/fileUpload" method="POST" enctype="multipart/form-data">
                    <input type="file" name="file"/>
                    <input type="submit"/>
                </form>
                """
        
        if __name__ == '__main__':
            app.run()
        ```
        | 일반 사용자 | 공격자 |
        |:---:|:---:|
        | <img width="400" alt="정상적인 File Upload Request" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b4ed2395-b24a-4211-87f5-4accd54935a3"> | <img width="400" alt="악의적인 File Upload Request" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a653de65-4bc0-49bb-a66d-ebf0cac1b5d3"> |

<br/>

* File Upload Vulnerability 2: 악성 파일 업로드
    - **WebShell** (웹셸)
        | | 설명 |
        |:---:|------|
        | 전제 조건 | 웹 서버가 서비스하는 경로에 파일을 업로드할 수 있어야 함 <br/> &nbsp;&nbsp; → 웹 서버가 파일을 실행하는 시기 = 사용자의 요청이 들어온 후 파일 시스템에서 해당 파일을 찾아 실행할 때 |
        | 공격 과정 | 웹 서비스가 동작하는 경로에 사용자가 원하는 파일 내용과 파일명을 업로드할 수 있음 <br/> → CGI를 통해 서비스하는 서버가 엔진에 요청하는 확장자를 업로드하여 서버의 웹 어플리케이션에 원하는 코드를 실행함 |
        | 파급 효과 | 원하는 시스템 커맨드를 실행하는 **원격 코드 실행 취약점**을 유발할 수 있음 <br/> &nbsp;&nbsp; - 웹 어플리케이션이 실행하는 코드를 조작할 수 있어 웹 어플리케이션 언어에 내장된 OS 명령어 등을 사용할 수 있어야 가능함 |
    - 악의적인 웹 리소스
        + 웹 브라우저는 **파일의 확장자**나 Response(응답)의 **Content-Type**에 따라 요청을 다양하게 처리함
            | 예시 | 결과 |
            |------|:---:|
            | 요청한 파일의 확장자가 **.html**인 경우, <br/> Response의 Content-Type이 **text/html**인 경우 | HTML 엔진로 처리됨 |
            | 요청한 파일의 확장자가 **.jpg**, **.png** 등의 이미지 확장자인 경우, <br/> Response의 Content-Type이 **image/png** 등인 경우 | 이미지로 렌더링됨 |
            - 공격자가 <U>악의적인 스크립트를 삽입한 html 파일</U>을 업로드한 경우 **XSS 공격**(Stored XSS)으로 이어질 수 있음
    
<br/>

* File Upload Vulnerability 방지 대책
    - 업로드 디렉터리를 웹 서버에서 직접 접근할 수 있도록 하거나 업로드 디렉터리에서는 CGI가 실행되지 않도록 해야 함
    - 업로드한 파일 이름을 그대로 사용하지 않고, ```basepath```와 같은 함수를 통해 파일 이름을 검증한 후 사용함
    - 허용할 확장자를 명시해 그 외의 확장자는 업로드할 수 없도록 해야 함
        + 동적 리소스의 확장자를 제한하면 파일 업로드 취약점을 통한 RCE 공격으로부터 서버를 보호할 수 있음
            | 리소스의 종류 | 설명 |
            |:---:|------|
            | **정적 리소스** (Static Resource) | 웹 리소스 중 이미지나 비디오와 같이 서버에서 실행되지 않는 리소스를 말함 |
            | **동적 리소스** (Dynamic Resource) | 웹 리소스 중 PhP, JSP처럼 서버에서 실행되닌 리소스를 가리킴 |
    - AWS, Azure, GCP 등의 정적 스토리지를 이용함
        + 서버의 파일 시스템을 이용하지 않음 → 파일 업로드 취약점이 웹 서버 공격으로 이어지는 것을 막을 수 있음

<br/><br/>

### File Download Vulnerability
| | 설명 |
|:---:|------|
| 정의 | 웹 서비스의 파일 시스템에 존재하는 파일을 다운로드하는 과정에서 발생하는 보안 취약점 |
| 취약점 발생 상황 | 이용자가 다운로드할 파일의 이름을 임의로 정할 수 있을 때 발생함 <br/> &nbsp;&nbsp; - **<U>사용자가 입력한 파일 이름을 검증하지 않은 채 그대로 다운로드를 제공할 때</U> 가장 흔하게 발생함** |
| 공격 효과 | 웹 어플리케이션의 소스 코드, 관리자의 패스워드, 서비스 키, 설정 파일, 데이터베이스 백업본 등 민감한 정보을 탈취할 수 있음 <br/> &nbsp;&nbsp; ***→ 다운로드받은 파일을 이용하여 2차 공격을 수행할 수도 있음*** |

<br/>

* Background: **File Downlaod**(파일 다운로드)
    - 웹 서비스의 파일 시스템에 존재하는 파일을 다운로드하는 기능
        + 사용자가 업로드한 파일을 다른 사용자와 공유하기 위해서 필요함
        + 다양한 방법으로 파일 다운로드 기능을 구현할 수 있음

<br/>

* File Download Vulnerability: File Path에서 발생하는 <U>Path Traversal를 이용한 공격</U>
    | | 설명 |
    |:---:|------|
    | 정의 | 웹 서비스가 **특정 디렉터리에 있는 파일만 접근하도록 허용하는 제약을 우회**하여, 파일 이름을 직접 입력 받아 임의 디렉터리에 <br/>있는 파일을 다운로드받을 수 있는 취약점 |
    | 취약점 <br/> 발생 원인 | 사용자가 입력한 파일 이름을 별다른 검증 없이 사용할 때 발생함 <br/> (*Path Traversal은 Injection의 일종*) |
    | 공격 방법 | 상위 디렉터리로 이동하는 ```..``` 메타 문자 등을 사용하여 다른 디렉터리에 있는 파일을 다운로드받을 수 있음 |
    - 예시: File Download Code
        ```python
        #...
        @app.route("/download")
        def download():
            # 사용자가 입력한 filename의 인자 값을 가져와 검증 없이 사용하고 있음 → File Download Vulnerability 발생
            filename = request.args.get("filename", "")
            return open("uploads/" + filename, "rb").read()
        ```
        | | 예시 설명 |
        |:---:|------|
        | 일반 사용자 | ```http://example.com/download?filename=docs.pdf``` 압룍 <br/> &nbsp;&nbsp; → **/uploads** 디렉터리에 존재하는 docs.pdf 파일을 다운로드 받음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (***/uploads/docs.pdf 파일 다운로드***)
        | 공격자 | ```http://example.com/download?filename=../../../../etc/passwd``` 입력 <br/> &nbsp;&nbsp; → 상위 디렉터리로 이동하는 과정을 반복하여 다른 경로에 존재하는 시스템 계정 파일을 다운로드 받음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (***/etc/passwd 파일 다운로드***) |

<br/>

* File Download Vulnerability 방지 대책
    - 📌 ***기본적으로 인자에 다운로드을 받으려는 파일의 경로나 이름을 넘기지 않는 것이 좋음***
        + 반드시 이름을 넘기는 방식으로 구현해야 하는 경우 <U>상대경로로 접근하는 데 사용될 수 있는 ```..```, ```/```, ```\\```를 적절하게 필터링해야 함</U>
        + ⚠️ 필터링을 잘못 구현한 경우 **필터링을 우회할 수 있음**
            | 예시 | 설명 |
            |:---:|------|
            | 단순히 ```../```만 <br/> 필터링하는 경우| 상위 경로로 올라가는 키워드를 없애면 상위 경로로 올라가지 못하기 때문에 안전할 것이라고 착각함 <br/> &nbsp;&nbsp; - ```..././file```와 같은 형식으로 요청하면 필터링에 의해 ```../```가 삭제되어 새로운 ```../```가 생성됨 <br/> &nbsp;&nbsp; - 윈도우 ```..\\```으로도 상위 경로에 접근할 수 있음 |
    - 데이터베이스에 다운로도 될 파일의 경로와 그에 해당하는 랜덤 키를 생성하여 1:1로 매칭해서 저장해두고 해당 랜덤 값(키)이 인자로 넘어왔을 때 데이터베이스에 존재하는 파일인지를 먼저 식별하고 다운로드를 수행함
        + 예시 코드
            ```python
            @app.route("/download")
            def donwload():
                file_id = requests.args.get("file_id", "") # file_id = 쉽게 유추하지 못하는 랜덤한 값
                file_path = find_path_from_database(file_id) # 데이터베이스에서 file_id와 맵핑된 파일 경로를 반환하는 find_path_from_database() 함수를 정의했다고 가정
                
                if file_path is None: # 맵핑되는 파일 경로가 존재하지 않는 경우
                    return "incorrect file id"
                return open(file_path, "rb").read() # 맵핑되는 파일 경로가 존재하는 경우
            ```
    - 요청된 파일 이름을 ```basepath```와 같은 함수를 통해 검증함

<br/><br/><br/>

## Business Logic Vulnerability
정상적인 비즈니스 로직을 악용하는 취약점

<br/>

* Background: **Business Logic** (비즈니스 로직)
    + 규칙에 따라 데이터를 생성﹒표시﹒저장﹒변경하는 로직, 알고리즘 등을 의미함
    + 예시: 게시판 서비스
        - 회원가입/로그인, 게시물 작성/수정/삭제 등 **다양한 로직이 모여 하나의 서비스가 완성됨**
        - 게시물을 수정하는 비즈니스 로직
            | 순서 | 설명 |
            |:---:|------|
            | 1 | 사용자가 게시물 수정을 요청함 |
            | 2 | 로그인된 사용자인지 확인함 |
            | 3 | 수정을 요청한 사용자가 해당 게시물을 수정할 수 있는 권한이 있는지 확인함 <br/> &nbsp;&nbsp; → ⚠️ 3번 과정이 **설계 과정의 실수**로 인해 누락될 경우 악의적인 공격자는 **다른 사용자의 게시물도 수정할 수 있음** <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (비즈니스 로직 취약점 발생!) |
            | 4 | 2, 3번 과정이 확인되면 데이터베이스에 사용자가 입력한 정보로 수정함 |

<br/>

* 다른 Server-side 취약점과 비즈니스 로직 취약점의 차이점
    | 취약점 종류 | 차이점 설명 |
    |:---:|------|
    | **인젝션, 파일 관련 취약점** | 사용자의 악의적인 데이터가 서버의 시스템 상에서 악영향을 미치는 공격을 수행함 |
    | **비즈니스 로직 취약점** | 서비스의 기능에서 적용되어야 할 로직이 없거나 잘못 설계된 경우 발생함 |

<br/>

* Businness Logic 관련 취약점
    | 종류 | 설명 |
    |:---:|------|
    | ***Business Logic Vulnerability*** | 정상적인 흐름에서 검증 과정의 부재 및 미흡으로 인해 정상적인 흐름이 악용되는 취약점 |
    | ***Insecure Direct Object Reference*** <br/> (IDOR) | 변조된 파라미터 값이 다른 사용자의 오브젝트 값을 참조할 때 발생하는 취약점 |
    | ***Race Condition*** | 비즈니스 로직의 순서가 잘못되거나, 한 오브젝트에 여러 요청이 동시에 처리되는 상황에서 발생하는 취약점 |

<br/><br/>

### Business Logic Vulnerability
| | 설명 |
|:---:|------|
| 정의 | 비즈니스 로직의 구현 과정에서 검증 과정의 부재 및 미흡으로 인해 정상적인 로직의 수행을 악용하는 취약점 |
| 취약점 발생 상황 | 어플리케이션의 **검증 부재 및 미흡**의 이유로 발생함 |
| 공격 결과 | 서비스에 따라 다양한 형태의 로직이 포함되어 각각의 형태에 따라 다양한 비즈니스 로직 취약점이 발생할 수 있음 |
| 방지 방법 | 📌 비즈니스 로직을 확실하게 이해하고, **설계 및 개발 단계에서 어떤 위협이 발생할 수 있는지 위협을 파악하고 방어**하는 것이 중요함 <br/> &nbsp;&nbsp; *→ 서비스의 구조와 형태에 올바른 비즈니스 로직을 선택하여 구현해야 함* |

<br/>

* Business Logic Vulnerability 예시: 후기 기능에서 발생할 수 있는 취약점
    - 후기 기능의 비즈니스 로직과 이를 구현한 코드
        + 후기 작성 기능
            | 순서 | 설명 |
            |:---:|------|
            | 01 | 사용자가 후기 작성을 요청함 *→ ```review_write()``` 함수를 호출함* |
            | 02 | 로그인한 사용자의 세션값과 후기 내용을 가져옴 *→ ```review_write()``` 함수의 1, 2번째 코드*|
            | 03 | 후기를 작성한 사용자의 세션이 이미 존재하는지 확인함 *→ ```review_write()``` 함수의 첫 번째 if 조건문 부분* |
            | 04 | 3번 과정을 통과하면 해당 사용자의 후기를 저장하고 100포인트를 지급함 *→ ```review_write()``` 함수의 조건문 이후 나머지 부분* |
            ```python
            @app.route('/reviewWrite')
            def review_wirte():
                userName = session['username'] # 세션에서 username(키)에 해당하는 값을 가져옴
                contents = requests.form['content'] # 사용자가 입력한 content를 가져옴

                if review_Check(userName): # 이미 해당 userName으로 작성된 후기가 존재한다면 Already write를 출력하고 후기 기능을 종료함
                    return "Already write"
                
                result = review_Insert(userName, content) # userName(사용자 이름)과 content(후기 내용)을 review_Insert() 함수로 저장함
                pointResult = userPoint(userName, 100) # 해당 이용자(userName)의 포인트를 100 증가시킴 (100포인트 지금)

                if result = pointResult: # 리뷰 작성과 포인트 지급이 완료된 경우
                    return "write success."
                else:
                    return "write fail."
            ```
        + 후기 삭제 기능
            | 순서 | 설명 |
            |:---:|------|
            | 01 | 사용자가 후기 삭제를 요청함 *→ ```review_delete()``` 함수를 호출함* |
            | 02 | 로그인한 사용자의 세션값과 삭제할 후기의 인덱스를 가져옴 *→ ```review_delete()``` 함수의 1, 2번째 코드* |
            | 03 | 삭제하려는 후기를 작성한 사용자와 후기 삭제를 요청한 사용자가 동일한지 확인함 *→ ```review_delete()``` 함수의 첫 번째 if 조건문 <br/>부분* |
            | 04 | 3번 과정을 통과하면 해당 후기를 삭제함 *→ ```review_delete()``` 함수의 조건문 이후 나머지 부분* <br/> &nbsp;&nbsp; → **⚠️ 저장된 후기를 삭제하지만 지급한 포인트는 차감하지 않아 후기 작성/삭제를 반복하여 포인트를 무제한으로 적립받을 수 있음** |
            ```python
            @app.route('/reviewDelete')
            def review_delete():
                userName = session['username'] # 세션에서 username(키)에 해당하는 값을 가져옴
                idx = request.args.get('idx') # 사용자의 idx 인자 값을 가져옴

                if review_owner_check(userName): # 삭제하려는 후기가 후기 삭제를 요청한 사용자가 작성한 것인지 확인함 → 다르다면 후기 삭제 기능을 종료함
                    return "owner check fail."
                
                result = review_Delete(idx) # idx(후기 번호)를 이용해 해당 후기를 삭제함

                if result: # 후기 삭제를 성공적으로 완료한 경우
                    return "delete success."
                else:
                    return "delete fail."
            ```
    - 취약점 패치 방법
        | | 설명 |
        |:---:|------|
        | 방법 1 | **후기 삭제 코드**에서 10번째 라인(```result = review_Delete(idx)``` 뒤)에 ```userPoint(userName, -100)``` 코드를 추가함 |
        | 방법 2 | 최초 1회에만 포인트를 지급하는 형태로 구현함 |
    - ⚠️ 전체 서비스의 기능 중 하나인 후기 기능에서 발생할 수 있는 취약점만 다루고 있음 ***→ 서비스 전체로 생각한다면 또 다른 취약점이 발생할 수 있음***
        + 사용자가 후기 이벤트를 통해 얻은 포인트를 쇼핑/송금과 같이 포인트를 사용하는 다른 기능에서 모든 포인트를 사용한 후기를 삭제하는 경우 등
        + 후기 삭제로 인해 다른 취약점이 발생하는 것을 방지하기 위해 **후기 삭제 기능을 지원하지 않도록 구현함**

<br/><br/>

### IDOR (Insecure Direct Object Reference, 안전하지 않은 객체 참조)
| | 설명 |
|:---:|------|
| 정의 | 객체 참조 시 사용하는 객체 참조 키가 사용자에 의해 조작됐을 때 조작된 객체 참조 키를 통해 객체를 참조하고, 해당 객체 정보를 <br/>기반으로 로직이 수행되는 것 |
| 취약점 발생 상황 | 사용자의 입력 데이터에 의해 참조하는 객체가 변하는 기능에서 **사용자가 참조하고자 하는 객체에 대한 권한 검증이 올바르지 않아** 발생함 |
| 공격 결과 | 비즈니스 로직에 따라 조회/삭제/수정/추가 등의 다양한 형태를 다른 사용자의 객체로 수행할 수 있음 |
| 취약점 방지 | 📌 **객체 참조 시 사용자의 권한을 검증하는 게 가장 중요함** <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = 사용자가 의도한 권한을 벗어나서 행동할 수 없도록 권한 등을 분리해서 관리해야 함 |

<br/>

* IDOR 취약점 예시: 계좌 정보에 대한 검증 없이 사용하는 잔액 조회/송금 기능 (계좌번호를 데이터베이스에 숫자로 저정함)

<br/><br/><br/>

## Language specific Vulnerability

<br/><br/><br/>

## Misconfiguration
| | 설명 |
|:---:|------|
| 취약점 <br/> 발생 위치 | 웹 서버(Apache, Nginx), 데이터베이스 서버(mysql, postgresql, mongodb), 캐시 서버(redis), 웹 프레임워크(django, spring), <br/>컨테이너(Docker, k8s) 등 **모든 어플리케이션 계층에서 발생할 수 있음 |
| 발생 원인 <br/> 및 방지 대책 | 소스 코드 상에 존재하는 복잡한 로직에서 발생하는 취약점이 아닌 ***간단한 설정 오류로 인해 발생함*** <br/> &nbsp;&nbsp; → <u>검수할 때는 기본 메뉴얼에 존재하는 룰을 잘 따랐는지</u>만 확인하면 됨 ⇒ 자동화 도구로 쉽게 발견할 수 있음 |
| 효과 | 공격자는 허가되지 않은 동작을 수행할 수 있음 |
| 분류 | 발생하는 원인에 따라 **부주의로 인해 발생하는 문제점**, **편의성을 위한 설정에 의해 발생하는 문제점**, **메뉴얼과 실제 구현체의 차이로 <br/>인해 발생하는 문제점**, **해당 코드나 설정에 대한 이해 없이 사용해 발생하는 문제점**으로 나눌 수 있음 |

<br/><br/>

### 부주의로 인해 발생하는 문제점
메뉴얼에 존재하지 않는 설정이거나 개발자의 부주의로 인해 설정 유무를 알지 못한 상태에서 서비스를 할 경우 자주 발생함

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***권한 설정 문제***
    - 웹 서비스를 구성하는 요소들 운용 시 시스템 또는 어플리케이션 상에서 권한 설정이 잘못된 경우
    - 대표적인 권한 설정 문제
        | 권한 설정 문제 | 설명 |
        |:---:|------|
        | ***기본 계정*** | 프레임워크 등에서 기본적으로 제공하는 계정을 삭제하지 않아 문제가 발생할 수 있음 <br/> &nbsp;&nbsp; - 프레임워크 등을 통해 제공되는 계정 정보는 **ID/PW 등의 정보가 메뉴얼을 통해 공개되어 있음** <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 이를 통해 특별한 취약점을 사용하지 않고 권한을 획득할 수 있음|
        | ***잘못된 권한 설정*** | 웹 서버를 운용하는 시스템 또는 어플리케이션 상에서 <u>필요 이상의 권한을 부여</u>하는 경우 문제가 발생할 수 있음 <br/><br/> 서비스를 운용하는 OS의 사용자 권한이 일반 유저 또는 시스템 운용을 위해 생성된 계정을 사용하지 않고 <u>**superuser**(root) <br/>권한으로 사용될 경우 서비스의 로직이 superuser의 권한으로 수행됨</u> <br/> &nbsp;&nbsp; ***e.g.*** 웹 서버가 superuser 권한으로 동작하며, 파일 관련 취약점이 존재할 경우 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 일반 사용자 권한이 접근하지 못하는 시스템 파일 등에도 접근할 수 있게 됨 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 이를 통해 시스템 장악 등의 문제로 연계될 수 있음 <br/><br/> DB, Docker 등의 어플리케이션 운용 시에도 해당 어플리케이션의 <u>사용자 권한을 필요 이상으로 부여할 경우</u> 문제가 발생할 <br/>수 있음 <br/> &nbsp;&nbsp; ***e.g.*** DBMS 내에서 root 계정을 사용하는 경우 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 기본적으로 모든 데이터베이스에 접근 가능하며, 일반 유저가 사용하지 못하는 함수들도 사용할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → sql injection 취약점 발생 시 함수들을 통해 다른 취약점으로 연계할 수 있는 위협이 존재할 수 있음 |

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***기본 서비스***
    - 서비스 요소들 운용 시 사용하는 시스템 또는 어플리케이션에서 **개발자가 설정하지 않아도 기본적으로 제공되는 기능**에 의해 문제가 발생할 수 있음
        + 대부분 개발 당시에는 필요한 기능이거나, 내부적으로 사용해야 하는 기능들임
        + ***운영 시에 의도하지 않은 경로 또는 기능이 노출되는 위협이 발생할 수 있음***
    - 기본 서비스 예시
        | 서비스 종류 | 예시 |
        |:---:|------|
        | 관리 서비스 | Tomcat - Web Application Manager <br/> Docker Registry API |
        | 모니터링 서비스 | Srping Boot Actuator - metrics <br/> Apache mod_status |

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***임시/백업 파일***
    - 웹 서버 디렉터리 내에 임시/백업 파일이 존재한다면 디렉터리 스캐너를 통해 파일이 유출될 수 있음
        + ❓ **디렉터리 스캐너**: 디렉터리 내 자주 쓰이는 파일명이나 확장자를 자동으로 스캐닝해주는 도구
    - 공격자에게 악용될 수 있는 임시/백업 파일
        | 확장자 | 설명 |
        |:---:|------|
        | **swp** | 임시 파일, 파일 내용(소스 코드)가 포함되어 있음 |
        | **bak** | 백업 파일, 대부분의 에디터에서 사용함 |
        | **config** | 설정(configuration) 파일, 비밀 키들이 존재하는 경우가 많음 |
        | **sql** | sql schema 파일, 데이터베이스 구조를 알아낼 수 있음 |
        | **sh** | shell script 파일 |
        | **~** | bluefish 에디터 백업 파일 |
        + 예시: vim 에디터와 swp 임시 파일을 이용한 파일 유출
            - ***hello.php*** 파일: password가 일치하면 임의 명령을 실행해주는 php 파일
                ```php
                <?php
                    if ($_GET['password'] === 'this_is_secret_password') { # 공격자는 소스코드에 정의되어 있는 'this_is_the_secret_password'를 알 수 없음
                        system($_GET['cmd']);
                    } else {
                        echo 'Nope';
                    }
                ?>
                ```
            - vim 에디터로 hello.php 파일을 수정 중에 생긴 **.swp**를 통해 소스코드를 획득하는 방법
                | 순서 | 설명 |
                |:---:|------|
                | 01 | 개발자가 hello.php 파일을 vim 에디터로 열게 되면 **.hello.php.swp**라는 임시 파일이 생성됨 <br/> &nbsp;&nbsp; - 📌 **웹 서버의 동작** <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ***.php 확장자*** → php script로 인식해 실행 결과를 응답함 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ***.swp 확장자*** → 바이너리 파일로 인식해 파일 내용(소스 코드)가 포함된 임시 파일을 응답함 |
                | 02 | 공격자는 .swp 파일을 다운로드 받아 ```vim hello.p``` 명령어를 입력한 후 R(Recover) <br/>키를 눌러 복구를 진행함 |
                | 03 | 복구가 되었다는 메시지와 함께 원본 소스코드를 반환함 → 이를 통해 공격자는 소스코드를 획득할 수 있음 |
    - 조치 방안
        + 서비스하기 이전에 서버 및 어플리케이션의 설정 및 파일들을 점검하여 서비스와는 무관한 파일을 제거해야 함
        + <u>위험한 확장자에 대한 요청이 들어올 경우 거부하는 방법</u>으로 위협을 없앰
            - 예시: Nginx에서 적용 가능한 설정
                ```nginx
                location ~ (?:\.(?:bak|config|sql|fla|psd|ini|log|sh|inc|swp|dist)|~)$ {
	            deny all;
	            access_log off;
	            log_not_found off;
                }
                ```

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***개발 관련 파일***
    - 버전 관리 도구(VCS)를 사용해 소스코드를 다운로드 받거나 웹 폴더 내에서 직접 개발을 하다보면 해당 VCS가 자동으로 생성하는 정보 파일이 남음
        | VCS | 설명 |
        |:---:|------|
        | git | **.git** 폴더가 생성됨 → 해당 폴더 내에 트래킹되고 있는 파일들의 소스 코드를 쉽게 획득할 수 있음 <br/> &nbsp;&nbsp; - ```init``` 과정보다 ```get clone ..```을 통해 저장소를 받아온 후 웹 서버가 바로 서빙되는 경우 발생하기 쉬움 |
        | hg | **.hg** 폴더가 생성됨 → 해당 폴더 내에 트래킹되고 있는 파일들의 소스 코드를 쉽게 획득할 수 있음 |
        + git, hg 등을 포함한 다양한 VCS 도구가 생성하는 정보를 진단하는 도구 → [🔗](https://github.com/kost/dvcs-ripper) 참고
    - 조치 방안
        + 서비스하기 이전에 점검하여 서비스와는 무관한 파일들을 제거해야 함
        + <u>웹 URL를 통해 접근하는 것을 차단하는 방법</u> → **웹 서버의 설정으로 쉽게 막을 수 있음**
            + 예시: Nginx 설정
                ```nginx
                location ~ /\.(git|hg) {
                deny all;
                }
                ```

<br/><br/>

### 편의성을 위한 설정에 의해 발생하는 문제점
개발자를 위한 설정이 서비스 환경에서도 켜져 있어 공격자가 이를 통해 시스템 내부의 정보를 알아내 추가적인 공격에 사용될 수 있음

<br/>

* 편의성을 위한 설정에 의해 발생하는 Misconfiguration: ***debug***
    | | 설명 |
    |:---:|------|
    | 발생하는 경우 | 디버그 모드 설정 또는 디버그 목적으로 **코드 상에서 특정 정보를 사용자에게 제공할 경우**에 발생함 <br/> &nbsp;&nbsp; - 서버 환경설정 또는 프레임워크 등의 어플리케이션 구동 시 **환경 설정의 Debug 옵션을 설정**해 운용하는 경우 |
    | 공격 결과 | 정보를 기반으로 서버의 환경 및 정보가 노출될 수 있음 <br/> &nbsp;&nbsp; - Debug 옵션 설정 시 사용자의 입력 데이터에 의해 에러가 발생하게 되면 해당 에러에 대한 정보가 노출될 수 있음 <br/><br/>  ***→ 공격자는 이런 정보들을 통해 서버를 공격하기 위한 기반 정보들을 획득함*** |
    - 문제점 발생 예시: Debug 모드가 설정된 코드
        ```python
        #!usr/bin/env python3
        from flask import Flask, request

        app = Flask(name)

        @app.route('/')
        def index():
            return 'hi'

        @app.route('/download')
        def download():
            file = request.args.get("file")
            return open(file).read()
        
        app.run(host='0.0.0.0', port=8000, debug=True) # debug=True → 어플리케이션 구동 시 Debug 옵션을 설정해 운용하고 있음
        ```
        + ⚠️ debug 모드가 설정된 상태에서 에러가 발생하게 되면 에러의 정보와 에러가 발생하는 서버 코드가 함께 노출되는 위험이 발생하기도 함

<br/>

* 편의성을 위한 설정에 의해 발생하는 Misconfiguration: ***Error Message Disclosure***
    | | 설명 |
    |:---:|------|
    | 발생하는 경우 | 코드 상에서 사용하는 변수 또는 정보가 디버그 목적 등으로 사용자들에게 노출되는 경우에 발생함 |
    | 공격 결과 | ⚠️ 디버그 목적으로 사용한 후 삭제하지 않거나 그대로 사용할 경우 **악의적인 공격자에게 정보가 노출될 수 있음** |
    - 문제점 발생 예시: 에러에 대한 상세한 내용을 출력하여 디버그하는 경우
        ```python
        #!usr/bin/env/ python3
        from flask import Flask, request

        app = Flask(__name__)

        @app.route('/')
        def index():
            return 'hi'
            
        @app.route('/download')
        def download():
            try:
                file = request.args.get("file")
                return open(file).read()
            except Exception as e:
                return str(e.args) # 에러에 대한 상세한 내용을 출력하고 있음
            
        app.run(host='0.0.0.0', port=8000)
        ```

<br/>

* 편의성을 위한 설정에 의해 발생하는 Misconfiguration: ***0.0.0.0으로 바인딩된 네트워크 설정***
    | | 설명 |
    |:---:|------|
    | 발생 위치 | 내부 서비스 네트워크 또는 특정 네트워크에서만 접근할 수 있도록 접근 제어가 이루어져야 할 서버들에서 발생함 |
    | 발생하는 경우 | 개발 및 운용 상의 편의를 위해 ```0.0.0.0```으로 바인딩되어 있다가 서비스 환경으로 변경되었음에도 바인딩 주소를 그대로 <br/>사용할 때 발생함 <br/> &nbsp;&nbsp; - 💡 ```0.0.0.0```은 **모든 IPv4 주소**를 의미함 |
    | 공격 결과 | 이를 통해 악의적인 공격자는 인증이 없거나 취약한 내부 서비스에 접근할 수 있게 됨 <br/> ***⇒ "공격자는 기본적으로 인증을 사용하지 않거나 취약한 기본 계정을 자주 사용하는 서비스를 대상으로 공격함"을 의미함*** |
    - 웹 환경 개발 시 자주 쓰이는 내부 서비스
        | 서비스명 | 용도 |
        |:---:|----|
        | redis | cache 서버 |
        | mysql | RDBMS(관계형 DBMS) 서버 |
        | k8s master mode | 클러스터 관리 |
    - 공격 예시
      <br/><br/>
      <img width="1512" alt="취약한 redis server 설정" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6c94f3bb-fcac-4298-a0c8-4b1b8b201eeb">
      <br/><br/>
    - 조치 방안
        | | 방법 |
        |:---:|------|
        | 01 | ***허용할 포트를 제외한 설정은 모두 삭제하는 것*** <br/><br/> - 클라우드 환경에서 제공하는 방화벽 기능을 이용하여 포트를 제어할 수 있음 <br/> - ⚠️ 인스턴스 이미지를 복사해 방화벽이 허용된 곳에서 사용한다면 취약함 |
        | 02 | ***설정 파일에서 취약한 부분을 찾아 패치하는 것*** ← 근본적인 방법 <br/><br/> - EX: Ubuntu 환경에서 apt로 redis 패키지를 설치 → ```/etc/redis/redis.conf``` 경로에 설정 파일이 존재함 <br/> &nbsp;&nbsp; [과정 1] ```0.0.0.0```으로 바인딩되어 있는 라인을 찾아서 삭제 <br/> &nbsp;&nbsp; [과정 2] ```bind 127.0.0.1 ::1```로 설정 → 로컬 머신에서만 접근 가능 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 접속하는 대상의 아이피를 명시할 수 있음 → 내부 서비스 네트워크에서만 접속 가능 |

<br/><br/>

### 메뉴얼과 실제 구현체의 차이로 인해 발생하는 문제점
메뉴얼에 설명되어있는 대로 사용했지만 중의적 표현이 존재하거나 잘못 설명되어 있을 때 발생함

<br/>

* 메뉴얼과 실제 구현체의 차이로 인해 발생하는 Misconfiguration: ***Nginx alias Path Traversal 취약점***
    - Nginx에서 경로를 설정할 때 사용하는 2가지 방식
        | 방식 | 설명 |
        |:---:|------|
        | ```alias``` | 요청한 경로를 지시한 **다른 경로로 변경**하는 역할을 함 <br/> → Nginx docs: alias [🔗](https://nginx.org/en/docs/http/ngx_http_core_module.html#alias) 참고 |
        | ```root``` | 해당 경로의 **root 경로를 명시**해주는 역할을 함 <br/> → Nginx docs: root [🔗](https://nginx.org/en/docs/http/ngx_http_core_module.html#root) 참고 |
    - ```alias```를 사용하는 경우 발생할 수 있는 문제점
        ```Nginx
        # Defines  a replacement for the specified location. For example, with the following configuration

        location /i/ {
            alias /data/w3/images/;
        }

        # on request of "/i/top.gif", the file /data/w3/images/top.gif will be sent. (alias)
        ```
        + 사용자가 ```location``` 지정 시 ```/i/```와 ```/i```의 차이점
            | | 차이점 설명 |
            |:---:|------|
            | ```location /i/``` <br/> 사용 | ```http://server.com/i/hello.png```로 요청 시 <br/>```/data/w3/images/hello.png```를 가져옴 |
            | ```location /i``` <br/> 사용 | ```http://server.com/i/hello.png```로 요청 시 ```/data/w3/images//hello.png```를 가져옴 <br/> ```http://server.com/i../hello.png```로 요청 시 ```/data/w3/images/../hello.png```를 가져옴 <br/> ***→ 📌 한 단계 상위 디렉토리의 파일을 가져올 수 있음*** |
    - 조치 방안
        | | 방법 |
        |:---:|------|
        | 01 | ***alias 대신에 root directive를 사용함*** (권장) |
        | 02 | ***location 마지막에 /를 붙이거나 alias 맨 뒤의 /를 삭제함*** <br/> - 정상적인 요청은 ```openat```이 성공하고, 악의적인 요청은 ```ENOENT``` 에러와 함께 실패함 |

<br/><br/>

### 해당 코드나 설정에 대한 이해 없이 사용해 발생하는 문제점
취약점이 존재하는 예제 코드, StackOverFlow 답변 등을 이해 없이 복사 붙여넣기 해 사용하거나 메뉴얼에 존재하는 권고 설정을 무시한 채 사용해 발생함

<br/>

* 해당 코드나 설정에 대한 이해 없이 사용해 발생하는 Misconfiguration: ***StackOverflow 답변 등을 복사하여 사용 (Nginx proxy SSRF)***
    - Nginx에서 외부 이미지를 다운받아 화면에 보여주는 URL을 아래와 같이 구성하는 엔드포인트를 만드는 경우에 대한 StackOverflow 답변 링크
        | | 설명 |
        |:---:|------|
        | URL 구성 | ```https://dreamhack.io/image/{image URL}``` <br/> ```https://dreamhack.io/image/http://imageviewr.com/941ba06f-fb7e-46e1-bcba-55a1306d6aa7.png``` |
        | StackOverflow | StackOverflow: Reverse image porxy without specifying host [🔗](https://stackoverflow.com/questions/47404893/reverse-image-proxy-without-specifying-host) <br/> StackOverflow: nginx proxy_pass and URL decoding [🔗](https://stackoverflow.com/questions/28995818/nginx-proxy-pass-and-url-decoding) |
        + StackOverflow 내용
            > The question is quite vague, but, based on the error message, what you're trying to do is perform a ```proxy_pass``` entirely based on the user input, by using the complete URL specified after the ```/image/``` prefix of the URI.
            >
            > ⚠️ **Bascially, this is very bad idea, as you're opening yourself to become an open proxy.** However, the reason it doesn't work as in the conf you supplied is due to URL normalisation, which, in your case, compacts ```https://example``` into ```http:/example``` (double slash becomes single), which is different in the context of ```proxy_pass```.
            >
            > If you don't care about security, you can just change ```merge_slashes``` from the default of ```on``` to ```off```:
            >   ```nginx
            >   merge_slashes off;
            >   location ...
            >   ```
            >
            > Another possibility is to somewhat related to nginx proxy pass and URL decoding:
            >   ```nginx
            >   location ~ ^/image/.+{
            >       rewrite ^ $request_uri;
            >       rewrite ^/image/(.*) $1 break;
            >       return 400;
            >       proxy_pass $uri: # will result in an open-proxy, don't try at home
            >   }
            >   ```
            - ⚠️ **Open-Proxy가 될 수 있다고 답변 작성자는 경고하고 있음** → 제대로 읽지 않고 그대로 사용할 경우 ```/image/``` location에서 SSRF 취약점이 발생해 nginx를 통해 내부 서비스 네트워크에 접근할 수 있음
                + ```http://dreamhack.io/image/http://127.0.0.1:1234```를 접속하게 되면 로컬 호스트에서 연결이 맺어지는 것을 확인할 수 있음
                  <br/><br/>
                  <img width="1512" alt="Nginx proxy ssrf" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/bd02568a-6906-4939-8a63-de79e1275cac">
		  <br/><br/>
    - 공격 시나리오
      <br/><br/>
      <img width="1512" alt="공격 시나리오" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/63eb0969-81cb-4410-98b1-0a38e6dd9091">
      <br/><br/>
    - 조치 방안
        + ***SSRF 취약점으로부터 안전해지기 위해서는 설정 파일을 수정해야 함***
            - 개발자가 미리 정의한 호스트에서만 요청할 수 있도록 변경함
                ```nginx
                location ~ ^/image/(https?):/((?:hostA|hostB)(?::\d+)?)/(.*) {
                    proxy_pass $1://$2/$3; # hostA, hostB를 제외한 어떠한 호스트도 프록시 패스를 허용하지 않음
                }
                ```

<br/>

* 해당 코드나 설정에 대한 이해 없이 사용해 발생하는 Misconfiguration: ***모든 도메인을 허용한 CORS 설정***
