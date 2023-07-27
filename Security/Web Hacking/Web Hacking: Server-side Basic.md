# Web Hacking: Server-side Basic
🔖 출처: [Dreamhack Lecture] Server-side Basic [🔗](https://dreamhack.io/lecture/courses/15)

<br/><br/>

## Server-side(서버 사이드) 취약점
| | 설명 |
|:---:|------|
| 정의 | 서버에서 사용자가 요청한 데이터를 해석하고 처리한 후 사용자에게 응답하는 과정에서 **웹 어플리케이션이나 데이터베이스와 <br/>같은 서버의 자원을 사용해 처리할 때** 사용자의 요청 데이터에 의해서 발생하는 취약점 |
| 공격 대상 | 웹 서비스를 제공하는 <U>서버</U> |
| 공격 목적 | 서버 내에 존재하는 사용자들의 정보를 탈취하거나, 서버의 권한을 장악함 |
| 발생 원인 | - HTTP 요청 시 악의적인 사용자가 메소드나 요청 헤더 등을 포함한 모든 데이터를 조작하여 전송할 수 있기 때문에 발생함 <br/> - **개발과 운영 상의 실수**로 사용자의 데이터에 대한 검증 과정의 부재 또는 올바르지 않은 검증으로 인해 발생함 |
| 기본 방지 대책 | 📌 ***서버는 사용자로부터 받는 모든 입력을 신뢰하지 않도록 해야 함*** |

<br/>

* Server-side 취약점의 분류
    | 분류 | 설명 |
    |:---:|------|
    | ***Injection*** <br/> (인젝션) | 서버의 처리 과정 중 사용자가 입력한 데이터가 처리 과정의 구조나 문법적으로 사용되어 <br/>발생하는 취약점 |
    | ***File Vulnerability*** | 서버의 파일 시스템에 사용자가 원하는 행위를 할 수 있을 때 발생하는 취약점 |
    | ***Business Logic Vulnerability*** <br/> (비즈니스 로직 취약점) | 인젝션, 파일 관련 취약점들과는 다르게 정상적인 흐름을 악용하는 것 |
    | ***Language specific Vulnerability*** <br/> (PHP, Python, NodeJS) | 웹 어플리케이션에서 사용하는 언어의 특성으로 인해 발생하는 취약점 |
    | ***Misconfiguration*** | 잘못된 설정으로 인해 발생하는 취약점 |

<br/><br/><br/>

## Injection

<br/><br/><br/>

## File Vulnerability

<br/><br/><br/>

## Business Logic Vulnerability

<br/><br/><br/>

## Language specific Vulnerability

<br/><br/><br/>

## Misconfiguration
