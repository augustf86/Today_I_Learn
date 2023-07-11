# [Dreamhack Wargame] Textbook-DH
### [🚩Textbook-DH](https://dreamhack.io/wargame/challenges/120/)
<img width="1073" alt="textbook_dh_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9be5a1ad-e52e-4f5d-bafc-39285525397e">


<br/><br/>

## 문제 파일(challenge.py) 분석
```python
#!/usr/bin/python3
# Crypto 모듈을 사용하기 위해서는 pip install pycryptodome으로 해당 모듈 설치 필요
from Crypto.Util.number import getPrime
from Crypto.Util.Padding import pad, unpad
from Crypto.Cipher import AES
import hashlib
import random

class Person(object):
    def __init__(self, p):
        # 소수 p, 1 <= g <= (p-1)를 만족하는 적당한 수 g, 그리고 1 <= a <= (p-1)를 만족하는 적당한 수 a를 선정함
        self.p = p
        self.g = 2 # 1 <= g <= (p=1)를 만족하는 적당한 수 g가 2로 고정
        self.x = random.randint(2, self.p - 1)
    
    def calc_key(self):
        self.k = pow(self.g, self.x, self.p) # self.g의 self.x 제곱을 self.p로 나눈 나머지 (g^x mod p)
        return self.k

    def set_shared_key(self, k):
        self.sk = pow(k, self.x, self.p) # k의 self.x 제곱을 self.p로 나눈 나머지 (k^x mod p)
        aes_key = hashlib.md5(str(self.sk).encode()).digest()
        self.cipher = AES.new(aes_key, AES.MODE_ECB)

    def encrypt(self, pt): # 키로 암호화
        return self.cipher.encrypt(pad(pt, 16)).hex()

    def decrypt(self, ct): # 키로 복호화
        return unpad(self.cipher.decrypt(bytes.fromhex(ct)), 16)

flag = open("flag", "r").read().encode()

prime = getPrime(1024) # p >= 2^{1024}를 만족하는 소수 p를 설정
print(f"Prime: {hex(prime)}")
alice = Person(prime)
bob = Person(prime)

alice_k = alice.calc_key() # Alice가 A = g^a mod p(= 2^a mod p)를 계산함
print(f"Alice sends her key to Bob. Key: {hex(alice_k)}")
print("Let's inturrupt !")
alice_k = int(input(">> ")) # 사용자의 입력으로 A를 대체함
if alice_k == alice.g:
    exit("Malicious key !!")
bob.set_shared_key(alice_k) # Bob이 A^b mod p(Alice와 공유하는 키)를 계산함 → (사용자의 입력값)^b mod p

bob_k = bob.calc_key() # Bob이 B = g^b mod p(= 2^b mod p)를 계산함
print(f"Bob sends his key to Alice. Key: {hex(bob_k)}")
print("Let's inturrupt !")
bob_k = int(input(">> ")) # 사용자의 입력으로 B를 대체함
if bob_k == bob.g:
    exit("Malicious key !!")
alice.set_shared_key(bob_k) # Alice가 B^b mod p(Bob과 공유하는 키)를 계산함 → (사용자의 입력값)^a mod p

print("They are sharing the part of flag") # Alice와 Bob은 flag를 나눠 가지고 있음 (Alice는 앞부분을, Bob은 뒷부분을 가지고 있음)
print(f"Alice: {alice.encrypt(flag[:len(flag) // 2])}")
print(f"Bob: {bob.encrypt(flag[len(flag) // 2:])}")
```

<br/><br/>

## 문제 풀이
### 취약점 분석
Diffie-Hellman 알고리즘을 이용하여 Alice와 Bob이 키를 교환하고 있음 → ⚠️ Diffie-Hellman 키 교환을 수행하는 중에 **서로의 신원을 확인하는 과정이 없어 중간자 공격(Man In The Middle Attack)에 취약함**
* 문제의 코에서 ```"Lets inturrupt !"``` 다음에 이어지는 ```>> ``` 뒤에 사용자의 입력을 받아 Alice와 Bob이 계산한 A(```g^a mod p```)와 B(```g^b mod p```)를 대신하고 있음
    - 공격자는 ```Prime: ``` 뒤에 출력되는 소수 p를 탈취하고, A와 B를 원하는 값으로 대신하여 통신에서 사용되는 AES 키를 계산해낼 수 있음
        - ```>> ``` 뒤에 ```1```를 입력하면 ```1^a mod p```, ```1^b mod p```가 됨 → ```a```, ```b```의 값에 상관없이 결과는 ```1```이 됨 (```1 mod p)``` <br/> &nbsp;&nbsp; ⇒ 따라서 AES 키는 ```hashlib.md5(str("1").encode()).digiest()```가 됨
    - 계산한 AES 키로 암호화된 플래그들을 복호화하여 합치면 플래그를 획득할 수 있음

<br/><br/>

### 익스플로잇
