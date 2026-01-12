<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<title>Pong Arcade</title>

<style>
body {
    margin: 0;
    background: radial-gradient(circle at top, #111, #000);
    color: white;
    font-family: Arial, sans-serif;
    text-align: center;
    overflow: hidden;
}

/* ---------- ANIMAZIONI ---------- */
@keyframes pulse {
    0% { transform: scale(1); opacity: 0.8; }
    50% { transform: scale(1.05); opacity: 1; }
    100% { transform: scale(1); opacity: 0.8; }
}

@keyframes fadeSlide {
    from { opacity: 0; transform: translateY(30px); }
    to { opacity: 1; transform: translateY(0); }
}

/* ---------- MENU ---------- */
#menu, #gameOver {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    animation: fadeSlide 0.8s ease;
}

#menu h1 {
    font-size: 56px;
    animation: pulse 2s infinite;
    color: #0ff;
    text-shadow: 0 0 20px #0ff;
}

input, button {
    padding: 12px 20px;
    font-size: 18px;
    margin: 8px;
    border-radius: 6px;
    border: none;
}

button {
    background: #0ff;
    cursor: pointer;
}

#credit {
    position: absolute;
    bottom: 15px;
    font-size: 14px;
    opacity: 0.6;
}

/* ---------- HUD ---------- */
#hud {
    position: absolute;
    top: 10px;
    width: 100%;
    display: flex;
    justify-content: space-around;
    font-size: 18px;
}

/* ---------- CANVAS ---------- */
canvas {
    display: none;
    margin: auto;
    background: linear-gradient(#000, #111);
    border: 3px solid #0ff;
    box-shadow: 0 0 25px #0ff;
    touch-action: none;
}

/* ---------- GAME OVER ---------- */
#gameOver {
    display: none;
    background: rgba(0,0,0,0.85);
}

#gameOver h2 {
    color: #f44;
    text-shadow: 0 0 15px #f44;
}

#scores {
    margin-top: 15px;
    line-height: 1.6;
}
</style>
</head>

<body>

<div id="menu">
    <h1>🏓 PONG ARCADE</h1>
    <input id="playerName" placeholder="Il tuo nome">
    <button onclick="startGame()">INIZIA</button>
    <div id="credit">Creato da Edobe</div>
</div>

<div id="hud" style="display:none">
    <div>Giocatore: <span id="name"></span></div>
    <div>Punteggio: <span id="score">0</span></div>
</div>

<canvas id="game" width="720" height="420"></canvas>

<div id="gameOver">
    <h2>💀 GAME OVER</h2>
    <div id="finalScore"></div>
    <div id="scores"></div>
    <button onclick="location.reload()">RIGIOCA</button>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const highscoresKey = "pong_highscores";

let paddle, ball, score, speed, player, playing;
let up=false, down=false;

/* ---------- START ---------- */
function startGame() {
    player = document.getElementById("playerName").value || "Anonimo";
    document.getElementById("menu").style.display = "none";
    document.getElementById("hud").style.display = "flex";
    canvas.style.display = "block";
    document.getElementById("name").textContent = player;

    paddle = { x: 20, y: 170, w: 12, h: 90, speed: 9 };
    ball = { x: 360, y: 210, r: 8, vx: 5, vy: 5 };
    score = 0;
    speed = 1;
    playing = true;

    requestAnimationFrame(loop);
}

/* ---------- INPUT ---------- */
document.addEventListener("keydown", e => {
    if (e.key==="ArrowUp" || e.key==="w") up=true;
    if (e.key==="ArrowDown" || e.key==="s") down=true;
});
document.addEventListener("keyup", e => {
    if (e.key==="ArrowUp" || e.key==="w") up=false;
    if (e.key==="ArrowDown" || e.key==="s") down=false;
});
canvas.addEventListener("touchmove", e => {
    const r = canvas.getBoundingClientRect();
    paddle.y = e.touches[0].clientY - r.top - paddle.h/2;
});

/* ---------- LOOP ---------- */
function loop() {
    if (!playing) return;

    ctx.clearRect(0,0,canvas.width,canvas.height);

    if (up && paddle.y>0) paddle.y -= paddle.speed;
    if (down && paddle.y<canvas.height-paddle.h) paddle.y += paddle.speed;

    ctx.setLineDash([10,15]);
    ctx.strokeStyle="#0ff";
    ctx.beginPath();
    ctx.moveTo(canvas.width/2,0);
    ctx.lineTo(canvas.width/2,canvas.height);
    ctx.stroke();
    ctx.setLineDash([]);

    ctx.fillStyle="#0ff";
    ctx.shadowColor="#0ff";
    ctx.shadowBlur=15;
    ctx.fillRect(paddle.x,paddle.y,paddle.w,paddle.h);

    ctx.beginPath();
    ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
    ctx.fill();

    ctx.shadowBlur=0;

    ball.x += ball.vx * speed;
    ball.y += ball.vy * speed;

    if (ball.y<=0 || ball.y>=canvas.height) ball.vy*=-1;

    if (
        ball.x-ball.r<=paddle.x+paddle.w &&
        ball.y>=paddle.y &&
        ball.y<=paddle.y+paddle.h
    ) {
        ball.vx*=-1;
        score++;
        speed+=0.04;
        document.getElementById("score").textContent=score;
    }

    if (ball.x<0) endGame();
    if (ball.x>canvas.width) ball.vx*=-1;

    requestAnimationFrame(loop);
}

/* ---------- GAME OVER ---------- */
function endGame() {
    playing=false;
    saveHighscore();
    showGameOver();
}

function saveHighscore() {
    let list = JSON.parse(localStorage.getItem(highscoresKey)) || [];
    list.push({name:player, score});
    list.sort((a,b)=>b.score-a.score);
    localStorage.setItem(highscoresKey, JSON.stringify(list.slice(0,5)));
}

function showGameOver() {
    document.getElementById("gameOver").style.display="flex";
    document.getElementById("finalScore").textContent =
        "Punteggio finale: " + score;

    let list = JSON.parse(localStorage.getItem(highscoresKey)) || [];
    let html="<h3>🏆 Highscores</h3>";
    list.forEach((e,i)=>html+=`${i+1}. ${e.name} - ${e.score}<br>`);
    document.getElementById("scores").innerHTML=html;
}
</script>

</body>
</html>
