# Web Hacking: Server-side Advanced - NoSQL
🔖 출처
* [Dreamhack Lecture] Server-side Advanced - NoSQL [🔗](https://dreamhack.io/lecture/courses/29)

<br/><br/>

## Overview: NoSQL
* Background: **NoSQL** = Non-Realtional DBMS(NRDBMS, 비관계형 데이터베이스)
    - NoSQL의 의미에 대해 NonSQL, Non relational SQL, Not Only SQL 등 다양한 의미가 존재함 → 현재는 Not Only SQL로 많이 사용되고 있음
        + **Not Only SQL**: *"데이터를 다루기 위해 꼭 SQL을 사용하지 않아도 데이터를 다룰 수 있다"는 의미*
    - **키-값**(Key-Value)를 사용해 데이터를 저장함
    - 데이터를 조회/추가/삭제하기 위해 SQL을 사용하지 않음 <br/> &nbsp;&nbsp; → 📌 RDBMS와는 달리 **복잡하지 않는 데이터를 다룬다는 것**이 가장 큰 특징임
        + NoSQL 데이터베이스는 단순 검색 및 추가 작업을 위한 매우 최적화된 저장 공간임
        + 다루는 데이터의 구조와 특징에 따라 여러 종류를 구분할 수 있음 [🔗](https://en.wikipedia.org/wiki/NoSQL#Types_and_examples) <br/> &nbsp;&nbsp; → Redis, Dynamo, CouchDB, MongoDB 등 다양한 DBMS별로 구조와 사용 문법을 배워야 함

<br/>
    
* NoSQL Injection
    - NoSQL 데이터베이스(NRDBMS) 또한 RDBMS와 같이 회원 계정, 비밀글 등과 같은 민감한 정보가 포함되어 있을 수 있음
        + ⚠️ 공격자는 데이터베이스 파일 탈취, NoSQL Injection 공격 등으로 민감한 정보를 확보하고 금전적인 이득을 취할 수 있음 <br/> &nbsp;&nbsp; → **임의 정보 소유자 이외의 이용자에게 민감한 정보가 노출되지 않도록 해야 함**
    - SQL Injection과 NoSQL Injection의 비교
        | | 설명 |
        |:---:|------|
        | 유사점 | 공격 목적 및 방법이 매우 유사함 <br/> &nbsp;&nbsp; - 이용자의 입력값이 쿼리에 포함되면서 발생하는 문제점 <br/> &nbsp;&nbsp; - 주로 **이용자의 입력값에 대한 검증이 불충분**할 때 발생함 |
        | 차이점 | - SQL Injection의 경우 SQL을 이해하고 있다면 모든 RDBMS에 대한 공격 수행이 가능함 <br/> - NoSQL Injection은 사용하는 DBMS에 따라 요청 방식과 구조가 다름 → 각각의 NRDBMS에 대해 이해하고 있어야 함 |

<br/><br/><br/>

## MongoDB
* Background: MongoDB
    - **key-value**의 쌍을 가지는 JSON objects 형태인 도큐먼트(Document)를 저장함
    - MongoDB의 특징
        + Schema가 존재하지 않아 각 콜렉션에 대한 특별한 정의를 하지 않아도 됨
            - MongoDBd에서는 SQL의 테이블과 유사한 개념으로 콜렉션(Collection)을 사용함
        + JSON 형식으로 쿼리를 작성할 수 있음
        + ```_id``` 필드가 Primary Key 역할을 함
    - MongoDB의 Query Operator [🔗](https://www.mongodb.com/docs/manual/reference/operator/query/) <br/> &nbsp;&nbsp; ↳ MongoDB는 **```$``` 문자**를 통해 연산자를 사용할 수 있음
        + Comparison
            | Operator | 설명 |
            |:---:|------|
            | ```$eq``` | 지정된 값과 같은 값을 찾음 (equal) |
            | ```$gt``` | 지정된 값보다 큰 값을 찾음 (greater than) |
            | ```$gte``` | 지정된 값보다 크거나 같은 값을 찾음 (greater than equal) |
            | ```$in``` | 배열 안의 값들과 일치하는 값을 찾음 (in) |
            | ```$lt``` | 지정된 값보다 작은 값을 찾음 (less than) |
            | ```$lte``` | 지정된 값보다 작거나 같은 값을 찾음 (less than equal) |
            | ```$ne``` | 지정된 값과 같지 않은 값을 찾음 (not equal) |
            | ```$nin``` | 배열 안의 값들과 일치하지 않은 값을 찾음 (not in) |
        + Logical
            | Operator | 설명 |
            |:---:|------|
            | ```$and``` | 논리적 AND → 각각의 쿼리를 모두 만족하는 문서가 반환됨 |
            | ```$or``` | 논리적 OR → 각각의 쿼리 중 하나 이상 만족하는 문서가 반환됨 |
            | ```$not``` | 쿼리 식의 효과를 반전시킴 → 쿼리 식과 일치하지 않은 문서를 반환함 |
            | ```$nor``` | 논리적 NOR → 각각의 쿼리를 모두 만족하지 않는 문서가 반환됨 |
        + Element
            | Operator | 설명 |
            |:---:|------|
            | ```$exists``` | 지정된 필드가 있는 문서를 찾음 |
            | ```$type``` | 지정된 필드가 지정된 유형인 문서를 선택함 |
        + Evaluation
            | Operator | 설명 |
            |:---:|------|
            | ```$expr``` | 쿼리 언어 내에서 집계식을 사용할 수 있음 |
            | ```$jsonSchema``` | 주어진 JSON 스키마에 대해서 문서를 검증함 |
            | ```$mod``` | 필드 값에 대해 mod 연산을 수행하고 지정된 결과를 가진 문서를 선택함 |
            | ```$regex``` | 지정된 정규식과 일치하는 문서를 선택함 |
            | ```$text``` | 지정된 텍스트를 검색함 |
            | ```$where``` | 지정된 Javascript 식을 만족하는 문서와 일치함 |
    - MongoDB의 문법

<br/><br/><br/>

## Redis

<br/><br/><br/>

## CouchDB
