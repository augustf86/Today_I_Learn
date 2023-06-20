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

<br/><br/>

## 익스플로잇
