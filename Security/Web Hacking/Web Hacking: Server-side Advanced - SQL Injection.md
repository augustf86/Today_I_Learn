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

<br/><br/><br/>

## Exploit Techique
