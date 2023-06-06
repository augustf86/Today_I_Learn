# [Dreamhack Wargame] file-download-1
### [🚩file-download-1](https://dreamhack.io/wargame/challenges/37/)
  <img width="1069" alt="file-download-1_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/3b9efb40-bfdc-4a94-84e0-a609de15fdf9">

<br/><br/>

## 문제 파일(app.py) 분석
```python
#!/usr/bin/env python3
import os
import shutil

from flask import Flask, request, render_template, redirect

from flag import FLAG

APP = Flask(__name__)

UPLOAD_DIR = 'uploads' # 파일을 업로드할 디렉토리


# 인덱스 페이지
@APP.route('/')
def index():
    files = os.listdir(UPLOAD_DIR) # os.listdir() 메서드를 통해 UPLOAD_DIR 디렉토리 내의 모든 파일과 디렉토리의 리스트를 반환받음
    return render_template('index.html', files=files) # 파일 목록을 출력하는 인덱스 페이지(index.html)를 화면에 출력함



# upload 페이지
@APP.route('/upload', methods=['GET', 'POST'])
def upload_memo():
    if request.method == 'POST': # POST 메소드로 요청 시
        filename = request.form.get('filename') # 이용자가 전달한 filename을 가져옴
        content = request.form.get('content').encode('utf-8') # 이용자가 전달한 content(utf-8로 인코딩을 수행함)를 가져옴

        if filename.find('..') != -1: # filename에 '..'이 포함되어 있을 경우 (상위 디렉토리로 이동하는 메타 문자 '..'에 대한 필터링을 수행함 → Path Traversal 방지
            return render_template('upload_result.html', data='bad characters,,') # "bad characters"를 출력하는 페이지(upload_result.html)를 화면에 출력함

        with open(f'{UPLOAD_DIR}/{filename}', 'wb') as f: # uploads에 filename 파일을 wb(write + binary) 모드로 열어 content를 파일에 작성함
            f.write(content)

        return redirect('/') # 인덱스 페이지로 이동함

    return render_template('upload.html') # GET 메소드로 요청 시 이용자에겡 filename, content를 입력받는 페이지(upload.html)를 화면에 출력함



# read 페이지
@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '') # 이용자가 입력한 name 값을 가져옴

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f: # filename(이용자의 입려값)을 rb(read+binary) 모드로 열어 파일의 데이를 읽음
            data = f.read()
    except (IsADirectoryError, FileNotFoundError): # filename에 해당하는 파일을 발견할 수 없을 경우 에러 발생
        error = True

    # 파일의 정보를 출력하는 페이지(read.html)를 출력함
    return render_template('read.html',
                           filename=filename,
                           content=data.decode('utf-8'), # content의 내용은 utf-8로 인코딩되어 있으므로 utf-8로 디코딩해야 함
                           error=error)


if __name__ == '__main__':
    if os.path.exists(UPLOAD_DIR): # uploads 디렉토리가 존재할 경우 삭제
        shutil.rmtree(UPLOAD_DIR)

    os.mkdir(UPLOAD_DIR) # uploads 디렉토리를 생성함

    APP.run(host='0.0.0.0', port=8000)
```

<br/><br/>

## 문제 풀이
### 취약점이 존재하는 부분
* upload_memo 함수를 보면 입력 받은 파일에서 ```..``` 메타 문자가 존재하는지 확인하고 이를 필터링하는 부분이 존재하기 때문에 Path Traversal(File Upload Vulnerability)을 방지하고 있음
* read_memo 함수에서 전달 받은 filename을 아무런 검사 없이 사용하고 있음 → 해당 페이지에서 Path Traversal(File Download Vulnerability)으로 공격자는 웹 서비스에 존재하는 임의의 파일을 다운로드할 수 있음
    - read 페이지의 URL에서 name 파라미터의 값에 메타 문자를 포함시켜 flag.py로 접근하면 해당 파일을 읽을 수 있음

<br/><br/>

### 익스플로잇
file-download-1 문제 사이트의 URL의 인덱스 페이지에서 URL에 ```/read?name=../flag.py```를 덧붙인 다음 새로고침하면 read 페이지에 flag.py의 내용이 출력됨 (flag 획득 가능)
  <img width="1001" alt="fd_exploit_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/4d9d2745-4b99-450d-9d8f-1bca30fc4eb2">
  <img width="1001" alt="fd_exploit_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/986b5788-c24a-49c2-a2b0-d4cd184c8e2b">
