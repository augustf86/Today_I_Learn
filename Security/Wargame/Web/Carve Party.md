# [Dreamhack Wargame] Carve Party
* 출처: 🚩 Carve Party [🔗](https://dreamhack.io/wargame/challenges/96/)
* Reference: 브라우저 개발자 도구(DevTools) - Elements, Console
* 문제 설명
  <br/><br/>
  <img width="1068" alt="carve_party_description" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/22039356-9f05-4639-b94b-38372c277f2e">

<br/><br/>

## 문제 파일(jack-o-lantern.html) 중 <script> 태그 부분 분석
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
  $('#jack-target').click(function () {
    counter += 1;
    if (counter <= 10000 && counter % 100 == 0) {
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

<br/><br/>

## 문제 풀이
1. 다운로드한 문제 파일(jack-o-lantern.html)를 크롬 브라우저로 열고 개발자 도구의 Console 탭으로 이동함
    - 10000번을 클릭해야 플래그를 획득하므로 Console 탭에서 자바스크립트 코드를 입력하여 반복문을 수행함
    - 클릭해야 하는 이미지의 id는 Elements 탭에서 Select 기능을 통해 ```svg>``` 태그의 id 속성('jack-target')에서 확인할 수 있음
      <img width="1340" alt="carve_party_1" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/577f5970-e60a-4b8a-a8d6-f60398d49c60">

2. Console 탭에서 아래와 같은 자바스크립트 코드를 입력하면 플래그를 획득할 수 있음
    ```javascript
    for (var i = 0; i <= 10000; i++) {
        $('#jack-target').click()
    }
    ```
  <img width="1555" alt="carve_party_2" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/6be543a8-f48d-4db9-b1d9-7e0c7a0a578e">
