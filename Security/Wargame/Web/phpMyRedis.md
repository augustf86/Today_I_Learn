# [Dreamhack Wargame] phpMyRedis
### [🚩phpMyRedis](https://dreamhack.io/wargame/challenges/420/)
  <img width="1070" alt="설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/20e47dbe-a6ed-4fdc-ac35-dcb7dbd45492">

<br/><br/>

## 문제 파일(index.php의 두번째 <?php...?> 부분) 분석
* index.php (두번째 ```<?php...?>``` 부분)
    ```php
    <?php 
        if(isset($_POST['cmd'])){
            $redis = new Redis();
            $redis->connect($REDIS_HOST);
            $ret = json_encode($redis->eval($_POST['cmd'])); # 이용자에게 받은 cmd를 Redis에 저장함
            echo '<h1 class="subtitle">Result</h1>';
            echo "<pre>$ret</pre>";
            if (!array_key_exists('history_cnt', $_SESSION)) {
                $_SESSION['history_cnt'] = 0;
            }
            $_SESSION['history_'.$_SESSION['history_cnt']] = $_POST['cmd'];
            $_SESSION['history_cnt'] += 1;

            if(isset($_POST['save'])){ // SAVE command(save 체크박스에 체크하면 SAVE 명령어를 실행함)
                $path = './data/'. md5(session_id());
                $data = '> ' . $_POST['cmd'] . PHP_EOL . str_repeat('-',50) . PHP_EOL . $ret;
                file_put_contents($path, $data);
                echo "saved at : <a target='_blank' href='$path'>$path</a>";
            }
        }
    ?>
    ```
    - PHP의 ```eval(string $code)``` 함수: 소괄호 안의 문자열(string)을 php에서 실행시키는 함수 (코드를 동작시키는 시스템 함수)
* config.php (두 번째 ```<?php...?>``` 부분)
    ```php
    <?php 
        if(isset($_POST['option'])){
            $redis = new Redis();
            $redis->connect($REDIS_HOST);
            if($_POST['option'] == 'GET'){ // GET command (GET KEY → key의 값을 받아옴)
                $ret = json_encode($redis->config($_POST['option'], $_POST['key']));
            }elseif($_POST['option'] == 'SET'){ // SET command (SET key value → 새로운 데이터 추가)
                $ret = $redis->config($_POST['option'], $_POST['key'], $_POST['value']);
            }else{ // GET, SET 이외는 모두 에러를 발생시킴
                die('error !');
            }                        
            echo '<h1 class="subtitle">Result</h1>';
            echo "<pre>$ret</pre>";
        }
    ?>
    ```
    - ```GET```을 선택하면 Key 입력값에 해당하는 값을 가져올 수 있음
    - ```SET```을 선택하면 새로운 데이터를 추가할 수 있음(config를 수정할 수 있음)

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
config.php의 코드를 보면 이용자의 입력값을 별다른 검증 없이 사용하여 ```SET``` 명령어로 저장되는 파일의 경로와 이름 등을 변경할 수 있음
* 파일 저장 경로를 html 경로(```/html```)로 하고, 파일 이름을 "exploit.php"로 지정한 다음 셸을 실행하는 PHP 코드를 작성하고 SAVE 명령어로 저장시키면 이를 통해서 플래그를 획득할 수 있음


<br/><br/>

### 익스플로잇
1. /config.php 페이지에서 GET을 통해 현재 파일 저장 경로를 확인하여 html 경로로 변경한 후 저장되는 파일 이름 또한 "exploit.php"로 변경함
    - GET으로 파일 저장 경로(dir)의 값을 확인해 본 결과 html 경로이므로 변경하지 않아도 됨
        <img width="1154" alt="파일경로확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/674f5a92-f063-4ebe-af45-af4e8e3e54cc">
    - SET으로 파일 이름(dbfilename)의 값을 "exploit.php"로 지정하고 변경사항이 적용된 것을 GET을 통해 확인함
        <img width="1154" alt="저장되는 파일 이름 변경" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/ce1f9754-aaec-4396-8829-007e169cad22">
        <img width="1154" alt="변경된 파일 이름 확인" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/a7b4df46-e757-46c5-9249-7d135d1a5047">

2. 
