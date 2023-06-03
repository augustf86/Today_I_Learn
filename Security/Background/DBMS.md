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
    - RDBMS: 스키마를 정의하고 해당 규격에 맞는 데이터를 2차원 테이블 형태로 저장함
        + 복잡하고, 저장해야 하는 데이터의 수가 늘어나면 용량의 한계에 다다를 수 있다는 단점이 존재함 → 이러한 단점을 해결하기 위해 **비관계형 데이터베이스(Non-Relational DBMS, NRDBMS, NoSQL)**임
    - RDBMS, NRDBMS 모두 SQL Injection이 발생할 수 있음
        + 이용자의 입력값을 통해 동적으로 쿼리를 생성해 데이터를 저장하는 점은 동일하기 때문

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

## Non-Relational DBMS (NRDBMS, 비관계형 데이터베이스)(NoSQL = Not Only SQL)
* 키-값(Key-Value)를 사용해 데이터를 저장함
* SQL을 사용하지 않고 복잡하지 않은 데이터를 저장함 → 단순 검색 및 추가 검색 작업을 위해 매우 최적화된 저장 공간임
    + SQL이라는 정해진 문법을 통해 데이터를 저장하는 RDBMS는 한 가지 언어로 다양한 DBMS를 사용할 수 있음
    + Redis, Dynamo, CouchDB, MongoDB 등 **다양한 DBMS별로 구조와 사용 문법을 배워야 한다**는 단점이 있음

<br/>

