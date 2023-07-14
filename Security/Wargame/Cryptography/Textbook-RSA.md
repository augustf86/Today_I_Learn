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
