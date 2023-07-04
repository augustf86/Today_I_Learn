# 블록 암호: DES (Data Encryption Standard)

* 미국의 국가 안보국(National Security Agency, NSA)에서 IBM의 루시퍼 알고리즘을 개량하여 만든 대칭키 암호
    - 128비트 키를 사용하던 루시퍼와는 달리 DES의 키 길이로 56비트를 사용하고, 내부에서 사용하는 알고리즘도 일부 변경함
    - 미국 국립표준기술연구소(National Institute of Standards and Technology, NIST)는 DES를 1976년부터 2002년까지 표준 블록 암호로 인정함
        + ⚠️ 현대에는 DES에 대한 공격 기법이 많이 연구되어 더 이상 블록 암호의 표준으로 사용하지 않음
* 8바이트(64비트)를 한 블록으로 하는 블록 암호
    - DES의 전체 구조
      <br/><br/>
      <img width="2560" alt="DES 전체 구조" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/5e8320bb-34f7-4799-b834-3b011effcfa6"><br/>


<br/><br/>

## DES의 원리
* 곱 암호(Product Cipher): 치환이나 순열 같은 단순한 연산들로 한 라운드를 구성하고, 각 라운드를 여러 번 반복하여 암호학적 안전성을 확보하는 암호
  <br/><br/><img width="2560" alt="product cipher" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/57075222-2200-4855-a7c8-2a64fad4a5cd"><br/><br/>
    - DES가 현대 암호 시스템의 두 가지 성질을 만족하기 위해 사용하는 연산들
        | 성질 | DES 연산 |
        |---|---|
        | 혼돈(Confusion) | 치환(Substitution) |
        | 확산(Diffusion) | 순열(Permutation) |
        + 치환과 순열은 매우 단순한 연산이므로 **평문에 이 연산들을 한 번 적용한다고 해서 암호학적 효과를 기대할 수 없음** <br/> &nbsp;&nbsp; 📌 치환과 순열을 여러 번 교차해서 반복 적용히면 혼돈과 확산의 성질을 모두 만족할 수 있음
        + DES는 여러 라운드에 걸쳐 치환과 순열을 반복하는 곱 암호의 일종
    - DES 외에도 많은 현대 블록 암호들은 곱 암호의 구조를 차용하고 있음

<br/>

* 페이스텔(Feistel) 구조
    - DES에서 라운드 함수를 적용하는 전체 과정은 페이스텔 구조를 이루고 있음
        + DES의 페이스텔 구조
          <br/><br/>
          <img width="2560" alt="fiestel" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/7d3f4703-5699-4285-ba03-38ab4cebfa31"><br/><br/>
            | 순서 | 설명 |
            |:---:|------|
            | 1 | 입력으로 들어온 블록을 동일한 길이의 왼쪽 블록 L과 오른쪽 블록 R으로 나눔 |
            | 2 | 각 라운드마다 오른쪽 블록은 다음 라운드의 왼쪽 블록으로 입력됨 (Red Line) |
            | 3 | 각 라운드마다 왼쪽 블록은 오른쪽 블록에 라운드 함수 F를 적용한 결과와 XOR되어 다음 라운드의 오른쪽 <br/>블록으로 입력됨 (Green Line) |
    - 페이스텔 암호의 특징
        | | 설명 |
        |---|------|
        | 장점 | 📌 블록 암호의 특징: 일반적으로 암호화를 구성하는 각 함수들에 역함수가 존재함 <br/> &nbsp;&nbsp; - 페이스텔 구조를 사용하면 라운드 함수 F가 복호화과정에서 XOR로 상쇄되므로 **역함수가 존재하지 않아도 됨** <br/> &nbsp;&nbsp; - 암호화와 복호화의 구조가 동일함(입력 순서가 반대) <br/> &nbsp;&nbsp;&nbsp;&nbsp; → **암호화에 사용한 라운드 키를 역순으로 입력**하면 복호화가 이뤄짐 |
        | 단점 | 비페이스텔 암호와 같은 안전성을 갖기 위해 두 배 정도 라운드를 사용해야 함 <br/> &nbsp;&nbsp; - 오른쪽 블록은 다음 라운드의 왼쪽 블록으로 어떠한 처리도 없이 입력되기 때문 |

<br/><br/>

## DES의 과정



<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [블록암호: DES](https://dreamhack.io/lecture/courses/72)
