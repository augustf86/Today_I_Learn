# [Dreamhack Wargame] Textbook-RSA
### [🚩Textbook-RSA](https://dreamhack.io/wargame/challenges/131/)
<img width="1071" alt="Textbook-RSA_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b354b3c8-c484-4169-b62c-8f7ab9f6e02e">

<br/><br/>

## 문제 파일(challenge.py) 분석
```python
#!/usr/bin/python3
from Crypto.Util.number import getStrongPrime, bytes_to_long, inverse


class RSA(object):
    def __init__(self): # RSA 키 생성 과정
        # 서로 다른 소수 p, q를 선택함 (일반적으로 p, q는 1024비트 이상에서 길이가 같은 수로 생성함)
        self.p = getStrongPrime(512)
        self.q = getStrongPrime(512)
        self.N = self.p * self.q # n = p * q로 n을 설정함 (n = modulus → 공개키)
        self.e = 0x10001 # 𝜑(n)보다 작은 수 중 𝜑(n)과 서로소인 수 e를 0x10001(16진수 → 10진수 변환: 65537)로 설정 (e = 공개 지수, public exponent → 공개키)
        
        # self.N = self.p - self.q + 1 → p * q - p - q + 1 = 𝜑(n)
        self.d = inverse(self.e, self.N - self.p - self.q + 1) # inverse of e mod 𝜑(n) → e^(-1) mod 𝜑(n)로 d를 설정함 (d = 비밀 지수, private exponent → 비밀키)

    def encrypt(self, pt): # RSA 암호화
        return pow(pt, self.e, self.N) # pt는 평문을 의미 → pt^e mod N으로 암호화함

    def decrypt(self, ct): # RSA 복호화
        return pow(ct, self.d, self.N) # ct는 암호문을 의미 → ct^d mod N으로 복호화함


rsa = RSA()
FLAG = bytes_to_long(open("flag", "rb").read())
FLAG_enc = rsa.encrypt(FLAG)

print("Welcome to dream's RSA server")

while True:
    print("[1] Encrypt")
    print("[2] Decrypt")
    print("[3] Get info")

    choice = input() # 사용자의 입력값에 따라 [1]암호화, [2]복호화, [3]info를 선택함

    if choice == "1": # Encrypt (암호화)
        print("Input plaintext (hex): ", end="")
        pt = bytes_to_long(bytes.fromhex(input()))
        print(rsa.encrypt(pt))

    elif choice == "2": # Decrypt (복호화)
        print("Input ciphertext (hex): ", end="")
        ct = bytes_to_long(bytes.fromhex(input()))
        if ct == FLAG_enc or ct > rsa.N:
            print("Do not cheat !")
        else:
            print(rsa.decrypt(ct))

    elif choice == "3": # GET info (공개키 n, e와 암호화된 FLAG를 획득할 수 있음)
        print(f"N: {rsa.N}") # N = p * q
        print(f"e: {rsa.e}") # e = 0x10001
        print(f"FLAG: {FLAG_enc}") # 암호화된 FLAG

    else:
        print("Nope")
```

<br/><br/>

## 문제 풀이
### 취약점 분석
* 사용자의 입력값에 따라 RSA 암호화와 복호화를 모두 수행할 수 있음 → 원하는 암호문을 복호화할 수 있으므로 **선택 암호문 공격**을 사용하여 공격할 수 있음
    - 📝 **선택 암호문 공격**(Chosen Ciphertext Attack, CCA)
        | | 설명 |
        |:---:|------|
        | 정의 | 임의로 선택한 암호문과 일치하는 평문으로부터 암호키를 알아내기 위해 시도하는 공격 |
        | 특징 | CCA에서는 **평문 값과 암호문 값을 모두** 선택함 <br/> &nbsp;&nbsp; → 공격자는 선택하는 모든 평문에 대응하는 암호문을 얻고, 공격자가 선택한 각 암호문에 대응하는 평문을 얻음 <br/> &nbsp;&nbsp; ⇒ 이를 이용해 복호화에 사용되는 개인키를 복구할 수 있으며, 다른 관측된 암호문에 해당하는 평문 도출이 가능함 <br/> 💡 *CCA의 전제 조건: 암호분석가가 복호화 장치에 접근 가능한 상황이어야 함* |
        | 출처 | 📚 [Wikipedia: Chosen-ciphertext attack](https://en.wikipedia.org/wiki/Chosen-ciphertext_attack) |
    - RSA에서의 선택 암호문 공격(CCA)
        + RSA가 갖는 "곱셈에 대한 준동형사상(Homomorphism) 성질"을 이용하여 공격함 <br/> &nbsp;&nbsp; ⇒ **RSA에서 같은 키로 생성된 서로 다른 암호문 2개를 곱하면, 평문 2개의 곱을 암호화한 것과 그 결과가 같다는 것**을 이용함
            - 준동형성질 → 함수 $f(x)$에 대해 $f(a*b) = f(a) * f(b)$가 성립함
        + 수행 과정
            - 가정: 암호화된 flag(```FLAG_enc```)를 알고 있고, 암﹒복호화 기능을 사용할 수 있어야 함
            - 과정
                | 순서 | 설명 |
                |:---:|------|
                | 01 | 숫자 2를 암호화한 것과 flag를 암호화한 겂을 곱한 다음 이를 n으로 나눈 나머지 값을 구함 <br/> (```(pow(2, e) * flag_enc) % n```) |
                | 02 | 이를 복호화 기능을 이용해 복호화를 수행함 → ```2 * flag``` 값을 구할 수 있음 |
                | 03 | 2번의 결과값을 2로 나누면 ```flag```를 구할 수 있음 |
            

<br/><br/>

### 익스플로잇
* 위의 코드를 참고하여 textbook_rsa_exploit.py를 작성하고 실행함
    ```python
    from pwn import *
    from Crypto.Util.number import bytes_to_long, inverse, long_to_bytes

    p = remote("host3.dreamhack.games", 13633) # url, port 정보는 드림핵 워게임 문제의 접속 정보를 확인

    # GET Info 페이지에서 공개키 n, e와 암호화된 FLAG를 획득함
    p.sendlineafter("info\n", "3")

    p.recvuntil("N: ")
    n = int(p.recvline()[:-1])

    p.recvuntil("e: ")
    e = int(p.recvline()[:-1])

    p.recvuntil("FLAG: ")
    flag_enc = int(p.recvline()[:-1])

    print("N: " + str(n))
    print("\ne: " + str(e))
    print("\nFLAG_enc: " + str(flag_enc))

    # RSA가 가지는 "곱셈에 대한 준동형사상" 성질을 이용하여 선택 암호문 공격을 수행함
    # 취약점 분석에서 과정의 1번에 해당함
    exploit = (pow(2, e) * flag_enc) % n

    print("Exploit_flag: " + str(exploit))

    # 취약점 분석에서 과정의 2번에 해당함
    p.sendlineafter("info\n", "2")
    p.sendlineafter("(hex): ", hex(exploit)[2:])

    # 취약점 분석에서 과정의 3번에 해당함
    flag = (int(p.recvline()[:-1])) // 2

    print("Real flag: " + str(long_to_bytes(flag)))
    ```
    <br/>
    <img width="1900" alt="익스플로잇 결과" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/0d767441-d179-4702-8753-60b9c6dee14f">
