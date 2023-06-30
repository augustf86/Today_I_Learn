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

<br/><br/>

## CSP 우회: base-uri 미지정


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: CSP Bypass](https://dreamhack.io/lecture/courses/322)
