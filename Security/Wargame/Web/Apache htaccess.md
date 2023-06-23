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
