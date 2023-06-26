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
        + ```<meta>``` 태그의 ```http-equiv``` 속성의 값을 ```"Content-Security-Policy```로 하고, ```content``` 속성의 값이 정책 지시문을 작성하여 CSP 구문을 추가할 수 있음
        + 예시 (CSP 구문 예시를 ```<meta>``` 태그를 이용해 추가)
            ```html
            <meta http-equiv="Content-Security-Policy" content="default-src 'self' https://example.test.com">
            ```
* CSP를 올바르게 정의하였는지 검사하기 위해 구글에서 제공하는 [CSP Evaluator](https://csp-evaluator.withgoogle.com)
    - 직접 작성한 CSP가 올바른지 검사할 수 있음
      <img width="2560" alt="CSP Evaluator" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b514aeba-b7cb-4bc8-a4a9-1629605e67c4">

<br/><br/>

## Content Security Policy 기본 정책


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Background: Content Security Policy](https://dreamhack.io/lecture/courses/321)
