# 해시(Hash)

* **해시 함수**(Hash Function): 임의 크기의 데이터를 입력으로 받아서, 고정된 크기의 데이터를 반환하는 함수
    - 해시 함수의 반환값 = **해시 값**(Hash Value)
    - 📌 웹, 네트워크, 블록 체인 등 컴퓨터와 관련된 여러 분야에서 **무결성을 입증하는 수단**으로 사용되고 있음

<br/><br/>

## 암호학적 해시 함수 (Cryptographic Hash Function)
* 해시 함수 중에서 아래의 성질을 만족하는 함수
    | 성질 | 설명 |
    |---|------|
    | **제1 역상 저항성** <br/> (Preimage Resistance) | 암호학적 해시 함수 $H$에 대해 $y$가 주어졌을 때 $H(x) = y$를 만족하는 $x$를 찾는 것이 어려움 <br/> &nbsp;&nbsp; → 암호학적 해시 함수가 **일방향 함수**여야 함을 의미함 |
    | **제2 역상 저항성** <br/> (Second Preimage Resistance) | 암호학적 해시 함수 $H$에 대해 $x$가 주어졌을 때 $x≠x^′$, $H(x) = H(x^′)$을 만족하는 $x^′$를 찾는 것이 어려움 |
    | **충돌 저항성** <br/> (Collision Resistance) | 암호학적 해시 함수 $H$에 대해 $x≠x^′$, $H(x) = H(x^′)$을 만족하는 $x$, $x^′$를 찾는 것이 어려움 |
    - 추가로 현대에는 **눈사태 효과**(Avalanche Effect)를 이상적인 해시 함수의 조건 중 하나로 보기도 함
        + **눈사태 효과**: 대칭키 암호 시스템의 확산과 비슷하게 입력에 조그만 변화가 발생하면, 해시값에도 큰 변화가 발생하는 성질

<br/>

### 일방향 함수(One-way Function)
* 임의의 인자에 대한 함수 값은 쉽게 계산할 수 있지만, 함수 값으로부터 함수의 인자를 알아내기 어려운 함수
  <br/><br/>
  <img width="2560" alt="일방향 함수" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/c19f5978-83b7-4841-adac-a5633209c5cb">


<br/>

### 생일 역설(Birthday Paradox)
* **생일 역설 문제**: 일반적인 직관과는 다르게 전체 인원이 적어도, 그 안에 생일이 같은 학생이 높은 확률로 존재하는 현상
  <br/><br/>
  <img width="2560" alt="생열 역설 문제" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9d8996a6-b58a-4b4b-a4ac-5f55644fbfea">
  <br/><br/>
* 📌 암호학적 해시 함수의 **충돌 저항성**과 관련된 이론 중 하나
    - ⚠️ 아무리 정교한 해시 함수를 만들더라도, **공역의 크기가 작으면 해시 함수는 충돌 저항성을 만족하기 어려움** <br/> &nbsp;&nbsp; *→ 이를 고려하여 공역의 크기를 크게 만들어야 함*
        | 해시 함수 | 공역의 크기 | 충돌 쌍을 찾을 수 있는 연산의 횟수 |
        |-----|:---:|------|
        | 입력이 256비트의 출력에 대응되는 해시 함수 | $2^{256}$ | 대략 $2^{128} ≃ 10^{36}$번 이상의 연산이 필요 <br/> &nbsp;&nbsp; *→ 현실적으로 불가능함 (= 충돌 저항성을 만족)* |
      
<br/>

### 암호학적 해시 함수의 사용
1. 어떤 통신의 무결성(Integrity)를 보이기 위해 사용할 수 있음
    - 통신의 무결성(Integrity): 데이터가 무결하게 송﹒수신되는 성질
        + 📌 **수신한 데이터가 원본과 동일**할 때 이 데이터는 '무결하다'고 함
    - 암호학적 해시 함수는 충돌 저항성을 만족함 → 임의 데이터의 해시 값은 **그 데이터의 고윳값**이라고 볼 수 있음
        + 암호학적 해시 함수를 이용한 무결성 확인 방법
            | 순서 | 설명 |
            |:---:|------|
            | 01 | 송신자가 데이터와 함께 데이터의 해시 값을 수신자에게 전송함 |
            | 02 | 데이터와 데이터의 해시 값을 받은 수신자는 받은 데이터로부터 해시값을 생성함 |
            | 03 | 생성한 해시값과 송신자가 보낸 해시값을 비교함 → 이를 통해 받은 데이터가 변조되지 않았는지 확인할 수 있음 <br/> &nbsp;&nbsp; - 해시 값이 일치하면 데이터가 변조되지 않았음을 의미 <br/> &nbsp;&nbsp; - 해시 값이 일치하지 않는다면 데이터가 변조되었음을 의미 |
2. 민감한 데이터를 보관할 때 사용할 수 있음
    - EX: 웹 서버의 데이터베이스에서 민감한 정보인 사용자 비밀번호를 저장하는 경우
        | 저장 형태 | 설명 |
        |---|------|
        | 평문으로 저장 | 관리자가 비밀번호를 유출하거나, 해킹 사고로 사용자의 계정 정보를 모두 도난당할 위험이 있음 |
        | 해시값으로 저장 | 암호학적 해시 함수는 **일방향 함수**이므로 데이터베이스가 유출돼도 비밀번호의 원본이 알려질 가능성이 거의 없음 <br/> &nbsp;&nbsp; - 사용자 인증 시에는 사용자가 입력한 비밀번호의 해시 값을 데이터베이스에 저장된 해시 값과 비교함 |


