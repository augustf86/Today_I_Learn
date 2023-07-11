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


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [공개키 암호: RSA](https://dreamhack.io/lecture/courses/76)
