# Blind SQL Injection (Advanced)

* Blind SQL Injection의 단점
    - 수많은 쿼리를 전송하기 때문에 실제 환경에서는 방화벽에 의해 접속 IP가 차단될 수 있음
    - 데이터의 길이가 길면 길수록 실행해야 하는 쿼리의 수도 증가하고 공격에 들이는 시간이 길어짐

<br/>

## Blind SQL Injection을 통해 데이터베이스의 내용을 효율적으로 알아내기 위한 방법
### Binary Search(이진 탐색)
이미 **정렬된 리스트**에서 검색 범위를 절반씩 좁혀가며 데이터를 탐색하는 방법 (효율적인 탐색을 위한 알고리즘)
* 이진 탐색의 과정: 찾으려는 데이터와 중간점에 위치하는 데이터를 반복적으로 비교해서 원하는 데이터를 찾는 것
    | 과정 | 설명 |
    |---|-------|
    | 1. 범위 지정 | 범위에서 한 숫자만이 정답일 때 범위의 중간값을 지정함 |
    | 2. 범위 조절 | 정답이 중간값보다 큰 값인지 확인함 <br/> &nbsp; → 큰 값이라면 범위를 (중간값 + 1) ~ (마지막 값)으로 조절하고, 작은 값이라면 범위를 (첫 번째 값) ~ (중간값 - 1)으로 조절함 |
    - 1번과 2번 과정을 반복하다보면 범위를 크게 좁힐 수 있고, 최종적으로 정답을 찾아낼 수 있음

<br/>

#### *Binary Search를 이용한 Blind SQL Injection 공격 예시*
* users 데이터베이스의 구성
    | username | password |
    |---|---|
    | admin | P@ssword |
* Binary Search를 이용한 공격 방법
    - 비밀번호에 포함될 수 있는 아스키에서 출력 가능한 문자의 범위(32 ~ 126)을 미리 확인하고 Binary Search를 통해 해당 범위를 좁혀가며 탐색을 진행함
    - 공격 코드 예시 (첫 번째 자리가 32와 126의 중간값인 79보다 큰 값인지 비교)
        ```sql
        SELECT * FROM users WHERE username='admin' AND ASCII(SUBSTR(password, 1, 1)) > 79;
        # P는 80이므로 79보다 크기 때문에 결과가 출력됨
        ```
        + 이러한 과정을 반복하다보면 비밀번호를 모두 구할 수 있음

<br/><br/>

### Bit 연산
* 7개의 비트에 대해 1인지 비교하여 총 7번의 쿼리로 임의 데이터의 한 바이트를 알아내는 방법
    - Bit 연산 방법의 출발 배경
        + 하나의 비트는 0과 1로 이루어져 있음
        + ASCII: 0부터 127 범위의 문자를 표현할 수 있음 → 7개의 비트를 통해 하나의 문자를 나타낼 수 있음을 의미
* MySQL의 ```BIN``` 함수와 ```ORD``` 함수를 이용함
    | 함수 | 형태 | 설명 |
    |---|---|------|
    | BIN | ```BIN(숫자)``` | 인자로 입력된 숫자를 2진수로 반환함 |
    | ORD | ```ORD(문자)``` | 인자로 입력된 문자(```'A'``` 등)의 아스키 코드 값을 반환함 |
    - 사용 예시
        ```sql
        SELECT BIN(ORD('A'))l # 'A'(문자)를 아스키 코드 값(숫자) 형태로 변환하고, 이를 다시 비트의 형태로 변환함
        ```
        
<br/>

#### *Bit 연산을 이용한 Blind SQL Injection 공격 예시*
* users 데이터베이스의 구성
    | username | password |
    |---|---|
    | admin | P@ssword |
* 비트 연산을 이용한 공격 방법
    - SUBSTR과 BIN을 통해서 총 7번의 쿼리를 실행해 한 바이트를 알아낼 수 있음 → 비밀번호의 길이만큼 반복하여 전체 비밀번호를 알아낼 수 있음
    - 공격 코드 예시 (관리자 계정의 첫 번째 바이트를 알아내는 방법)
        ```SQL
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 1, 1)=1; # 결과가 출력됨 (첫번째 자리 1)
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 2, 1)=1; # 결과가 출력되지 않음 (두번째 자리 0)
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 3, 1)=1; # 결과가 출력됨 (세번째 자리 1)
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 4, 1)=1; # 결과가 출력되지 않음 (네번째 자리 0)
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 5, 1)=1; # 결과가 출력되지 않음 (다섯번째 자리 0)
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 6, 1)=1; # 결과가 출력되지 않음 (여섯번째 자리 0)
        SELECT * FROM users username='admin' AND SUBSTR(BIN(ORD(password)), 7, 1)=1; # 결과가 출력되지 않음 (일곱번째 자리 0)
        # 비밀번호의 첫 번째 바이트: 1010000 → 10진수로 표현하면 80 (문자로 표현하면 'P')
        ```

<br/><br/><br/><br/>

### 🔖 출처
* [드림핵 Web Hacking Advanced - Server Side] 📌 [ExploitTech: Blind SQL Injection Advanced](https://dreamhack.io/lecture/courses/304)
