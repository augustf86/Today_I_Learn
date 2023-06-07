# SQL

#### SQL에서 제공하는 3가지 질의 언어
* 데이터 정의어(Data Definition Language, DDL): 테이블이나 관계의 구조를 생성하는 데 사용함
    - CREATE, SELECT, DROP 문 등이 있음
* 데이터 조작어(Data Manipulation Language, DML): 테이블에 데이터를 검색, 삽입, 수정, 삭제하는 데 사용함
    - SELECT, INSERT, DELETE, UPDATE 문 등이 있음
    - SELECT 문은 특별히 질의어(query)라고 부름
    - **우리가 웹 서비스에서 이용하는 대다수의 기능이 DML을 통해 처리됨**
* 데이터 제어어(Data Control Language, DCL): 데이터의 사용 권한을 관리하는 데 사용함
    - GRANT, REVOKE 문 등이 있음

#### SQL문의 특징
* 실행 순서가 없는 비절차적인 언어(non-procedural) 언어로, 찾는 데이터만 기술하고 어떻게 찾는지 그 절차(실행 순서)는 기술하지 않음
    - 이 부분은 DBMS가 내부적으로 처리함
* 세미콜론(;)과 함께 끝나지만 이 세미콜론은 생략 가능함
    - 가능하다면 세미콜론은 생략하지 않고 작성하는 것을 권장함
* 대소문자의 구분은 없지만 SQL의 예약어는 대문자로, 테이블이나 속성 이름은 소문자로 적어주면 읽기 쉬움 (관례)

<br/><br/>

