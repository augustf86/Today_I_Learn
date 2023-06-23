# DBMS Misconfiguration

DBMS의 설정 항목을 잘못 설정했을 때 발생할 수 있는 문제점
* 각각의 DBMS는 설정 파일을 통해 특정 기능을 활성화하거나 접근 가능한 디렉토리 등을 설정할 수 있음
    - 개발 과정에서 마주하는 오류를 쉽게 해결하고 접근성을 위해서 **보안을 위해 기본적으로 비활성화되어 있는 일부 기능들을 활성화하는 경우가 존재함** → 이를 이용해 공격자는 공격을 수행할 수 있음

<br/>

> ### OWASP TOP 10 2021: "Security Misconfiguration"
> * 계정 및 권한이 적절하게 분리되지 않았거나 불필요한 기능의 활성화, 그리고 데이터베이스의 보안 설정이 미흡한 경우 등을 포함함

<br/><br/>

## Out of DBMS
DBMS에서 제공하는 특별한 함수 또는 기능 등을 이용해 **파일 시스템, 네트워크, OS 명렁어 등에도 접근**할 수 있음 <br/> &nbsp;&nbsp; → 이를 통해 SQL Injection으로 단순히 데이터베이스의 정보만 획득하는 것이 아니라 **파일 시스템, 네트워크 시스템 모두 장악할 수 있음**
* DBMS의 버전과 설정에 따라 정상적으로 동작하지 않을 수 있음
    - DBMS의 버전이 올라갈수록 위험한 함수나 기능을 제거하거나, 기본 설정/권한으로 접근하지 못하게 하는 등 다양한 방법으로 이러한 공격에 대한 패치를 진행하고 있기 때문
    
<br/>

### Out of DBMS: MySQL
* MySQL에서의 파일 관련 작업
    - mysql 권한으로 수행되며, "my.cnf" 설정 파일의 **secure_file_priv** 값에 영향을 받음
    - [secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_secure_file_priv)
        + mysql 쿼리 내에서 ```LOAD_FILE```, ```OUTFILE```을 이용해 파일에 접근할 때 접근할 수 있는 파일 경로에 대한 정보를 가지고 있음 (FILE 권한이 있는 사용자에게만 허용됨)
        + 설정값의 종류
            | Values | 형태 | 설명 |
            |-----|-----|----------|
            | empty string | ```secure_file_priv = "" ``` | 미설정 = 해당 변수가 영향을 미치지 않음 (보안 설정이 아님) |
            | dirname | ```secure_file_priv = "\tmp"``` | 파일에 대한 작업을 해당 디렉토리의 하위 경로 및 파일에 대해서만 작업하도록 제한함 (존재하는 디렉토리로 설정해야 함) |
            | NULL | ```secure_file_priv = NULL``` | 파일에 대한 작업을 비활성화함 |
            - 기본값: STANDALONE이면 비어있고(empty), DEB/RPM/SVR4의 경우 **var/lib/mysql-files**로 되어 있음
        + 조회 방법
            ```sql
            SELECT @@secure_file_priv; -- 결과: /var/lib/mysql-files;
            ```
* ```LOAD_FILE``` 함수: 인자로 전달된 파일을 읽고 출력함
    - 인자로 전달된 파일은 전체 경로를 입력해야 하며 해당 파일에 접근 권한이 있어야 함
    - 예시
        ```sql
        -- /var/lib/mysql-files/test를 조회 (test 파일 내에는 12345678이 작성되어 있음)
        SELECT LOAD_FILE('/var/lib/mysql-files/test'); -- 결과: 12345678이 출력됨
        ```
* ```SELECT ... INTO``` 형식의 쿼리문: 쿼리 결과를 변수나 파일에 쓸 수 있음
    - ```SELECT ... INTO```의 종류
        | 쿼리문 | 설명 |
        |---|------|
        | ```SELECT ... INTO var_list``` | 열 값을 선택하여 변수에 저장함 |
        | ```SELECT ... INTO OUTFILE 'filename'``` | 쿼리 결과의 행을 파일에 기록함 (column/line terminators를 지정하여 특정 출력 포맷을 생성할 수 있음) |
        | ```SELECT ... INTO DUMPFILE 'filename'``` | 쿼리 결과를 포맷 없이 단일 행(single row)을 파일에 기록함 |
    - secure_file_priv 값이 올바르지 않아 임의 경로에서 파일 작업을 수행할 수 있다면 이를 통해 웹 셸을 업로드하는 등의 공격이 가능함
        ```sql
        SELECT '<?=`ls`?>' INTO OUTFILE '/tmp/a.php';
        /* <?php include $_GET['page'].'.php'; // "?page=../../../tmp/a" */
        ```
        

<br/>

