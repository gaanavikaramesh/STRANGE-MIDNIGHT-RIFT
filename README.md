# STRANGE-MIDNIGHT-RIFT----------if(gameState==="start"){
    ctx.fillStyle="black";
    ctx.fillRect(0,0,canvas.width,canvas.height);

    // subtle glow
    ctx.shadowBlur=25;
    ctx.shadowColor="red";

    ctx.fillStyle="red";
    ctx.textAlign="center";
    ctx.font="70px Arial";
    ctx.fillText("STRANGE",canvas.width/2,canvas.height/2-120);

    ctx.font="90px Arial";
    ctx.fillText("MIDNIGHT",canvas.width/2,canvas.height/2-30);

    ctx.font="70px Arial";
    ctx.fillText("RIFT",canvas.width/2,canvas.height/2+60);

    ctx.shadowBlur=0;

    ctx.fillStyle="white";
    ctx.font="22px Arial";
    ctx.fillText("Switch Worlds. Survive the Rift.",
                 canvas.width/2,canvas.height/2+120);

    ctx.font="18px Arial";
    ctx.fillText("Press Any Key or Tap to Begin",
                 canvas.width/2,canvas.height/2+160);

    return;
}
Strange Midnight Rift is a retro neon arcade browser game featuring dual-world switching mechanics, particle effects, dynamic difficulty scaling, and high score saving. Developed using pure HTML5 and JavaScript --------<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Strange Midnight Rift</title>
<style>
body { margin:0; overflow:hidden; background:black; font-family:Arial; }
canvas { display:block; }
</style>
</head>
<body>
<canvas id="game"></canvas>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");
canvas.width=window.innerWidth;
canvas.height=window.innerHeight;

/* ---------------- GAME VARIABLES ---------------- */

let gameState="start";
let world=1;
let score=0;
let highScore=localStorage.getItem("riftHigh")||0;
let speed=6;
let gravity=0.6;

let player={x:120,y:canvas.height-150,w:40,h:40,vy:0};
let obstacles=[];
let particles=[];
let stars=[];

for(let i=0;i<120;i++){
stars.push({x:Math.random()*canvas.width,
            y:Math.random()*canvas.height,
            size:Math.random()*2});
}

/* ---------------- MUSIC ---------------- */

const audioCtx=new(window.AudioContext||window.webkitAudioContext)();

function synth(freq){
let o=audioCtx.createOscillator();
let g=audioCtx.createGain();
o.type="sawtooth";
o.frequency.value=freq;
g.gain.value=0.02;
o.connect(g);
g.connect(audioCtx.destination);
o.start();
o.stop(audioCtx.currentTime+0.15);
}

setInterval(()=>{if(gameState==="playing")synth(world===1?120:200)},400);

/* ---------------- CONTROLS ---------------- */

document.addEventListener("keydown",e=>{
if(gameState==="start") startGame();
else if(gameState==="gameover") resetGame();
else{
if(e.code==="Space") switchWorld();
if(e.code==="ArrowUp" && player.y>=canvas.height-150)
player.vy=-12;
}
});

canvas.addEventListener("touchstart",()=>{
if(gameState==="start") startGame();
else if(gameState==="gameover") resetGame();
else switchWorld();
});

/* ---------------- GAME FLOW ---------------- */

function startGame(){
audioCtx.resume();
gameState="playing";
}

function resetGame(){
if(score>highScore){
highScore=score;
localStorage.setItem("riftHigh",highScore);
}
score=0;
speed=6;
world=1;
player.y=canvas.height-150;
player.vy=0;
obstacles=[];
particles=[];
gameState="playing";
}

function switchWorld(){
world=world===1?2:1;
for(let i=0;i<20;i++){
particles.push({
x:player.x+20,
y:player.y+20,
vx:(Math.random()-0.5)*8,
vy:(Math.random()-0.5)*8,
life:30
});
}
}

function spawnObstacle(){
obstacles.push({
x:canvas.width,
y:canvas.height-150,
w:40,
h:40,
world:Math.random()>0.5?1:2
});
}

