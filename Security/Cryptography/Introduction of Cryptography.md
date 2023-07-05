# Introduction of Cryptography

## 암호학(Cryptography)
* 좁은 의미의 암호확: 제3자로부터 정보를 보호하는 방법에 대한 연구
    - 핵심이 되는 연구 주제들
        | 주제 | 설명 |
        |---|------|
        | 키 생성 <br/> (Key Generation) | 암호화 및 복호화에 사용할 키를 만드는 과정
        | 암호화 <br/> (Encryption) | 키를 이용해 평문(Plaintext)을 암호문(Ciphertext)으로 변환하는 과정 |
        | 복호화 <br/> (Decryption) | 송신자가 전송한 암호문을 키를 이용해 다시 평문으로 변환하는 과정 |
    - 암호 시스템(Cryptosystem): 암호화와 복호화로 정보가 전달되는 체계
      <br/><br/>
      <img width="2560" alt="암호 시스템" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/18997b48-fd26-4baf-a362-5590f7f47963"><br/>

<br/>

* 현대의 암호학의 역할
    - 수신자와 송신자가 서로의 신원을 확인하는 방법, 메시지가 중간에 조작되지 않았음을 보증하는 방법 등을 연구하는 것이 폭넓게 암호학의 분야로 연구되고 있음

<br/><br/>

## Background: 배타적 논리합
* 배타적 논리합(eXclusive OR, XOR): 입력으로 들어온 두 인자가 서로 다를 때, 참을 반환하는 연산
  <br/><br/>
  <img width="2560" alt="배타적 논리합" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f359c09b-d1c1-4181-a100-42b09c4207f8"><br/><br/>
    - 암호학에서 배타적 논리합은 일반적으로 **비트 단위**로 이루어짐
    - 두 입력 값을 2진법으로 표기했을 때 각 자릿수의 값이 다르면 1, 같으면 0이 출력됨
    - 📌 **임의의 정수를 자기 자신과 배타적 논리합 하면 결과 값은 0이 됨**
        + 비트 간 배타적 논리합은 특정 비트의 반전을 의미함 → XOR 연산을 두 번 반복하면 원래 값을 반환함

<br/><br/>

## Background: 합동식
* 합동식: 두 정수 a, b를 각각 정수 m으로 나눴을 때 나머지가 같은지를 판별하는 식
    <br/><br/>
    <img width="2560" alt="합동식" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/92f8506a-d80e-48ad-9bdc-a6e4bfb672cf"><br/><br/>
    - **a와 b 각각을 m으로 나눈 나머지가 같을 때**, 수학적으로 a와 b가 **mod m에 대해 합동**(congruent)이라고 표현함
        + a, b가 mod m에 대해 합동일 경우 a, b 각각에 정수 x를 더하거나 빼거나 곱해도 여전히 합동임 (나눗셈은 성립하지 않음)

<br/>

* 합동식에서 곱셈의 역원
      <br/><br/>
      <img width="2560" alt="역원" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8727973f-4a03-478d-a8e2-b9ae9986d0df">


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [머릿말](https://dreamhack.io/lecture/courses/69)
* 처음 배우는 암호화, 장필리프 오마송, 한빛미디어 _ Page 30 (Chapter 1. 암호화 - 1.1 기초)
