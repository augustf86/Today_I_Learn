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

<br/><br/><br/><br/>
### 🔖 출처
* [드림핵 Reverse Engineering] 📌 [x86 Assembly: Essential Part(1)](https://dreamhack.io/lecture/courses/57)
* [드림핵 Reverse Engineering] 📌 [x86 Assembly: Essential Part(2)](https://dreamhack.io/lecture/courses/38)
* [드림핵 System Hacking] 📌 [x86 Assembly: Essential Part(2)](https://dreamhack.io/lecture/courses/63)
* [드림핵 Reverse Engineering] ❓ [Quiz: x86 Assembly 1](https://dreamhack.io/learn/quiz/17)
* [드림핵 Reverse Engineering] ❓ [Quiz: x86 Assembly 2](https://dreamhack.io/learn/quiz/25)