### Out of DBMS: MSSQL
* ```xp_cmdshell``` 기능을 이용해 OS 명령어를 실행할 수 있음
    - SQL Server 2005 버전 이후부터는 기본적으로 **비활성화**되어 있어 임의로 활성화하지 않는 이상 해당 기능을 이용한 공격은 불가능함 → 해당 기능이 활성화되어 있는지 확인해야 함
    - ```xp_cmdshell``` 기능의 활성화 여부를 판단하는 쿼리
        ```sql
        SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell'
        -- 결과로 1이 반환되면 활성화, 0이 반환되면 비활성화된 상태임을 의미함
        ```
    - ```xp_cmdshell```이 활성화되어 있는 경우 OS 명령어를 실행시키는 방법
        ```sql
        -- EXEC [저장프로시저 이름] [매개변수1], [매개변수2], ...;
        -- 저장 프로시저의 이름에 xp_cmdshell을, 매개변수의 위치에 실행시킬 OS 명령어를 문자열로 전달함
        EXEC xp_cmdshell "net user";
        EXEC master.dbo.xp_cmdshell 'ping 127.0.0.1';
        ```

<br/><br/>

## DBMS 권한 문제 주의사항
* DBMS 애플리케이션 작동 권한
    - 리눅스 서버는 이용자 별로 권한을 나누어 관리함 → 서버에서 DBMS 작동 시에는 **DBMS 전용 계정**(nologin)을 만들어 사용해야 함
        + 루트 계정이나 다른 애플리케이션 권한(www-data)가 탈취되면 큰 문제가 발생할 수 있기 때문
        + 계정을 분리하지 않을 경우 파일 시스템, 네트워크에 접근이 가능하고 OS 명령어를 실행할 수 있게 됨 

<br/>

* DBMS 계정 권한
    - DBMS 내부에 계정과 권한이 존재함 → 루트 계정의 사용을 지양하고, **서비스 및 기능 별로 계정을 생성해 사용**해야 함
        + 웹 애플리케이션에서 DBMS의 루트 계정을 사용하고, 여러 서비스에서 같은 DBMS를 사용할 때 접속 계정을 분리하지 않는 경우 문제가 발생함
        + 한 서비스에서 SQL Injection 취약점이 발생하면 서비스에서 사용하는 DBMS 계정으로 다른 데이터베이스에 접근할 수 있음

<br/><br/>

## DBMS 문자열 비교 주의사항
* 각각의 DBMS는 문자열을 비교하는 방법이 다름 → 웹 애플리케이션과 DBMS에서 문자열을 비교하는 방식이 다를 수 있어 개발자의 의도와 다르게 동작하는 경우가 존재함 (DBMS에 따라 우회가 가능할 수 있음)
    | CASE | 설명 |
    |---|------|
    | 대소문자 구분 | MySQL, MSSQL 등 일부 DBMS의 비교 연산에서 대소문자를 구분하지 않음 <br/> &nbsp;&nbsp; → 소문자와 대문자를 비교한 연산에서 참을 반환함 |
    | 공백 문자로 끝나는 문자열 비교 | MySQL, MSSQL 등 일부 DBMS의 비교 연산에서 할당된 컬럼의 크기에 맞게 공백 문자를 채운 후에 비교함 <br/> &nbsp;&nbsp; → 공백을 추가하지 않은 문자와 공백을 추가한 문자를 비교한 연산에서 참을 반환함 |

    - DBMS 별 문자열 비교 예시
        | CASE | MySQL DBMS | MSSQL DBMS | 결과 |
        |---|-----|-----|---|
        | 대소문자 구분 | ```SELECT 'a'='A';``` | ```SELECT 1 FROM test WHERE 'a'='A';``` | 1 (참) |
        | 공백 문자로 끝나는 문자열 비교 | ```SELECT 'a'='a ';``` | ```SELECT 1 FROM test WHERE 'a'='a ';``` | 1 (참) |

<br/><br/>

## DBMS 다중 쿼리 주의사항
* 다중 쿼리(Multi Query): 하나의 요청에 다수의 쿼리 구문을 사용하는 것
    - 대부분의 웹 애플리케이션은 DBMS에 쿼리를 전송할 때 다중 쿼리를 지원하지 않음
        + SQL Injection 취약점이 발생했을 때 해당 쿼리를 벗어나서 새로운 쿼리를 작송하지 못하도록 하여 공격을 방지하기 위함
    - 애플리케이션에서 다중 쿼리를 지원할 경우 공격자는 **본래 실행되는 쿼리문에 새로운 쿼리를 작성**해 삭제/추가/수정 등의 행위가 가능함
        + 예시: PHP의 PHP Data Object(PDO)
            | PDO 함수 | 설명 |
            |---|------|
            | ```query``` 함수 | 다중 쿼리를 지원하지 않음 |
            | ```exec``` 함수 | 다중 쿼리를 지원함 |
            - ```exec``` 함수 사용 시 다중 쿼리를 실행시킬 수 있음 (사용하지 않아야 함)

<br/><br/><br/><br/>

### 🔖 출처
* [드림핵 Web Hacking Advanced - Server Side] 📌 [ExploitTech: DBMS Misconfiguration](https://dreamhack.io/lecture/courses/288)
