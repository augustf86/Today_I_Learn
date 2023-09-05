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

<br/><br/><br/>

## Exploit Technigue

<br/><br/><br/>

## Windows 환경

<br/><br/><br/>

## Command Injection Bug Cases
