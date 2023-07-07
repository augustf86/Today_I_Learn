# CSRF Token 오용

* **Cross Site Request Forgery**(CSRF): 임의 이용자의 권한으로 임의 주소에 HTTP 요청을 보낼 수 있는 취약점
    - CSRF 공격의 피해자는 **자신의 의지와는 무관하게** 공격자가 의도한 행위를 특정 웹 사이트에 요청하게 됨
        + 의도치 않은 요청을 통해 임의 비밀번호 변경, 게시글 등록 및 삭제, 임의 메시지 전송 등 **HTTP 요청으로 이뤄지는 행위에 대해 인지하지 못한 채 피해를 당할 수 있음**
    - 📌 일반적으로 CSRF 취약점을 방어하기 위해 **CSRF Token**을 사용함

<br/><br/>

## CSRF 방어 기법: CSRF Token
* 동일한 오리진에서만 접근 가능한 형태로 저장해두고, HTTP 요청을 전송할 때 함께 전송되는 Token
    - 웹 서버는 전송된 Token을 이용하여 제3자가 아닌 이용자로부터 요청이 왔다는 것을 인증할 수 있음
    - CSRF Token 값의 사용
        - 보통 HTML ```<form>``` 태그의 ```hidden``` 속성에 입력됨
        - 동적 요청에서도 사용될 수 있음
    - CSRF Token의 장﹒단점
        | | 설명 |
        |:---:|------|
        | 장점 | 캡차나 암호화 방식과 달리 추가적인 사용자 상호작용이 불필요함 |
        | 단점 | XMLHttpRequest 또는 Fetch API 등을 통해 ```Authorization```과 같은 이용자 인증 헤더를 설정하여 <br/>통신하는 것에 비해 여러 가지 보안 문제의 원인이 되기도 함 |

<br/>

### CSRF Token을 이용한 CSRF 탐지 코드 예시
```php
<?php
    if (!isset($_SESSION["csrftoken"])) { // 세션에 CSRF Token이 없는 경우 CSRF Token을 생성한 후 세션에 저장함
        $csrftoken = bin2hex(random_bytes(32));
        $_SESSION["csrftoken"] = $csrftoken;
    } else { // 세션에 CSRF Token이 존재하는 경우 해당 CSRF Token을 사용함
        $csrftoken = $_SESSION["csrftoken"];
    }

    $method = $_SERVER["HTTP_METHOD"];
    if ($method !== "GET" && $method !== "HEAD") { // GET, HEAD 메소드 외의 메소드로 요청을 받은 경우에 아래 실행문을 실행함

        // csrftoken 값이 없거나 세션에 저장된 csrftoken 값과 비교해봤을 때 그 값이 다른 경우
        if (!isset($_POST["csrftoken"]) || !hash_equals($csrftoken, $_POST["csrftoken"])) {
            header("HTTP/1.1 403 Forbidden");
            die("CSRF detected"); // CSRF 공격 탐지
        }

        // POST 요청을 전달받은 csrftoken 값이 세션에 저장된 csrftoken 값과 동일한 경우
        echo "Input value: ";
        echo htmlentities($_POST["query"], ENT_QUOTES|ENT_HTML5, 'utf-8');
    }
?>

<form action="" method="POST">
    <input name="csrftoken" type="hidden" value="<? =htmlentities($csrftoken, ENT_QUOTES|ENT_HTML5, 'utf-8'); ?>"/>
    <input name="query" type="text" />
    <input type="submit"/>
</form>
```
* 세션에 CSRF Token을 저장하고, HTTP 요청을 통해 전송된 CSRF Token과 세션에 저장된 Token이 같은지 확인하여 CSRF 공격을 탐지함

<br/><br/>

## CSRF Token 사용 시 주의 사항 및 오용 시 발생할 수 있는 문제점
### 짧은 CSRF Token
* 📌 CSRF Token은 외부자가 예측할 수 없도록 설계되어야 함
    - Token을 모르는 공격자가 무차별 대입 공격(Brute-force attack)으로 Token을 획득할 수 없도록 **Token의 길이가 충분히 길어야 함**

<br/>

### 예측 가능한 CSRF Token (PRNG 등)
* ⚠️ Token의 길이가 충분히 길어도 **Token의 값이 예측 가능**하다면 공격자는 Token을 추론하여 올바른 CSRF Token을 생성해낼 수 있음
    | 공격자가 Token 값을 예측할 수 있는 경우와 방어법 | 
    |---|
    | 현재 시간 등 공격자가 추측 가능한 데이터를 기반으로 Token을 생성하는 경우 <br/> → 공격자가 예측할 수 없는 값을 이용해 Token을 생성해야 함 |
    | 암호학적으로 안전하지 않은 의사 난수 생성기(Pseudorandom Number Generator, PRNG)를 사용해 Token을 생성하는 경우 <br/> → 충분한 안전성이 보장된 난수 생성기(Cryptographically-Secure Pseudorandom Number Generator, CSPRNG)를 사용하여 Token을 생성해야 함 |

<br/>

### CSRF Token 유출
* CSRF Token이 제공하는 보안의 전제 = *"공격자가 Token을 알지 못함"* → ⚠️ Token이 **기타 다른 경로로 제3자에게 노출되지 않도록** 주의해야 함
    - Token이 다른 경로로 노출되는 예시: CSRF Token이 URL의 쿼리 파라미터로 넘겨지는 경우
        + 이후 다른 링크를 방문하였을 때 ```Referer``` 헤더에 Token이 그대로 노출됨 → 공격자는 이를 이용해 역으로 Token을 획득할 수 있음


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Web Hacking Advanced - Client Side] 📌 [Exploit Tech: CSRF Token 오용](https://dreamhack.io/lecture/courses/323)
