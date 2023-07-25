# [Dreamhack Wargame] Carve Party
* 출처: 🚩 Carve Party [🔗](https://dreamhack.io/wargame/challenges/96/)
* Reference: 브라우저 개발자 도구(DevTools) - Elements, Console
* 문제 설명
  <br/><br/>
  <img width="1068" alt="carve_party_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/22039356-9f05-4639-b94b-38372c277f2e">

<br/><br/>

## 문제 파일(jack-o-lantern.html) 중 <script> 태그 부분 및 취약점 분석
```html
<!-- jack-o-lantern.html의 <script> 태그 부분 (크롬 개발자 도구의 Elements 탭을 통해서도 확인 가능) -->
<script>
var pumpkin = [ 124, 112, 59, 73, 167, 100, 105, 75, 59, 23, 16, 181, 165, 104, 43, 49, 118, 71, 112, 169, 43, 53 ];
var counter = 0; // pumpkin을 클릭하는 횟수
var pie = 1;

function make() { 
  // counter가 특정 범위에 속하면 css를 변경하여 style을 적용시킴
  if (0 < counter && counter <= 1000) {
    $('#jack-nose').css('opacity', (counter) + '%');
  }
  else if (1000 < counter && counter <= 3000) {
    $('#jack-left').css('opacity', (counter - 1000) / 2 + '%');
  }
  else if (3000 < counter && counter <= 5000) {
    $('#jack-right').css('opacity', (counter - 3000) / 2 + '%');
  }
  else if (5000 < counter && counter <= 10000) {
    $('#jack-mouth').css('opacity', (counter - 5000) / 5 + '%');
  }

  if (10000 < counter) {
    $('#jack-target').addClass('tada');
    var ctx = document.querySelector("canvas").getContext("2d"),
    dashLen = 220, dashOffset = dashLen, speed = 20,
    txt = pumpkin.map(x=>String.fromCharCode(x)).join(''), x = 30, i = 0;

    ctx.font = "50px Comic Sans MS, cursive, TSCu_Comic, sans-serif"; 
    ctx.lineWidth = 5; ctx.lineJoin = "round"; ctx.globalAlpha = 2/3;
    ctx.strokeStyle = ctx.fillStyle = "#1f2f90";

    (function loop() {
      ctx.clearRect(x, 0, 60, 150);
      ctx.setLineDash([dashLen - dashOffset, dashOffset - speed]); // create a long dash mask
      dashOffset -= speed;                                         // reduce dash length
      ctx.strokeText(txt[i], x, 90);                               // stroke letter

      if (dashOffset > 0) requestAnimationFrame(loop);             // animate
      else {
        ctx.fillText(txt[i], x, 90);                               // fill final letter
        dashOffset = dashLen;                                      // prep next char
        x += ctx.measureText(txt[i++]).width + ctx.lineWidth * Math.random();
        ctx.setTransform(1, 0, 0, 1, 0, 3 * Math.random());        // random y-delta
        ctx.rotate(Math.random() * 0.005);                         // random rotation
        if (i < txt.length) requestAnimationFrame(loop);
      }
    })();
  }
  else {
    $('#clicks').text(10000 - counter); // 화면 하단의 '... more clicks to go!' 부분의 숫자를 10000에서 클릭한 수만큼 뺀 값으로 변경함
  }
}

$(function() {
  $('#jack-target').click(function () { # jack-target를 id 값으로 가지는 이미지를 클릭했을 때 수행하는 함수
    counter += 1; # counter를 1 증가시킴
    if (counter <= 10000 && counter % 100 == 0) { # counter가 10000 이하이고, 100으로 나누었을 때 나머지가 0인 경우에만 if문 내 실행문 실행
      for (var i = 0; i < pumpkin.length; i++) {
        pumpkin[i] ^= pie;
        pie = ((pie ^ 0xff) + (i * 10)) & 0xff;
      }
    }
    make();
  });
});
</script>
```
* 10000번 클릭을 수행하기 위해서는 ```$('#jack-target').click()```을 10000번 반복하여 수행하는 코드를 작성하면 됨
  - 자바스크립트를 실행하고 그 결과를 확인할 수 있는 Console 기능을 이용하여 아래와 같이 입력하면 플래그를 획득할 수 있음
    ```javascript
    for (var i = 0; i <= 10000; i++) { // for문을 이용하여 반복을 수행하고 있음
      $('#jack-target').click()
    }
    ```

<br/><br/>

## 문제 풀이
1. 다운로드한 문제 파일(jack-o-lantern.html)를 크롬 브라우저를 이용하여 열고 개발자 도구의 Console 탭으로 이동하여 클릭해야 하는 이미지의 id 속성 값을 확인함
   <br/><br/>
   <img width="1512" alt="Carve Party_문제 풀이 1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/0244275f-d309-4466-b347-cbf38e2b4561">

<br/>

2. 개발자 도구의 Console 탭에서 위의 자바스크립트 코드를 입력하면 10000번 클릭하라는 조건을 만족하여 화면에서 FLAG를 획득할 수 있음
   <br/><br/>
   <img width="1512" alt="Carve Party_문제 풀이 2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/ec963b6f-4223-4fd5-a248-3ae7fedcdce9">
