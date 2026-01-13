<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<title>La stanza delle necessità</title>

<style>
body{
    margin:0;
    background:radial-gradient(circle at top,#111,#000);
    color:white;
    text-align:center;
    overflow:hidden;
    font-family:Arial,sans-serif;
}

/* ---------- MENU ---------- */
#menu{
    position:absolute;
    inset:0;
    display:flex;
    flex-direction:column;
    justify-content:center;
    align-items:center;
}

#menu h1{
    font-size:60px;
    letter-spacing:4px;
    color:#cfa;
    text-shadow:0 0 30px #0f0;
    margin-bottom:30px;
}

#playerInput{
    padding:10px;
    font-size:18px;
    margin-bottom:20px;
    border-radius:6px;
    border:none;
}

#grid{
    display:grid;
    grid-template-columns:repeat(2,180px);
    gap:25px;
}

.tile{
    height:120px;
    border:2px solid #0ff;
    box-shadow:0 0 20px #0ff;
    display:flex;
    justify-content:center;
    align-items:center;
    cursor:pointer;
    font-size:22px;
    background:rgba(0,0,0,.6);
    transition:.3s;
}

.tile:hover{
    transform:scale(1.08);
}

#credit{
    position:absolute;
    bottom:15px;
    opacity:.6;
}

/* ---------- CHAT ---------- */
#chat{
    display:none;
    position:absolute;
    inset:0;
    padding:20px;
}

#chat h2{
    font-size:46px;
    margin-bottom:20px;
    color:#0ff;
    text-shadow:0 0 25px #0ff;
}

#messages{
    height:60%;
    border:2px solid #0ff;
    overflow-y:auto;
    padding:10px;
    margin-bottom:10px;
    text-align:left;
}

/* ---------- HUD ---------- */
#hud{
    display:none;
    position:absolute;
    top:10px;
    width:100%;
    justify-content:space-around;
}

/* ---------- CANVAS ---------- */
canvas{
    display:none;
    margin:100px auto 0;
    background:#000;
    border:3px solid #0ff;
    box-shadow:0 0 25px #0ff;
}

/* ---------- GAME OVER ---------- */
#gameOver{
    display:none;
    position:absolute;
    inset:0;
    background:rgba(0,0,0,.85);
    justify-content:center;
    align-items:center;
    flex-direction:column;
}

#scores{
    margin-top:15px;
    line-height:1.6;
}

/* ---------- CLASSIFICA ---------- */
#leaderboard{
    display:none;
    position:absolute;
    inset:0;
    padding-top:80px;
}

#leaderboard h2{
    font-size:48px;
    color:#0ff;
    text-shadow:0 0 20px #0ff;
}
</style>
</head>

<body>

<!-- MENU -->
<div id="menu">
    <h1>La stanza delle necessità</h1>
    <input id="playerInput" placeholder="Nome giocatore">
    <div id="grid">
        <div class="tile" onclick="openPong()">🏓 Pong</div>
        <div class="tile" onclick="openChat()">✉ Messaggi</div>
        <div class="tile" onclick="openLeaderboard()">🏆 Classifica</div>
        <div class="tile">?</div>
    </div>
    <div id="credit">Creato da Edobe</div>
</div>

<!-- CHAT -->
<div id="chat">
    <h2>Per quelli che vengono dopo</h2>
    <div id="messages"></div>
    <input id="chatName" placeholder="Nome">
    <input id="chatText" placeholder="Messaggio">
    <button onclick="sendMessage()">Invia</button><br><br>
    <button onclick="backToMenu()">⬅ Torna al menù</button>
</div>

<!-- HUD -->
<div id="hud">
    <div>Punteggio: <span id="score">0</span></div>
</div>

<!-- PONG -->
<canvas id="game" width="720" height="420"></canvas>

<!-- GAME OVER -->
<div id="gameOver">
    <h2>💀 GAME OVER</h2>
    <div id="finalScore"></div>
    <div id="scores"></div><br>
    <button onclick="backToMenu()">⬅ Torna al menù</button>
</div>

<!-- LEADERBOARD -->
<div id="leaderboard">
    <h2>Classifica dei Custodi</h2>
    <div id="leaderboardList"></div><br>
    <button onclick="backToMenu()">⬅ Torna al menù</button>
</div>

<script>
/* ---------- UTIL ---------- */
function backToMenu(){
    document.querySelectorAll(
        "#chat,#leaderboard,#gameOver,#hud,canvas"
    ).forEach(e=>e.style.display="none");
    document.getElementById("menu").style.display="flex";
}

