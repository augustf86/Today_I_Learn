# [Dreamhack Wargame] [wargame.kr] fly to the moon
* 출처: 🚩 [wargame.kr]fly to the moon [🔗](https://dreamhack.io/wargame/challenges/324/)
* Reference: 브라우저 개발자 도구(DevTools) - Element 탭 (Tools: Burp Suite)
* 문제 설명
  <br/><br/>
  <img width="826" alt="fly to the moon_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/57f2811a-e606-4f2a-90a2-f142b6774280">

<br/><br/>

## 문제 풀이 (익스플로잇)
1. 문제 사이트의 화면에서 게임의 완료 조건이 점수(score)가 31337인 것을 확인할 수 있으며, 개발자 도구의 Element 탭에서 ```src``` 속성의 값이 ```"text/javascript```인 ```<script>``` 태그 부분을 확인하면 난독화된 자바스크립트 코드를 얻을 수 있음
   <br/><br/>
   <img width="1512" alt="fly to the moon_문제 풀이 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/47751cf4-3695-4193-9512-095812b76823">

<br/>

2. 난독화된 자바스크립트 코드를 난독화 해제 사이트를 이용해서 해제한 후 내용을 확인해보면 POST 메소드로 /high-scores.php에 게임 종료 시 body 데이터에 ```token```과 ```score``` 정보를 담아 Request를 보내고 있음을 알 수 있음
   <br/><br/>
   <img width="1512" alt="fly to the moon_문제 풀이 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/1d98ccf0-7fba-4ec8-b849-9e8978eb8c12">

<br/>

3. Burp Suite의 Target 탭에서 /high-scores.php로 보내는 POST 메소드의 Request를 Repeater 탭으로 복사한 다음, token과 score 값을 조작하여 [Send] 버튼을 누르면 조건을 만족하여 Response에서 FLAG를 획득할 수 있음
   <br/><br/>
   <img width="1512" alt="fly to the moon_문제 풀이 3" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/63a9fbba-2945-4d5b-9029-63b0404379b6">
