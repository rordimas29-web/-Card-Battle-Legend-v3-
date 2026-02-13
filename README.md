<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>🔥 Card Battle Legend V3 🔥</title>

<style>
body{
  margin:0;
  font-family:Arial;
  background:linear-gradient(135deg,#141e30,#243b55);
  color:white;
  text-align:center;
  overflow-x:hidden;
}

/* Tombol */
button{
  padding:10px 20px;
  margin:10px;
  border:none;
  border-radius:10px;
  background:orange;
  color:white;
  font-weight:bold;
  cursor:pointer;
  transition:0.3s;
}
button:hover{transform:scale(1.1);}
.hidden{display:none;}

/* Overlay Cara Bermain */
#instructionsOverlay{
  position:fixed;
  top:0; left:0; width:100vw; height:100vh;
  background:rgba(0,0,0,0.85);
  display:flex; justify-content:center; align-items:center;
  flex-direction:column;
  font-size:22px;
  text-align:center;
  z-index:1000;
}
#instructionsOverlay button{
  margin-top:20px;
  padding:10px 20px;
  font-size:20px;
  cursor:pointer;
  border:none;
  border-radius:8px;
  background:orange; color:white;
}

/* MODE TEBAK */
#mode1{
  min-height:400px;
  background:radial-gradient(circle at top,#1f4037,#0f2027);
  padding:20px;
  border-radius:20px;
  margin:20px;
}

.scene{
  display:flex;
  justify-content:center;
  margin-top:20px;
}

.card{
  width:90px;
  height:140px;
  perspective:1000px;
  margin:10px;
  transition:0.3s;
}
.card:hover{
  transform:scale(1.1);
  box-shadow:0 0 25px gold;
}

.card-inner{
  width:100%;
  height:100%;
  position:relative;
  transform-style:preserve-3d;
  transition:0.6s;
}
.card.flip .card-inner{transform:rotateY(180deg);}

.card-front,.card-back{
  position:absolute;
  width:100%;
  height:100%;
  border-radius:15px;
  backface-visibility:hidden;
  display:flex;
  justify-content:center;
  align-items:center;
  font-size:26px;
  font-weight:bold;
}

.card-front{
  background:linear-gradient(145deg,#ffffff,#dddddd);
  color:black;
  transform:rotateY(180deg);
}

.card-back{
  background:repeating-linear-gradient(
    45deg,#b30000,#b30000 10px,#ff1a1a 10px,#ff1a1a 20px
  );
  color:white;
}
.card-back::after{content:"♠ BATTLE ♠";}

/* Flash */
.flashWin{animation:winFlash 0.6s;}
.flashLose{animation:loseFlash 0.6s;}
@keyframes winFlash{0%{background:lime;}100%{background:none;}}
@keyframes loseFlash{0%{background:red;}100%{background:none;}}

/* MODE TEPUK */
#mode2{margin:20px;}

#barContainer{
  width:300px;
  height:20px;
  background:red;
  margin:20px auto;
  border-radius:10px;
  position:relative;
}

#greenZone{
  position:absolute;
  left:110px;
  width:80px;
  height:20px;
  background:lime;
  border-radius:10px;
}

#slider{
  position:absolute;
  width:10px;
  height:20px;
  background:white;
}

.stack{
  position:relative;
  width:100px;
  height:140px;
  margin:20px auto;
}

.stackCard{
  position:absolute;
  width:100px;
  height:140px;
  border-radius:15px;
  background:repeating-linear-gradient(
    45deg,#b30000,#b30000 10px,#ff1a1a 10px,#ff1a1a 20px
  );
  box-shadow:0 5px 15px rgba(0,0,0,0.5);
}

.fall{animation:fallAnim 0.8s forwards;}
@keyframes fallAnim{
  to{transform:translateY(300px) rotate(360deg);opacity:0;}
}

/* Shake */
@keyframes shake{
0%{transform:translateX(0);}
25%{transform:translateX(-10px);}
50%{transform:translateX(10px);}
75%{transform:translateX(-10px);}
100%{transform:translateX(0);}
}
</style>
</head>
<body>

<!-- Overlay Cara Bermain -->
<div id="instructionsOverlay">
<h1>🎮 Cara Bermain 🔥</h1>
<p>Mode Tebak Kartu: Pilih kartu yang lebih besar dari 2 kartu lain.<br>
Salah pilih → tombol Restart muncul.<br>
Benar pilih → lanjut kartu baru.</p>
<p>Mode Tepuk Kartu: Klik kartu pas masuk ke zona hijau.</p>
<button onclick="closeInstructions()">× Tutup</button>
</div>

<h1>🔥 CARD BATTLE LEGEND V3 🔥</h1>
<h3>Kartu Kamu: <span id="cardCount">17</span></h3>
<h4>Level Tebak: <span id="levelTebak">1</span></h4>
<h4>Level Tepuk: <span id="levelTepuk">1</span></h4>

<button onclick="showMode(1)">Mode Tebak</button>
<button onclick="showMode(2)">Mode Tepuk</button>

<div id="mode1">
<div class="scene" id="cards"></div>
<p id="result1"></p>
</div>

<div id="mode2" class="hidden">
<div class="stack" id="stack"></div>
<div id="barContainer">
<div id="greenZone"></div>
<div id="slider"></div>
</div>
<button onclick="hit()">TEPUK!</button>
<p id="result2"></p>
</div>

