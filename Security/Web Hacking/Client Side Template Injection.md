# Client Side Template Injection

* 현재에는 개발의 편의성을 위해 다양한 프론트엔드 프레임워크가 사용됨
    - 프레임워크 사용 시 얻을 수 있는 개발의 편리함
        + DOM 내의 데이터를 수정하기에 간편함
        + 웹 페이지를 컴포넌트화 하여 작성할 수 잇음
    - 보편적으로 사용되는 프레임워크의 종류: Vue, React, AngularJS 등

<br/>

* **Client Side Template Injection**: 이용자의 입력값이 client side template framework에 의해 템플릿으로 해석되어 랜더링될 때 발생하는 취약점
    - ⚠️ 개발의 편의성을 높여주는 프레임워크를 잘못된 방식으로 사용할 경우 이를 통해 **XSS 취약점까지 연계**하는 것이 가능함
        + 여기서는 Vue와 AngularJS만 설명하지만 그 외 프레임워크에서도 발생할 수 있음!
    - 꺽쇠와 같은 특수문자가 필터링되어 있더라도 XSS로 연계할 수 있음 → **파급력이 높은 취약점**
    - 📌 Template Injection 방지책
        + 반드시 production mode로 코드를 빌드하여 사용함
        + 이용자의 입력이 템플릿으로 인식되지 않도록 유의해야 함

<br/><br/>