/* ---------------- UPDATE ---------------- */

let spawnTimer=0;
let difficultyTimer=0;

function update(){
if(gameState!=="playing") return;

spawnTimer++;
difficultyTimer++;

if(spawnTimer>90){
spawnObstacle();
spawnTimer=0;
}

if(difficultyTimer>600){
speed+=0.5;
difficultyTimer=0;
}

player.vy+=gravity;
player.y+=player.vy;

if(player.y>canvas.height-150){
player.y=canvas.height-150;
player.vy=0;
}

obstacles.forEach(obs=>{
obs.x-=speed;

if(obs.world===world &&
player.x<obs.x+obs.w &&
player.x+player.w>obs.x &&
player.y<obs.y+obs.h &&
player.y+player.h>obs.y){
gameState="gameover";
}
});

obstacles=obstacles.filter(o=>o.x>-50);

particles.forEach(p=>{
p.x+=p.vx;
p.y+=p.vy;
p.life--;
});
particles=particles.filter(p=>p.life>0);

score++;
}

/* ---------------- DRAW ---------------- */

function drawNeonRect(x,y,w,h,color){
ctx.shadowBlur=20;
ctx.shadowColor=color;
ctx.fillStyle=color;
ctx.fillRect(x,y,w,h);
ctx.shadowBlur=0;
}

function drawStars(){
ctx.fillStyle="white";
stars.forEach(s=>{
s.x-=1;
if(s.x<0) s.x=canvas.width;
ctx.fillRect(s.x,s.y,s.size,s.size);
});
}

function draw(){
ctx.fillStyle=world===1?"#111":"#200";
ctx.fillRect(0,0,canvas.width,canvas.height);

drawStars();

if(gameState==="start"){
ctx.shadowBlur=25;
ctx.shadowColor="red";
ctx.fillStyle="red";
ctx.textAlign="center";

ctx.font="70px Arial";
ctx.fillText("STRANGE",canvas.width/2,canvas.height/2-120);

ctx.font="90px Arial";
ctx.fillText("MIDNIGHT",canvas.width/2,canvas.height/2-30);

ctx.font="70px Arial";
ctx.fillText("RIFT",canvas.width/2,canvas.height/2+60);

ctx.shadowBlur=0;

ctx.fillStyle="white";
ctx.font="22px Arial";
ctx.fillText("Switch Worlds. Survive the Rift.",
canvas.width/2,canvas.height/2+120);

ctx.font="18px Arial";
ctx.fillText("Press Any Key or Tap",
canvas.width/2,canvas.height/2+160);
return;
}

drawNeonRect(player.x,player.y,player.w,player.h,
world===1?"cyan":"magenta");

obstacles.forEach(obs=>{
if(obs.world===world)
drawNeonRect(obs.x,obs.y,obs.w,obs.h,"red");
});

particles.forEach(p=>{
ctx.fillStyle="white";
ctx.fillRect(p.x,p.y,4,4);
});

ctx.fillStyle="white";
ctx.font="20px Arial";
ctx.textAlign="left";
ctx.fillText("Score: "+score,20,40);
ctx.fillText("High: "+highScore,20,70);
ctx.fillText("World: "+world,20,100);

if(gameState==="gameover"){
ctx.fillStyle="rgba(0,0,0,0.7)";
ctx.fillRect(0,0,canvas.width,canvas.height);

ctx.fillStyle="white";
ctx.textAlign="center";
ctx.font="60px Arial";
ctx.fillText("GAME OVER",
canvas.width/2,canvas.height/2-20);

ctx.font="30px Arial";
ctx.fillText("Score: "+score,
canvas.width/2,canvas.height/2+30);

ctx.fillText("Press Any Key",
canvas.width/2,canvas.height/2+80);
}
}

/* ---------------- LOOP ---------------- */

function loop(){
update();
draw();
requestAnimationFrame(loop);
}
loop();

</script>
</body>
</html>
