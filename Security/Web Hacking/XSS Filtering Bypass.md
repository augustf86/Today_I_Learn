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
