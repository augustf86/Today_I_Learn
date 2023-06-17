# NoSQL Injection

* NRDBMS 또한 RDBMS와 같이 회원 계정, 비밀글 등과 같은 민감한 정보가 포함되어 있을 수 있음
    - 공격자는 데이터베이스 파일 탈취, NoSQL Injection 공격 등으로 민감한 정보를 확보하고 악용하여 금전적인 이득을 취할 수 있음
    - 임의 정보 소유자 이외의 이용자에게 민감한 정보가 노출되지 않도록 해야 함

* SQL Injection과 NoSQL Injection의 비교
    - 유사점 - 공격 목적 및 방법이 매우 유사함
        + 이용자의 입력값이 쿼리에 포함되면서 발생하는 문제점
        + 주로 이용자의 입력값에 대한 검증이 불충분할 때 발생함
    - 차이점
        | SQL Injection | NoSQL Injection |
        |---|---|
        | SQL을 이해하고 있다면 모든 RDBMS에 대한 공격 수행 가능 | **사용하는 DBMS에 따라 요청 방식과 구조가 다름** → 각각의 DBMS에 대해 이해하고 있어야 함 |

<br/><br/>

### [MongoDB] NoSQL Injection
* 문자열, 정수, 날짜, 실수 외에도 오브젝트, 배열 타입을 저장하는 데이터의 자료형으로 사용할 수 있음
    - **오브젝트 타입의 입력값**을 처리할 때에는 [쿼리 연산자](https://www.mongodb.com/docs/manual/reference/operator/query/type/)를 사용할 수 있음 → 이를 통해 다양한 행위가 가능함

<br/>

#### MongoDB NoSQL Injection 예제 코드
* user 콜렉션에서 이용자가 입력한 ```uid```와 ```upw```에 해당하는 데이터를 찾고 출력함
    ```javascript
    const express = require('express');
    const app = express();

    const mongoose = require('mongoose');
    const db = mongoose.connection;
    mongoose.connect('mongodb://localhost:27017/', { useNewUrlParser: true, useUnifiedTopology: true });

    app.get('/query', function(req, res) {
        // 이용자가 입력한 uid, upw에 해당하는 데이터를 찾고 이를 출력함
        // 이용자의 입력값에 대한 타입 검증을 수행하지 않아 오브젝트 타입의 값을 입력할 수 있음
        db.collection('user').find({
            'uid': req.query.uid,
            'upw': req.query.upw
        }).toArray(function(err, result) {
                if (err) throw err;
                res.send(result);
        });
    });
    const server = app.listen(3000, function() {
        console.log('app.listen');
    });
    ```
    - 오브젝트 타입의 값을 입력할 수 있다면 쿼리 연산자를 사용할 수 있음
        + ```$ne``` 연산자(not equal, 입력한 데이터와 일치하지 않는 데이터를 반환함)를 이용하면 계정 정보를 모르더라도 해당 정보를 알아낼 수 있음
            | 정상적인 POST Data | 악의적인 POST Data |
            |----|----|
            | ```db.user.find({"uid": "guest", "upw": "guest"})``` <br/> → 입력받은 uid("guest"), upw("guest")에 해당하는 데이터인 guest 계정을 찾아 이를 반환함 | ```db.user.find({"uid": "admin", "upw": {"$ne": ""}})``` <br/> → upw의 조건을 무시하도록 만들어 admin 계정의 데이터를 조회할 수 있음 |

            - ```{"$ne": ""}```의 의미 - ```$ne```는 지정된 값(```""```)과 일치하지 않은 값을 찾으므로, upw의 조건을 무시하도록 만들 수 있음

<br/><br/>

### [MongoDB] Blind NoSQL Injection
SQL Blind Injection과 같이 참/거짓 반환 결과 또는 정상적인 쿼리문을 보냈을 때와 삽입 공격이 성공했을 때의 차이점(시간 차이 등)으로 데이터베이스의 정보를 알아내거나 취약성 여부를 판단하는 공격 기법
* MongoDB의 ```$regex```, ```$where``` 연산자를 사용해 Blind NoSQL Injection 공격을 수행할 수 있음
    - ```$regex``` 연산자: 정규식을 사용해 식과 일치하는 데이터를 조회함
        + 쿼리 예시 - user 컬렉션의 ```upw```에서 각 문자로 시작하는 데이터(```{$regex: "^알파벳"}```)를 조회
            ```
            db.user.find({upw: {$regex: "^a"}})
            db.user.find({upw: {$regex: "^b"}})
            ...
            db.user.find({upw: {$regex: "g"}})
            ```
            + 정규식 ```^```는 입력의 시작 부분을 나타냄
                - ```^알파벳```은 첫 번째 글자가 해당 알파벳인 데이터를 조회한다는 의미
        + 사용할 수 있는 정규식 표현
            | 정규식 | 설명 |
            |---|------|
            | ^ | 입력의 시작 부분을 나타냄 (```^a```, ```^ab```와 같이 각 위치에 맞는 알파벳을 찾으면 다음 위치에서 다시 하나씩 알파벳을 조회하여 비밀번호를 알아낼 수 있음) |
            | .{숫자} | ```.```은 모든 문자 하나와 일치하고, ```{...}```은 반복을 나타냄 (중괄호 안의 숫자를 변경시켜가며 비밀번호의 길이를 획득할 수 있음) |

    - ```$where``` 연산자: 인자로 전달한 Javascript 표현식을 만족하는 데이터를 조회함
        + ```substring``` 함수 - Blind SQL Injectiond에서 한 글자씩 비교했던 것과 같이 데이터를 알아낼 수 있음
            | 함수 | 설명 |
            |---|------|
            | ```substring(start), end)``` | start에서 end까지 문자열을 자름 (```end - 1```까지 문자열을 자름)

            -  쿼리 예시 - user 컬렉션의 ```upw```의 첫 번째 글자(```upw.substring(0, 1)```)를 비교해 데이터를 알아냄
                ```
                db.user.fimd({$where: "this.upw.substring(0, 1)=='a'"})
                db.user.find({$where: "this.upw.substring(0, 1)=='b'"})
                ...
                db.user.find({$where: "this.upw.substring(0, 1)=='g'"})
                ```
        
        + ```sleep``` 함수를 이용한 Time based Injection
            - Javascript 표현식과 함께 사용하면 지연 시간을 통해 참/거짓 결과를 확인할 수 있음
                + Javascript 표현식과 ```&&```(논리 AND)로 연결하여 참이 될 때만 ```sleep(sec)``` 함수를 실행시킬 수 있음

        + Error based Injection
            - 에러를 기반으로 데이터를 알아내는 기법
                + 올바르지 않은 문법을 입력해 고의로 에러를 발생시킴

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking] 📌 [ServerSide: NoSQL Injection](https://dreamhack.io/lecture/courses/189)