## Vue Template Injection
### Vue (Vue.js)
* 2014년에 Google의 연구원인 Evan You가 개발한 프론트엔드 프레임워크
    - 오픈소스 자바스크립트 프레임워크 → 이용자 인터페이스나 Single Page Application을 빌드할 때 사용됨
    - Vue에 대한 상세한 정보는 [Vue.js 홈페이지](https://vuejs.org/)에서 확인 가능

<br/>

* Vue 예제 코드 (Vue 3 버전)
    ```html
    <!-- 참고: Vue.js Basic - Hello World(https://vuejs.org/examples/#hello-world) -->
    <script src="https://unpkg.com/vue@3"></script>

    <div id="app">{{ message }}</div> <!-- Hello Vue!가 화면에 출력됨-->
    <script>
        Vue.createApp({
            data() {
                return {
                    message: 'Hello Vue!'
                }
            }
        }).mount('#app')
    </script>
    ```
    - ```{{ }}```로 감싸진 부분(뷰 템플릿) 내에서 문자열을 표시하거나 자바스크립트 표현식을 실행할 수 있음 <br/> &nbsp;&nbsp; → ⚠️ 이 템플릿 내부에 공격자의 입력이 들어가 **Template Injection이 발생**한다면 **XSS 공격으로 이어질 수 있음**

<br/>

* 취약한 Vue 예제 코드
    ```php
    <script src="https://unpkg.com/vue@3"></script>

    <div id="app">
        <?php echo htmlspecialchars($_GET['msg']); ?>
    </div>

    <script>
        Vue.createApp({
            data() {
                return {
                    message: 'Hello Vue!'
                }
            }
        }).mount('#app');
    </script>
    ```
    - 이용자의 입력을 그대로 페이지에 출력하여 Vue Template Injection에 취약한 예제 코드
        + 이용자로부터 입력받은 GET 메소드의 ```msg``` 파라미터를 ```htmlspecialchars``` 함수를 통해 꺽쇠 문자를 모두 인코딩한 후 출력함 → 일반적인 XSS 공격은 불가능힘
        + 출력되는 값이 Vue의 템플릿으로 사용될 수 있음 → ⚠️ *Template Injection이 발생함*

<br/>

### Vue Template Injection
<img width="2560" alt="Vue Template Injection" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/d02d073f-4f85-4bef-8bfe-67e34946ae22">
<br/><br/>

* Vue Template Injection 발생 여부를 확인하는 가장 간단한 방법 → 📌 **탬플릿을 이용한 산술 연산의 수행**
    + ```{{1+1}}```과 같은 산술 연산 형태의 템플릿을 입력했을 때 연살이 실행된 형태인 ```2```가 출력됨 → Template Injection이 발생함!

<br/>

* Template Injection 발생 시 임의 자바스크립트 코드 실행으로 연계하는 방법으로 보통 **생성자**(Constructor)를 이용함
    - 📌 **Vue 템플릿 컨텍스트 내에서 생성자에 접근하여 임의 코드에 해당하는 함수를 생성하고 이를 호출하는 방식**으로 XSS 공격이 가능함
    - Vue 템플릿 컨텍스트 내에서 생성자에 접근하는 방법은 다양함
        | 대표적인 접근 방법 | 이를 이용한 익스플로잇 코드 예시 |
        |---|---|
        | ```{{_Vue.h.constructor}}```를 이용 | ```{{_Vue.h.constructor("alert(1)")()}}``` |

<br/><br/>

## AngularJS Template Injection
### AngularJS
* Google에서 개발되어 2010년에 출시된 프론트엔드 프레임워크
    - 타입스크립트 기반의 오픈소스 프레임워크
    - CLI 도구에서 다양한 기능을 제공함 → 개발을 편리하게 해주는 프레임워크 중 하나
    - AngularJS에 대한 상세한 정보는 [AngularJs Docs](https://angular.io/docs)에서 확인 가능

<br/>

* AngularJS 예제 코드 (AngularJS 1.8.2 버전)
    ```html
    <!doctype html>
    <html ng-app>
        <head>
            <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
        </head>
        <body>
            <div>
                <label>Name:</label>
                <input type="text" ng-model="yourName" placeholder="Enter a name here">
                <hr>
                <h1>Hello {{yourName}}</h1>
            </div>
        </body>
    </html>
    ```
    - ```{{ }}```로 감싸진 부분(AngularJS 템플릿) 내에서 문자열을 표시하거나, 자바스크립트 표현식을 실행할 수 있음 <br/> &nbsp;&nbsp; → ⚠️ 이 템플릿 내부에 공격자의 입력이 들어가 **Template Injection 취약점이 발생**한다면 **XSS 공격**으로 이어질 수 있음

<br/>

* 취약한 AngularJS 예제 코드
    ```html
    <!doctype html>
    <html ng-app>
        <head>
            <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
        </head>
        <body>
            <?php echo htmlspeicalchars($_GET['msg']); ?>
        </body>
    </html>
    ```
    - 이용자의 입력을 그대로 페이지에 출력하여 AngularJS Template Injection에 취약한 예제 코드
        + 이용자로부터 입력받은 GET 메소드의 ```msg``` 파라미터를 ```htmlspecialchars``` 함수를 통해 꺽쇠 문자를 모두 인코딩한 후 출력함 → 일반적인 XSS 공격은 불가능힘
        + 출력되는 값이 AngularJS의 템플릿으로 사용될 수 있음 → ⚠️ *Template Injection이 발생함* 

<br/>

### Angular Template Injection
<img width="2560" alt="AngularJS Template Injection" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/c296bb33-121a-417d-a94d-bd3d02275b5c">
<br/><br/>

* Angular Template Injection 발생 여부를 확인하는 가장 간단한 방법 → 📌 **탬플릿을 이용한 산술 연산의 수행**
    - ```{{1+1}}```과 같은 산술 연산 형태의 템플릿을 입력했을 때 연살이 실행된 형태인 ```2```가 출력됨 → Template Injection이 발생함!

<br/>

* Template Injection 발생 시 임의 자바스크립트 코드 실행으로 연계하는 방법으로 보통 **생성자**(Constructor)를 이용함
    - 📌 **Angular 템플릿 컨텍스트 내에서 생성자에 접근하여 임의 코드에 해당하는 함수를 생성하고 이를 호출하는 방식**으로 XSS 공격이 가능함
    - Angular 템플릿 컨텍스트 내에서 생성자에 접근하는 방법
        | 대표적인 접근 방법 | 이를 이용한 익스플로잇 코드 예시 |
        |---|---|
        | ```{{ constructor.constructor }}```를 이용 | ```{{constructor.constructor("alert(1)")()}}``` |



<br/><br/><br/><br/>

### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: Client Side Template Injection](https://dreamhack.io/lecture/courses/329)
