<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Escape Prison Obby</title>

<style>
body {
  margin:0;
  font-family:Arial;
  background:#111;
  color:white;
  text-align:center;
}

#gate, #game, #win {
  display:none;
}

button {
  padding:10px 15px;
  margin:5px;
  border:none;
  border-radius:8px;
  background:orange;
  color:white;
}

canvas {
  background:#222;
}

#ui {
  display:flex;
  justify-content:space-around;
  font-size:14px;
}
</style>
</head>

<body>

<!-- TELEGRAM GATE -->
<div id="gate">
  <h2>🔒 Join Channel First</h2>
  
  <a href="https://t.me/robloxcityfighter" target="_blank">
    <button>Join Channel</button>
  </a>

  <br>
  <button onclick="unlockGame()">I Joined</button>
</div>

<!-- GAME -->
<div id="game">
  <div id="ui">
    <span>📶 Data: <span id="score">0</span></span>
    <span>📏 Distance: <span id="distance">0</span></span>
    <span>⏱ Time: <span id="time">0</span></span>
    <span>🎯 Level: <span id="level">1</span></span>
  </div>

  <canvas id="canvas" width="300" height="400"></canvas>

  <div>
    <button onclick="changeSkin()">🎨 Change Skin</button>
  </div>
</div>

<!-- WIN -->
<div id="win">
  <h1>🎉 ESCAPED!</h1>
  <p id="finalStats"></p>
</div>

<script>
let canvas = document.getElementById("canvas");
let ctx = canvas.getContext("2d");

let skins = ["orange","blue","green","purple"];
let skinIndex = 0;

let player = {x:140,y:350,size:20,color:skins[0]};
let obstacles = [];
let coins = [];

let score = 0;
let distance = 0;
let level = 1;
let time = 0;

let gameRunning = false;

function unlockGame(){
  document.getElementById("gate").style.display="none";
  document.getElementById("game").style.display="block";
  startGame();
}

function startGame(){
  gameRunning = true;

  setInterval(()=>{
    if(gameRunning){
      time++;
      document.getElementById("time").innerText=time;
    }
  },1000);

  spawnObjects();
  gameLoop();
}

function spawnObjects(){
  obstacles = [];
  coins = [];

  for(let i=0;i<5+level;i++){
    obstacles.push({x:Math.random()*260,y:Math.random()*400});
    coins.push({x:Math.random()*260,y:Math.random()*400});
  }
}

function gameLoop(){
  if(!gameRunning) return;

  ctx.clearRect(0,0,300,400);

  // player
  ctx.fillStyle = player.color;
  ctx.fillRect(player.x,player.y,player.size,player.size);

  // obstacles
  ctx.fillStyle="red";
  obstacles.forEach(o=>{
    o.y += 2 + level;

    if(o.y > 400){
      o.y = 0;
      o.x = Math.random()*260;
      distance++;
    }

    ctx.fillRect(o.x,o.y,20,20);

    if(collide(player,o)){
      gameOver();
    }
  });

  // coins (data)
  ctx.fillStyle="cyan";
  coins.forEach((c,i)=>{
    c.y += 2 + level;

    if(c.y > 400){
      c.y = 0;
      c.x = Math.random()*260;
    }

    ctx.fillRect(c.x,c.y,10,10);

    if(collide(player,c)){
      score++;
      document.getElementById("score").innerText=score;
      coins.splice(i,1);

      // level up
      if(score % 5 === 0){
        level++;
        document.getElementById("level").innerText=level;
        spawnObjects();
      }
    }
  });

  document.getElementById("distance").innerText=distance;

  // win condition
  if(level >= 5){
    winGame();
    return;
  }

  requestAnimationFrame(gameLoop);
}

function collide(a,b){
  return a.x < b.x+20 &&
         a.x+20 > b.x &&
         a.y < b.y+20 &&
         a.y+20 > b.y;
}

function gameOver(){
  alert("💀 Caught!");
  saveScore();
  location.reload();
}

function winGame(){
  gameRunning = false;
  document.getElementById("game").style.display="none";
  document.getElementById("win").style.display="block";

  saveScore();

  document.getElementById("finalStats").innerText =
    "Time: "+time+"s | Distance: "+distance+" | Data: "+score;
}

function changeSkin(){
  skinIndex = (skinIndex + 1) % skins.length;
  player.color = skins[skinIndex];
}

// leaderboard (local)
function saveScore(){
  let scores = JSON.parse(localStorage.getItem("leaderboard")) || [];
  scores.push({score, time, distance});
  localStorage.setItem("leaderboard", JSON.stringify(scores));
}

document.addEventListener("keydown", e=>{
  if(e.key=="ArrowLeft") player.x -= 20;
  if(e.key=="ArrowRight") player.x += 20;
});

// mobile touch
document.addEventListener("touchstart", e=>{
  let x = e.touches[0].clientX;
  if(x < window.innerWidth/2) player.x -= 20;
  else player.x += 20;
});

document.getElementById("gate").style.display="block";
</script>

</body>
</html>
