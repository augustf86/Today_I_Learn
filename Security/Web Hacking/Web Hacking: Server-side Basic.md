# Web Hacking: Server-side Basic
🔖 출처: [Dreamhack Lecture] Server-side Basic [🔗](https://dreamhack.io/lecture/courses/15) <br/>
🔖 출처: [Dreamhack Web Hacking] Server Side: File Vulnerability [🔗](https://dreamhack.io/lecture/courses/202)

<br/><br/>

## Server-side(서버 사이드) 취약점
| | 설명 |
|:---:|------|
| 정의 | 서버에서 사용자가 요청한 데이터를 해석하고 처리한 후 사용자에게 응답하는 과정에서 **웹 어플리케이션이나 데이터베이스와 <br/>같은 서버의 자원을 사용해 처리할 때** 사용자의 요청 데이터에 의해서 발생하는 취약점 |
| 공격 대상 | 웹 서비스를 제공하는 <U>서버</U> |
| 공격 목적 | 서버 내에 존재하는 사용자들의 정보를 탈취하거나, 서버의 권한을 장악함 |
| 발생 원인 | - HTTP 요청 시 악의적인 사용자가 메소드나 요청 헤더 등을 포함한 모든 데이터를 조작하여 전송할 수 있기 때문에 발생함 <br/> - **개발과 운영 상의 실수**로 사용자의 데이터에 대한 검증 과정의 부재 또는 올바르지 않은 검증으로 인해 발생함 |
| 기본 방지 대책 | 📌 ***서버는 사용자로부터 받는 모든 입력을 신뢰하지 않도록 해야 함*** |

<br/>

* Server-side 취약점의 분류
    | 분류 | 설명 |
    |:---:|------|
    | ***Injection*** <br/> (인젝션) | 서버의 처리 과정 중 사용자가 입력한 데이터가 처리 과정의 구조나 문법적으로 사용되어 <br/>발생하는 취약점 |
    | ***File Vulnerability*** | 서버의 파일 시스템에 사용자가 원하는 행위를 할 수 있을 때 발생하는 취약점 |
    | ***Business Logic Vulnerability*** <br/> (비즈니스 로직 취약점) | 인젝션, 파일 관련 취약점들과는 다르게 정상적인 흐름을 악용하는 것 |
    | ***Language specific Vulnerability*** <br/> (PHP, Python, NodeJS) | 웹 어플리케이션에서 사용하는 언어의 특성으로 인해 발생하는 취약점 |
    | ***Misconfiguration*** | 잘못된 설정으로 인해 발생하는 취약점 |

<br/><br/><br/>

## Injection

<br/><br/><br/>

## File Vulnerability
파일을 업로드하거나 다운로드할 때 발생하는 취약점
* File Vulnerability의 분류
    | 종류 | 설명 |
    |:---:|------|
    | ***File Upload Vulnerabiltiy*** <br/> (파일 업로드 취약점) | 서버의 파일 시스템에 사용자가 원하는 경로 또는 파일명 등으로 업로드가 가능하여 악영향을 미칠 수 있는 파일이 업로드되는 취약점 |
    | ***File Download Vulnerability*** <br/> (파일 다운로드 취약점) | 서버의 기능 구현 상 의도하지 않은 파일을 다운로드할 수 있는 취약점 |
    - ⚠️ **File Vulnerability가 발생하는 원리는 단순하지만, 공격에 사용되면 <U>웹 애플리케이션에 임의 명령어를 실행하거나 중요 파일을 탈취할 수 있는 만큼</U> 매우 치명적임**

<br/><br/>

### File Upload Vulnerability
| | 설명 |
|:---:|------|
| 정의 | 공격자의 파일을 웹 서비스의 파일 시스템에 업로드하는 과정에서 발생하는 보안 취약점 |
| 취약점 발생 상황 | 파일 시스템 상 임의 경로에 원하는 파일을 업로드하거나 악성 확장자를 갖는 파일을 업로드할 수 있을 때 발생함 <br/> &nbsp;&nbsp; → ***이용자가 원하는 파일의 이름을 임의로 정할 수 있을 때 발생함*** |
| 분류 | 크게 <U>Injection의 일종인 Path Traversal를 이용한 공격</U>과 <U>악성 파일 업로드</U>로 구분됨 |

<br/>

* Background
    - **File Upload**(파일 업로드): 웹 서비스를 통해 파일을 서버에 업로드하는 기능
        + 사용자의 사진, 문서와 같은 파일을 서버에 업로드하여 <u>다른 사용자와 공유하기 위한 목적</U>으로 사용됨
        + ⚠️ 파일 업로드 기능은 사용자의 파일이 서버의 파일 시스템에 저장되어 처리됨 → ***File Upload Vulnerability 취약점 발생***
    - **CGI**(Common Gateway Interface): 사용자의 요청을 받은 서버가 동적인 페이지를 구성하기 위해 엔진에 요청을 보내고 엔진이 처리한 결과를 서버에 반환하는 기능
        + php, jsp, asp 등과 같이 CGI를 통해 서비스를 하는 형태에서는 <U>**확장자를 통해** 웹 어플리케이션 엔진에 요청 여부를 판단함</U>
        + 예시: Apache 설정 파일
            ```Apache
            <FilesMatch ".+\.ph(p[3457]?|t|tml)$">
                SetHandler application/x-httpd-php
            </FilesMatch>
            ```
            - 정규표현식을 만족하는 .php, .php3/4/5/6, .phtml 등의 확장자를 가진 파일들은 php에서 처리되도록(x-httpd-php) 핸들링되어 있음
                | | 설명 |
                |:---:|-------|
                | 정규표현식 | 문자열의 끝이 .php, .php3, .php4, .php5, .php7, .pht, phtml인 경우 <br/>해당 정규표현식을 만족한다는 의미 |
                | ```x-httpd-php``` | PHP 엔진으로 요청한 파일을 실행하고 그 결과를 반환함 | 
            - Apache는 사용자가 서버로 보낸 요청을 해석함 → 사용자가 요청하는 리소스의 확장자가 php 엔진을 사용하도록 설정되어 있다면 **mod_php(CGI)를 통해 사용자의 요청을 php 엔진이 처리 및 실행하도록 요청함**
        + 💡 Python의 Flask, django와 NodeJS의 express 등과 같은 프레임워크
            - 서버, CGI, 웹 어플리케이션 등이 통합되어 있음
            - <U>미리 라우팅된 경로에만 접근이 가능하도록 설정되어 서비스됨</U>

<br/>

* File Upload Vulnerability 1: <U>File Path에서 발생하는 Path Traversal를 이용한 공격</U>
    | | 설명 |
    |:---:|------|
    | 정의 | 파일 업로드를 허용하는 대개의 서비스가 **특정 디렉터리에만 업로드를 허용하는 제약을 우회**하여, 임의 디렉터리에 <br/>파일을 업로드할 수 있는 취약점 |
    | 취약점 <br/> 발생 원인 | 이용자가 입력한 파일 이름을 별다른 검증 없이 그대로 사용할 때 발생함 <br/> (*Path Traversal은 Injection의 일종*) |
    | 공격 방법 | 상위 디렉터리로 이동하는 ```..``` 메타 문자 등을 사용하여 다른 디렉터리에 파일을 업로드할 수 있음 |
    - 예시: File Upload Code
        ```python
        from flask import Flask, request
        app = Flask(__name__)

        @app.route('/fileUpload', methods=['GET', 'POST'])
        def upload_file():
            if request.method == 'POST': # POST 메소드로 요청하는 경우
                f = request.files['file'] # 사용자가 업로드한 file을 가져옴
                f.save("./uploads/" + f.filename) # 사용자가 입력한 파일명으로 /uploads 디렉터리에 업로드함
                return 'Upload Success'
            else: # GET 메소드로 요청하는 경우
                return """
                <form action="/fileUpload" method="POST" enctype="multipart/form-data">
                    <input type="file" name="file"/>
                    <input type="submit"/>
                </form>
                """
        
        if __name__ == '__main__':
            app.run()
        ```
        | 일반 사용자 | 공격자 |
        |:---:|:---:|
        | <img width="400" alt="정상적인 File Upload Request" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b4ed2395-b24a-4211-87f5-4accd54935a3"> | <img width="400" alt="악의적인 File Upload Request" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a653de65-4bc0-49bb-a66d-ebf0cac1b5d3"> |

<br/>

* File Upload Vulnerability 2: 악성 파일 업로드
    - **WebShell** (웹셸)
        | | 설명 |
        |:---:|------|
        | 전제 조건 | 웹 서버가 서비스하는 경로에 파일을 업로드할 수 있어야 함 <br/> &nbsp;&nbsp; → 웹 서버가 파일을 실행하는 시기 = 사용자의 요청이 들어온 후 파일 시스템에서 해당 파일을 찾아 실행할 때 |
        | 공격 과정 | 웹 서비스가 동작하는 경로에 사용자가 원하는 파일 내용과 파일명을 업로드할 수 있음 <br/> → CGI를 통해 서비스하는 서버가 엔진에 요청하는 확장자를 업로드하여 서버의 웹 어플리케이션에 원하는 코드를 실행함 |
        | 파급 효과 | 원하는 시스템 커맨드를 실행하는 **원격 코드 실행 취약점**을 유발할 수 있음 <br/> &nbsp;&nbsp; - 웹 어플리케이션이 실행하는 코드를 조작할 수 있어 웹 어플리케이션 언어에 내장된 OS 명령어 등을 사용할 수 있어야 가능함 |
    - 악의적인 웹 리소스
        + 웹 브라우저는 **파일의 확장자**나 Response(응답)의 **Content-Type**에 따라 요청을 다양하게 처리함
            | 예시 | 결과 |
            |------|:---:|
            | 요청한 파일의 확장자가 **.html**인 경우, <br/> Response의 Content-Type이 **text/html**인 경우 | HTML 엔진로 처리됨 |
            | 요청한 파일의 확장자가 **.jpg**, **.png** 등의 이미지 확장자인 경우, <br/> Response의 Content-Type이 **image/png** 등인 경우 | 이미지로 렌더링됨 |
            - 공격자가 <U>악의적인 스크립트를 삽입한 html 파일</U>을 업로드한 경우 **XSS 공격**(Stored XSS)으로 이어질 수 있음
    
<br/>

* File Upload Vulnerability 방지 대책
    - 업로드 디렉터리를 웹 서버에서 직접 접근할 수 있도록 하거나 업로드 디렉터리에서는 CGI가 실행되지 않도록 해야 함
    - 업로드한 파일 이름을 그대로 사용하지 않고, ```basepath```와 같은 함수를 통해 파일 이름을 검증한 후 사용함
    - 허용할 확장자를 명시해 그 외의 확장자는 업로드할 수 없도록 해야 함
        + 동적 리소스의 확장자를 제한하면 파일 업로드 취약점을 통한 RCE 공격으로부터 서버를 보호할 수 있음
            | 리소스의 종류 | 설명 |
            |:---:|------|
            | **정적 리소스** (Static Resource) | 웹 리소스 중 이미지나 비디오와 같이 서버에서 실행되지 않는 리소스를 말함 |
            | **동적 리소스** (Dynamic Resource) | 웹 리소스 중 PhP, JSP처럼 서버에서 실행되닌 리소스를 가리킴 |
    - AWS, Azure, GCP 등의 정적 스토리지를 이용함
        + 서버의 파일 시스템을 이용하지 않음 → 파일 업로드 취약점이 웹 서버 공격으로 이어지는 것을 막을 수 있음

<br/><br/>

### File Download Vulnerability
| | 설명 |
|:---:|------|
| 정의 | 웹 서비스의 파일 시스템에 존재하는 파일을 다운로드하는 과정에서 발생하는 보안 취약점 |
| 취약점 발생 상황 | 이용자가 다운로드할 파일의 이름을 임의로 정할 수 있을 때 발생함 <br/> &nbsp;&nbsp; - **<U>사용자가 입력한 파일 이름을 검증하지 않은 채 그대로 다운로드를 제공할 때</U> 가장 흔하게 발생함** |
| 공격 효과 | 웹 어플리케이션의 소스 코드, 관리자의 패스워드, 서비스 키, 설정 파일, 데이터베이스 백업본 등 민감한 정보을 탈취할 수 있음 <br/> &nbsp;&nbsp; ***→ 다운로드받은 파일을 이용하여 2차 공격을 수행할 수도 있음*** |

<br/>

* Background: **File Downlaod**(파일 다운로드)
    - 웹 서비스의 파일 시스템에 존재하는 파일을 다운로드하는 기능
        + 사용자가 업로드한 파일을 다른 사용자와 공유하기 위해서 필요함
        + 다양한 방법으로 파일 다운로드 기능을 구현할 수 있음

<br/>

* File Download Vulnerability: File Path에서 발생하는 <U>Path Traversal를 이용한 공격</U>
    | | 설명 |
    |:---:|------|
    | 정의 | 웹 서비스가 **특정 디렉터리에 있는 파일만 접근하도록 허용하는 제약을 우회**하여, 파일 이름을 직접 입력 받아 임의 디렉터리에 <br/>있는 파일을 다운로드받을 수 있는 취약점 |
    | 취약점 <br/> 발생 원인 | 사용자가 입력한 파일 이름을 별다른 검증 없이 사용할 때 발생함 <br/> (*Path Traversal은 Injection의 일종*) |
    | 공격 방법 | 상위 디렉터리로 이동하는 ```..``` 메타 문자 등을 사용하여 다른 디렉터리에 있는 파일을 다운로드받을 수 있음 |
    - 예시: File Download Code
        ```python
        #...
        @app.route("/download")
        def download():
            # 사용자가 입력한 filename의 인자 값을 가져와 검증 없이 사용하고 있음 → File Download Vulnerability 발생
            filename = request.args.get("filename", "")
            return open("uploads/" + filename, "rb").read()
        ```
        | | 예시 설명 |
        |:---:|------|
        | 일반 사용자 | ```http://example.com/download?filename=docs.pdf``` 압룍 <br/> &nbsp;&nbsp; → **/uploads** 디렉터리에 존재하는 docs.pdf 파일을 다운로드 받음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (***/uploads/docs.pdf 파일 다운로드***)
        | 공격자 | ```http://example.com/download?filename=../../../../etc/passwd``` 입력 <br/> &nbsp;&nbsp; → 상위 디렉터리로 이동하는 과정을 반복하여 다른 경로에 존재하는 시스템 계정 파일을 다운로드 받음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (***/etc/passwd 파일 다운로드***) |

<br/>

* File Download Vulnerability 방지 대책
    - 📌 ***기본적으로 인자에 다운로드을 받으려는 파일의 경로나 이름을 넘기지 않는 것이 좋음***
        + 반드시 이름을 넘기는 방식으로 구현해야 하는 경우 <U>상대경로로 접근하는 데 사용될 수 있는 ```..```, ```/```, ```\\```를 적절하게 필터링해야 함</U>
        + ⚠️ 필터링을 잘못 구현한 경우 **필터링을 우회할 수 있음**
            | 예시 | 설명 |
            |:---:|------|
            | 단순히 ```../```만 <br/> 필터링하는 경우| 상위 경로로 올라가는 키워드를 없애면 상위 경로로 올라가지 못하기 때문에 안전할 것이라고 착각함 <br/> &nbsp;&nbsp; - ```..././file```와 같은 형식으로 요청하면 필터링에 의해 ```../```가 삭제되어 새로운 ```../```가 생성됨 <br/> &nbsp;&nbsp; - 윈도우 ```..\\```으로도 상위 경로에 접근할 수 있음 |
    - 데이터베이스에 다운로도 될 파일의 경로와 그에 해당하는 랜덤 키를 생성하여 1:1로 매칭해서 저장해두고 해당 랜덤 값(키)이 인자로 넘어왔을 때 데이터베이스에 존재하는 파일인지를 먼저 식별하고 다운로드를 수행함
        + 예시 코드
            ```python
            @app.route("/download")
            def donwload():
                file_id = requests.args.get("file_id", "") # file_id = 쉽게 유추하지 못하는 랜덤한 값
                file_path = find_path_from_database(file_id) # 데이터베이스에서 file_id와 맵핑된 파일 경로를 반환하는 find_path_from_database() 함수를 정의했다고 가정
                
                if file_path is None: # 맵핑되는 파일 경로가 존재하지 않는 경우
                    return "incorrect file id"
                return open(file_path, "rb").read() # 맵핑되는 파일 경로가 존재하는 경우
            ```
    - 요청된 파일 이름을 ```basepath```와 같은 함수를 통해 검증함

<br/><br/><br/>

## Business Logic Vulnerability

<br/><br/><br/>

## Language specific Vulnerability

<br/><br/><br/>

## Misconfiguration
| | 설명 |
|:---:|------|
| 취약점 <br/> 발생 위치 | 웹 서버(Apache, Nginx), 데이터베이스 서버(mysql, postgresql, mongodb), 캐시 서버(redis), 웹 프레임워크(django, spring), <br/>컨테이너(Docker, k8s) 등 **모든 어플리케이션 계층에서 발생할 수 있음 |
| 발생 원인 <br/> 및 방지 대책 | 소스 코드 상에 존재하는 복잡한 로직에서 발생하는 취약점이 아닌 ***간단한 설정 오류로 인해 발생함*** <br/> &nbsp;&nbsp; → <u>검수할 때는 기본 메뉴얼에 존재하는 룰을 잘 따랐는지</u>만 확인하면 됨 ⇒ 자동화 도구로 쉽게 발견할 수 있음 |
| 효과 | 공격자는 허가되지 않은 동작을 수행할 수 있음 |
| 분류 | 발생하는 원인에 따라 **부주의로 인해 발생하는 문제점**, **편의성을 위한 설정에 의해 발생하는 문제점**, **메뉴얼과 실제 구현체의 차이로 <br/>인해 발생하는 문제점**, **해당 코드나 설정에 대한 이해 없이 사용해 발생하는 문제점**으로 나눌 수 있음 |

<br/><br/>

### 부주의로 인해 발생하는 문제점
메뉴얼에 존재하지 않는 설정이거나 개발자의 부주의로 인해 설정 유무를 알지 못한 상태에서 서비스를 할 경우 자주 발생함

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***권한 설정 문제***
    - 웹 서비스를 구성하는 요소들 운용 시 시스템 또는 어플리케이션 상에서 권한 설정이 잘못된 경우
    - 대표적인 권한 설정 문제
        | 권한 설정 문제 | 설명 |
        |:---:|------|
        | ***기본 계정*** | 프레임워크 등에서 기본적으로 제공하는 계정을 삭제하지 않아 문제가 발생할 수 있음 <br/> &nbsp;&nbsp; - 프레임워크 등을 통해 제공되는 계정 정보는 **ID/PW 등의 정보가 메뉴얼을 통해 공개되어 있음** <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 이를 통해 특별한 취약점을 사용하지 않고 권한을 획득할 수 있음|
        | ***잘못된 권한 설정*** | 웹 서버를 운용하는 시스템 또는 어플리케이션 상에서 <u>필요 이상의 권한을 부여</u>하는 경우 문제가 발생할 수 있음 <br/><br/> 서비스를 운용하는 OS의 사용자 권한이 일반 유저 또는 시스템 운용을 위해 생성된 계정을 사용하지 않고 <u>**superuser**(root) <br/>권한으로 사용될 경우 서비스의 로직이 superuser의 권한으로 수행됨</u> <br/> &nbsp;&nbsp; ***e.g.*** 웹 서버가 superuser 권한으로 동작하며, 파일 관련 취약점이 존재할 경우 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 일반 사용자 권한이 접근하지 못하는 시스템 파일 등에도 접근할 수 있게 됨 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 이를 통해 시스템 장악 등의 문제로 연계될 수 있음 <br/><br/> DB, Docker 등의 어플리케이션 운용 시에도 해당 어플리케이션의 <u>사용자 권한을 필요 이상으로 부여할 경우</u> 문제가 발생할 <br/>수 있음 <br/> &nbsp;&nbsp; ***e.g.*** DBMS 내에서 root 계정을 사용하는 경우 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - 기본적으로 모든 데이터베이스에 접근 가능하며, 일반 유저가 사용하지 못하는 함수들도 사용할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → sql injection 취약점 발생 시 함수들을 통해 다른 취약점으로 연계할 수 있는 위협이 존재할 수 있음 |

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***기본 서비스***
    - 서비스 요소들 운용 시 사용하는 시스템 또는 어플리케이션에서 **개발자가 설정하지 않아도 기본적으로 제공되는 기능**에 의해 문제가 발생할 수 있음
        + 대부분 개발 당시에는 필요한 기능이거나, 내부적으로 사용해야 하는 기능들임
        + ***운영 시에 의도하지 않은 경로 또는 기능이 노출되는 위협이 발생할 수 있음***
    - 기본 서비스 예시
        | 서비스 종류 | 예시 |
        |:---:|------|
        | 관리 서비스 | Tomcat - Web Application Manager <br/> Docker Registry API |
        | 모니터링 서비스 | Srping Boot Actuator - metrics <br/> Apache mod_status |

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***임시/백업 파일***
    - 웹 서버 디렉터리 내에 임시/백업 파일이 존재한다면 디렉터리 스캐너를 통해 파일이 유출될 수 있음
        + ❓ **디렉터리 스캐너**: 디렉터리 내 자주 쓰이는 파일명이나 확장자를 자동으로 스캐닝해주는 도구
    - 공격자에게 악용될 수 있는 임시/백업 파일
        | 확장자 | 설명 |
        |:---:|------|
        | **swp** | 임시 파일, 파일 내용(소스 코드)가 포함되어 있음 |
        | **bak** | 백업 파일, 대부분의 에디터에서 사용함 |
        | **config** | 설정(configuration) 파일, 비밀 키들이 존재하는 경우가 많음 |
        | **sql** | sql schema 파일, 데이터베이스 구조를 알아낼 수 있음 |
        | **sh** | shell script 파일 |
        | **~** | bluefish 에디터 백업 파일 |
        + 예시: vim 에디터와 swp 임시 파일을 이용한 파일 유출
            - ***hello.php*** 파일: password가 일치하면 임의 명령을 실행해주는 php 파일
                ```php
                <?php
                    if ($_GET['password'] === 'this_is_secret_password') { # 공격자는 소스코드에 정의되어 있는 'this_is_the_secret_password'를 알 수 없음
                        system($_GET['cmd']);
                    } else {
                        echo 'Nope';
                    }
                ?>
                ```
            - vim 에디터로 hello.php 파일을 수정 중에 생긴 **.swp**를 통해 소스코드를 획득하는 방법
                | 순서 | 설명 |
                |:---:|------|
                | 01 | 개발자가 hello.php 파일을 vim 에디터로 열게 되면 **.hello.php.swp**라는 임시 파일이 생성됨 <br/> &nbsp;&nbsp; - 📌 **웹 서버의 동작** <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ***.php 확장자*** → php script로 인식해 실행 결과를 응답함 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ***.swp 확장자*** → 바이너리 파일로 인식해 파일 내용(소스 코드)가 포함된 임시 파일을 응답함 |
                | 02 | 공격자는 .swp 파일을 다운로드 받아 ```vim hello.p``` 명령어를 입력한 후 R(Recover) <br/>키를 눌러 복구를 진행함 |
                | 03 | 복구가 되었다는 메시지와 함께 원본 소스코드를 반환함 → 이를 통해 공격자는 소스코드를 획득할 수 있음 |
    - 조치 방안
        + 서비스하기 이전에 서버 및 어플리케이션의 설정 및 파일들을 점검하여 서비스와는 무관한 파일을 제거해야 함
        + <u>위험한 확장자에 대한 요청이 들어올 경우 거부하는 방법</u>으로 위협을 없앰
            - 예시: Nginx에서 적용 가능한 설정
                ```nginx
                location ~ (?:\.(?:bak|config|sql|fla|psd|ini|log|sh|inc|swp|dist)|~)$ {
	            deny all;
	            access_log off;
	            log_not_found off;
                }
                ```

<br/>

* 부주의로 인해 발생하는 Misconfiguration: ***개발 관련 파일***
