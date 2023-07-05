# 블록 암호: AES(Advanced Encryption Standard)

* 연산 능력의 향상으로 DES가 더 이상 안전하지 않게 되자, 2001년에 새롭게 표준으로 선정된 블록 암호 알고리즘
    - 공모에 제안된 21개의 알고리즘 중 보안성, 효율성, 하드웨어 이식의 적합성, 유연성 등을 고려해서 심사한 결과 **Rijndael 구조**기 체텍됨
    - AES의 설계 원리를 이해하기 위해선 **갈루아 필드**(Galois Field), **체**(Field)와 **군**(Group)에 대한 이해가 필요함
* 현대에는 대칭키 암호 알고리즘을 사용할 때 일반적으로 AES가 사용됨
    - 표준으로 선정된 이후부터 지금까지 기밀성을 위협하는 치명적인 취약점이 발견되지 않음
    - CPU 제조사들이 AES 연산을 위한 명령어를 따로 정의해 주어서 암﹒복호화 성능이 뛰어남

<br/><br/>

## AES의 구조
### SPN(Substitution Permutation Network)
<img width="2560" alt="SPN" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/9a25d3fa-4493-44f5-94e5-7903f3a56733"><br/>
* AES에서 사용하는 암호 구조
    - 곱 암호의 일종
    - S-Box를 사용하는 치환(Substitution)과 P-Box를 사용하는 순열(Permutation)을 여러 라운드에 걸쳐 반복함
* 페이스텔 구조와 SPN 구조의 비교
    | 페이스텔 구조 | SPN 구조 |
    |---|---|
    | 입력을 반으로 나눠 왼쪽 블록에만 라운드 함수가 적용됨 <br/> (오른쪽 블록은 다음 라운드의 왼쪽 블록으로 어떠한 처리도 없이 입력됨) | 라운드마다 입력 전체에 라운드 함수를 적용함 |
    - 같은 수의 라운드를 사용할 때 SPN이 페이스텔 구조에 비해 2배의 암호학적 안전성을 갖는다고 할 수 있음

<br/>

### AES 전체 구조
* 라운드마다 128비트 크기의 블록을 암호화하는 블록 암호
    - 키의 길이와 라운드 수
        | 명칭 | 키 길이 | 라운드 수 |
        |:---:|:---:|:----:|
        | AES-128 | 128비트 | 10회 |
        | AES-192 | 192비트 | 12회 |
        | AES-256 | 256비트 | 14회 |
        + 키의 길이를 선택할 수 있고, 라운드 수는 키의 길이에 따라 결정됨
* AES 암호화 과정
  <br/><br/>
  <img width="2560" alt="AES 구조" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/711bb033-f8cd-40f1-9283-bb431a3ca827"><br/><br/>
    | STEP | 설명 |
    |:---:|------|
    | 01 | 각 블록을 4행 4열의 상태 배열(State)로 재구성함 <br/> &nbsp;&nbsp; - State의 각 칸에는 8비트(1바이트)가 저장됨 |
    | 02 | 재구성된 입력(State)에 대해 AddRoundKey 함수를 적용함 |
    | 03 | 마지막 라운드 전까지 매 라운드마다 SubBytes, ShiftRows, MixColumns, AddRoundKey 함수를 반복하여 적용함 |
    | 04 | 마지막 라운드에서는 MixColumns을 제외한 SubBytes, ShiftRows, AddRoundKey 함수만 적용함 |
* AES의 라운드 함수들에 대한 역함수를 이용하여 AES 복호화를 수행함


<br/><br/>

## AES 라운드 함수
### SubBytes
<img width="2560" alt="SubBytes" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/e68f62f5-55a1-4432-b03d-ebc72e292fa7"><br/>
* State의 각 바이트를 S-Box를 참조하여 치환하는 함수
    - 바이트의 상위 4비트가 행을, 하위 4비트가 열을 결정함

<br/>

### ShiftRows
<img width="2560" alt="ShiftRows" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/89533e86-753e-45c6-8fa9-f653a9849885"><br/>
* State의 각 행을 구성하는 바이트들을 쉬프트하는 함수
    - 4개의 라운드 함수 중에서 유일하게 **순열**의 역할을 수행함
    - 수행 과정
        | | 과정 |
        |:---:|------|
        | 암호화 | 2행은 왼쪽으로 1칸, 3행은 왼쪽으로 2칸, 4행은 왼쪽으로 3칸을 이동시킴 |
        | 복호화 | 2행은 오른쪽으로 1칸, 3행은 오른쪽으로 3칸, 4행은 오른쪽으로 4칸을 이동시킴 |

<br/>

### MixColumns
* 열 단위로 치환을 수행하는 함수
    - 치환은 갈루아 필드 내에서의 행렬 연산으로 구해짐 (자세한 내용은 생략)

<br/>

### AddRoundKey
* 키 생성 함수(Key Schedule)로 생성된 라운드 키의 state를 각 바이트별로 XOR하는 함수
    - 복호화할 때는 XOR의 성질을 이용하여 동일한 키를 state에 XOR함

<br/><br/>

## 키 생성 함수(Key Schedule)

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Cryptography] 📌 [블록암호: AES](https://dreamhack.io/lecture/courses/73)