## SQL DML
### SELECT
데이터를 조회하는 구문 (= 데이터를 검색하는 기본 문장, 질의어(query)라고 부름)
* MySQL 공식 홈페이지에서 제공하는 [SELECT 구문](https://dev.mysql.com/doc/refman/8.0/en/select.html) 형식
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
    - SELECT 구문의 간단한 문법 형식: ```SELECT 열1[, 열2, ...] FROM 테이블이름 [WHERE <검색조건>] [GROUP BY 속성이름] [ORDER BY 속성이름 [ASC | DESC]];```
        | 절 | 설명 |
        |---|-----|
        | SELECT | 해당 문자열을 시작으로, 조회하기 위한 표현식과 컬럼들에 대해 정의함 (열의 순서는 결과 테이블의 열 순서를 결정하며, 기본적으로 중복을 제거하지 않음) <br/>&nbsp; → ```SELECT *```은 테이블의 모든 열을 조회한다는 의미 |
        | FROM | 데이터를 조회할 테이블의 이름 |
        | WHERE | 조회할 데이터의 조건 (조건으로 사용할 수 술어(predicate)가 존재하며, 복합조건 사용 시 논리 연산자 AND, OR, NOT을 사용함) |
        | ORDER BY | 조회한 결과를 원하는 컬럼 기준으로 정렬함 (정렬을 원하는 열 이름을 순서대로 지정하며, 기본은 오름차순(ACS)임) |
        | LIMIT | 조회한 결과에서 행의 갯수와 오프셋을 지정함 |
        + WHERE 절에서 조건으로 사용할 수 있는 술어(predicate)
            - 비교: ```=, <>, <, <=, >, >=``` 등을 사용
            - 범위: ```BETWEEN``` 연산자를 사용하면 값의 범위를 지정할 수 있음 (논리 연산자 ```AND```을 함께 사용함)
                + 조건이 가격이 10000 이상 20000 이하일 경우 ```WHERE price BETWEEN 10000 AND 20000;```
            - 집합: 두 개 이상의 값을 비교하기 위해 ```IN```, ```NOT IN``` 연산자를 이용
                + ```IN``` 연산자: 집합의 원소인지 판단하는 연산자로, 문자 값들을 괄호 안에 포함시켜 비교하며 값이 이 중 하나와 같으면 선택함
            - 패턴: 문자열의 패턴 비교를 위해 ```LIKE``` 연산자를 사용함 (찾는 속성이 텍스트 혹은 날짜 데이터를 포함하면 비교값에는 반드시 영문 작은 따옴표로 둘러싸야 하며)
                | 와일드 문자 | 의미 | 사용 예시 |
                |---|-----|-----|
                | + | 문자열을 연결 | ```'admin'+user'```: 'adminuser' |
                | % | 0개 이상의 문자열과 일치 | ```'%admin%'```: admin을 포함하는 문자열 |
                | [] | 1개 이상의 문자열과 일치 | ```'[0-5]%'```: 0-5 사이 숫자로 시작하는 문자열 |
                | [^] | 1개 이상의 문자열과 불일치 | ```'[^0-5]%'```: 0-5 사이의 숫자로 시작하는 문자열 |
                | _ | 특정 위치의 1개의 문자와 일치 | ```'_d%'```; 두 번째 위치에 'd'가 들어가는 문자열 |
            - NULL
            - 복합 조건: 논리 연산자 AND, OR, NOT을 사용하면 복합 조건을 명시할 수 있음
* SELECT 구문 예시
    ```sql
    SELECT uid, title, boardcontent # uid, title, boardcontent 열을 조회 (결과의 열 순서도 동일함)
    FROM board # board 테이블에서
    WHERE boardcontent LIKE '%abc%' # 조건(패턴 비교): 'abc'를 포함하는 문자열
    ORDER BY uid DESC # 정렬: uid를 기준으로 내림차순(DESC) 정렬
    LIMIT 5; # 결과 테이블의 행의 수 = 5
    ```

<br/>

### INSERT
데이터를 추가하는 구문
* MySQL 공식 홈페이지에서 제공하는 [INSERT 구문](https://dev.mysql.com/doc/refman/8.0/en/insert.html) 형식
    ```sql
    INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
        [INTO] tbl_name
        [PARTITION (partition_name [, partition_name] ...)]
        [(col_name [, col_name] ...)]
        { {VALUES | VALUE} (value_list) [, (value_list)] ... }
        [AS row_alias[(col_alias [, col_alias] ...)]]
        [ON DUPLICATE KEY UPDATE assignment_list]
    ```
    - INSERT 구문의 간단한 문법 형식: ```INSERT INTO 테이블이름[(속성리스트)] VALUES (값 리스트);```
        | 절 | 설명 |
        |---|-----|
        | INSERT | 해당 문자열을 시작으로, 추가할 테이블과 데이터를 정의함 |
        | INTO | 데이터를 추가할 테이블의 이름과 컬럼을 정의함 (새로윤 행을 삽입할 때에는 속성의 이름을 생략할 수 있음) <br/> &nbsp; → **데이터의 입력 순서는 속성의 순서와 일치해야 함** |
        | VALUES | INTO 절에서 정의한 테이블의 컬럼에 명시한 데이터들을 추가함 |
* INSERT 구문 예시
    ```sql
    # 예시 1: 쿼리 한 줄로 두 개 이상의 데이터를 추가
    INSERT
        INTO board (title, boardcontent) # board 테이블의 title, boardcontent 열에 순서대로 추가
        VALUES ('title 1', 'content 1'), ('title 2', 'content 2'); # 추가할 값 (소괄호의 첫 번째 항목은 title 열에, 두 번째 항목은 boardcontent 열에 순서대로 추가됨)
    
    # 예시 2: 서브 쿼리를 통해 다른 테이블에 존재하는 데이터를 추갛
    INSERT
        INTO board (title, boardcontent) # board 테이블의 title, boardcontent 열에 순서대로 추가
        VALUES ('title 1', (SELECT upw FROM users WHERE uid='admin')); # 추가할 값 (소괄호의 두 번째 항목인 서브 쿼리의 결과를 boardcontent 열에 그대로 삽입함)
    ```

<br/>

### UPDATE
데이터를 수정하는 구문
* MySQL 공식 홈페이지에서 제공하는 [UPDATE 구문](https://dev.mysql.com/doc/refman/8.0/en/update.html) 형식
    ```sql
    UPDATE [LOW_PRIORITY] [IGNORE] table_reference
        SET assignment_list
        [WHERE where_condition]
        [ORDER BY ...]
        [LIMIT row_count]
    ```
    - UPDATE 구문의 간단한 문법 형식: ```UPDATE 테이블이름 SET 속성이름1=값1[, 속성이름2=값2, ...] [HWERE <검색조건>];```
        | 절 | 설명 |
        |---|-----|
        | UPDATE | 해당 문자열을 시작으로, 수정할 테이블을 정의함 |
        | SET | 수정할 컬럼과 데이터를 정의함 (데이터는 SELECT문을 사용하여 다른 테이블의 속성값을 이용할 수 있음) |
        | WHERE | 수정할 행의 조건을 명시함 |
* UPDATE 구문 예시
    ```sql
    UPDATE board # board 테이블을 수정할 것이라고 명시
        SET boardcontent = "update content 2" # (수정할 컬럼) = (수정할 데이터) 형식으로 정의함
        WHERE title = 'title 1'; # 조건: title 행의 값이 'title 1'인 경우
    ```

<br/>

### DELETE
데이터를 삭제하는 구문
* MysQL 공식 홈페이지에서 제공하는 [DELETE 구문](https://dev.mysql.com/doc/refman/8.0/en/delete.html) 형식
    ```sql
    DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name [[AS] tbl_alias]
        [PARTITION (partition_name [, partition_name] ...)]
        [WHERE where_condition]
        [ORDER BY ...]
        [LIMIT row_count]
    ```
    - DELETE 구문의 간단한 문법 형식: ```DELETE FROM 테이블이름 [WHERE <검색조건>];```
        | 절 | 설명 |
        |---|-----|
        | DELETE | 해당 문자열을 시작으로, 이후에 삭제할 테이블을 정의함 |
        | FROM | 삭제할 테이블을 정의함 |
        | WHERE | (생략 가능) 삭제할 행의 조건을 정의함 (없으면 모든 행을 삭제함) |
* DELETE 구문 예시
    ```sql
    DELETE FROM board # board 테이블에서 행 삭제함
        WHERE title='title 1'; # 조건: title의 값이 'title 1'인 경우
    ```

<br/><br/>


## SQL Features
