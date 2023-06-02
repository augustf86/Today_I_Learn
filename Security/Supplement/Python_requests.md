# Python의 requests 모듈
파이썬에서 HTTP 통신을 위해 제공하는 모듈 중 하나로, 다양한 메소드를 사용해 HTTP 요청을 보낼 수 있으며 응답 또한 확인할 수 있음
* GET, POST 메소드 이외에도 다양한 메소드를 사용해 요청을 전송할 수 있음
* 자세한 기능은 [Requests 모듈 공식 문서](https://docs.python-requests.org/en/latest/)를 참고

<br/>

### requests 모듈을 통해 HTTP의 GET 메소드 통신을 하는 예제 코드
```python
import requests

url = 'https://example.com/'

header = {
	'Content-Type': 'application/x-www-from-urlencoded',
	'User-Agent': 'USER_REQUEST'
}

params = {
  'test': 1,
}

for i in range(1, 5):
  c = requests.get(url + str(i), headers=headers, params=params)
  print(c.request.url)
  print(c.test)
```

* ```requests.get``` 함수: GET 메소드를 사용해 HTTP 요청을 보내는 함수
	- URL, Header, Parameter와 함께 요청을 전송할 수 있음

<br/>

### requests 모듈을 통해 HTTP의 POST 메소드 통신을 하는 예제 코드
```python
import requests

url = 'https://dreamhack.io/'

headers = {
	'Content-Type': 'application/x-www-from-urlencoded',
	'User-Agent': 'DREAMHACK_REQUEST'
}

data = {
	'test': 1,
}

for i in range(1, 5):
	c = requests.post(url + str(i), headers=headers, data=data)
	print(c.text)
```

* ```requests.post``` 함수: POST 메소드를 사용해 HTTP 요청을 보내는 함수
	- URL, Header, Body와 함께 요청을 전송할 수 있음
