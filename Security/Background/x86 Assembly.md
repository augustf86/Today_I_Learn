# x86 Assembly

* **어셈블리 언어**(Assembly Language): 0과 1로만 구성된 컴퓨터의 기계어(Machine Language)와 치환되는 언어
    - 기계어가 여러 종류라면 어셈블리어도 여러 종류여야 함
        + CPU에 사용되는 ISA는 종류가 굉장히 다양함 → ISA의 종류만큼 많은 수의 어셈블리어가 존재함
            | ISA | 어셈블리어 |
            |:---:|:---:|
            | x64(x86-64) | x64 어셈블리어 |
            | ARM | ARM 어셈블리어 |
            + 각 종류마다 어셈블리어가 상이함 *→ 많이 알면 알수록 리버스 엔지니어링에 유리함*
* **어셈블러**(Assembler)와 **디스어셈블러**(Disassembler)
    | | 설명 |
    |:---:|------|
    | 어셈블러 <br/> (Assembler) | **어셈블리어로 작성된 코드를 컴퓨터가 이해할 수 있는 기계어로 치환**해주는 역할을 수행함 <br/> &nbsp;&nbsp; → 어셈블리어가 기계어보다 이해하기 쉬웠으므로 개발자는 더욱 편리하게 소프트웨어를 개발할 수 있게 됨 |
    | 디스어셈블러 <br/> (Disassembler) | 컴퓨터가 이해할 수 있는 **기계어를 어셈블리어 코드로 번역**해주는 역할을 수행함 <br/> &nbsp;&nbsp; → 소프트웨어 분석가들은 소프트웨어를 분석하기 위해 기계어를 읽을 필요가 없어짐 |

<br/>

## x86 어셈블리 언어
* 한국어와 어셈블리 언어의 문법 구조 비교
    | 언어 | 비교 설명 |
    |:---:|------|
    | 한국어 <br/> (자연어) | 주어, 목적어, 서술어 등으로 이뤄진 문법 구조를 가지고 있음 <br/> &nbsp;&nbsp; - 어떤 문장을 읽을 때 주어와 목적어가 무엇인지 등을 파악하여 단어들에 문법적 의미를 부여하고, 문장을 이해함 |
    | 어셈블리 언어 | 📌 **명령어**(Operation Code, Opcode)와 **피연산자**(Operand)로 구성됨 (기본 형식: ```Opcode operand1 operand2```)<br/> &nbsp;&nbsp; - 한국어(자연어)보다는 **훨씬 단순한  문법 구조**를 가지고 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp; (명령어는 자연어의 동사에 해당되고, 피연산자는 자연어의 목적어에 해당됨) |

<br/>

### 기본 구조: 명령어(Operation Code, Opcode)
* 인텔의 x64에서 중요하게 다뤄지는 21개의 명령어
    | 분류 | 명령 코드 |
    |---|------|
    | 데이터 이동(Data Transfer) | ```mov```, ```lea``` |
    | 산술 연산(Arithmetic) | ```inc```, ```dec```, ```add```, ```sub``` |
    | 논리 연산(Logical) | ```and```, ```or```, ```xor```, ```not``` |
    | 비교(Comparison) | ```cmp```, ```test``` |
    | 분기(Branch) | ```jmp```, ```je```, ```jg``` |
    | 스택(Stack) | ```push```, ```pop``` |
    | 프로시저(Procedure) | ```call```, ```ret```, ```leave``` |
    | 시스템 콜(System call) | ```syscall``` |
    + 인텔의 x64에는 매우 많은 명령어가 존재함

<br/>

