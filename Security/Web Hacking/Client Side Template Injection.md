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



<br/><br/><br/><br/>

### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: Client Side Template Injection](https://dreamhack.io/lecture/courses/329)
