<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Colorful Memory Puzzle</title>
<style>
  :root{
    --card-size: min(12vw, 96px);
    --gap: 12px;
    --ui-bg: rgba(255,255,255,0.08);
    --glass: rgba(255,255,255,0.06);
  }
  html,body{height:100%; margin:0; font-family:Inter,system-ui,Arial; color:#fff;}
  /* Full-screen animated gradient + soft particles canvas behind */
  .bg-wrap{
    position:fixed; inset:0; z-index:0; overflow:hidden;
    background: linear-gradient(135deg,#41295a 0%, #2b5876 50%, #1dd3b0 100%);
  }
  canvas#particles{ position:absolute; inset:0; width:100%; height:100%; display:block; }
  /* subtle animated gradient overlay */
  .gradient {
    position:absolute; inset:0; z-index:0; pointer-events:none;
    background: linear-gradient(120deg, rgba(255,255,255,0.02), rgba(0,0,0,0.02));
    mix-blend-mode: overlay;
    animation: rotate 12s linear infinite;
    opacity:0.9;
  }
  @keyframes rotate { 
    0%{transform:rotate(0);} 50%{transform:rotate(180deg);} 100%{transform:rotate(360deg);} 
  }

  /* App container */
  .app{
    position:relative;
    z-index:2;
    min-height:100%;
    display:flex;
    align-items:center;
    justify-content:center;
    padding:32px;
    box-sizing:border-box;
  }

  .card-area {
    width: min(86vw, 720px);
    backdrop-filter: blur(6px) saturate(1.1);
    background: linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
    border-radius:14px;
    padding:22px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.45), inset 0 1px rgba(255,255,255,0.02);
  }

  header {
    display:flex; align-items:center; justify-content:space-between;
    gap:12px; margin-bottom:18px;
  }
  h1 { font-size:20px; margin:0; letter-spacing:0.4px; }
  .controls { display:flex; gap:8px; align-items:center; }
  .btn {
    border:0; padding:8px 12px; border-radius:10px; cursor:pointer;
    background:var(--ui-bg); color:#fff; font-weight:600; font-size:13px;
    transition:transform .15s ease, background .15s;
  }
  .btn:hover{ transform:translateY(-2px); background:var(--glass); }

  /* grid of cards */
  .grid {
    display:grid;
    grid-template-columns: repeat(4, var(--card-size));
    grid-auto-rows: var(--card-size);
    gap: var(--gap);
    justify-content:center;
    align-items:center;
  }

  /* card */
  .card {
    width:var(--card-size); height:var(--card-size);
    perspective:1000px;
    cursor:pointer;
  }
  .card-inner {
    width:100%; height:100%;
    position:relative; transform-style:preserve-3d;
    transition: transform 420ms cubic-bezier(.2,.9,.3,1);
    box-shadow: 0 6px 18px rgba(0,0,0,0.45);
    border-radius:10px;
  }
  .card.flipped .card-inner { transform: rotateY(180deg) scale(1.02); }
  .face {
    position:absolute; inset:0; display:flex; align-items:center; justify-content:center;
    backface-visibility: hidden; border-radius:10px; font-weight:700; font-size:22px;
  }
  .front {
    transform: rotateY(180deg);
    color:#111;
  }
  .back {
    background: linear-gradient(135deg, rgba(255,255,255,0.06), rgba(0,0,0,0.06));
    display:flex; align-items:center; justify-content:center;
    gap:6px;
    font-size:14px;
  }

  /* small info/footer */
  .meta { display:flex; gap:12px; align-items:center; font-size:13px; margin-top:14px; color: #eef; }
  .meta b { font-weight:700; color:#fff; }

  /* responsive: fewer columns on narrow screens */
  @media (max-width:640px){
    .grid { grid-template-columns: repeat(3, var(--card-size)); }
  }
</style>
</head>
<body>
  <div class="bg-wrap">
    <canvas id="particles"></canvas>
    <div class="gradient"></div>
  </div>

  <div class="app">
    <div class="card-area" role="application" aria-label="Colorful memory puzzle">
      <header>
        <h1>Colorful Memory Puzzle</h1>
        <div class="controls">
          <button id="newBtn" class="btn" title="Restart (R)">New Game</button>
          <button id="hintBtn" class="btn" title="Show a hint (H)">Hint</button>
        </div>
      </header>

      <main>
        <div id="grid" class="grid" tabindex="0"></div>
        <div class="meta">
          <div>Moves: <b id="moves">0</b></div>
          <div>Pairs: <b id="pairs">0 / 8</b></div>
          <div>Time: <b id="time">0s</b></div>
        </div>
      </main>
    </div>
  </div>

<script>
/* ====== Particle Background (canvas) ====== */
(() => {
  const c = document.getElementById('particles');
  const ctx = c.getContext('2d');
  let W = c.width = innerWidth;
  let H = c.height = innerHeight;
  window.addEventListener('resize', ()=>{ W = c.width = innerWidth; H = c.height = innerHeight; });

  const P = 70;
  const particles = [];
  for(let i=0;i<P;i++){
    particles.push({
      x: Math.random()*W,
      y: Math.random()*H,
      r: 0.8+Math.random()*3.2,
      vx: (Math.random()-0.5)*0.6,
      vy: (Math.random()-0.5)*0.6,
      hue: Math.floor(Math.random()*360)
    });
  }

  function draw(){
    ctx.clearRect(0,0,W,H);
    // soft vignette
    const g = ctx.createLinearGradient(0,0,W,H);
    g.addColorStop(0,'rgba(255,255,255,0.02)');
    g.addColorStop(1,'rgba(0,0,0,0.08)');
    ctx.fillStyle = g;
    ctx.fillRect(0,0,W,H);

    for(const p of particles){
      p.x += p.vx; p.y += p.vy;
      if(p.x < -20) p.x = W+20;
      if(p.x > W+20) p.x = -20;
      if(p.y < -20) p.y = H+20;
      if(p.y > H+20) p.y = -20;

      ctx.beginPath();
      const hue = (p.hue + (Math.sin(Date.now()*0.0005+p.x*0.0002)*30))|0;
      ctx.fillStyle = `hsla(${hue}, 85%, 60%, 0.10)`;
      ctx.arc(p.x, p.y, p.r, 0, Math.PI*2);
      ctx.fill();

      // glow
      ctx.beginPath();
      ctx.fillStyle = `hsla(${hue}, 85%, 60%, 0.06)`;
      ctx.arc(p.x-8, p.y-8, p.r*4.5, 0, Math.PI*2);
      ctx.fill();
    }

    // connect near particles
    ctx.beginPath();
    for(let i=0;i<particles.length;i++){
      for(let j=i+1;j<particles.length;j++){
        const a=particles[i], b=particles[j];
        const dx=a.x-b.x, dy=a.y-b.y, d=dx*dx+dy*dy;
        if(d < 10000){
          ctx.moveTo(a.x,a.y);
          ctx.lineTo(b.x,b.y);
        }
      }
    }
    ctx.strokeStyle = 'rgba(255,255,255,0.02)';
    ctx.stroke();

    requestAnimationFrame(draw);
  }
  draw();
})();

/* ====== Memory Game Logic ====== */
const palette = [
  '#FF6B6B','#FFD93D','#6BCB77','#4D96FF','#8E44AD','#FF7BAC','#00C2A8','#FF9F1C',
  '#2EC4B6','#F72585','#7209B7','#3A0CA3'
];

const gridEl = document.getElementById('grid');
const movesEl = document.getElementById('moves');
const pairsEl = document.getElementById('pairs');
const timeEl = document.getElementById('time');
const newBtn = document.getElementById('newBtn');
const hintBtn = document.getElementById('hintBtn');

let cards = [];
let first = null, second = null;
let lock = false;
let moves = 0;
let matches = 0;
let timer = null;
let seconds = 0;
let

