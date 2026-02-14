# genshin-game
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gacha Impact</title>

<style>
body{
 margin:0;
 font-family:Arial;
 background:#0e0f1a;
 color:white;
 text-align:center;
}

.screen{display:none;padding:10px;}
.active{display:block;}

button{
 padding:10px 14px;
 margin:6px;
 border:none;
 border-radius:10px;
 background:#6c5cff;
 color:white;
}

/* Баннер */
.banner{
 width:95%;
 max-width:420px;
 border-radius:12px;
 margin:10px;
}

/* Анимация */
#wishAnim{
 position:fixed;
 inset:0;
 background:black;
 display:none;
 align-items:center;
 justify-content:center;
 flex-direction:column;
}

.ray{
 width:12px;
 height:220px;
 background:white;
 animation:ray 1s linear infinite;
}

@keyframes ray{
 0%{transform:scaleY(0)}
 100%{transform:scaleY(1.4)}
}

/* Арена */
#arena{
 width:360px;
 height:500px;
 background:url("https://i.imgur.com/7b1Yq9B.jpg");
 background-size:cover;
 margin:auto;
 position:relative;
 border:2px solid white;
}

.char,.enemy{
 position:absolute;
 width:90px;
}

.hpBar{
 width:260px;
 height:16px;
 background:#333;
 margin:auto;
 border-radius:10px;
 overflow:hidden;
}

.hpFill{height:100%;background:lime;}
.enemyHP{background:red;}
</style>
</head>
<body>

<!-- ВЫБОР -->
<div id="select" class="screen active">

<h2>Выбери персонажа</h2>

<img width="140"
src="https://i.imgur.com/WP6Z6xV.png"
onclick="pickTwin('Люмин',this.src)">

<img width="140"
src="https://i.imgur.com/6X4q8wS.png"
onclick="pickTwin('Итер',this.src)">

</div>

<!-- МЕНЮ -->
<div id="menu" class="screen">

<h3>Примогемы: <span id="gems">1600</span> 💎</h3>

<button onclick="openScreen('banners')">Баннеры</button>
<button onclick="openScreen('inventory')">Персонажи / Оружие</button>
<button onclick="startFight()">Бой</button>

</div>

<!-- БАННЕРЫ -->
<div id="banners" class="screen">

<h2>Баннеры</h2>

<img class="banner"
src="https://i.imgur.com/k4KpK5F.jpg">

<img class="banner"
src="https://i.imgur.com/W6d7X6p.jpg">

<img class="banner"
src="https://i.imgur.com/8K9sK8R.jpg">

<img class="banner"
src="https://i.imgur.com/F9z3YpL.jpg">

<br>

<button onclick="wish(1)">1 крутка</button>
<button onclick="wish(10)">10 круток</button>
<button onclick="wish(19)">19 круток</button>

<br>

<button onclick="openScreen('history')">История</button>
<button onclick="openScreen('menu')">Назад</button>

</div>

<!-- АНИМАЦИЯ -->
<div id="wishAnim">
 <div id="ray" class="ray"></div>
 <h2 id="wishText"></h2>
</div>

<!-- ИСТОРИЯ -->
<div id="history" class="screen">
<h2>История</h2>
<div id="historyList"></div>
<button onclick="openScreen('banners')">Назад</button>
</div>

<!-- ИНВЕНТАРЬ -->
<div id="inventory" class="screen">
<h2>Мои персонажи и оружия</h2>
<div id="inv"></div>
<button onclick="openScreen('menu')">Назад</button>
</div>

<!-- БОЙ -->
<div id="fight" class="screen">

Игрок HP
<div class="hpBar"><div id="pHP" class="hpFill"></div></div>

Враг HP
<div class="hpBar"><div id="eHP" class="hpFill enemyHP"></div></div>

<div id="arena"></div>

<button onclick="atk()">Атака</button>
<button onclick="skill()">Ешка</button>
<button onclick="ult()">Ульта</button>

<br>
<button onclick="openScreen('menu')">Сбежать</button>

</div>

<script>
let gems=1600;
let twin=null;
let history=[];
let inventory=[];
let pity4=0;

/* ЭКРАН */
function openScreen(id){
 document.querySelectorAll(".screen")
 .forEach(s=>s.classList.remove("active"));
 document.getElementById(id).classList.add("active");
}

/* ВЫБОР */
function pickTwin(name,img){
 twin={name,img,atk:8};
 inventory.push(twin);
 openScreen("menu");
}

/* КРУТКА */
function wish(count){

 if(gems<160*count){
  alert("Нет гемов");
  return;
 }

 for(let i=0;i<count;i++) roll();

 updateUI();
}

/* РОЛЛ */
function roll(){

 gems-=160;
 pity4++;

 let rarity="3★";

 if(pity4>=10){
  rarity="4★";
  pity4=0;
 }
 else if(Math.random()<0.05){
  rarity="5★";
 }

 showAnim(rarity);

 if(rarity=="5★"){
  inventory.push({name:"Легендарный персонаж"});
 }
 else if(rarity=="4★"){
  inventory.push({name:"4★ оружие"});
 }
 else{
  inventory.push({name:"3★ оружие"});
 }

 history.push(rarity);
}

/* АНИМАЦИЯ */
function showAnim(rarity){

 let anim=document.getElementById("wishAnim");
 let ray=document.getElementById("ray");
 let text=document.getElementById("wishText");

 anim.style.display="flex";

 if(rarity=="5★") ray.style.background="gold";
 else if(rarity=="4★") ray.style.background="violet";
 else ray.style.background="white";

 text.innerText=rarity;

 setTimeout(()=>{
  anim.style.display="none";
 },900);
}

/* UI */
function updateUI(){

 document.getElementById("gems").innerText=gems;

 document.getElementById("historyList").innerHTML=
 history.slice().reverse().join("<br>");

 document.getElementById("inv").innerHTML=
 inventory.map(i=>i.name).join("<br>");
}

/* БОЙ */
let player,enemy;

function startFight(){

 openScreen("fight");

 player={hp:100,max:100};
 enemy={hp:120,max:120};

 arena.innerHTML=`
 <img class="char"
 src="${twin.img}"
 style="left:140px;top:350px">

 <img class="enemy"
 src="https://i.imgur.com/Z6X9X9s.png"
 style="left:120px;top:80px">
 `;

 loop();
}

/* КНОПКИ */
function atk(){enemy.hp-=10;}
function skill(){enemy.hp-=18;}
function ult(){enemy.hp-=30;}

/* ИИ ВРАГА */
setInterval(()=>{
 if(enemy) player.hp-=6;
},1500);

/* ЦИКЛ */
function loop(){

 pHP.style.width=(player.hp/player.max*100)+"%";
 eHP.style.width=(enemy.hp/enemy.max*100)+"%";

 if(enemy.hp<=0){
  gems+=40;
  alert("Победа +40 💎");
  openScreen("menu");
  updateUI();
  return;
 }

 if(player.hp<=0){
  alert("Поражение");
  openScreen("menu");
  return;
 }

 requestAnimationFrame(loop);
}
</script>
</body>
</html>