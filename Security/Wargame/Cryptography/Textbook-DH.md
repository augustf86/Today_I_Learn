# [Dreamhack Wargame] Textbook-DH
### [🚩Textbook-DH](https://dreamhack.io/wargame/challenges/120/)

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
        self.g = 2
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

alice_k = alice.calc_key() # Alice가 A = g^a mod p를 계산함
print(f"Alice sends her key to Bob. Key: {hex(alice_k)}")
print("Let's inturrupt !")
alice_k = int(input(">> "))
if alice_k == alice.g:
    exit("Malicious key !!")
bob.set_shared_key(alice_k) # Bob이 A^b mod p(Alice와 공유하는 키)를 계산함

bob_k = bob.calc_key() # Bob이 B = g^b mod p를 계산함
print(f"Bob sends his key to Alice. Key: {hex(bob_k)}")
print("Let's inturrupt !")
bob_k = int(input(">> "))
if bob_k == bob.g:
    exit("Malicious key !!")
alice.set_shared_key(bob_k) # Alice가 B^b mod p(Bob과 공유하는 키)를 계산함

print("They are sharing the part of flag")
print(f"Alice: {alice.encrypt(flag[:len(flag) // 2])}")
print(f"Bob: {bob.encrypt(flag[len(flag) // 2:])}")
```

<br/><br/>

## 문제 풀이
