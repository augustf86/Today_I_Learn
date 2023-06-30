# CSP Bypass
⚠️ Content Security Policy(CSP)는 XSS를 방어하기 위한 훌륭한 방어책이지만, CSP 구문을 올바르지 않게 설정할 경우 이를 우회히여 허용되지 않은 출처에서 자원을 로드할 수 있는 방법들이 존재함

<br/><br/>

## CSP 우회: 신뢰하는 도메인에 업로드
* CSP에서 자원을 불러올 수 있도록 허용한 출처가 **파일 업로드 및 다운로드 기능을 제공**할 경우 출처에 스크립트와 같은 자원을 업로드한 뒤 다운로드 경로로 웹 페이지 자원을 포함시킬 수 있음
    + 외부 자원 업로드 예시
        ```html
        <!-- 스크립트 태그의 출처를 현재 페이지와 같은 출처(self)로 제한하는 Policy Directive -->
        <meta http-equiv="Content-Security-Policy" content="script-src 'self'">

        <!-- ... -->

        <!-- download_file.php의 id 값을 177742인 것을 다운로드 하고 있음 (download_file.php은 현재 페이지와 같은 출처에 위치함) → CSP 우회 성공 -->
        <h1>검색 결과: <script src="/download_file.php?id=177742"></script></h1>
        ```

<br/><br/>

## CSP 우회: JSONP API
* CSP에서 허용한 출처가 JSONP API를 지원할 경우 ```callback``` 파라미터에 원하는 스크립트를 삽입하여 공격이 가능함
    - JSONP API를 이용한 CSP 우회 예시
        + 웹 페이지에서 ```*.google.com```에서 온 출처만 허용한다면, 구글에서 JSONP API를 지원하는 서버(Google Account 서비스)를 찾아 아래와 같이 ```callback```에 원하는 스크립트를 삽입할 수 있음
            ```
            https://accounts.google.com/o/oauth2/revoke?callback=alert(1);
            ```

<br/>

* JSONP API를 이용한 CSP 우회를 방지하기 위한 방법
    - JSONP API를 제공하는 서비스는 콜백 이름에 식별자를 제외한 문자를 거부함으로써 이를 추가적으로 방어할 수 있음
        + JSONP API의 문자 검사 예시
            ```html
            <!-- 스크립트 태그의 출처를 https://*.google.com/으로 제한하는 Policy Directive -->
            <meta http-equiv="Content-Security-Policy" content="script-src 'https://*.google.com/'">

            <!-- ... -->

            <!-- JSONP API를 지원하는 Google Account 서버에서 callback에 스크립트를 삽입해 CSP 우회를 시도함 -->
            <script src="https://accounts.google.com/o/oauth2/revoke?callback=alert(1);"></script>>
            ```
            - JSONP API가 문자 검사를 할 경우의 결과: 유효하지 않은 인자를 주었다는 오류(error) 메시지를 반환함
                ```
                alert(1);({
                    "error": {
                        "code": 400,
                        "message": "Invalid JSONP callback name: 'alert(1)'; only alphabet, number, '_', '$', '.', '[', and ']' are allowed.",
                        "status": "INVALID_ARGUMENT"
                    }
                });
                ```
    - 가능한 경우 JSONP보다는 **CORS를 지원하는 API를 사용하는 것**이 좋음

<br/><br/>

