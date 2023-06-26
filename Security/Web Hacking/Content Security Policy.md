# Content Security Policy (CSP)

### Content Security Policy(CSP)의 탄생 배경
* 웹 브라우저는 웹 서버로부터 받는 페이지의 컨텐츠가 모두 의도된 컨텐츠인지 알 수 없음 <br/> &nbsp;&nbsp; → **페이지의 컨텐츠에서 사용하는 자원들이 모두 웹 서버에서 의도한 자원이 맞는지 확인**하기 위해 CSP가 탄생함
    - [XSS 공격] 페이지 내에 자바스크립트 구문을 삽입하여 마치 원래의 컨텐츠인 것처럼 브라우저를 속이고 동일 출처 정책(SOP)를 우회함 → CSP를 통해 **웹 페이지의 컨텐츠와 자원들이 모두 서버에서 허용한 출처로부터 받아온 것인지 확인할 수 있음**

<br/>

### ⚠️ 주의사항
CSP는 XSS 공격으로부터 피해를 최소화할 수 있는 좋은 방안이지만, **XSS 공격의 피해를 완전히 무력화하기 위한 수단은 아님** <br/> &nbsp;&nbsp; → *XSS에 대한 자체적인 방어가 CSP와 병행되어야 함*
    

<br/><br/>

## Content Security Policy(CSP, 컨텐츠 보안 정책)
XSS나 데이터를 삽입하는 류의 공격이 발생하였을 때 피해를 줄이고 웹 관리자가 공격 시도를 보고 받을 수 있도록 새롭게 추가된 보안 계층
* XSS 공격과 CSP의 작동 원리
    | | 설명 |
    |---|------|
    | XSS 공격 | 브라우저가 서버로부터 전달받는 컨텐츠를 신뢰한다는 점을 이용 <br/> &nbsp;&nbsp; - 그 컨텐츠가 원래는 서버에서 추가된 것이 아니라 공격자에 의해 추가된 악성 컨텐츠라도 **서버로부터 전달받으면** 무조건 신뢰함 |
    | CSP | 웹 페이지에 사용될 수 있는 자원의 위치, 출처 등에 제약을 걸 수 있음 <br/> &nbsp;&nbsp; - 공격자가 웹 사이트에 본래 있지 않은 스크립트를 삽입하거나 공격자에게 권한이 있는 서버 등에 요청을 보내지 못하도록 막을 수 있음 |

<br/>

### CSP 헤더
1개 이상의 정책 지시문이 세미콜론(```;```)으로 분리된 형태로 이루어져 있음
* 정책 지시문
    - ```default-src```, ```script-src``` 등의 지시문과 1개 이상의 출처(```'self'```, ```https:```, ```*sample.com``` 등)가 **공백**으로 분리된 형태로 지정하여야 함
    - CSP 구문 예시
        ```
        default-src 'self' https://example.test.com
        ```
        + 페이지 내부의 자원(```default-src```)이 같은 오리진(```'self'```) 혹은 ```https://example.test.com```에서만 로드되어야 함을 의미
* CSP 헤더의 정의 방법
    - 방법 1: **```Content-Security-Policy``` HTTP 헤더**를 이용
        + ```Content-Security-Policy``` HTTP 헤더에 일반적으로 추가하여 적용할 수 있는 CSP 구문
            ```
            Content-Security-Policy: <policy-directive>; <policy-directive>
            ```
            - ```policy-directive``` 부분에 CSP를 정의하는 정책 디렉티브(```<지시문> <출처>``` 형태)를 작성함
        + 예시 (CSP 구문 예시를 HTTP 헤더를 이용해 추가함)
            ```
            Content-Security-Policy: default-src 'self' https://example.test.com
            ```
    - 방법 2: **```meta``` 태그의 엘리먼트**로 정의
        + ```<meta>``` 태그의 ```http-equiv``` 속성의 값을 ```"Content-Security-Policy"```로 하고, ```content``` 속성의 값에 정책 지시문을 작성하여 CSP 구문을 추가할 수 있음
        + 예시 (CSP 구문 예시를 ```<meta>``` 태그를 이용해 추가)
            ```html
            <meta http-equiv="Content-Security-Policy" content="default-src 'self' https://example.test.com">
            ```
