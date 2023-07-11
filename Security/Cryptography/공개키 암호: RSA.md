# 공개키 암호: RSA

* **RSA 암호 알고리즘**: 공개키 암호 시스템 중 하나, 전자 거래﹒금융 거래﹒인증서 등 다양한 분야에서 가장 보편적으로 사용되는 암호 및 인증 알고리즘
    - RSA의 역사
        + 1978년 MIT의 대학원생인 Ronald L. Rivest와 Adi Shamir, 그라고 그들의 지도 교수인 Leonard Adleman의 연구로 쳬계화된 알고리즘
        + 1983년에 특허로 등록되었지만, 현재는 만료되어 공공 및 민간 분야에서 널리 사용되고 있음
    - RSA 알고리즘의 안전성
        - 📌 아주 큰 두 소수의 곱으로 이루어진 합성수를 인수분해하기 어렵다는 **인수 분해 문제의 어려움**에 기반함 <br/> &nbsp;&nbsp; → RSA로 암호화할 때는 합성수의 소인수분해가 어려워지도록 각 인자를 적절히 설정해야 함
    - RSA 암﹒복호화 과정에서 대칭키 암호 알고리즘보다 훨씬 많은 연산을 필요로 함
        + 많은 데이터를 여러 번 암호화해야 하는 네트워크 통신에서는 잘 사용되지 않음

<br/><br/>

## RSA
* 대칭키 암호와 달리 두 개의 키(공개키와 비밀키)가 사용됨
    | 키 | 설명 |
    |:---:|------|
    | 공개키 (Public Key) | 모든 사용자에게 공개되며, **평문을 암호화할 때 사용됨** |
    | 비밀키 (Private Key) | 타인에게 노출되어서는 안되며, **공개키로 암호화된 암호문을 복호화할 때 사용됨** |

<br/>

### RSA Background: 오일러 정리
<img width="2555" alt="오일러 정리" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e4e2aec0-254f-484a-9cce-24760a4d15e0"><br/>

<br/>

### RSA 키 생성 과정
* 공개키 암호 알고리즘에서는 공개키와 개인키를 생성하는 키 생성 과정이 필요함
    | 암호의 종류 | 설명 |
    |---|------|
    | 대칭키 암호 알고리즘 | 임의의 난수를 선택하기만 하면 키 생성 가능 |
    | 공개키 암호 알고리즘(RSA) | 인수분해를 어렵게 만들기 위해 복잡한 연산을 거쳐 키를 생성해야 함 |
* RSA의 키 생성 과정
  <br/><br/>
  <img width="2560" alt="RSA 키 생성 과정" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e99fd886-a0fa-481a-8938-e06b6522c72f"><br/>
    - 키 생성 과정의 결괏값들
        | 결과 | 설명 |
        |:---:|------|
        | $n$ | modulus, 공개키로 사용됨 |
        | $e$ | 공개 지수(public exponent), 공개키로 사용됨 |
        | $d$ | 비밀 지수(private exponent), 비밀키로 사용됨 |

<br/>

### RSA 암﹒복호화
<img width="2560" alt="RSA 암호화, 복호화" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/caa5c4d1-606f-481c-83cd-28480b00b59f"><br/>

<br/><br/>

## RSA 공격
⚠️ 아래의 공격들을 방지하고 RSA의 안전한 사용을 위해서는 $n$, $e$, $d$를 주의해서 설정해야 함!

<br/>

### 작은 $e$ 설정 시 가능한 공격
* RSA 암호 알고리즘 구현 시 **빠른 암호화**를 위해 **공개 지수 $e$를 작게 설정**하기도 함 <br/> &nbsp;&nbsp; → ⚠️ $e$를 너무 작게 설정하면 Coppersmith 공격과 Hastad's Broadcast 공격 등에 취약해질 수 있음
* 공격의 종류
    - Coppersmith 공격과 Hastad's Braodcast 공격
      <br/><br/>
      <img width="2560" alt="작은 e 사용 시 가능한 공격" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/d69950b8-0251-4b66-a4ff-3cb604907ea5">
      <img width="2560" alt="중국인의 나머지 정리" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/43f3dcdc-a0c0-4973-bc05-b702392cfd31">
      <br/><br/>
    - 이 외에도 Coppersmith의 짧은 패드 공격(Short Pad Attack) 등에 취약함
* 📌 공개 지수를 너무 작은 값으로 설정하면 위와 같은 취약점이 발생하고, 너무 큰 값으로 설정하면 RSA 알고리즘의 효율성이 떨어지게 됨 <br/> &nbsp;&nbsp; → 일반적으로 공개 지수로 $2^{16}+1=65537$을 사용함

<br/>

### 공통 $n$ 사용 시 가능한 공격
* Common Modulus Attack
  <br/><br/>
  <img width="2560" alt="공통 n 사용 시 가능한 공격" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/5e6d904c-60b4-447d-9d7d-2a7d880bda7f">
  <br/><br/>
* 📌 수신자들이 같은 $n$을 사용하면 쉽게 공격받을 수 있음 → 수신자들이 $n$을 무작위 값으로 생성하여 사용해야 함

<br/>

### 작은 $d$ 사용 시 가능한 공격
* 비밀 지수 $d$가 작으면 가능한 공격들
  <br/><br/>
  <img width="2560" alt="작은 d 사용 시 가능한 공격" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/b2ef1b9a-8f48-49d5-ad6c-c33826e4ecfc">
  <br/><br/>
* 📌 수신자들이 비밀 지수 $d$를 작게 설정하면 여러 공격에 취약함 → 비밀 저수를 설정할 때는 $n$보다 적당히 큰 수가 되도록 해줘야 함


<br/><br/>

> ## 공개키 암호의 현재와 미래
> * 공개키 암호는 **어려운 수학 문제를 기반으로 하여 암호시스템의 안전성을 확보한다**는 점에서 대칭키 암호와 다름
>   | 공개키 암호 | 기반으로 하고 있는 수학 문제 |
>   |---|------|
>   | RSA | 인수분해의 어려움을 기반으로 함 |
>   | ElGamal 암호 | 이산대수의 어려움을 기반으로 함 |
>   | Elliptic Curve ElGamal 암호 | 타원곡선 상에서의 이산대수의 어려움을 기반으로 함 |
>   - ⚠️ "문제의 어려움"을 기초로 하기 때문에 관련된 문제들을 쉽게 풀 수 있게 하는 알고리즘이 제시되거나, 컴퓨터의 연산 능력이 향상되면 안전성을 위협받게 됨
>       + RSA의 경우 양자 컴퓨터가 등장하면 쇼어 알고리즘으로 쉽게 풀릴 수 있음 <br/> &nbsp;&nbsp; → 양자 컴퓨터가 등장한 이후의 암호 시스템(Post Quantum Cryptograph, PQC)에 대해서 활발히 연구 중에 있음


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [공개키 암호: RSA](https://dreamhack.io/lecture/courses/76)
