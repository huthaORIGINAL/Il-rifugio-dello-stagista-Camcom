<html lang="it">
<head>
<meta charset="UTF-8">
<title>La stanza delle necessità</title>

<style>
/* ---------- BASE ---------- */
body{
    margin:0;
    background: radial-gradient(circle at top, #1a0f0f, #000000);
    color: #f8f1e5;
    text-align:center;
    overflow:hidden;
    font-family: "Georgia", serif;
    position:relative;
}

/* ---------- LUCE SOFFUSA ---------- */
body::after {
    content: '';
    position: fixed;
    top:0; left:0; width:100%; height:100%;
    background: radial-gradient(circle, rgba(255,255,220,0.05) 0%, transparent 70%);
    pointer-events: none;
    animation: flicker 3s infinite;
}

@keyframes flicker {
    0%, 100% { opacity: 0.06; }
    50% { opacity: 0.12; }
}

/* ---------- PARTICELLE FLOTTANTI ---------- */
.particle{
    position:absolute;
    width:4px;
    height:4px;
    background: rgba(245,230,196,0.3);
    border-radius:50%;
    pointer-events:none;
    animation: floatUp linear infinite;
}

@keyframes floatUp{
    0% { transform: translateY(0) scale(1); opacity:0.2; }
    50% { opacity:0.5; }
    100% { transform: translateY(-600px) scale(0.5); opacity:0; }
}

/* ---------- MENU ---------- */
#menu{
    position:absolute;
    inset:0;
    display:flex;
    flex-direction:column;
    justify-content:center;
    align-items:center;
    z-index:1;
}

#menu h1{
    font-size:60px;
    letter-spacing:4px;
    color:#f5e6c4;
    text-shadow:0 0 20px #f5e6c4,0 0 50px rgba(245,230,196,0.2);
    margin-bottom:30px;
}

#playerInput{
    padding:10px;
    font-size:18px;
    margin-bottom:20px;
    border-radius:8px;
    border:1px solid #f5e6c4;
    background: rgba(0,0,0,0.5);
    color:#f5e6c4;
    box-shadow:0 0 8px rgba(245,230,196,0.3);
}

#grid{
    display:grid;
    grid-template-columns:repeat(2,180px);
    gap:25px;
}

.tile{
    height:120px;
    border:2px solid #f5e6c4;
    box-shadow:0 0 15px #f5e6c4,0 0 40px rgba(245,230,196,0.3);
    display:flex;
    justify-content:center;
    align-items:center;
    cursor:pointer;
    font-size:22px;
    background:rgba(0,0,0,.6);
    transition:.3s;
    border-radius:10px;
}

.tile:hover{
    transform: scale(1.05);
    box-shadow:0 0 30px #f5e6c4,0 0 60px rgba(245,230,196,0.4);
}

#credit{
    position:absolute;
    bottom:15px;
    opacity:.6;
    font-size:14px;
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
    color:#f5e6c4;
    text-shadow:0 0 25px #f5e6c4;
}

#messages{
    height:60%;
    border:2px solid #f5e6c4;
    overflow-y:auto;
    padding:10px;
    margin-bottom:10px;
    text-align:left;
    background: rgba(0,0,0,0.4);
    border-radius:6px;
}

#messages div{
    background: rgba(0,0,0,0.3);
    padding: 5px 8px;
    margin-bottom:4px;
    border-radius:4px;
    box-shadow: 0 0 5px rgba(245,230,196,0.2);
}

/* ---------- INPUT E BUTTON ---------- */
input, button{
    border-radius:8px;
    border:1px solid #f5e6c4;
    background: rgba(0,0,0,0.5);
    color: #f5e6c4;
    padding:8px 12px;
    font-size:16px;
    box-shadow: 0 0 8px rgba(245,230,196,0.3);
    transition: 0.2s;
}

input:focus, button:hover{
    box-shadow:0 0 20px #f5e6c4;
    outline:none;
}

