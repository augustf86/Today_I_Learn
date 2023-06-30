# CSP Bypass
⚠️ Content Security Policy(CSP)는 XSS를 방어하기 위한 훌륭한 방어책이지만, CSP 구문을 올바르지 않게 설정할 경우 이를 우회히여 허용되지 않은 출처에서 자원을 로드할 수 있는 방법들이 존재함

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

<br/><br/>

## CSP 우회: nonce 예측 가능

<br/><br/>

## CSP 우회: base-uri 미지정


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: CSP Bypass](https://dreamhack.io/lecture/courses/322)
