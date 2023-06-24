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