/* ---------- CHAT ---------- */
const chatKey="stanzachat";

function openChat(){
    document.getElementById("menu").style.display="none";
    document.getElementById("chat").style.display="block";
    loadMessages();
}

function loadMessages(){
    const box=document.getElementById("messages");
    box.innerHTML="";
    const msgs=JSON.parse(localStorage.getItem(chatKey))||[];
    msgs.forEach(m=>{
        box.innerHTML+=`<div><b>${m.n}</b>: ${m.t}</div>`;
    });
    box.scrollTop=box.scrollHeight;
}

function sendMessage(){
    const n=document.getElementById("chatName").value||"Anonimo";
    const t=document.getElementById("chatText").value.trim();
    if(!t) return;
    const msgs=JSON.parse(localStorage.getItem(chatKey))||[];
    msgs.push({n,t});
    localStorage.setItem(chatKey,JSON.stringify(msgs));
    document.getElementById("chatText").value="";
    loadMessages();
}

/* ---------- PONG + HIGHSCORES ---------- */
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");
const highscoresKey="pong_highscores";

let paddle,ball,score,playing,playerName;
let up=false,down=false;

function openPong(){
    playerName=document.getElementById("playerInput").value||"Anonimo";
    document.getElementById("menu").style.display="none";
    document.getElementById("hud").style.display="flex";
    canvas.style.display="block";

    paddle={x:60,y:170,w:12,h:90,speed:9};
    ball={x:360,y:210,r:8,vx:5,vy:5};
    score=0;
    playing=true;
    document.getElementById("score").textContent=score;

    requestAnimationFrame(loop);
}

document.addEventListener("keydown",e=>{
    if(e.key==="ArrowUp"||e.key==="w") up=true;
    if(e.key==="ArrowDown"||e.key==="s") down=true;
});
document.addEventListener("keyup",e=>{
    if(e.key==="ArrowUp"||e.key==="w") up=false;
    if(e.key==="ArrowDown"||e.key==="s") down=false;
});

function loop(){
    if(!playing) return;
    ctx.clearRect(0,0,canvas.width,canvas.height);

    if(up&&paddle.y>0) paddle.y-=paddle.speed;
    if(down&&paddle.y<canvas.height-paddle.h) paddle.y+=paddle.speed;

    ctx.fillStyle="#0ff";
    ctx.fillRect(paddle.x,paddle.y,paddle.w,paddle.h);

    ctx.beginPath();
    ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
    ctx.fill();

    ball.x+=ball.vx;
    ball.y+=ball.vy;

    if(ball.y<=0||ball.y>=canvas.height) ball.vy*=-1;

    if(
        ball.vx<0 &&
        ball.x-ball.r<=paddle.x+paddle.w &&
        ball.x>paddle.x &&
        ball.y>=paddle.y &&
        ball.y<=paddle.y+paddle.h
    ){
        ball.vx*=-1;
        score++;
        document.getElementById("score").textContent=score;
    }

    if(ball.x<0){
        endGame();
        return;
    }
    if(ball.x>canvas.width) ball.vx*=-1;

    requestAnimationFrame(loop);
}

function endGame(){
    playing=false;
    saveHighscore();
    showGameOver();
}

function saveHighscore(){
    let list=JSON.parse(localStorage.getItem(highscoresKey))||[];
    list.push({name:playerName,score});
    list.sort((a,b)=>b.score-a.score);
    localStorage.setItem(highscoresKey,JSON.stringify(list.slice(0,5)));
}

function showGameOver(){
    document.getElementById("hud").style.display="none";
    canvas.style.display="none";
    document.getElementById("gameOver").style.display="flex";
    document.getElementById("finalScore").textContent="Punteggio finale: "+score;
    renderScores("scores");
}

/* ---------- LEADERBOARD ---------- */
function openLeaderboard(){
    document.getElementById("menu").style.display="none";
    document.getElementById("leaderboard").style.display="block";
    renderScores("leaderboardList");
}

function renderScores(target){
    let list=JSON.parse(localStorage.getItem(highscoresKey))||[];
    let html="<h3>🏆 Highscores</h3>";
    list.forEach((e,i)=>{
        html+=`${i+1}. <b>${e.name}</b> — ${e.score}<br>`;
    });
    document.getElementById(target).innerHTML=html;
}
</script>

</body>
</html>
