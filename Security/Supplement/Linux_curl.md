# [Linux] curl

## curl
서버에서 또는 서버로 데이터를 전송하기 위한 도구
* 프로토콜을 이용해 인자로 받은 url로 데이터를 전송하여 서버에 데이터를 보내거나 가져옴
* 아래와 같이 다양한 프로토콜을 지원하며, Linux﹒Unix﹒Windows 등 다양한 OS에서 구동됨 → 유용하게 사용할 수 있음
    - curl이 지원하는 프로토콜들: DICT, FILE, FTP, FTPS, GOPHER, GOPHERS, **HTTP**, **HTTPS**, IMAP, IMAPS, LDAP, LDAPS, MQTT, POP3, POP3S, RTMP, RTMPS, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, TELNET, TFTP, WS, WSS
    - Linux와 MacOS에는 curl 패키지가 기본적으로 탑재되어 있음
        - Windows에는 같은 기능을 제공하는 편리한 프로그램들이 많기 때문에 잘 사용되지는 않음

### curl 명령어 설치
Linux에는 기본적으로 탑재되어 있으나 다음과 같은 명령어를 통해 curl 패키지를 설치할 수 있음
* 우분투
    ```
    sudo apt update
    sudo apt install curl
    ```
* CentOS
    ```
    sudo yum install curl
    ```

<br/><br/>

## curl 명령의 형식과 주요 옵션들
```
curl [options/URLs]
```
| 옵션 | 설명 |
|---|-----|
| -X (--request \<method\>) | HTTP 서버와 통신할 때 사용할 메소드를 지정함 (기본값은 GET) <br/> &nbsp;&nbsp; - \<method\>에 올 수 있는 값: GET, POST, PUT, PATCH, DELETE |
| -d (--data \<data\>) | 브라우저가 수행하는 것과 동일한 방식으로 POST 요청에 <\data\>를 담아 HTTP 서버로 전송함 <br/> &nbsp;&nbsp; - Content-type을 ```application/x-www-form-urlencoded```로 하여 서버로 전송함 |
| -o (--output \<file\>) | stdout 대신에 \<file\>이라는 이름을 가진 파일에 출력을 씀 | 
| -O (--remote-name) | 원격 파일과 동일한 이름의 로컬 파일에 출력을 씀 (경로를 제외한 원격 파일의 파일 이름만 사용함) <br/> &nbsp;&nbsp; - 파일은 현재 디렉토리에 저장됨 |
| -H (--header \<header/@file\>) | HTTP 요청 내에서 사용될 때 일반 요청 헤더에 추가되는 헤더를 작성함 |
| -T (--upload-file \<file\>) | \<file\>로 지정된 로컬 파일을 URL로 전송함 (HTTP PUT 메소드와 함께 사용함) <br/> &nbsp;&nbsp; - 주어진 파일 대신에 stdin을 사용하기 위해선 파일 이름 대신 ```-```(single dash)를 사용 |
| -v (--verbose) | 동작하면서 자세한 내용을 출력함 (디버깅 및 내부의 진행 상황을 확인하는데 유용함) <br/> &nbsp;&nbsp; - '>'로 시작하는 줄: curl에서 보낸 '헤더 데이터' <br/> &nbsp;&nbsp; - '<'로 시작하는 줄: curl에서 받은 '헤더 데이터' (일반적으로 숨겨져 있음) <br/> &nbsp;&nbsp; - '\*'로 시작하는 줄: curl에서 제공하는 추가 정보 |

* 사용할 프로토콜을 지정하지 않으면 curl 명령어에서 사용할 프로토콜을 추측함

<br/><br/>

## curl 명령을 이용한 HTTP 요청 예시
### HTTP GET 요청
```
curl http://www.example.com/get
curl -X GET http://www.example.com/get
```
* HTTP 메소드의 기본값이 GET이기 때문에 아무 옵션을 적지 않아도 ```-X GET```과 동일한 결과를 출력함
    - GET 메소드의 경우 Body가 없음
* HTTP GET 요청의 결과 ```http://www.example.com/get```의 소스 코드가 화면에 출력됨

<br/>

### HTTP POST 요청
```
curl -d "body 데이터" -H "Content-Type: 데이터에 따른 Content-type 헤더 값" -X POST http://www.example.com/post
```
* ```-d``` 옵션으로 body를 지정하고, ```-X POST```를 통해 POST 메소드를 사용할 것임을 알려줌
    - 파라미터는 '인코딩'된 상태여야 함
* Content-Type의 디폴트 값은 ```application/x-www-form-urlencoded```로, body의 형식에 따라 이를 변경해주어야 함
* body 형식 별 예시
    - url 형식의 데이터
        ```
        curl -d "key1=value&key2=value2&key3=value3" \
        -H "Content-Type: application/x-www-form-urlencoded" \
        -X POST http://www.example.com/post
        ```
        + Content-Type의 디폹트 값이 ```aplication/x-www-form-urlencoded```이므로 ```-H``` 옵션을 작성하든 작성하지 않든 동일한 결과를 반환함
    - JSON 형식의 데이터
        ```
        curl -d '{"key1":"value1", "key2":"value2", "key3":"value3"}' \
        -H "Content-Type: application/json" \
        -X POST http://www.example.com/post
        ```
        + Body 데이터의 형식이 JSON이므로 Content-Type의 값을 ```application/json```으로 주어야 함
    - 파일을 지정해서 보내는 경우
        ```
        curl -d "@data.txt" -X POST http://www.example.com/post
        curl -d "@data.json" -X POST http://www.example.com/post
        ```

<br/>

### HTTP PUT 요청
```
curl -X PUT -H "Content-Type: 데이터에 따른 Content-Type 헤더 값" -d "데이터" http://www.example.com/put
```
* ```-d``` 옵션으로 PUT 요청에 사용할 데이터를 지정하고, ```-X PUT```을 통해 PUT 메소드를 사용할 것임을 알려줌
* Content-Type의 디폴트 값은 ```application/x-www-form-urlencoded```로, ```-d``` 옵션의 형식에 따라 이를 변경해 주어야 함
* 데이터 형식별 예시
    - url 형식의 데이터
        ```
        curl -X PUT -H "Content-Type: application/x-www-form-urlencoded" \
        -d "key1=value1&key2=value2" http://www.example.com/post
        ```
        + ```-H``` 옵션은 작성하든 작성하지 않든 동일한 결과를 반환함
    - JSON 형식의 데이터
        ```
        curl -X PUT -H "Content-Type: application/json" \
        -d '{"key1": "value1", "key2": "value2"}' http://www.example.com/put
        ```
        + Content-Type에 ```application/json```을 작성하여 PUT 요청을 전송해야 함
    - 파일명으로 지정해서 보내는 경우
        ```
        curl -T filename.txt http://www.example.com/put
        ```
        + ```-T``` 옵션으로 로컬에서 원격으로 전송할 파일명을 작성함

<br/>

### HTTP DELETE 요청
```
curl -X DELETE http://www.example.com/users/user1
```

<br/><br/>


<br/><br/><br/><br/>
### 🔖 출처
* [curl - Man Page](https://curl.se/docs/manpage.html)