### 기본 구조: 피연산자(Operand)
* 아래의 3가지 종류 중 하나가 피연산자에 올 수 있음
    - 상수(Immdeiate Value)
    - 레지스터(Register)
    - 메모리(Memory)
        + ```[]```으로 둘러쌓은 것으로 표현되며, 앞에 크기 지정자(Size Directive) ```TYPE PTR```이 추가될 수 있음
            - ```TYPE```으로 지정할 수 있는 값
                | 타입 | 지정할 바이트의 크기 |
                |:---:|------|
                | ```BYTE``` | 1바이트의 크기를 지정함 |
                | ```WORD``` | 2바이트의 크기를 지정함 (→ **기존 인텔 아키텍처와의 호환성**을 위해 64비트 아키텍처에서 WORD를 64비트 크기로 <br/>지정하지 않고 16비트로 유지함) |
                | ```DWORD``` | 4바이트의 크기를 지정함 |
                | ```QWORD``` | 8바이트의 크기를 지정함 |
            - 메모리 피연산자 예시
                | 예시 | 설명 |
                |----|------|
                | ```QWORD PTR [0x8048000]``` | 0x8048000의 데이터를 8바이트(QWORD)만큼 참조 |
                | ```DWORD PTR [0x8048000]``` | 0x8048000의 데이터를 4바이트(DWORD)만큼 참조 |
                | ```WORD PTR [rax]``` | rax가 가리키는 주소에서 데이터를 2바이트(WORD)만큼 참조 |

<br/><br/>

## x86-64 어셈블리 명령어(opcode)
### 데이터 이동 명령어
어떤 값을 레지스터나 메모리 옮기도록 지시하는 명령어
* 데이터 이동 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```mov dst, src``` | src에 들어있는 값을 dst에 대입함 |
    | ```lea dst, src``` | src의 유효 주소(Effective Address, EA)를 dst에 저장함 |
* 📝 예제: 데이터 이동
  <br/><br/>
  <img width="2560" alt="데이터 이동 명령어 예시" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/5f59deb1-96b6-4fb2-8c73-e1cc458d6b81"><br/>

<br/>

### 산술 연산 명령어
덧셈, 뺄셈, 곱셈, 나눗셈 연산을 지시하는 명령어
* 산술 연산 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```add dst, src``` | dst에 src의 값을 더함 (→ ```dst += src```) |
    | ```sub dst, src``` | dst에 src의 값을 뺌 (→ ```dst -= src```) |
    | ```inc op``` | op의 값을 1 증가시킴 (→ ```op += 1```) |
    | ```dec op``` | op의 값을 1 감소시킴 (→ ```op -= 1```) |
* 📝 예제: 덧셈과 뺄셈
  <br/><br/>
  <img width="2560" alt="산술 연산 명령어 예시" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/638136c2-96da-4158-ad76-fbe88464292b"><br/>

<br/>

### 논리 연산 명령어
and, or, nor, neg 등의 비트 연산을 지시하는 명령어
* 논리 연산 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```and dst, src``` | dst와 src의 비트가 모두 1이면 1, 아니면 0 |
    | ```or dst, src``` | dst와 src의 비트 중 하나라도 1이면 1, 아니면 0 |
    | ```xor dst, src``` | dst와 src의 비트가 서로 다르면 1, 같으면 0 |
    | ```not op``` | op의 비트 전부 반전 |
    - 논리 연산은 **비트 단위로 이루어 짐**
* 📝 예제: 논리 연산
  <br/><br/>
  <img width="2560" alt="논리 연산 명령어 예시(and, or)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/574f5a0d-0439-4fa0-841f-a2b9cafe741b">
  <img width="2560" alt="논리 연산 명령어 예시 (xor, not)" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/f79761db-3933-403f-89ca-0c28d42efc73">
  <img width="2560" alt="논리 연산 명령어 예시 (xor, not) 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/12ffa383-7c14-411a-966c-4d8b25abb875"><br/>

<br/>

### 비교 명령어
두 피연산자의 값을 비교하고, 플래그를 설정하는 명령어
* 비교 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```cmp op1, op2``` | op1과 op2를 비교함 <br/> → **두 피연산자를 빼서 대소를 비교하고, 연산의 결과를 op1에 대입하지 않음** |
    | ```text op1, op2``` | op1과 op2를 비교함 <br/> → **두 피연산자에 AND 비트연산을 취해 비교하고, 연산의 결과를 op1에 대입하지 않음** |
* 📝 예제: 비교
  <br/><br/>
  <img width="2560" alt="비교 명령어 예시" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/85ed95f1-73d1-48f7-9904-58a166369503"><br/>

<br/>