## CSP 우회: nonce 예측 가능
* CSP의 ```nonce```를 이용하면 도메인이나 해시 등을 지정하지 않아도 **공격자가 예측할 수 없는 ```nonce``` 값이 태그 속성에 존재할 것을 요구함**으로써 XSS 공격을 방어할 수 있음
    - ```nonce``` 이용 시 주의해야 하는 사항
        | 📌 주의사항 | 설명 |
        |---|------|
        | ```nonce```가 공격자가 취득하거나 <br/>**예측할 수 없는** 값이어야 함 | - ```nonce```를 이용한 방어를 효과적으로 사용하기 위한 전제 조건 <br/> &nbsp;&nbsp;&nbsp;&nbsp; - ```nonce``` 값은 보통 요청마다 새로 생성되며, 공격자가 예측할 수 없는 난숫값임 <br/> &nbsp;&nbsp;&nbsp;&nbsp; - 보안 상 안전한 의사 난수 생성기(CSPRNG)를 사용하여 ```nonce``` 값을 생성해야 함 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - ⚠️ **```nonce```를 생성하는 알고리즘이 취약**하여 값을 예측할 수 있으면 ```nonce``` 값을 유추하여 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;악성 스크립트를 웹 사이트에 삽입할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - ⚠️ 현재 시각 등 **공격자가 알 수 있는 정보를 바탕으로 ```nonce``` 값을 생성**하는 것은 의미가 없음 |
        | ```nonce``` 값을 담고 있는 <br/> HTTP 헤더 또는 ```<meta>``` 태그가 <br/> 캐싱되지 않는지 주의해야 함 | ⚠️ PHP/CGI 계열의 스크립팅을 사용할 때에는 스크립트들이 ```/index.php/style.css```처럼 **뒤에<br/> 추가적인 경로를 붙여 접근될 수 있기 때문**에 특히 주의히야 함 <br/> &nbsp;&nbsp; - **캐시 서버가 확장자를 기반으로 캐시 여부를 판단**할 경우 ```.css```는 일반적으로 정적 파일이기 때문에 <br/> &nbsp;&nbsp;&nbsp;&nbsp; 동적 컨텐츠로 간주하지 않아 캐시에 저장할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → 캐시가 만료될 때까지 요청 시마다 같은 ```nonce```로 들어와 이를 통해 ```nonce```를 획득할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; → ⚠️ 콘텐츠가 캐시되어 서버 측 XSS는 발생하지 않지만, DOM XSS 등 **클라이언트 측에서 일어날 <br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;수 있는 공격에 취약**해지게 됨 |

<br/>

* 예시: Nginx와 PHP FastCGI SAPI(```php-fpm```) 사용
    ```
    location ~ \.php {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }
    ```
    - 취약한 ```snippets/fastcgi-php.conf```의 내용 일부 (Nginx 설정)
        ```
        # regex to split $uri to $fastcgi_script_name and $fastcgi_path
        fastcgi_split_path_info ^(.+\.php)(/.+)$;

        # Check that the PHP script exists before passing it
        try_files $fastcgi_script_name =404;

        # Bypass the fact that try_files resets $fastcgi_path_info
        # see: http://trac.nginx.org/nginx/ticket/321
        set $path_info $fastcgi_path_info;
        fastcgi_param PATH_INFO $path_info;

        fastcgi_index index.php;
        include fastcgi.conf;
        ```
        + 공격 방법
            | 순서 | 설명 |
            |---|------|
            | 01 | ```/dom_xss_vulnerable.php/style.css```의 주소로 접근하면 ```dom_xss_vulnerable.php``` 파일이 실행됨 |
            | 02 | 파일 실행 결과 ```nonce```가 ```<meta https-equiv="Content-Security-Policy" content="...nonce...">``` 태그로 출력됨 |
            | 03 | CDN(Content Delivery Network)은 보통 CSS, 스크립트 등 정적 파일을 캐싱함 <br/> &nbsp;&nbsp; → ⚠️ CDN에 의해 ```<meta>``` 태그로 출력된 ```nonce```가 캐싱됨 |
            | 04 | DOM XSS에 취약한 페이지의 ```nonce``` 값이 고정되어 공격자는 <br/> ```<script nonce="{고정된 nonce 값}">alert(1);</script>```과 같은 마크업을 사용하여 공격할 수 있음 |
    - 해결 방안
        + ```PATH_INFO``` 기능을 사용하지 않는 경우 ```location ~ \.php$```처럼 URL의 끝부분이 ```.php```일 때만 FastCGI로 넘어가게 수정되어야 함
        + URL에 ```a/b.php/c/d.php```와 같이 ```.php```가 중복 사용될 때를 대비하여 **fastcgi-php.conf 스니펫을 사용하지 않고** 아래와 같이 변경해야 함
            ```
            location ~ \.php$ {
                try_files $uri = 404;
                fastcgi_index index.php;
                include fastcgi.conf;
                fastcgi_pass unix;/run/php/php7.2-fpm.sock;
            }
            ```

<br/><br/>

## CSP 우회: base-uri 미지정


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: CSP Bypass](https://dreamhack.io/lecture/courses/322)
