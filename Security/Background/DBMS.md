# Background: DBMS

### 데이터베이스(Database)
*  정보를 기록하기 위해 사용함
	- 다수의 사람들이 데이터를 생성﹒참조﹒수정하는 웹 서버의 특수한 환경에 가장 적합한 자료구조로 채택됨
	- **최근의 웹 서비스에는 거의 필수적으로 포함되어야 함**
* 데이터베이스 관리 시스템(Database Management System, DBMS): 데이터베이스를 관리하는 애플리케이션

<br/>

### Database Management System(DBMS)
* 웹 서버가 데이터베이스에 정보를 저장하고, 이를 관리하기 위해서 사용함
	- 역할 > 데이터베이스에 새로운 정보를 기록하거나, 기록된 내용을 수정﹒삭제하는 역할을 함
	- 특징 > 다수의 사람이 동시에 데이터베이스에 접근할 수 있고, 복잡한 요구사항을 만족하는 데이터를 조회할 수 있음
* DBMS의 종류 - 관계형과 비관계형을 기준으로 분류함
	| 종류 | 대표적인 DBMS | 차이점 |
  |--------|---------|---------------------|
	| 관계형(Relational) | MySQL, MariaDB, PostgreSQL, SQLite | 행과 열의 집합인 테이블 형식으로 데이터를 저장함 |
	| 비관계형(Non-Relational) | MongoDB, CouchDB, Redis | 키-값(Key-Value) 형태로 값을 저장함 |

<br/><br/>

## Relational DBMS (RDBMS, 관계형 DBMS)
* 행(Row)과 열(Column)의 집합으로 구성된 테이블의 묶음 형식으로 데이터를 관리함
* 테이블 형식의 데이터를 조작할 수 있는 관계 연산자를 제공함
	- 관계 연산자로 **Structured Query Language**(SQL)라는 쿼리 언어를 사용함 → 쿼리를 통해 테이블 형식의 데이터를 조작함

<br/>

### SQL(Structured Query Language)
RDBMS의 데이터를 관리하기 위해 설계된 특수 목적의 프로그래밍 언어 <br/>
&nbsp;&nbsp; = 데이터베이스로부터 정보를 얻거나 수정하기 위한 표준 대화식 프로그래밍 언어 <br/>
&nbsp;&nbsp; = RDBMS의 데이터를 정의하고 질의, 수정하기 위해 고안된 언어
* 구조화된 형태를 가지고 있으며, 웹 애플리케이션이 DBMS와 상호작용할 때 사용함
	- 대다수의 상용 또는 공개 데이터베이스 프로그램은 SQL를 표준으로 함
	- 명령어는 대﹒소문자를 구분하지 않지만 가독성을 위해 명령어는 대문자로, 데이터베이스나 테이블, 컬럼 이름 등은 소문자로 나타냄
* 사용 목적과 행위에 따라 다양한 구조가 존재함
	| 언어 | 설명 |
  |------|--------------|
	| DDL (Data Definition Language) | 데이터를 정의하기 위한 언어<br/> → 데이터를 저장하기 위한 스키마, 데이터베이스의 생성/수정/삭제 등의 행위를 수행함 |
	| DML (Data Manipulation Language) | 데이터를 조작하기 위한 언어<br/> → 실제 데이터베이스 내에 존재하는 데이터에 대해 조회﹒저장﹒수정﹒삭제 등의 행위를 수행함 |
	| DCL (Data Control Language) | 데이터베이스의 접근 권한 등을 설정하기 위한 언어<br/> → 데이터베이스 내의 이용자의 권한을 부여하기 위한 ```GRANT```와 권한을 박탈하는 ```REVOKE```가 대표적임 |

#### DDL (Data Definition Language)
* DDL의 종류에는 ```CREATE```(생성), ```ALTER```(수정), ```DROP```(삭제)가 있으며, 해당 키워드 뒤에 ```DATABASE```, ```TABLE```, ```INDEX```를 명시하여 무엇을 정의할 것인지 구분함
	| 종류 | 설명 | 종류 | 설명 |
	|----|----|----|----|
	| CREATE DATABASE | 새로운 데이터베이스 생성 | ALTER DATABASE | 데이터베이스 수정 |
	| CREATE TABLE | 새로운 테이블 생성 | ALTER TABLE | 테이블 수정 |
	| DROP TABLE | 테이블 삭제 | | |
	| CREATE INDEX | 인덱스(검색 키) 생성 | DROP INDEX | 인덱스 삭제 |

* RDBMS의 기본적인 구조: *데이터베이스 → 테이블 → 데이터* 구조
	- 데이터를 다루기 위해서는 데이터베이스와 테이블을 생성해야 함 → DDL의 ```CREATE```를 사용해 새로운 데이터베이스 또는 테이블을 생성할 수 있음
* DDL 예시
	```SQL
	CREATE DATABASE MyDB; # MyDB라는 데이터베이스를 생성

	USE MyDB; #. MyDB 데이터베이스의
	CREATE TABLE Board { # Board 테이블 생성
		idx INT AUTO_INCREMENT,
		boardTitle VARCHAR(100) NOT NULL,
		boardContent VARCHAR(2000) NOT NULL,
		PRIMARY KEY(idx)
	);
	```
#### DML (Data Manipulation Language)
* 생성된 테이블에 데이터를 조회﹒저장﹒수정﹒삭제하기 위해 사용함
	| 종류 | 설명 |
	|-----|----------|
	| SELECT | 데이터베이스에서 데이터를 추출(검색/조회) |
	| UPDATE | 데이터베이스에 있는 데이터 수정(갱신) |
	| DELETE | 데이터베이스에서 데이터 삭제 |
	| INSERT | 데이터베이스에 새로운 데이터 추가(삽입) |

* DML 예시
	- 테이블 데이터 생성(삽입)
		```SQL
		# Board 테이블에 데이터를 삽입하는 쿼리문
		INSERT INTO Board(boardTitle, boardContent, createdDate)
		VALUES (
			‘Title1’, ‘Hello World!’, Now()
		);
		```
		+ ```INSERT INTO Board``` 다음의 소괄호에 나열된 열과 ```VALUES``` 다음의 소괄호에 나열된 값들이 순서대로 일대일 대응되어 삽입됨
	- 테이블 데이터 조회(검색)
		```SQL
		# Board 테이블의 데이터를 조회하는 쿼리문 (idx의 값이 1인 행의 boardTitle, boardContent 열에 들어있는 데이터를 테이블 형식으로 출력함)
		SELECT boardTitle, boardContent FROM Board WHERE idx=1;
		```
		+ 테이블에 있는 특정 데이터를 조회하기 위해 ```WHERE``` 구문을 사용함
		+ **웹 애플리케이션에서는 ```SELECT``` 명령어로 데이터를 조회하는 경우가 많으므로 ```SELECT``` 명령어를 확실하게 알아두는 것이 좋음**
	- 테이블 데이터 변경
		```SQL
		# Board 테이블의 컬럼 값을 변경하는 쿼리문 (idx의 값인 1인 행의 boardContent 열에 들어있는 데이터를 ‘Change!’로 변경함)
		UPDATE Board SET boardContent=‘Change!’ WHERE idx=1;
		```
		+ **주의**: ```UPDATE```문에 반드시 ```WHERE``` 구문을 추가해서 특정 사용자만 업데이트해야 함

