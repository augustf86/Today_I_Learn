# [Dreamhack Wargame] Flying Chars
### [🚩Flying Chars](https://dreamhack.io/wargame/challenges/850/)
<img width="1064" alt="Flying Chars_문제 설명" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/12cb6cca-280b-4296-bfb8-8c6f4c56e710">

<br/><br/>

## 문제 풀이
### 문제 사이트 분석
문제 사이트에 접속해보면 이미지들이 빠른 속도로 이동하는 것을 확인할 수 있음
<br/><br/>
<img width="1000" alt="문제 사이트 분석" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/79a102e0-ca7f-4834-afca-ae28b12aaea5"><br/>

* 개발자 도구의 Elements 탭에서 javascript 코드를 살펴본 결과 ```anim()``` 함수를 이용해 무작위로 위치를 변경하고 있음을 알 수 있음
    ```html
    <script type="text/javascript">
    const img_files = ["/static/images/10.png", "/static/images/17.png", "/static/images/13.png", "/static/images/7.png","/static/images/16.png", "/static/images/8.png", "/static/images/14.png", "/static/images/2.png", "/static/images/9.png", "/static/images/5.png", "/static/images/11.png", "/static/images/6.png", "/static/images/12.png", "/static/images/3.png", "/static/images/0.png", "/static/images/19.png", "/static/images/4.png", "/static/images/15.png", "/static/images/18.png", "/static/images/1.png"];
    var imgs = [];
    for (var i = 0; i < img_files.length; i++){
      imgs[i] = document.createElement('img');
      imgs[i].src = img_files[i]; 
      imgs[i].style.display = 'block';
      imgs[i].style.width = '10px';
      imgs[i].style.height = '10px';
      document.getElementById('box').appendChild(imgs[i]);
    }

    const max_pos = self.innerWidth;
    function anim(elem, pos, dis){
      function move() {
        pos += dis;
        if (pos > max_pos) {
          pos = 0;
        }
        elem.style.transform = `translateX(${pos}px)`;
        requestAnimationFrame(move);
      }
      move();
    }

    for(var i = 0; i < 20; i++){  // i: 이미지 배열의 인덱스 (0 ~ 19)
      anim(imgs[i], 0, Math.random()*60+20); // Math.random()*60+20으로 dis에 랜덤한 인자를 전달하고 있음
    }
  </script>
    ```

<br/><br/>

### 익스플로잇
1. 브라우저 개발자 도구의 Console 탭에서 아래의 자바스크립트 코드를 입력함
    ```javascript
    for (var i = 0; i < 20; i++) {
        anim(img[i], 0, 1); // dis의 인자로 1을 주어 랜덤한 거리를 이동하는 게 아니라 1씩 이동하게 만듦
    }
    ```

<br/>

2. 해당 자바스크립트 코드를 실행하면 이미지 파일이 위에서부터 순서대로 한 열에 출력되며 해당 열 전체가 한꺼번에 움직이고 있는 것을 확인할 수 있음 <br/> &nbsp;&nbsp;&nbsp;&nbsp; → 이를 캡쳐하여 문제의 설명에서 나온 조건에 맞게 하나씩 조합하면 FLAG(```DH{...}```)를 획득할 수 있음
   <br/><br/><img width="1000" alt="플래그 획득" src="https://github.com/augustf86/Today_I_Learn/assets/122844932/534a5809-a17e-432f-9fb1-d729a8632e9e"><br/>
