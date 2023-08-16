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

<br/><br/><br/>

## SQL DML 구문에 대한 이해

<br/><br/><br/>

## Exploit Techique