<!-- Musik & Sound -->
<audio id="bgMusic" src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3" loop autoplay></audio>

<script>
/* ===== SOUND SIMPLE ===== */
let audioCtx=new (window.AudioContext||window.webkitAudioContext)();
function playSound(freq,duration){
  let osc=audioCtx.createOscillator();
  let gain=audioCtx.createGain();
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  osc.frequency.value=freq;
  osc.type="square";
  osc.start();
  gain.gain.setValueAtTime(0.2,audioCtx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001,audioCtx.currentTime+duration);
  osc.stop(audioCtx.currentTime+duration);
}

/* ===== GLOBAL ===== */
let playerCards=17;
let values=[];
let levelTebak=1;
let levelTepuk=1;
let slider=document.getElementById("slider");
let pos=0;
let direction=1;
let interval;
let speed=5;

let levelTebakSpan=document.getElementById("levelTebak");
let levelTepukSpan=document.getElementById("levelTepuk");
let cardCount=document.getElementById("cardCount");

function updateUI(){
  cardCount.innerText=playerCards;
  levelTebakSpan.innerText=levelTebak;
  levelTepukSpan.innerText=levelTepuk;
}

/* Tutup overlay */
function closeInstructions(){
  document.getElementById("instructionsOverlay").remove();
}

function showMode(m){
  mode1.classList.add("hidden");
  mode2.classList.add("hidden");
  document.getElementById("mode"+m).classList.remove("hidden");
  if(m===2){createStack();startBar();}
}

/* ===== MODE TEBAK ===== */
function createCards(){
  values=[];
  cards.innerHTML="";
  let maxNumber=5+(levelTebak*2);
  if(maxNumber>25) maxNumber=25;

  for(let i=0;i<3;i++){
    let val=Math.floor(Math.random()*maxNumber)+1;
    values.push(val);
    let card=document.createElement("div");
    card.className="card";
    card.innerHTML=`
      <div class="card-inner">
        <div class="card-front">${val}</div>
        <div class="card-back"></div>
      </div>`;
    card.onclick=function(){flipCard(card,val)};
    cards.appendChild(card);
  }
}

function flipCard(card,val){
  playSound(400,0.1);
  card.classList.add("flip");
  setTimeout(()=>{checkWin(val)},600);
}

function checkWin(selected){
  let win=true;
  for(let v of values){
    if(v!==selected && selected<=v) win=false;
  }

  // HAPUS reset button lama dulu
  let oldBtn = document.getElementById("resetBtn");
  if(oldBtn) oldBtn.remove();

  if(win){
    playerCards+=13;
    levelTebak++;
    playSound(900,0.2);
    result1.innerText="🔥 MENANG! LEVEL NAIK!";
    mode1.classList.add("flashWin");
    setTimeout(()=>{
      createCards();
      result1.innerText="";
    },800);
  } else {
    playerCards--;
    playSound(150,0.4);
    result1.innerText="💀 KALAH!";
    mode1.classList.add("flashLose");

    // bikin tombol restart cuma 1
    let btn=document.createElement("button");
    btn.id="resetBtn";
    btn.innerText="RESTART MODE TEBAK";
    btn.onclick=function(){
      levelTebak=1;
      createCards();
      result1.innerText="";
      this.remove();
    };
    mode1.appendChild(btn);
  }

  setTimeout(()=>{mode1.classList.remove("flashWin","flashLose");},600);
  updateUI();
}

/* ===== MODE TEPUK ===== */
function createStack(){
  stack.innerHTML="";
  for(let i=0;i<levelTepuk+2;i++){
    let c=document.createElement("div");
    c.className="stackCard";
    c.style.top=(i*4)+"px";
    c.style.left=(i*2)+"px";
    stack.appendChild(c);
  }
  let size=80-(levelTepuk*5);
  if(size<25) size=25;
  greenZone.style.width=size+"px";
  speed=5+(levelTepuk*1.5);
}

function startBar(){
  pos=0;direction=1;
  clearInterval(interval);
  interval=setInterval(()=>{
    pos+=speed*direction;
    if(pos>=290||pos<=0) direction*=-1;
    slider.style.left=pos+"px";
  },20);
}

function hit(){
  let left=110;
  let right=left+greenZone.offsetWidth;

  // hapus reset button lama dulu
  let oldBtn = document.getElementById("resetBtn");
  if(oldBtn) oldBtn.remove();

  if(pos>=left&&pos<=right){
    playSound(1000,0.2);
    result2.innerText="🔥 PAS! LEVEL NAIK!";
    let stackCards=document.querySelectorAll(".stackCard");
    stackCards.forEach(c=>c.classList.add("fall"));
    levelTepuk++;
    playerCards+=5;
    updateUI();
    setTimeout(()=>{createStack();startBar();},800);
  }else{
    playSound(100,0.5);
    result2.innerText="💀 GAGAL!";
    playerCards--;
    updateUI();
    clearInterval(interval);
    document.body.style.animation="shake 0.5s";
    setTimeout(()=>{document.body.style.animation="";},500);

    let btn=document.createElement("button");
    btn.id="resetBtn";
    btn.innerText="RESTART LEVEL";
    btn.onclick=function(){
      result2.innerText="";
      this.remove();
      createStack();
      startBar();
    };
    mode2.appendChild(btn);
  }
}

updateUI();
createCards();

</script>
</body>
</html>