* CSP를 올바르게 정의하였는지 검사하기 위해 구글에서 제공하는 [CSP Evaluator](https://csp-evaluator.withgoogle.com)
    - 직접 작성한 CSP가 올바른지 검사할 수 있음
      <img width="2560" alt="CSP Evaluator" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b514aeba-b7cb-4bc8-a4a9-1629605e67c4">

<br/><br/>

## Content Security Policy 기본 정책

### 인라인 코드(Inline Code)
📌 CSP는 **인라인 코드를 유해하다**고 간주함 → CSP를 사용할 때에는 기본적으로 인라인 코드를 사용할 수 없음
* CSP는 인라인 코드 형태를 지양하고, **```src``` 속성에 코드 경로를 정의하는 방식**을 권장함
    | CSP | 예시 |
    |---|------|
    | 지양 (인라인 코드 형태) | ```<script>alert(1);</script>``` <br/> &nbsp;&nbsp; → ```<script>``` 태그 내엔 코드를 직접 삽입하는 형태(인라인 코드) |
    | 권장 (```src``` 속성으로 코드 로드) | ```<script src="alert.js"></script>``` <br/> &nbsp;&nbsp; → ```<script>``` 태그의 ```src``` 속성으로 코드 경로를 정의함 |
    - CSP가 인라인 코드로 간주하는 것들 (허용하지 않음)
        | 인라인 코드으로 간주하고 허용하지 않는 것들 | 예시 (*CSP 적용 시 사용 불가*) |
        |---|-----|
        | ```<script>``` 태그 내에 코드를 삽입하는 것 | ```<script>alert(1);<script>``` |
        | ```on*``` 이벤트 핸들러 속성 | ```<img src="valid.jpg" onload="alert(1)">``` |
        | ```javascript:``` URL 스키마 | ```<a href="javascript:alert(1)">Click me</a>``` |
* CSP는 ```CSS``` 또한 인라인 코드를 허용하지 않음
    - ```style``` 속성과 ```style``` 태그 모두 외부 스타일시트로 통합하는 것을 권장함 <br/> &nbsp;&nbsp; = 인라인 스타일(Inline style)과 내부 스타일 시트(Internal style sheet) 사용은 지양하고, 외부 스타일 시트(External style sheet)의 사용을 권장함
        | CSS 적용 방법 | 설명 |
        |---|------|
        | 인라인 스타일<br/>(Inline style) | HTML 요소 내부에 ```style``` 속성을 사용하여 CSS 스타일을 적용하는 방법 (인라인 코드) <br/> &nbsp;&nbsp; - 해당 요소에만 스타일을 적용할 수 있음 <br/> &nbsp;&nbsp; - 예시: ```<p style="color:blue text-decoration:underline"></p>``` |
        | 내부 스타일 시트<br/>(Internal style sheet) | HTML 문서 내의 ```<head>``` 태그에 ```<style>``` 태그를 사용하여 CSS 스타일을 적용하는 방법 <br/> &nbsp;&nbsp; - 해당 HTML 문서에만 스탕리을 적용할 수 있음 <br/> &nbsp;&nbsp; - 예시: ```<style>p {color: blue; text-decoration: underline;}</style>``` |
        | 외부 스타일 시트<br/>(External style sheet) | ```.css``` 확장자로 저장된 스타일 시트 파일을 ```<head>``` 태그에 ```<link>``` 태그를 사용하여 외부 스타일 시트를 포함시켜 CSS 스타일을 적용하는 방법 <br/> &nbsp;&nbsp; - 웹 사이트 전체의 스탕리을 하나의 파일에서 변경할 수 있도록 해줌 <br/> &nbsp;&nbsp; - 예시: ```<head><link rel="stylesheet" href="/css/externalSheet.css"></head>``` |

<br/>

### Eval
📌 CSP는 기본적으로 **문자열 텍스트를 실행 가능한 자바스크립트 코드 형태로 변환하는 메커니즘을 유해하다**고 간주함
* **문자열로부터 코드를 실행**하는 경우 의도와는 다르게 XSS 공격을 통해 삽입된 공격 코드로 변조되어 실행될 가능성이 존재함 → CSP는 이를 권장하지 않음
    - ```eval```, ```new Function()```, ```setTimeout([string], ...)```, ```setInterval([string], ...)```과 같이 문자열 형태로 입력을 받는 함수의 실행은 모두 차단됨
        | 함수 | 설명 | Docs |
        |---|------|---|
        | ```eval(script)``` | 문자열로 표현된 자바스크립트 코드를 실행하고 그 결과 값을 반환함 | 📚 [eval() docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval) |
        | ```Function()``` constructor | Function 객체를 만듦 <br/> &nbsp;&nbsp; - 인자(들)과 함수 정의(제일 마지막에 위치)를 모두 문자열로 표현하여 넘겨줌 <br/> &nbsp;&nbsp;&nbsp;&nbsp;(functionbody에 문자열로 표현된 자바스크립트 코드가 들어감) | 📚 [Function() constructor docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/Function) |
        | ```setTimeout(code, delay)``` | 타이머가 완료되면 함수 또는 지정된 코드를 실행하는 타이머를 설정함 <br/> &nbsp;&nbsp; - ```code```: 타이머가 완료되면 실행할 문자열로 표현된 자바스크립트 코드 <br/> &nbsp;&nbsp; - ```delay```: (Optional) 코드를 실행하기 전에 타이머가 대기해야 하는 시간(millisecs) | 📚 [setTimeout() docs](https://developer.mozilla.org/en-US/docs/Web/API/setTimeout) |
        | ```setInterval(code, delay)``` | 주어진 시간 간격마다 함수 또는 코드를 반복적으로 실행함 <br/> &nbsp;&nbsp; - ```code```: 정해진 시간 간격마다 실행할 문자열로 표현된 자바스크립트 코드 <br/> &nbsp;&nbsp; - ```delay```: 지정된 시간 간격(millisecs)(default: 0) | 📚 [setInterval() docs](https://developer.mozilla.org/en-US/docs/Web/API/setInterval) |

    - **예외** > 해당 함수에 문자열 입력이 아닌 **인라인 함수의 형태로 파라미터가 전달될 때에는 차단되지 않음**
        | 차단 | 허용 |
        |---|---|
        | ```setTimeout("alert(1)", ...)``` | ```setTimeout(function(){ alert(1) }, ...)``` |

<br/><br/>

## Policy Directive


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Background: Content Security Policy](https://dreamhack.io/lecture/courses/321)
