# OWASP TOP 10

취약점이 악용될 가능성, 허술한 보안 통제로 인해 취약점이 악용되었을 때 비즈니스에 미치는 영향을 토대로 해당 취약점의 위험도를 파악하여 만든 상위 10가지의 웹 애플리케이션 보안 취약점 목록
* OWASP에서 발표하며, 3~4년에 한 번씩 정기적으로 업데이트됨
    - OWASP(Open Web Application Security Project): 웹 애플리케이션 보안에 대한 정보를 공유하고 체계를 세우는 자발적인 온라인 정보 공유 사이트 (온라인을 통한 오픈 프로젝트)


<br/><br/>

## List of OWASP Top 10 2021
<img width="468" alt="OWASP_Top10_2021" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6f6f8d09-3bec-418b-9669-622fc0ca411e"><br/>

| OWASP Top 10 2021 | Avg Incidence Rate | Total CVEs | Documentation |
|---------|-----:|-----:|---|
| A01:2021 - Broken Access Control | 3.81% | 19,013 | 📚[A01:2021](https://owasp.org/Top10/A01_2021-Broken_Access_Control/) |
| A02:2021 - Cryptographic Failures | 4.49% | 3,075 | 📚[A02:2021](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/) |
| A03:2021 - Injection | 3.37% | 32,078 | 📚[A03:2021](https://owasp.org/Top10/A03_2021-Injection/) |
| A04:2021 - Insecure Design | 3.00% | 2,691 | 📚[A04:2021](https://owasp.org/Top10/A04_2021-Insecure_Design/) |
| A05:2021 - Security Misconfiguration | 4.51% | 789 | 📚[A05:2021](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/) |
| A06:2021 - Vulnerable and Outdated Components | 8.77% | 0 | 📚[A06:2021](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/) |
| A07:2021 - Identification and Authentication Failures | 2.55% | 3,897 | 📚[A07:2021](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/) |
| A08:2021 - Software and Data Integrity Failures | 2.05% | 1,152 | 📚[A08:2021](https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/) |
| A09:2021 - Security Logging and Monitoring Failures | 6.51% | 242 | 📚[A09:2021](https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/) |
| A10:2021 - Server-Side Request Forgery(SSRF) | 2.72% | 385 | 📚[A10:2021](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/) |

<br/><br/>

### A01: Broken Access Control (잘못된 접근 통제, 접근 권한 취약점)
* 2017년 버전에서 5위였던 Broken Access Control(잘못된 접근 통제)가 2021년 버전에서는 1위로 올라감
* 접근 통제(Access Control)은 사용자가 권한을 벗어나 행동할 수 없도록 정책을 시행함
    - 접근 통제가 취약할 경우 공격자가 할 수 있는 행동
        + 주어진 권한 밖의 모든 데이터를 무단으로 열람﹒수정﹒삭제 등의 행위가 가능함
        + 주어진 권한을 벗어난 기능을 사용할 수 있음
    - 일반적으로 개발자가 코드를 작성할 때 접근 통제 로직을 누락해서 발생함
* 📌 Notable CWE(Common Weakness Enuemration)
    - CWE-200: Exposure of Sensitive Information to an Unauthorized Actor(비인가된 행위자에 민감한 정보 노출)
    - CWE-201: Insertion of Sensitive Information Into Sent Data(전송된 데이터에 민감한 정보의 삽입)
    - CWE-352: Cross-Site Request Forgery(CSRF)

<br/>

### A02: Cryptographic Failures (암호화 오류)
* 2017년 버전의 Sensitive Data Exposure(민감한 데이터 노출)의 명칭이 2021년 버전에서 Cryptographic Failures(암호화 오류)로 변경됨
* **암호화와 관련된 오류**에 중점을 두며, 적절한 암호화가 이루어지지 않으면 민감한 데이터의 노출이나 시스템 손상으로 이어질 수 있음
    - 개인정보, 패스워드, 시스템 접속 정보 등 암호화로 보호해야 하는 정보가 그대로 노출되는 경우
    - 암호화를 수행했지만 너무 쉽게 복호화가 되는 경우 (암호화 부족)
* 📌 Notable CWE(Common Weakness Enumeration)
    - CWE-259: Use of Hard-coded Password(하드 코딩된 패스워드 사용)
    - CWE-327: Broken or Risky Crypto Algorithm(취약하거나 위험한 암호화 알고리즘)
    - CWE-331: Insufficient Entropy(불충분한 엔트로피)

<br/>

### A03: Injection (인젝션)
* 2017년 버전의 1위였던 Injection(인젝션)이 2021년 버전에서 3위로 내려감
* 내부 데이터베이스와 연동한 결과로 웹 애플리케이션에 보내주는 부분에서 주로 발생함
    - 신뢰할 수 없는 데이터가 명령어나 쿼리문의 일부분으로써 인터프리터로 보내질 때 발생
* Injection의 종류
    - SQL, NoSQL, OS Command, Object Relational Mapping(ORM), LDAP, Expression Language(EL), Object Graph Navigation Library(OGNL) Injection
* 📌 Notable CWE(Common Weakness Enumeration)
    - CWE-79: Cross-site Scripting(XSS)
    - CWE-89: SQL Injection
    - CWE-73: External Control of File Name or Path (파일 이름 또는 경로의 외부 제어)

<br/>

### A04: Insecure Design (안전하지 않은 설계)
* 2021년 버전에서 새롭게 추가됨
* 설계 결함과 관련된 위험에 중점을 두며, 누락되거나 비효율적인 제어 설계로 표현되는 다양한 취약점을 나타내는 광범위한 카테고리
    - 안전하지 않은 설계(Insecure Design)과 안전하지 않은 구현(Insecure Implementation)에는 차이가 있음
        + 안전하지 않은 설계에서 취약점으로 이어지는 구현 결함이 있을 수 있음
        + 안전하지 않은 설계는 특정 공격을 방어하기 위해 필요한 보안 통제가 없음 → 구현 단계에서 수정이 어려움
    - 일반적으로 개발 중인 소프트웨어 또는 시스템에 내재된 위험 프로파일링(분석)이 불충분하여 필요한 보안 설계 수준을 제대로 결정하지 못했을 때 발생함
    - 위협 모델링, 보안 설계 패턴 및 원칙, 참조 아키텍처와 같은 설계가 훌륭한 보안 환경을 위해 요구됨
* 📌 Notable CWE(Common Weakness Enumeration)
    - CWE-209: Generation of Error Message Containing Sensitive Information(민감한 정보가 포함된 오류 메시지 생성)
    - CWE-256: Unprotected Storage of Credentials(비보호된 자격 증명 저장소)
    - CWE-501: Trust Boundary Violation(신뢰 경계의 위반)
    - CWE-522: Insufficiently Protected Credentials(자격 증명의 불출분한 보호)

<br/>

### A05: Security Misconfiguration (보안 설정 오류)
* 2017년 버전의 XML External Entities(XXE, XML 외부 개체)가 2021년 버전에서는 Security Misconfiguration(보안 설정 오류)에 포함됨
* Security Misconfiguation(보안 설정 오류)가 발생하는 경우
    - 애플리케이션 스택의 적절한 보안 강화가 누락되었거나 클라우드 서비스에 대한 권한이 적절하지 않게 구성된 경우
    - 불필요한 기능이 활성화되거나 설치된 경우
    - 기본 계정 및 암호를 변경하지 않고 그대로 사용하는 경우
    - 지나치게 상세한 오류 메시지를 노출하는 경우
    - 최신 보안 기능이 비활성화되거나 안전하지 않게 구성되었을 경우 (최신 보안 패치를 업데이틓하지 않은 경우)
    - 소프트웨어가 오래되었거나 취약한 경우
* 📌 Noteable CWE(Common Weakness Enumeration)
    - CWE-16: Configuration(설정)
    - CWE-611: Improper Restriction of XML External Entity Reference(XML 외부 개체 참조의 부적절한 제한)

<br/>

### A06: Vulnerable and Outdated Components (취약하고 오래된 요소)
* 2017년 버전에서 9위였던 Using Components with Known Vulnerabilities(알려진 취약점을 이용한 컴포넌트의 사용)가 2021년 버전에서는 명칭이 Vulnerable and Outdated Components(취약하고 오래된 요소)로 변경되고 6위로 올라감
* Vulnerable and Outdated Components(취약하고 오래된 요소)가 발생하는 경우
    - 지원이 종료되거나 오래된 버전을 사용하는 경우
        - 애플리케이션뿐만 아니라 DBMS, API 및 모든 구성 요소, 런타임 환경 및 라이브러라가 포함됨
    - 정기적으로 취약성을 스캔하여 확인하지 않으며, 구성 요소와 관련된 보안 공지를 확인하지 않는 경우
    - 개발자가 업데이트, 업그레이드 또는 패치된 라이브러리의 호환성을 테스트하지 안흔ㄴ 경우
    - 구성 요소의 구성을 보호하지 않는 경우 (*A05: Security Misconfiguration과 연관*)
* 취약한 구성 요소를 공격하면 데이터의 손상이나 서버 장악 문제 등이 일어날 수 있음
* 📌 Notable CWE(Commmon Weakness Enumeration)
    - CWE-1104: Use of Unmaintained Third-Party Components(유지관리되지 않은 타사 구성 요소의 사용)
    - The two CWEs from TOP 10 2013 and 2017 (2013년 및 2017년 상위 10개 CWE 2개)

<br/>

### A07: Identification and Authentication Failures (식별 및 인증 오류)
* 2017년 버전에서 2위였던 Broken Authentication(잘못된 인증)가 2021년 버전에서는 명칭이 Identification and Authentication Failures(식별 및 인증 오류)로 바뀌고 7위로 내려감
    - Broken Authentication(취약한 인증)에서 Identification Failures(식별 실패)까지 포함하여 더 넓은 범위를 포함할 수 있도록 변경됨
* 웹 애플리케이션이 사용자에 대한 인증 및 세션 처리를 잘못했을 때 발생함
    - 사용자의 신원 확인, 인증 및 세션 관리는 인증 관련 공격으로부터 보호하는데 중요함 → 적절히 되지 않을 경우 취약점이 발생할 가능성이 높음
* 📌 Notable CWE(Common Weakness Enumeration)
    - CWE-297: Improper Validation of Certificate with Host Mismatch (호스트 불일치가 있는 인증서의 부적절한 유효성 검사)
    - CWE-287: Improper Authentication (부적절한 인증)
    - CWE-384: Session Fixation (세션 고정)

<br/>

### A08: Software and Data Integrity Failures (소프트웨어 및 데이터 무결성 오류)
* 2021년 버전에서 새롭게 추가됨
    - 2017년 버전에서 8위였던 Insecure Deserialization(안전하지 않은 역직렬화)가 이 카테고리에 포함됨
* 무결성을 확인하지 않고 소프트웨어 업데이트, 중요 데이터 및 CI/CD 파이프라인과 관련된 가정을 하는데 중점을 둠
    - 소프트웨어 및 데이터 무결성 오류는 무결성 오류로부터 보호하지 않는 코드 및 인프라와 관련이 있음
* 📌 Notable CWE(Common Weakness Enumeration)
    - CWE-829: Inclusion of Functionality from Untrusted Control Sphere(신뢰할 수 없는 제어 영역의 기능을 포함)
    - CWE-494: Download of Code Without Integrity Check(무결성 검사 없이 코드를 다운로드함)
    - CWE-502: Deserialization of Untrusted Data(신뢰할 수 없는 데이터의 역직렬화)

<br/>

### A09: Security Logging and Monitoring Failures (보안 로깅 및 모니터링 실패)
* 2017년 버전에서 10위였던 Insufficient Logging & Monitoring(충분하지 않은 로깅 및 모니터링)이 2021년 버전에서는 명칭이 Security Logging and Monitoring Failures(보안 로깅 및 모니터링 실패)로 바뀌고 9위로 올라감
* Security Logging and Monitoring Failures의 특징
    - 해당 취약점은 테스트하기 어렵지만, 해당 활동 없이는 공격 활동을 인지할 수 없음
    - 진행 중인 공격을 감지 및 대응하는데 도움이 됨
    - 이 카테고리가 잘못되면 침해사고 발생 시 포렌식 조사에 필요한 데이터와 로그를 얻기 힘듦
* 📌 Notable CWE(Common Weakness Enumeration)
    - CWE-778: Insufficient Logging (불충분한 로깅)
    - CWE-117: Improper Output Neutralization for Log (로그에 대한 부적절한 출력 중립화)
    - CWE-223: Omission of Security-relevant Information (보안 관련 정보 누락)
    - CWE-532: Insertion of Sensitive Information into Log File (로그 파일에 민감한 정보가 삽입됨)

<br/>

### A10: Server-Side Request Forgery (서버 측 요청 위조)
* 2021년 버전에서 새롭게 추가됨
    - 커뮤니티 설문조사에서 1위로 평가되어 이번에 추가됨
    - 실제 데이터를 봤을 때에는 평균 이상의 검사 결과를 보였으나 상대적으로 발생 가능성은 매우 낮고 공격 가능성은 평균 이상의 등급을 보임 (보안 커뮤니테서는 중요한 위험으로 여겨지고 있음)
* SSRF: 서버에서 이뤄지는 요쳥을 공격자가 변조하여 서버로 하여금 공격자가 원하는 행동을 하도록 하는 위험
    - SSRF 결함은 웹 애플리케이션이 사용자가 제공한 URL의 유효성을 검사하지 않고 원격 리소스를 가져올 때마다 발생함
        + 공격자는 SSRF를 통해 방화벽, VPN 또는 다른 유형의 네트워크 ACL(Access Control List)에 의해 보호되는 경우에도 응용 프로그램이 조작된 요청을 예기치 않은 대상으로 보내도록 강제할 수 있음
    - 최신 애플리케이션이 최종 사용자에게 편리한 기능을 제공하기 위해 URL을 가져오는 것이 일반적이게 되면서 SSRF 발생률이 증가하고 있는 추세이며, 클라우드 서비스와 아키텍처의 복잡성으로 인해 SSRF의 심각성 또한 높아지고 있음




<br/><br/><br/><br/>
### 🔖 출처
* [OWASP TOP 10 2021](https://owasp.org/Top10/)
* 인터넷 해킹과 보안[4판], 김경곤, 한빛아카데미 _ Page 073 ~ 077 (Chapter 02-3. 웹 애플리케이션의 취약점 - OWASP TOP 10)
