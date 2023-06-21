# [Dreamhack Wargame] Command Injection Advanced
### [🚩Command Injection Advanced](https://dreamhack.io/wargame/challenges/413/)
<img width="1069" alt="CommandInjectionAdvanced_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9012fc35-0864-40c2-b060-5054290718a8">

<br/><br/>

## 문제 파일(index.php) 분석
```php
<html>
    <head></head>
    <link rel="stylesheet" href="/static/bulma.min.css" />
    <body>
        <div class="container card">
        <div class="card-content">
        <h1 class="title">Online Curl Request</h1>
        <?php
            if(isset($_GET['url'])){
                $url = $_GET['url']; // 사용자가 입력한 url 인자값을 가져옴
            
                // url 입력값이 http로 시작하는지 검사
                if(strpos($url, 'http') !== 0 ){ // http이 포함되어 있지 않은 경우
                    die('http only !');
                }else{ // http이 포함되어 있는 경우
                    // shell_exec 함수를 통해 curl 명령어가 수행됨 (curl의 인자로 url이 전달됨)
                    // → escapeshellcmd 함수를 사용함 (셸 메타문자를 사용할 수 없지만, 명령어의 옵션 또는 인자는 조작할 수 있음)
                    $result = shell_exec('curl '. escapeshellcmd($_GET['url']));
                
                    // url의 값을 MD5 알고리즘으로 해싱한 결과를 파일 이름로 하여 curl 명령어의 실행 결과를 cache 디렉터리에 저장함
                    $cache_file = './cache/'.md5($url);
                    file_put_contents($cache_file, $result);

                    echo "<p>cache file: <a href='{$cache_file}'>{$cache_file}</a></p>";
                    echo '<pre>'. htmlentities($result) .'</pre>';
                    return;
                }
            }else{
        ?>
            <form>
                <div class="field">
                    <label class="label">URL</label>
                    <input class="input" type="text" placeholder="url" name="url" required>
                </div>
                <div class="control">
                    <input class="button is-success" type="submit" value="submit">
                </div>
            </form>
        <?php
            }
        ?>
        </div>
        </div>
    </body>
</html>
```

<br/><br/>

## 문제 풀이
## 취약점이 존재하는 부분
```curl``` 명령어의 인자로 ```url```이 전달될 때 ```escapeshellcmd``` 함수를 사용하기 때문에 메타 문자를 사용한 Command Injection은 불가능하지만, 실행하려는 명령어의 옵션 또는 인자는 조작할 수 있음
* ```escapeshellcmd``` 함수는 '-' 문자에는 이스케이프 처리를 하지 않음 → ```curl``` 명령어의 옵션 중 하나인 ```-o```를 사용해 임의 디렉토리에 파일을 생성할 수 있음
    | 옵션 | 설명 |
    |---|-----|
    | -o (--output \<file\>) | stdout 대신에 \<file\>이라는 이름을 가진 파일에 출력을 씀 <br/> &nbsp;&nbsp; - curl 명령어의 실행 결과(방문한 url의 소스코드)가 파일의 내용이 됨 |
    - ```-o``` 옵션을 사용해 임의 위치에 파일을 생성할 수 있는지 확인해야 함
        + 코드를 보면 ```cache``` 디렉터리에 데이터를 쓸 수 있으므로 해당 위치에 파일을 생성하고 생성한 페이지에 접속함
        + 기본적으로 아파치 웹 문서의 파일 경로는 "/var/www/html"이므로 "/var/www/html/cache/test"를 ```-o``` 옵션의 <file>에 입력함
        + 수행 과정
            1. 사이트에서 ```http://127.0.0.1/ -o /var/www/html/cache/test```를 URL에 입력하고 submit 버튼을 누르면 "cache file:"이 화면에 출력됨
                <img width="984" alt="명령어 옵션 test" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/60b43109-8559-42ab-b441-160eac1ceb99">
                <img width="984" alt="명령어 옵션 test 결과" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/355e371e-2bd6-467d-8fc5-5da6b0dec53f">
            2. ```http://host3.dreamhack.games:20849/cache/test```(접속 정보는 문제 사이트에서 확인)에 접속하여 성공적으로 수행되는 것을 확인함
                <img width="984" alt="명령어 옵션 test 사이트 접속" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/60024bec-180c-4cc0-b966-9aaff7cd1dbc">

<br/><br/>

## 익스플로잇
