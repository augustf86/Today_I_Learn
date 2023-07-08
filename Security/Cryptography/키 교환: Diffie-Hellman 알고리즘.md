# 키 교환: Diffie-Hellman 알고리즘

## 키 교환(Key Exchange)의 필요성
* 📌 대칭키 암호의 전제 조건: 데이터를 주고 받기 위해서는 **송신자와 수신자가 같은 키를 공유하고 있어야 함** <br/> &nbsp;&nbsp;&nbsp;&nbsp; = 두 사람이 대챙 키 암호 시스템을 이용하여 통신하려면, **데이터를 교환하기 전에 키 교환이 이뤄져야 함**
    - 송﹒수신자가 물리적으로 가까운 거리에 있음 → 직접 만나서 신원을 확인한 후 키를 교환하면 됨
    - 일반적으로 아주 먼 거리의 대상과 통신하는 현대 유﹒무선 환경에서는 안전한 키 교환이 쉽지 않음
        | 키 전달 방법 | 공격 가능 방법 |
        |---|------|
        | 한 사람이 키를 생성한 후 <br/> 전화로 상대방에게 키를 전달함 | 통화를 도﹒감청하는 사람이나 기관은 키를 알 수 있음 |
        | 한 사람이 키를 생성한 후 <br/> 이메일, SNS의 DM 등으로 키를 전달함 | 둘 사이의 통신을 도청하면 해당 키를 알 수 있음 |

<br/>

* ⚠️ 대칭키 암호는 키에서 비롯됨 → **키를 안전하게 공유할 수 없는 환경에서 대칭키 암호는 무용지물임**
    - 해결책
        + 암호학자들이 네트워크와 같은 공개된 채널을 통해 키를 교환해도 외부인은 키를 알 수 없게 하는 **공개 키 교환 알고리즘**을 고안함 <br/> &nbsp;&nbsp; → 키 알고리즘의 등장으로 네트워크를 이용한 두 통신자는 안전하게 키를 교환할 수 있게 됨
        + 현대에 널리 사용되는 방식: 키 교환 알고리즘으로 대칭키를 교환함 → 교환한 대칭키로 암호화된 통신을 수행함
    - 네트워크 통신이 안전하지 않음 → 금융 서비스와 같은 **보안이 중요한 서비스들이 온라인 환경으로 이식되지 못했음**
        + 의식하기는 어렵지만 암호 기술의 발전은 개인의 생활에 많은 영향을 미치고 있음

<br/><br/>

## Diffie-Hellman 키 교환 절차(Diffie-Hellman key exchange protocol)
* 1976년 Whitfield Diffie와 Martin Hellman이 고안한, 공개된 채널로 키를 교환할 수 있는 알고리즘
    - 최초의 공개 키 교환 알고리즘이며, 현재도 널리 사용되고 있음
        + 최근에는 더욱 뛰어난 성능을 보여주는 **타원 곡선 Diffie-Hellman**(Elliptic-Curve Diffie-Hellman, ECDH) 알고리즘으로 발전함
    - Diffie-Hellman 알고리즘을 이해하기 위해서는 다양한 수학적 배경지식이 필요함

<br/>

### Background: 수학적 원리
* 모듈로 연산에서의 거듭제곱
  <br/><br/>
  <img width="2560" alt="모듈러 연산에서의 거듭제곱" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9174afee-2ed5-4d3d-b876-93492d1c386a">
  <br/>
  
    - Diffie Helmman 키 교환 알고리즘의 연산량을 줄여주는데 이 방법이 사용됨
        + ```2^(2048)```번 가까이 제곱된 어떤 수의 모듈러 값을 구해야 함 → 여기에 위의 방법을 적용하면 2048번 정도만 연산해도 그 값을 구할 수 있음

<br/>

* 페르마의 소정리(Fermat's Little Theorem)
  <br/><br/>
  <img width="2554" alt="페르마의 소정리" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/8426199b-5ed7-4c5b-bbca-f8ab96d4e266">

<br/>

* 이산 로그 문제(Discrete Logarithm Problem)
  <br/><br/>
  <img width="2560" alt="이산 로그 문제" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/03240b52-f5bd-4b26-8973-fb7ada9b0eac">
  <br/>

    - Diffie Hellman 알고리즘의 안전성은 **이산 로그 문제의 어려움**에 바탕을 두고 있음
        + 키를 모르는 공격자가 키를 구하려면 ```m```이 ```2^2048``` 정도 되는 이산 로그 문제를 풀어야함 → 현재의 연산 능력으로는 불가능하다고 알려져 있음

<br/>

### 키 교환 절차


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [키 교환: Diffie-Hellman 알고리즘](https://dreamhack.io/lecture/courses/75)