/* ---------- HUD ---------- */
#hud{
    display:none;
    position:absolute;
    top:10px;
    width:100%;
    justify-content:space-around;
    color:#f5e6c4;
    font-size:18px;
    text-shadow: 0 0 10px #f5e6c4;
}

/* ---------- CANVAS ---------- */
canvas{
    display:none;
    margin:100px auto 0;
    background:#000;
    border:3px solid #f5e6c4;
    box-shadow:0 0 25px #f5e6c4;
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
    color:#f5e6c4;
    text-shadow:0 0 15px #f5e6c4;
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
    color:#f5e6c4;
}

#leaderboard h2{
    font-size:48px;
    text-shadow:0 0 20px #f5e6c4;
}

/* ---------- CARTELLA FILE (HOME) ---------- */
#fileFolder{
    position: fixed;
    bottom:20px;
    right:20px;
    width:200px;
    background: rgba(0,0,0,0.6);
    border:2px solid #f5e6c4;
    border-radius:10px;
    padding:10px;
    box-shadow:0 0 20px #f5e6c4;
    z-index:2;
}

#fileFolder h3{
    margin:0 0 10px 0;
    font-size:18px;
    color:#f5e6c4;
    text-shadow:0 0 10px #f5e6c4;
}

#fileList div{
    margin-bottom:5px;
}
</style>
</head>

<body>

<!-- PARTICELLE -->
<script>
for(let i=0;i<40;i++){
    const p=document.createElement('div');
    p.className='particle';
    p.style.left=Math.random()*window.innerWidth+'px';
    p.style.top=Math.random()*window.innerHeight+'px';
    p.style.animationDuration=2+Math.random()*3+'s';
    p.style.width=2+Math.random()*3+'px';
    p.style.height=2+Math.random()*3+'px';
    document.body.appendChild(p);
}
</script>

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
    <h2>Per quelli che verranno</h2>
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

<!-- CARTELLA FILE -->
<div id="fileFolder">
    <h3>Archivio Segreto</h3>
    <input type="file" id="fileInput"><br><br>
    <button onclick="uploadFile()">Carica</button>
    <div id="fileList"></div>
</div>

<script>
/* ---------- FILE UPLOAD ---------- */
function uploadFile(){
    const input=document.getElementById("fileInput");
    if(!input.files.length) return alert("Seleziona un file!");
    const file=input.files[0];
    const reader=new FileReader();
    reader.onload=function(e){
        let files=JSON.parse(localStorage.getItem("archivio"))||[];
        files.push({name:file.name,data:e.target.result});
        localStorage.setItem("archivio",JSON.stringify(files));
        renderFiles();
    };
    reader.readAsDataURL(file);
}

function renderFiles(){
    const list=document.getElementById("fileList");
    const files=JSON.parse(localStorage.getItem("archivio"))||[];
    list.innerHTML=files.map(f=>`<div>${f.name} - <a href="${f.data}" download="${f.name}">Scarica</a></div>`).join('');
}

renderFiles();

/* ---------- UTIL ---------- */
function backToMenu(){
    document.querySelectorAll("#chat,#leaderboard,#gameOver,#hud,canvas")
        .forEach(e=>e.style.display="none");
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

    ctx.fillStyle="#f5e6c4";
    ctx.fillRect(paddle.x,paddle.y,paddle.w,paddle.h);

    ctx.beginPath();
    ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
    ctx.fill();

    ball.x+=ball.vx;
    ball.y+=ball.vy;

    if(ball.y<=0||ball.y>=canvas.height) ball.vy*=-1;

    if(ball.vx<0 && ball.x-ball.r<=paddle.x+paddle.w &&
       ball.x>paddle.x && ball.y>=paddle.y && ball.y<=paddle.y+paddle.h){
        ball.vx*=-1;
        score++;
        document.getElementById("score").textContent=score;
    }

    if(ball.x<0){ endGame(); return; }
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
