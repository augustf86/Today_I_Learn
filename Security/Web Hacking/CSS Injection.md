# CSS Injection

### Cascading Style Sheet (CSS)
* HTML이 표시되는 방법을 정의하는 스타일 시트 언어
    - HTML로 정의된 문서를 다채롭게 하는 역할을 수행함
    - 이미지, 태그, 글자 등의 다양한 HTML 요소들이 사용자에게 어떻게 보여질지 정의할 수 있음

<br/>

### CSS Injection
* XSS와 비슷하게 웹 페이지 로딩 시 악의적인 문자열을 삽입하여 악의적인 동작을 이끄는 공격 <br/> &nbsp;&nbsp; = 표현에 사용될 임의의 CSS 코드를 주입시켜 의도하지 않은 속성이 정의되는 것
    - 공격자가 임의 CSS 속성을 삽입해 CSS Injection 공격을 수행할 수 있음
        | 공격자가 수행할 수 있는 동작 | 설명 |
        |---|------|
        | 웹 페이지의 UI(외관)을 변조 | 특정 버튼, ```<h1>``` 태그의 크기와 색깔 등을 변경할 수 있음
        | CSS 속성의 다양한 기능을 통해 <br/> 웹 페에지 내의 데이터를 외부로 훔침 | CSRF Token, 피해자의 API Key 등 웹 페이지에 직접적으로 보여지는 값처럼 <br/>**CSS 선택자를 통해 표현이 가능한 데이터만 탈취**할 수 있음 <br/> &nbsp;&nbsp; → CSS 선택자로 표현이 불가능한 ```script``` 태그 내 데이터들은 탈취할 수 없음! |
        + HTML (style) 영역에 공격자가 임의 입력 값을 넣을 수 있거나 임의 HTML를 삽입할 수 있지만 Content-Security-Policy로 인해 자바스크립트를 실행할 수 없는 등 다양한 환경에서 사용할 수 있음
    - 악의적인 CSS의 위험성은 임의 Javascript 삽입의 위험성보다 잘 알려져있지 않아 대개 무시되고 있음 → 📌 많은 회사에서 CSS Injection 취약점이 발생
        + 그 파급력과 심각성에 대해 대중에 널리 알려지지 않고 다양한 웹 방화벽에서 차단하지 않는 입력값을 가지고 있음
            | 키워드 | 설명 |
            |---|------|
            | ```"<script>"```, ```"alert"```, ```eval``` 등의 키워드 | 보통의 웹 방화벽의 경우 임의 Javascript의 실행을 막기 위한 <br/>키워드를 차단함 | 
            | ```"background:"```, ```"url``` 등의 키워드 | CSS Injection에 주로 사용되는 키워드는 차단하지 않음 |

<br/><br/>

## CSS Injection: 기초


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: CSS Injection](https://dreamhack.io/lecture/courses/327)
