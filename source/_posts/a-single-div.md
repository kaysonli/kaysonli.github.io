---
title: 只用 div 和 css 能玩出什么花来？请扶好下巴！
date: 2020-07-03 11:12:00
tags: CSS
---
HTML，CSS 和 JavaScript 是 Web 页面的“三驾马车”，分别负责网页的结构、样式和行为，共同组成了丰富多彩的 Web 世界。一个完整的网页，通常少不了其中任何一个。

提到网页绘图，我们往往会想到用 canvas、svg 或者 WebGL，甚至直接用各种图片格式。但是，你想过只用 HTML 和 CSS 能实现多复杂的视觉效果吗？做过几年前端开发就知道，CSS 3 确实很强大，能实现很多意想不到的效果。最近在网上看到一位大神的作品，更是惊掉了下巴：只用一个 `div` 元素，加上 CSS 代码，就实现了很多惊艳的图形和动画！

让我们一起来膜拜学习吧！
![特斯拉电动卡车](https://s1.ax1x.com/2020/07/06/UC3Hdx.md.png)
<!-- more -->
HTML 代码：
```
<div class="entry wide" id="cybertruck"><div></div></div>
```
CSS 代码：
```
#cybertruck {
    background-color: black;
    background-image: -webkit-gradient(linear,left bottom, left top,color-stop(32%, #0c0e11),color-stop(32%, black));
    background-image: linear-gradient(to top,#0c0e11 32%,black 32%);
}

#cybertruck div {
    width: 650px;
    height: 210px;
    margin-left: -265px;
    margin-top: -105px;
    background-repeat: no-repeat;
    background-image: linear-gradient(161deg,transparent 11%,#1d232a 11.5%),linear-gradient(161deg,transparent 50%,#555 50%),linear-gradient(to right,#1d232a,#1d232a),radial-gradient(circle at center,#ff0 8%,#f00 30%,transparent 80%),linear-gradient(to right,transparent,rgba(255,0,0,0.6) 30%,rgba(255,0,0,0.6) 70%,transparent),linear-gradient(to bottom,transparent,rgba(255,0,0,0.6) 30%,rgba(255,0,0,0.6) 70%,transparent),radial-gradient(circle at center bottom,transparent 45%,rgba(255,255,255,0.1) 45%,rgba(255,255,255,0.1) 50%,transparent 50%),linear-gradient(to right,rgba(255,255,255,0.1),rgba(255,255,255,0.1)),linear-gradient(40deg,transparent 40%,rgba(255,255,255,0.1) 41%,rgba(255,255,255,0.1) 44%,transparent 45%),linear-gradient(-40deg,transparent 40%,rgba(255,255,255,0.1) 41%,rgba(255,255,255,0.1) 44%,transparent 45%),linear-gradient(320deg,transparent 40%,rgba(0,0,0,0.7) 41%,rgba(0,0,0,0.7) 44%,transparent 45%),linear-gradient(-320deg,transparent 40%,rgba(0,0,0,0.7) 41%,rgba(0,0,0,0.7) 44%,transparent 45%),radial-gradient(circle,#0f1215 50%,transparent 50%),radial-gradient(circle at center top,rgba(255,255,255,0.1) 50%,transparent 50%),radial-gradient(circle at center top,rgba(0,0,0,0.4) 50%,transparent 50%),radial-gradient(circle at center bottom,rgba(255,255,255,0.1) 50%,transparent 50%),radial-gradient(circle,rgba(0,0,0,0.3) 50%,transparent 50%),radial-gradient(circle,#1d232a 50%,transparent 50%),linear-gradient(70deg,transparent 10%,#4a596b 11%),linear-gradient(-60deg,transparent 10%,#4a596b 11%),linear-gradient(127deg,transparent 40%,rgba(0,0,0,0.2) 50%,#333d49 51%,#333d49 62%,black 63%),linear-gradient(-125deg,transparent 35%,rgba(0,0,0,0.2) 45%,#333d49 46%,#333d49 57%,black 58%),linear-gradient(to right,black,black),radial-gradient(circle at center bottom,transparent 45%,rgba(255,255,255,0.1) 45%,rgba(255,255,255,0.1) 50%,transparent 50%),linear-gradient(to right,rgba(255,255,255,0.1),rgba(255,255,255,0.1)),linear-gradient(40deg,transparent 40%,rgba(255,255,255,0.1) 41%,rgba(255,255,255,0.1) 44%,transparent 45%),linear-gradient(-40deg,transparent 40%,rgba(255,255,255,0.1) 41%,rgba(255,255,255,0.1) 44%,transparent 45%),linear-gradient(320deg,transparent 40%,rgba(0,0,0,0.7) 41%,rgba(0,0,0,0.7) 44%,transparent 45%),linear-gradient(-320deg,transparent 40%,rgba(0,0,0,0.7) 41%,rgba(0,0,0,0.7) 44%,transparent 45%),radial-gradient(circle,#0f1215 50%,transparent 50%),radial-gradient(circle at center top,rgba(255,255,255,0.1) 50%,transparent 50%),radial-gradient(circle at center top,rgba(0,0,0,0.4) 50%,transparent 50%),radial-gradient(circle at center bottom,rgba(255,255,255,0.1) 50%,transparent 50%),radial-gradient(circle,rgba(0,0,0,0.3) 50%,transparent 50%),radial-gradient(circle,#1d232a 50%,transparent 50%),linear-gradient(60deg,transparent 10%,#4a596b 11%),linear-gradient(-60deg,transparent 10%,#4a596b 11%),linear-gradient(127deg,transparent 32%,rgba(0,0,0,0.2) 42%,#333d49 43%,#333d49 53%,black 54%),linear-gradient(-125deg,transparent 22%,rgba(0,0,0,0.2) 32%,#1d232a 33%,#1d232a 43%,black 44%),linear-gradient(to right,black,black),linear-gradient(89deg,transparent 50%,black 51%,black 53%,transparent 54%),linear-gradient(170deg,transparent 50%,black 50%),linear-gradient(100deg,transparent 50%,black 51%,black 52%,transparent 53%),linear-gradient(90deg,transparent 50%,black 51%,black 52%,transparent 53%),linear-gradient(to right,rgba(255,255,255,0.05),rgba(255,255,255,0)),linear-gradient(98deg,transparent 55%,black 56%),linear-gradient(176deg,transparent 32%,#1d232a 33%),linear-gradient(179deg,#1d232a 85%,black 86%),linear-gradient(179deg,transparent 40%,#1d232a 45%,#1d232a 50%,transparent 55%),linear-gradient(105deg,transparent 50%,#1d232a 51%,#1d232a 53%,transparent 54%),linear-gradient(100deg,transparent 50%,#1d232a 51%,#1d232a 53%,transparent 54%),linear-gradient(81deg,transparent 50%,#1d232a 51%,#1d232a 53%,transparent 54%),linear-gradient(81deg,transparent 38%,black 40%,black 49%,#ddd 51%),linear-gradient(to right,rgba(0,0,0,0.3),transparent),linear-gradient(to left,rgba(0,0,0,0.3),transparent),linear-gradient(161deg,black 50%,#ddd 51%,#ddd 55%,transparent 56%),linear-gradient(-174deg,black 33%,#ddd 34%,#ddd 40%,transparent 41%),linear-gradient(to right,#ddd,#ddd),linear-gradient(176deg,transparent 34%,black 35%,black 45%,#ddd 46%,#ddd 60%,transparent 60%),linear-gradient(161deg,black 56%,transparent 57%),linear-gradient(-174deg,black 41%,transparent 42%),linear-gradient(105deg,transparent 30%,rgba(255,255,255,0.05) 31%,rgba(255,255,255,0.05) 40%,rgba(0,0,0,0.6) 41%,rgba(0,0,0,0.6) 45%,rgba(255,255,255,0.1) 46%,rgba(255,255,255,0.1) 55%,transparent 56%),linear-gradient(103deg,transparent 30%,rgba(255,255,255,0.1) 31%,rgba(255,255,255,0.1) 34%,rgba(0,0,0,0.3) 35%,rgba(0,0,0,0.3) 38%,transparent 39%),linear-gradient(93deg,transparent 29%,rgba(0,0,0,0.2) 30%,rgba(0,0,0,0.2) 33%,rgba(255,255,255,0.1) 34%,rgba(255,255,255,0.1) 50%,transparent 51%),linear-gradient(to right,rgba(0,0,0,0.3) 20%,rgba(255,255,255,0.3) 21%),linear-gradient(to top,rgba(255,255,255,0.9) 50%,rgba(255,255,255,0) 65%),linear-gradient(to right,rgba(255,255,255,0),rgba(255,255,255,0.3) 60%,rgba(255,255,255,0.7));
    background-size: 20px 45px,5px 5px,26px 10px,12px 12px,55px 1px,1px 55px,130px 65px,10px 7px,16px 16px,16px 16px,16px 16px,16px 16px,20px 20px,24px 12px,34px 16px,34px 16px,90px 90px,130px 130px,40px 6px,40px 6px,35px 38px,40px 51px,78px 60px,130px 65px,10px 7px,16px 16px,16px 16px,16px 16px,16px 16px,20px 20px,24px 12px,34px 16px,34px 16px,90px 90px,130px 130px,40px 9px,40px 9px,60px 58px,60px 55px,75px 60px,30px 80px,150px 30px,30px 80px,30px 95px,45px 45px,20px 120px,600px 87px,500px 50px,254px 20px,30px 30px,30px 30px,30px 30px,25px 22px,200px 100px,200px 100px,230px 80px,370px 80px,220px 60px,550px 50px,230px 80px,370px 80px,30px 80px,30px 80px,30px 80px,10px 80px,7px 80px,350px 60px;
    background-position: 5px 80px,0 125px,0 128px,right 25px top 35px,right top 40px,right 30px top 15px,25px 90px,85px 123px,68px 130px,96px 130px,68px 160px,96px 160px,80px 145px,78px 156px,73px 156px,73px 140px,45px 110px,25px 90px,50px 89px,90px 89px,20px 89px,124px 89px,52px 89px,435px 90px,495px 123px,478px 130px,506px 130px,478px 160px,506px 160px,490px 145px,488px 156px,483px 156px,483px 140px,455px 110px,435px 90px,460px 78px,500px 78px,416px 78px,512px 78px,465px 78px,150px 70px,500px 115px,250px 65px,380px 55px,25px 80px,right 25px top 15px,25px 39px,115px 90px,165px 145px,148px 60px,257px 52px,380px 42px,380px 23px,25px 0,right 25px top 0,25px 0,right 25px top 0,right 25px top 20px,50px 29px,25px 0,right 25px top 0,170px 30px,260px 10px,277px 10px,382px 10px,375px 0,50px 0;
}
```
![香蕉](https://s1.ax1x.com/2020/07/06/UC8T1g.md.png)
HTML 代码：
```
<div class="entry wide" id="ripe"><div></div></div>
```
CSS 代码：
```
#ripe {
    background-color: #222;
    overflow: hidden;
}

#ripe div {
    width: 200px;
    height: 220px;
    margin-left: 100px;
    margin-top: -110px;
    border-radius: 50%;
    -webkit-box-shadow: -35px 3px 0 -8px rgba(0,0,0,0.1),-60px 0 0 #90ee90,-85px 3px 0 -5px rgba(0,0,0,0.1),-110px 0 0 #adff2f,-135px 3px 0 -5px rgba(0,0,0,0.1),-160px 0 0 #ff0,-185px 3px 0 -5px rgba(0,0,0,0.1),-210px 0 0 #ffd700,-235px 3px 0 -5px rgba(0,0,0,0.1),-258px 0 0 #daa520;
    box-shadow: -35px 3px 0 -8px rgba(0,0,0,0.1),-60px 0 0 #90ee90,-85px 3px 0 -5px rgba(0,0,0,0.1),-110px 0 0 #adff2f,-135px 3px 0 -5px rgba(0,0,0,0.1),-160px 0 0 #ff0,-185px 3px 0 -5px rgba(0,0,0,0.1),-210px 0 0 #ffd700,-235px 3px 0 -5px rgba(0,0,0,0.1),-258px 0 0 #daa520;
}
```
还有更绝的：麻将！
![麻将](https://s1.ax1x.com/2020/07/06/UC8W7t.md.png)
HTML 代码：
```
<div class="entry wide" id="dragon"><div></div></div>
```
CSS 代码：
```
#dragon div:before,#dragon div:after {
    width: 120px;
    height: 160px;
    background-color: white;
    background-repeat: no-repeat;
    border-radius: 7px;
    -webkit-box-shadow: 0 15px 5px rgba(0,0,0,0.05),0 30px 0 #eee,0 35px 5px rgba(0,0,0,0.05),0 40px 0 #00fa9a,0 45px 0 rgba(0,0,0,0.1);
    box-shadow: 0 15px 5px rgba(0,0,0,0.05),0 30px 0 #eee,0 35px 5px rgba(0,0,0,0.05),0 40px 0 #00fa9a,0 45px 0 rgba(0,0,0,0.1)
}

#dragon div:before {
    left: -140px;
    background-image: linear-gradient(to right,#191970 7%,transparent 7%,transparent 93%,#191970 93%),linear-gradient(to bottom,#191970 5%,transparent 5%,transparent 95%,#191970 95%),linear-gradient(to right,#191970 7%,transparent 7%,transparent 93%,#191970 93%),linear-gradient(to bottom,#191970 5%,transparent 5%,transparent 95%,#191970 95%),linear-gradient(-45deg,transparent 10%,#191970 10%,#191970 13%,transparent 13%,transparent 87%,#191970 87%,#191970 90%,transparent 90%),linear-gradient(45deg,transparent 10%,#191970 10%,#191970 13%,transparent 13%,transparent 87%,#191970 87%,#191970 90%,transparent 90%);
    background-size: 70% 75%,70% 75%,52% 62%,52% 62%,52% 62%,52% 62%;
    background-position: center center
}

#dragon div:after {
    right: -140px;
    background-image: linear-gradient(-30deg,#006400 40%,transparent 40%),linear-gradient(-30deg,#006400 40%,transparent 40%),linear-gradient(110deg,white 50%,transparent 50%),linear-gradient(120deg,transparent 45%,#006400 45%,#006400 57%,transparent 57%),linear-gradient(55deg,transparent 35%,#006400 35%,#006400 50%,transparent 50%),linear-gradient(-45deg,transparent 30%,#006400 30%,#006400 50%,transparent 50%),linear-gradient(-30deg,white 40%,transparent 40%),linear-gradient(-45deg,transparent 30%,#006400 30%,#006400 50%,transparent 50%),linear-gradient(50deg,transparent 40%,white 40%),linear-gradient(60deg,transparent 40%,#006400 40%,#006400 60%,transparent 60%),linear-gradient(-22deg,transparent 20%,#006400 20%,#006400 45%,transparent 45%),linear-gradient(-20deg,transparent 20%,#006400 20%,#006400 45%,transparent 45%),radial-gradient(circle at bottom left,transparent 50%,white 50%),linear-gradient(-16deg,transparent 10%,#006400 10%,#006400 35%,transparent 35%),linear-gradient(110deg,transparent 25%,#006400 25%,#006400 50%,transparent 50%),linear-gradient(60deg,white 50%,transparent 50%),linear-gradient(45deg,#006400 60%,transparent 60%),linear-gradient(90deg,#006400,#006400),linear-gradient(100deg,transparent 10%,#006400 10%,#006400 30%,transparent 30%),linear-gradient(20deg,#006400 30%,transparent 30%),linear-gradient(100deg,transparent 5%,#006400 5%,#006400 27%,transparent 27%),linear-gradient(45deg,#006400 66%,transparent 66%),linear-gradient(-35deg,white 35%,transparent 35%),linear-gradient(-48deg,transparent 40%,#006400 40%,#006400 60%,transparent 60%),linear-gradient(30deg,transparent 30%,white 30%),linear-gradient(40deg,transparent 40%,#006400 40%,#006400 60%,transparent 60%);
    background-size: 15px 15px,12px 20px,30px 40px,40px 50px,20px 20px,20px 20px,20px 10px,30px 20px,50px 50px,50px 40px,18px 20px,18px 20px,10px 10px,22px 20px,20px 25px,20px 20px,20px 20px,7px 5px,20px 18px,10px 20px,20px 20px,35px 6px,20px 20px,30px 30px,30px 35px,30px 30px;
    background-position: 36% 25%,25% 37%,15% 50%,17% 45%,67% 28%,51% 28%,48% 41%,51% 36%,83% 35%,79% 43%,37% 45%,35% 51%,43% 56%,33% 54%,39% 69%,23% 72%,25% 72%,56% 47%,67% 51%,63% 51%,58% 52%,62% 61%,47% 77%,54% 75%,62% 65%,58% 72%
}
```
还有更多的图案和动效，这里就不一一贴出来了。光看这几个，就不得不佩服作者的想象力和 CSS 功夫了。这位大神叫  [Lynn Fisher](https://twitter.com/lynnandtonic)，这个项目 Demo 网站是[https://a.singlediv.com/](https://a.singlediv.com/)，正如域名的含义：只用一个 `div`。是不是很神奇？

在图形绘制方面跟 SVG 相比，这样的实现方式虽然不太实用，而且还可能有浏览器兼容问题。但是不得不说，这样的思路确实很清奇，可以看出 CSS 的特性和潜力。