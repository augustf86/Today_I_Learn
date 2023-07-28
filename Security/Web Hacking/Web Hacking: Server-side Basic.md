# Web Hacking: Server-side Basic
🔖 출처: [Dreamhack Lecture] Server-side Basic [🔗](https://dreamhack.io/lecture/courses/15)

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
            if request.method == 'POST':
                f = request.files['file']
                f.save("./uploads/" + f.filename)
                return 'Upload Success'
            else:
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
    
<br/><br/>

### File Download Vulnerability


<br/><br/><br/>

## Business Logic Vulnerability

<br/><br/><br/>

## Language specific Vulnerability

<br/><br/><br/>

## Misconfiguration