<br/><br/>

## 해시 함수의 종류
### MD5
* Ronald Lorin Rivest가 1991년에 만들어낸 해시 함수
    - ⚠️ 현 시점에서는 **다양한 취약점이 발견**되어 더는 안전한 해시 함수로 여겨지지 않음
    - 몇몇 구형의 시스템에서는 아직 MD5가 사용되고 있음
* 임의 입력으로부터 128비트(= 16바이트)의 값을 생성하는 함수
    - 임의 길이의 입력을 블록 암호와 비슷하게 512비트 단위로 쪼갠 후 연산을 거쳐 값을 생성함
    - Python에서 제공하는 ```hashlib``` 모듈에 구현된 MD5 해시 함수
        ```python
        #!/usr/bin/env python3
        # Name: md5.py (MD5 예시)

        import hashlib
        print(hashlib.md5(b'dreamhack').hexdigiest())
        print(hashlib.md5(b'dreamhacc').hexdigiest())

        # MD5 hash of file
        with open('/path/to/file', 'rb') as f:
            print(hashlib.md5(f.read()).hexdigiest())
        ```

<br/>

### SHA256
* 미국 표준 기술 연구소(NIST)에서 만들어낸 해시 함수
    - 현재까지 취약점이 발견되지 않아 해시가 필요한 대부분의 곳에서 사용되고 있음
    - SHA256 이전에 존재하던 SHA0, SHA1는 모두 취약점이 발견됨 → ⚠️ 현재는 사용하지 않을 것이 권장됨
* 임의 입력으로부터 256비트(= 32바이트)의 출력을 내는 함수
    - MD5에 비해 길이가 2배로 늘어남 → **충돌 저항성이 크게 증가함**
        + SAH256이 만들어진 이후로 수많은 데이터들의 해시값이 생성되었지만 지금까지 충돌이 발생한 사례는 알려지지 않고 있음
    - Python에서 제공하는 ```hashlib``` 모듈에 구현된 SHA256 해시 함수
        ```python
        #!/usr/bin/env python3
        # Name: sha256.py (SAH256 해시)

        import hashlib
        print(hashlib.sha256(b'dreamhack').hexdigiest())
        print(hashlib.sha256(b'dreamhacc').hexdigiest())

        # SHA256 hash of file
        with open('/path/to/file', 'rb') as f:
            print(hashlib.sha256(f.read()).hexdigiest())
        ```

<br/><br/>

## MAC (메세지 인증 코드, Message Authentication Code)
* 데이터와 함께 보내는 추가적인 정보
    - 해시 함수를 응용함
    - MAC의 역할
        + 데이터의 무결성을 보장할 수 있음
        + 현재 통신 중인 상대방이 위장한 공격자가 아니라는 사실을 알아낼 수 있음
    - MAC 생성 방법은 크게 **암호학적 해시 함수를 이용하는 방법**과 **블록 암호를 이용하는 방법**으로 나눌 수 있음
* MAC을 이용한 메시지 무결성 인증 방법
    | | 설명 |
    |:---:|------|
    | MAC 생성의 전제 조건 | 메시지와 키가 필요함 <br/> &nbsp;&nbsp; - 키는 송신자와 수신자 사이에서 사전에 공유되어 있어야 함 |
    | 과정 1 | 송신자가 수신자에게 메시지를 보낼 때 **메시지와 키를 이용해 계산된 MAC 값**을 같이 전송함 |
    | 과정 2 | 수신자는 기존에 알고 있던 키를 이용해 수신한 메시지의 MAC을 계산하고, 이를 전송받은 MAC과 일치하는지 비교함 <br/> &nbsp;&nbsp; - 두 MAC 값이 일치한다면 메시지가 변조되지 않았음을 의미함 <br/> &nbsp;&nbsp; - 두 MAC 값이 다르다면 메시지가 변조되었음을 의미함 |
    - 공격자가 메시지를 위조하고 싶으면 **위조된 메시지에 대한 올바른 MAC값**을 알아내야 함 *→ 📌 키를 모르는 공격자는 MAC을 계산할 수 없음*

<br/>

### HMAC
* 해시 함수를 기반으로 하는 MAC
    - HMAC 함수 설명 예시
      <br/><br/>
      <img width="2553" alt="HMAC" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e0e35d7c-a7b4-422e-a142-1ae72abdb00d">
      <br/><br/>
        + ⚠️ 실제 표준으로 정의된 HMAC은 키의 길이, 블록의 길이를 인자로 하는 복잡한 함수로, 위의 예시는 이를 다소 간소화한 것임
    - Python에서 제공하는 ```hmac``` 모듈
        ```python
        #!/usr/bin/env python3
        # Name: hamc.py (HMAC)

        import hashlib, hmac

        # key: b'secret', msg: b'hello world'
        h = hmac.new(b'secret', b'hello world', hashlub.sha256).hexdigest()
        print(h)
        ```
* HMAC 사용 시 얻을 수 있는 효과
    - 메시지를 도청당해도 역상 저항성으로 인해 공격자가 HMAC에 사용된 키를 알 수 없음
    - 메시지를 위조하면 위조한 메시지에 대한 올바른 HMAC를 생성할 수 없음


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [해시](https://dreamhack.io/lecture/courses/77)
