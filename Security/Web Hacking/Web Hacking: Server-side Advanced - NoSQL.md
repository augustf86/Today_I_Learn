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

<br/><br/><br/>

## Redis

<br/><br/><br/>

## CouchDB