### 분기 명령어
rip를 이동시켜 실행 흐름을 바꾸는 명령어
* 분기 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```jmp addr``` | addr로 rip를 이동시킴 |
    | ```je addr``` | 직전에 비교한 두 피연산자가 같으면 점프 (jump if equal) |
    | ```jg addr``` | 직전에 비교한 두 연산자 중 전자가 더 크면 점프 (jump if greater) |
    - ⚠️ 분기문은 굉장히 많은 경우의 수를 가지고 있음 → 전부 다르기 보다는 실제 코드를 분석하면서 감을 익혀나가는 것이 중요함
* 📝 예제: 분기
  <br/><br/>
  <img width="2560" alt="분기 명령어 예시 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/63bbdb73-6253-4cbb-891e-7285796604a1">
  <img width="2560" alt="분기 명령어 예시 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/97ff995a-272b-4d93-8f9b-c4603701ff9a"><br/>

<br/>

### 스택 명령어
운영체제의 핵심 자료구조인 스택(Stack)을 조작할 수 있는 명령어
* 스택 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```push val``` | val를 스택 최상단에 쌓음 |
    | ```pop reg``` | 스택 최상단의 값을 꺼내서 reg에 대입 |
* 📝 예제: 스택
  <br/><br/>
  <img width="2560" alt="스택 명령어 예시 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/5d1cfaa4-749a-4e96-84e2-5b8e848efbb3">
  <img width="2560" alt="스택 명령어 예시 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/80733758-37d2-4c05-b000-ada4d4778aa6"><br/>

<br/>

### 프로시저 명령어
프로시저의 호출과 반환을 위해 사용하는 명령어
* 컴퓨터 과학에서의 **프로시저**(Procedure): 특정 기능을 수행하는 코드 조각 (C 언어의 함수에 대응됨)
    - 프로시저 사용 시 얻을 수 있는 이점
        + 반복되는 연산을 프로시저 호출로 대체할 수 있음 → 전체 코드의 크기를 줄일 수 있음
        + 기능 별로 코드 조각에 이름을 붙일 수 있게 됨 → 코드의 가독성을 크게 높일 수 있음
    - 프로시저의 행위
        | 행위 | 설명 |
        |---|------|
        | 호출(Call) | 프로시저를 부르는 행위 |
        | 반환(Return) | 프로시저에서 돌아오는 것 |
        + ❗️ 프로시저 호출 시에는 프로시저를 실행하고 나서 원래의 실행 흐름으로 돌아와야 함 <br/> &nbsp;&nbsp; → **call 다음의 명령어 주소**(return address, 반환 주소)를 스택에 저장하고 프로시저로 rip를 이동시킴
* 프로시저 명령어의 종류
    | 명령어 형식 | 설명 |
    |---|------|
    | ```call addr``` | addr에 위치한 프로시저 호출(call) |
    | ```leave``` | 스택프레임 정리 |
    | ```ret``` | return address(반환 주소)로 반환 |
    - ❓ **스택프레임**을 사용하는 이유: 아래와 같은 문제를 막고, **함수별로 서로가 사용하는 스택의 영역을 구분**하기 위해 사용함
        - 스택 = 함수별로 자신의 지역변수 또는 연산 과정에서 부차적으로 생겨나는 임시 값들을 저장하는 영역 <br/> &nbsp;&nbsp; → 스택 영역을 아무런 구분 없이 사용할 경우 서로 다른 두 함수가 같은 메모리 영역을 공유하여 정상적인 연산을 수행할 수 없게 됨
* 📝 예제: 프로시저

<br/>

### 시스템 콜 명령어


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Reverse Engineering] 📌 [x86 Assembly: Essential Part(1)](https://dreamhack.io/lecture/courses/57)
* [드림핵 Reverse Engineering] 📌 [x86 Assembly: Essential Part(2)](https://dreamhack.io/lecture/courses/38)
* [드림핵 System Hacking] 📌 [x86 Assembly: Essential Part(2)](https://dreamhack.io/lecture/courses/63)
* [드림핵 Reverse Engineering] ❓ [Quiz: x86 Assembly 1](https://dreamhack.io/learn/quiz/17)
* [드림핵 Reverse Engineering] ❓ [Quiz: x86 Assembly 2](https://dreamhack.io/learn/quiz/25)
