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


<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Reverse Engineering] 📌 [x86 Assembly: Essential Part(1)](https://dreamhack.io/lecture/courses/57)
* [드림핵 Reverse Engineering] 📌 [x86 Assembly: Essential Part(2)](https://dreamhack.io/lecture/courses/38)
* [드림핵 System Hacking] 📌 [x86 Assembly: Essential Part(2)](https://dreamhack.io/lecture/courses/63)
* [드림핵 Reverse Engineering] ❓ [Quiz: x86 Assembly 1](https://dreamhack.io/learn/quiz/17)
* [드림핵 Reverse Engineering] ❓ [Quiz: x86 Assembly 2](https://dreamhack.io/learn/quiz/25)
