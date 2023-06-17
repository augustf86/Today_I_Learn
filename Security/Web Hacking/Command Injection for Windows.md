# Command Injection for Windows

<br/>

## Windows: 공격 사전 지식
### 윈도우 메타 문자
* 윈도우에서 명령어를 실행하기 위해 **cmd.exe** 또는 **파워셸**(Powershell)을 사용할 수 있음
    - 같은 운영체제 상에서 구동되지만 각 인터프리터에서 제공하는 문자와 기능은 서로 다름
* 윈도우와 리눅스의 메타 문자 비교
    | Linux | Windows (cmd, powershell) | 설명 |
    |---|---|---|
    | ```-A```, ```--A``` | ```/c``` | 커맨드 라인 옵션 |
    | ```$PATH``` | ```%PATH``` | 환경 변수 |
    | ```$ABCD``` | ```$ABCD``` (powershell only) | 쉘 변수 |
    | ```;``` | ```&```(cmd only), ```;```(powershell only) | 명령어 구분자 |
    | ```echo $(id)``` | ```for /f "delims=" %a in ('whoami') do echo %a``` | 명령어 치환 |
    | ```> /dev/null``` | ```> NUL```(cmd only), ```\| Out-Null```(powershell only) | 출력 제거 |
    | ```command \|\| true``` | ```command & rem```(cmd only), <br/>```command -ErrorAction SilentlyContinue```(poershell Cmdlet only) | ```command``` 명령어 오류 무시 |

<br/>

### 윈도우 명령어
* 윈도우와 리눅스의 명령어 비교
    | Linux | Windows | 설명 |
    |---|---|------|
    | ls | dir | 디렉토리(폴더) 파일 목록 출력 |
    | cat | type | 파일 내용 출력 |
    | cd | cd | 디렉토리(폴더) 이동 |
    | rm | del | 파일 삭제 |
    | mv | move | 파일 이동 |
    | cp | copy | 파일 복사 |
    | ifconfig | ipconfig | 네트워크 설정 |
    | env, export | set | 환경변수 설정 |
    - 자세한 윈도우 명령어는 [Windows Command 추가 자료](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Supplement/Windows%20Command.md)를 참고

<br/><br/>

## 윈도우 리버스 셸


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Server Side] 📌 [ExploitTech: Command Injection for Windows](https://dreamhack.io/lecture/courses/301)
