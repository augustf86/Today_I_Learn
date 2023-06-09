# System Table & DBMS Fingerprinting

SQL Injection 취약점이 발생할 때 데이터베이스의 테이블과 컬럼 등 데이터베이스가 저장하고 있는 중요 정보를 수집하는 방법
* 핑거프린팅(Fingerprinting)
    - 공격 대상의 정보를 수집하는 과정
        + 공격 대상이 사용하는 웹 서버와 개발 언어, 그리고 DBMS의 내부 정보 등을 알아내는 과정 등이 해당됨
    - Penetration Testing Execution Standard(PTES)에서 정의한 모의 해킹의 7단계 중 1단계에 해당
        + 1단계: 공격 대상을 정하고 대상의 정보를 수집함 (가장 중요한 단계 중 하나)

<br/><br/>

## System Tables
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