### MongoDB
* JSON 형태의 도큐먼트(Document)를 저장함
    - MongoDB의 특징
        + 스키마를 따로 정의하지 않음 → 각 콜렉션(Collection)에 대한 정의가 필요하지 않음
            - 콜렉션(Collection): 데이터베이스의 하위에 속하는 개념 (RDBMS의 테이블과 유사함)
        + JSON 형식으로 쿼리를 작성할 수 있음
        + ```_id``` 필드가 Primary Key 역할을 함
    - [📚 MongoDB Manual](https://www.mongodb.com/docs/manual/)

#### MongoDB의 연산자
MongoDB는 ```$```문자를 통해 연산자를 사용할 수 있음 (참고: [Operators](https://www.mongodb.com/docs/manual/reference/operator/query/#std-label-query-selectors))
* Comparison(비교)
    | 연산자 | 설명 |
    |-----|---------|
    | $eq | 지정된 값과 같은 값을 찾음 (eq = equal) |
    | $in | 배열 안의 값들과 일치하는 값을 찾음 (in) |
    | $ne | 지정된 값과 같지 않은 값을 찾음 (ne = not equal) |
    | $nin | 배열 안의 값들과 일치하지 않은 값을 찾음 (not in) |

* Logical(논리)
    | 연산자 | 설명 |
    |-----|---------|
    | $and | 논리적 AND (각각의 쿼리를 모두 만족하는 문서가 반환됨) |
    | $not | 쿼리 식의 효과를 반전시킴 (= 쿼리와 일치하지 않는 문서를 반환함) |
    | $nor | 논리적 NOR (각각의 쿼리를 모두 만족하지 않는 문서가 반환됨) |
    | $or | 논리적 OR (각각의 쿼리 중 하나 이상 만족하는 문서가 반환됨) |

* Element
    | 연산자 | 설명 |
    |-----|---------|
    | $exists | 지정된 필드가 있는 문서를 찾음 |
    | $type | 지정된 필드가 지정된 유형인 문서를 선택함 |

* Evalutaion
    | 연산자 | 설명 |
    |-----|---------|
    | $expr | 쿼리 언어 내에서 집계 식을 사용할 수 있음 |
    | $regex | 지정된 정규식과 일치하는 문서를 반환함 |
    | $text | 지정된 텍스트를 검색함 |

#### MongoDB와 SQL의 문법 비교
* SELECT (조회)
    | SQL | MongoDB |
    |-----|-----|
    | ```SELECT * FROM account``` → account 테이블 전체 조회 | ```db.account.find()``` → db의 account 컬렉션을 검색 |
    | ```SELECT * FROM account WHERE user_id="admin"``` → account 테이블에서 user_id가 "admin"인 데이터를 조회 | ```db.account.find({user_id: "admin"})``` → db의 account 컬렉션에서 user_id(key)가 "admin"(value)인 데이터를 검색 |
    | ```SELECT user_idx FROM account WHERE user_id="admin"``` → account 테이블에서 user_id가 "admin"인 데이터의 user_idx 열에 해당하는 데이터를 조회 | ```db.acount.find({user_id: "admin"}, {user_idx:1, _id: 0})``` → db_account 셀력센에서 user_id가 "admin"인 데이터의 user_idx 정보를 조회 (```user_idx:1```는 포함한다는 의미이고, ```_id:0```은 제외한다는 의미) |

    + MongoDB의 ```db.collection.find()```
        - Definition
            ```
            db.collection.find(query, projection, options)
            ```
        - Parameter
            | Parameter | Type | Description |
            |----|----|----|
            | query | document | (Optional) 연산자를 사용하여 선택 필터를 지정함 (모든 문서를 반환시키기 위해서는 매개변수 생략 또는, 빈 문서(```{}```)를 전달함) |
            | projection | document | (Optional) query와 일치하는 문서에서 반환할 필드를 지정함 (일치하는 문서의 모든 필드를 반환하려면 매개변수를 생략함) |
            | options | document | (Optional) query에 대한 추가 옵션을 지정함 (쿼리 동작과 결과가 반환되는 방식을 수정함) |
        - [📚 Reference](https://www.mongodb.com/docs/manual/reference/method/db.collection.find/#std-label-find-projection)

* INSERT (삽입, 추가)
    | SQL | MongoDB |
    |-----|-----|
    | ```INSERT INTO account(user_id, user_pw) VALUES ("guest", "guest")``` → account 테이블의 user_id(첫 번째 열)에 "guest"(첫 번째 값)를, user_pw(두 번째 열)에 "guest"(두 번째 값)를 삽입 | ```db.account.insert({user_id: "guest"}, {user_pw: "guest"})``` → db의 account 셀렉션의 user_id(key)에 "guest"(value)를, user_pw(key)에 "guest"(value)를 삽입 |

    + MongoDB의 ```db.collection.insert()```
        - Definition
            ```
            db.collection.insert(
                    <documnet or array of documents>,
                    {
                        writeConcern: <document>,
                        orderer: <boolean>
                    }
            )
            ```
        - Parameter
            | Parameter | Type | Description |
            |----|----|----|
            | document | document/array | 컬렉션에 삽입할 문서 또는 문서 배열 |
            | writeConcern | document | (Optional) 기본으로 설정 시 생략 가능 (참고: [Write concern](https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/#std-label-insert-wc)) |
            | ordered | boolean | (Optioanl) true면 순서대로 삽입하고, false면 정렬하지 않고 삽입을 수행함 |
        - [📚 Reference](https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/)

* DELETE (삭제)
    | SQL | MongoDB |
    |-----|-----|
    | ```DELETE FROM account``` → account 테이블의 데이터 전체를 삭제 | ```db.account.remove()``` → db의 account 셀렉션 내의 모든 키-값을 삭제 |
    | ```DELETE FROM account WHERE user_id="guest"``` → account 테이블에서 user_id가 "guest"인 데이터를 삭제함 | ```db.account.remove({user_id: "guest"})``` → db의 account 셀렉션에서 user_id(key)가 "guest"(value)인 데이터를 삭제함 |

    + MongoDB의 ```db.account.remove()```
        - Definition
            ```
            db.collection.remove(
                <query>,
                <justOne>
            )
            ```
        - Parameter
            | Parameter | Type | Description |
            |----|----|----|
            | query | document | 연산자를 사용하여 삭제 기준을 지정함 (모든 문서를 삭제하려면 빈 문서(```{}```)를 전달) |
            | justOne | boolean | (optional) 삭제를 하나의 문서로 제한하려면 true, 일치하는 모든 문서를 삭제하려면 false |
        - [📚 Reference](https://www.mongodb.com/docs/manual/reference/method/db.collection.remove/)

* UPDATE (수정, 갱신)
    | SQL | MongoDB |
    |-----|-----|
    | ```UPDATE account SET user_id="guest2" WHERE user_idx=2``` → account 테이블에서 user_idx가 2인 데이터의 user_id를 "guest2"로 변경함 | ```db.account.update({user_idx: 2}. {$set: {user_id: "guest2"}})``` → db의 account 셀력센여세 user_id가 2인 데이터의 user_id를 "guest2"로 설정($set)하고 변경사항을 반영함 |

    + MongoDB의 ```db.account.update()```
        - Definition
            ```
            db.collection.update(query, update, options)
            ```
        - Parameter
            | Parameter | Type | Description |
            |----|----|----|
            | query | document | 업데이트 선택 기준 |
            | update | document/pipeline | 적용할 수정 사항을 지정함 (연산식, 대체 문서, 파이프라인) |
            | options | document | (Optional) query에 대한 추가 옵션을 지정함 (쿼리 동작과 결과가 반환되는 방식을 수정함) |
        - [📚 Reference](https://www.mongodb.com/docs/manual/reference/method/db.collection.update/)

<br/>

### Redis
