# [Dreamhack Wargame] NoSQL-CouchDB
### [🚩NoSQL-CouchDB](https://dreamhack.io/wargame/challenges/419/)
  <img width="1073" alt="NoSQL-CouchDB_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/803839cc-7825-4819-9d2f-1e3b77c78b9a">

<br/><br/>

## 문제 파일(app.js) 분석
```javascript
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');

const nano = require('nano')(`http://${process.env.COUCHDB_USER}:${process.env.COUCHDB_PASSWORD}@couchdb:5984`);
const users = nano.db.use('users');
var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

/* GET home page. */
// 인덱스(/) 페이지
app.get('/', function(req, res, next) {
  res.render('index');
});

/* POST auth */
// 로그인(/auth) 페이지: 아이디(uid)와 패스워드(upw)를 입력받아 인증 과정을 수행함
app.post('/auth', function(req, res) { // POST 메소드를 통해 아이디와 패스워드를 /auth 페이지에 전달함
    users.get(req.body.uid, function(err, result) { // get 함수를 이용해 전달받은 uid에 해당하는 데이터를 조회하고, 반환된 에러와 결과를 아래의 if문, if-else문으로 비교함
        if (err) { // 에러가 발생한 경우
            console.log(err);
            res.send('error');
            return;
        }
        if (result.upw === req.body.upw) { // 조회한 uid의 upw와 이용자가 입력한 upw를 비교해 일치할 경우 플래그를 출력함
            res.send(`FLAG: ${process.env.FLAG}`);
        } else { // 일치하지 않을 경우 인증 실패
            res.send('fail');
        }
    });
});

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
```get``` 함수의 인자로 전달된 이용자의 입력값 ```uid```에 대해 어떠한 검사도 하지 않으므로 CouchDB의 특수 구성 요소를 전달할 수 있는 취약점이 존재함
* nano 패키지의 ```get``` 함수의 구현을 살펴보면 전달된 인자에 대해 특수 구성 요소의 포함 여부를 검사하지 않음 → 개발자가 이용자의 입력값을 검사하는 코드를 별도로 작성하지 않는 이상 애플리케이션 내에서 특수 구성 요소를 사용할 수 있음

<br/><br/>

### 익스플로잇
1. 플래그를 획득하기 위해 애플리케이션 로직을 파악한 후 공격에 사용할 특수 구성 요소를 정함
    - 애플리케이션 로직 (```auth``` 코드)
        1. 이용자가 입력한 ```uid```에 해당하는 데이터를 조회하고 에러와 결과를 ```err```와 ```result``` 변수에 저장함
        2. (if 문) 에러가 발생했다면 에러를 출력한 후 함수를 종료하고, 발생하지 않있디면 3번(if-else 문)을 진행함
        3. (if-else 문) 조회 결과의 ```upw```와 이용자가 입력한 ```upw```의 값을 비교하여 일치한다면 플래그를 출력하고, 그렇지 않다면 fail을 출력함
            - 이때 조회한 결과에서 ```upw```에 해당하는 키의 값이 존재하지 않으면 ```result.upw```는 ```undefined```이 됨
    - 특수 구성 요소 선정
        + 데이터베이스의 ```_all_docs```에 접근해 저장된 데이터베이스의 정보를 획득할 수 있음

2. 파악한 애플리케이션 로직을 토대로 ```uid```에 ```_all_docs```을, ```upw```(키)와 값을 입력하지 않은 상태로 로그인을 요청하면 플래그를 획득할 수 있음
    - ```upw```의 입력값 별 결과
        | ```upw```의 입력값 | 설명 |
        |-----|------|
        | ```{"uid": "_all_docs", "upw": "undefined"}``` | ```upw```에 "undefined" 문자열을 삽입하고 요청하면 애플리케이션은 전달받은 값을 단순 문자열로 판단함 → ```undefined == "undefined"```가 되어버려 인증 조건을 만족할 수 없음 (fail) |
        | ```{"uid": "_all_docs", "upw": ""}``` | ```upw```에 어떠한 문자도 삽입하지 않은 채 요청하면 애플리케이션은 ```upw```가 비어있다고 판단함 → ```undefined == ""```이 되어버려 인증 조건을 만족할 수 없음 (fail) |
        | ```{"uid": "_all_docs"}``` | ```upw``` 자체를 입력하지 않고 요청하면(*= POST Body에 ```upw``` 키 자체를 제거하고 요청하면*) 애플리케이션은 ```upw```가 정의되지 않은 상태로 판단하고 ```upw```를 ```undefined```으로 판단함 → ```undefined == undefined```가 되어 인증 조건을 만족함 (FLAG 출력) |
      <img width="911" alt="NoSQL-CouchDB_Result" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/56347781-cb4b-49f6-9623-dda55c8660f8">
