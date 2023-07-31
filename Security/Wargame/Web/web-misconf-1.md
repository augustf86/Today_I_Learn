# [Dreamhack Wargame] web-misconf-1
* 출처: 🚩 web-misconf-1 [🔗](https://dreamhack.io/wargame/challenges/45)
* Reference: Misconfiguration - 부주의로 인해 발생하는 문제점 (권한 설정 문제 - **기본 계정**)
* 문제 설명
  <br/><br/>
  <img width="812" alt="web-misconf-1_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/ae19fe28-86b6-4d7d-ba10-af4586858acb">

<br/><br/>

## 문제 파일(defaults.ini) 및 취약점 분석
<img width="1512" alt="web-misconf-1_문제 파일 분석" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/5b6f0047-2acb-4047-985b-6d8df778d56c"><br/>
* defaults.ini 파일을 열면 Granfana의 Paths, Server, Database, Security 등 기본 설정을 확인할 수 있음
    - VS code의 검색 기능(cmd+f)으로 'admin'을 검색한 결과 관리자(admin) 계정의 username과 password를 확인할 수 있음 <br/> &nbsp;&nbsp; → 기본적으로 제공하고 있는 계정 정보를 삭제하지 않아 admin 계정으로 로그인할 수 있음

<br/><br/>

## 문제 풀이(익스플로잇)
