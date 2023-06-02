# SQL Injection

DBMS가 관리하는 데이터베이스에는 회원계정과 같은 민감한 정보가 포함되어 있을 수 있어 임의 정보 소유자 이외의 이용자에게 해당 정보가 노출되지 않도록 해야 함
* 공격자는 데이터베이스 파일 탈취, SQL Injection 공격 등으로 해당 정보를 확보하고 금전적인 이득을 얻을 수 있음

<br/>

#### Injection(인젝션)
인젝션 공격: 이용자의 입력값이 애플리케이션 처리 과정에서 구조나 문법적인 데이터로 해석되어 발생하는 취약점
* 공격자는 악의적인 입력값을 주입해 의도하지 않은 행위를 일으킴
* **SQL Injection**: DBMS에서 사용하는 질의 구문인 SQL(쿼리)를 삽입하여 데이터베이스의 정보를 획득하는 공격
	- 알려진 지 꽤 되었으나 여전히 수많은 웹 사이트가 SQL Injection 공격에 취약함
	- 많은 개발자도 이를 간과한 채 웹 애플리케이션을 개발하고 있음

<br/><br/>

### SQL Injection
이용자가 SQL 구문에 임의 문자열을 삽입하는 행위
* SQL Injection 공격에 취약점이 발생하는 곳 = 웹 애플리케이션과 데이터베이스가 연동되는 부분
	- 웹 서비스는 로그인 시의 ID와 PW, 게시글의 제목과 내용 등과 같은 이용자의 입력을 SQL 구문에 포함해 요청하는 경우가 있음
	- 임의로 작성한 SQL 구문을 삽입하는 것이므로 SQL에 대한 이해가 필수적임
		+ SQL에 대해 깊이 알수록 다양한 SQL Injection 공격과 필터링 우회가 가능함
* SQL Injection 발생 시 조작된 쿼리로 인증을 우회하거나, 데이터베이스의 정보를 유출할 수 있음
* 이용자의 입력값이 SQL 구문으로 해석되도록 하는 것이 제일 중요함

<br/>

#### SQL Injection 실습 - SQL Injection을 이용한 인증 우회
아이디(```uid```)와 비밀번호(```upw```)를 입력받고 DBMS의 user_table(```uid```와 ```upw```의 정보를 가지고 있는 테이블)에 조회하기 위한 쿼리(아래의 SQL 문)을 생성 및 실행하는 로그인 모듈
```SQL
# 로그인 페이지에서 uid와 upw에 아무것도 입력하지 않았을 때의 쿼리문
SELECT uid FROM user_table WHERE uid=‘’ AND upw=‘’
```
<br/>

1. 쿼리 질의를 통해 admin 결과를 반환하는 방법
	- 이용자의 입력값을 문자로 나타내기 위해 ```’``` 문자를 사용하는 경우 ```’``` 문자를 입력하여 SQL 구문으로 해석되도록 만듦
	    | uid | upw |
      |-----|-----|
      | admin’ or '1 | 아무것도 입력하지 않음 |

      + 생성되는 쿼리문
      ```SQL
      SELECT uid FROM user_table WHERE uid=‘admin’ or ‘1’ AND upw=‘’
      ```
      
		+ WHERE 절(조건)
		  - [조건1] ```uid=‘admin’``` = ```uid```가 ‘admin’d인 데이터 → admin의 결과를 반환함
		  - [조건2] ```’1’ and upw = ‘’```  = 참(True)이고 ```upw```가 없는(Null)인 경우 → 아무런 결과도 반환하지 않음 
	  + 두 조건이 ```or```로 결합되어 있으므로 결과적으로 ```uid```가 ‘admin’인 데이터를 반환함 → admin 결과를 반환하게 만들 수 있음
	- 이 외에도 주석(```—```, ```#```, ```/**/```)을 사용하는 등 다양한 방법으로 SQL Injection을 시도할 수 있음

<br/>

2. 쿼리 질의를 통해 admin의 upw를 알아내는 방법
	- ```UNION``` 문을 이용하여 로그인 모듈에 아래와 같이 입력
	    | uid | upw |
      |-----|-----|
      | 아무것도 입력하지 않음 | ‘ UNION SELECT upw FROM user_table WHERE uid=‘admin |
    
      + 생성되는 쿼리문
      ```SQL
      SELECT uid FROM user_table WHERE uid=‘’ AND upw ='' UNION SELECT upw FROM user_table WHERE uid=‘admin’
      ```
      
<br/>

> ### UNION
> * 2개 이상의 ```SELECT```문의 결과를 결합하는 데 사용함 <br/>&nbsp;&nbsp; = 한 번에 여러 테이블의 데이터를 조회할 때 사용함
>   - 특히 SQL Injection 공격에서 다른 테이블의 내용을 조회할 때 유용함
> * 결합 조건 (**모두 만족해야 ```UNION``` 문이 실행됨**)
>   - ``UNION``` 내의 모든 ```SELECT``` 문은 동일한 수의 열을 가져야 함 (동일하지 않으면 Error 발생)
>   - 열에는 유사한 데이터 유형이 존재해야 함
>   - 모든 SELECT 문의 열의 순서가 같아야 함
> * 기본적으로 고유한 값만 선택함 (중복 불가)
>   - 중복된 값을 허용하려면 ```UNION``` 대신 ```UNION ALL```을 사용함

<br/>

3. 쿼리 질의를 통해 admin 계정의 upw를 변경하는 방법
	- 2개의 SQL 구문 삽입 - 세미콜론(;)을 이용해 로그인 모듈에 아래와 같이 입력함
	    | uid | upw |
      |-----|-----|
      | admin’ | ‘; UPDATE user_table SET upw=‘admin’ WHERE uid=‘admin |

      + 생성되는 쿼리문
      ```SQL
      SELECT uid FROM user_table WHERE uid=‘admin’ AND upw=‘’; UPDATE user_table SET upw=‘admin’ WHERE uid=‘admin’
      ```

<br/>

> ### 세미콜론(;)
> 데이터베이스 쿼리문을 한 번 더 실행할 수 있는 구분자
> * 사용 시 쿼리문을 연달아 실행되도록 만들 수 있음

<br/>

#### DBMS의 종류에 따른 SQL Injection 공격
SQL Injection 공격을 할 때에는 데이터베이스의 종류에 따라 조금씩 다른 기법이 필요함
* 기업이 웹 애플리케이션을 구축할 때 가장 많이 사용하는 데이터베이스 - Microsoft SQL Server, 오라클
	- 각 데이터베이스마다 SQL Injection 공격이 상이할 수 있음

<br/><br/>

### Blind SQL Injection
