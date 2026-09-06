[Uploading alkkagi.html…]()
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1, user-scalable=no">
<title>보드게임 아케이드</title>
<style>
  :root{
    --bg-deep:#1b140f; --bg-mid:#2b2017; --panel:#332417;
    --wood-a:#8a5a2e; --wood-b:#5a3616;
    --board-tan-a:#f2d9a8; --board-tan-b:#dcb877;
    --accent:#e8a33d; --accent-2:#e2c08d;
    --ink:#f3e8d6; --ink-dim:#c9b696;
    --black-stone:#1c1c1c; --white-stone:#f7f2e6;
    --red:#c0392b; --blue:#1a4fa0;
    --good:#5fbf6a; --bad:#e0574c;
  }
  *{ box-sizing:border-box; }
  html,body{ margin:0; padding:0; height:100%;
    font-family:"Apple SD Gothic Neo","Malgun Gothic","Pretendard",sans-serif;
    color:var(--ink); overflow:hidden; -webkit-user-select:none; user-select:none;
    background: radial-gradient(circle at 50% -10%, var(--bg-mid), var(--bg-deep) 70%);
  }
  #app{ display:flex; flex-direction:column; align-items:center; height:100%; padding:10px 8px; gap:8px; }

  /* ---- Menu ---- */
  #menuScreen{ display:flex; flex-direction:column; align-items:center; gap:16px; width:100%; max-width:560px; margin-top:4vh; }
  #menuScreen h1{ margin:0; font-size:26px; letter-spacing:3px; font-weight:800; color:var(--accent); }
  #menuScreen p.sub{ margin:0; font-size:12px; color:var(--ink-dim); }
  #cardGrid{ display:grid; grid-template-columns:1fr 1fr; gap:12px; width:100%; padding:0 6px; }
  .card{
    background:linear-gradient(180deg,var(--panel),#241a10);
    border:1px solid rgba(255,255,255,.08);
    border-radius:14px; padding:16px 10px; text-align:center; cursor:pointer;
    box-shadow:0 4px 10px rgba(0,0,0,.35);
    transition:transform .15s, box-shadow .15s;
  }
  .card:active{ transform:scale(.97); }
  .card:hover{ box-shadow:0 0 0 2px var(--accent), 0 6px 14px rgba(0,0,0,.4); }
  .card .emoji{ font-size:34px; display:block; margin-bottom:6px; }
  .card .name{ font-size:15px; font-weight:800; color:var(--ink); }
  .card .desc{ font-size:11px; color:var(--ink-dim); margin-top:4px; line-height:1.4; }

  /* ---- Game screen ---- */
  #gameScreen{ display:none; flex-direction:column; align-items:center; width:100%; height:100%; gap:8px; }
  #topbar{ display:flex; align-items:center; justify-content:space-between; width:100%; max-width:560px; }
  #backBtn{
    font-family:inherit; font-size:12px; font-weight:700; color:var(--ink);
    background:rgba(255,255,255,.08); border:1px solid rgba(255,255,255,.15);
    border-radius:999px; padding:6px 12px; cursor:pointer;
  }
  #gameTitle{ font-size:15px; font-weight:800; color:var(--accent); }
  #turnInfo{ font-size:12px; font-weight:700; padding:4px 12px; border-radius:999px; background:rgba(255,255,255,.08); min-width:96px; text-align:center; }
  #turnInfo.you{ color:var(--good); }
  #turnInfo.ai{ color:var(--bad); }

  #boardWrap{ position:relative; width:min(84vmin,520px); height:min(84vmin,520px); flex-shrink:0; }
  canvas{ width:100%; height:100%; display:block; touch-action:none; cursor:pointer; border-radius:10px; }

  #controls{ display:flex; gap:8px; }
  .ctlBtn{
    font-family:inherit; font-size:12px; font-weight:700; color:#4a2f0c;
    background:linear-gradient(180deg,#ffe373,#f6b93b); border:2px solid #b9781a;
    border-radius:999px; padding:7px 16px; cursor:pointer; box-shadow:0 2px 0 rgba(0,0,0,.25);
  }
  .ctlBtn:active{ transform:translateY(1px); box-shadow:none; }
  .ctlBtn.secondary{ background:linear-gradient(180deg,#d9d9d9,#aaa); border-color:#777; color:#222; }

  #overlay{ position:absolute; inset:0; display:none; align-items:center; justify-content:center; flex-direction:column; gap:12px; background:rgba(10,8,6,.72); border-radius:10px; text-align:center; padding:14px; }
  #overlay.show{ display:flex; }
  #overlayText{ font-size:20px; font-weight:900; color:var(--accent); text-shadow:0 2px 6px rgba(0,0,0,.6); white-space:pre-line; }
  #overlayBtns{ display:flex; gap:8px; }
  #hint{ font-size:11px; color:var(--ink-dim); text-align:center; max-width:520px; margin:0; }
  #aiThinking{ font-size:11px; color:var(--bad); min-height:14px; }
  @media (max-width:420px){
    #menuScreen h1{ font-size:21px; }
    .card .emoji{ font-size:28px; }
  }
</style>
</head>
<body>
<div id="app">

  <div id="menuScreen">
    <h1>보드게임 아케이드</h1>
    <p class="sub">게임을 선택하세요 · 상대는 컴퓨터(AI)입니다</p>
    <div id="cardGrid">
      <div class="card" data-game="alkkagi"><span class="emoji">⚪</span><div class="name">알까기</div><div class="desc">돌을 튕겨 상대 돌을<br>판 밖으로 밀어내기</div></div>
      <div class="card" data-game="baduk"><span class="emoji">⚫</span><div class="name">바둑</div><div class="desc">18×18 · 상대 돌을 에워싸<br>더 넓은 집을 차지하기</div></div>
      <div class="card" data-game="janggi"><span class="emoji">🀄</span><div class="name">장기</div><div class="desc">상대 궁(장군)을<br>먼저 잡기</div></div>
      <div class="card" data-game="omok"><span class="emoji">⚫⚪</span><div class="name">오목</div><div class="desc">18×18 · 가로·세로·대각선<br>5개를 먼저 잇기</div></div>
    </div>
  </div>

  <div id="gameScreen">
    <div id="topbar">
      <button id="backBtn">← 게임 선택</button>
      <div id="gameTitle"></div>
      <div id="turnInfo">-</div>
    </div>
    <div id="boardWrap">
      <canvas id="c" width="600" height="600"></canvas>
      <div id="overlay"><div id="overlayText"></div>
        <div id="overlayBtns">
          <button class="ctlBtn" id="retryBtn">다시하기</button>
          <button class="ctlBtn secondary" id="menuBtn2">게임 선택으로</button>
        </div>
      </div>
    </div>
    <div id="controls"></div>
    <div id="aiThinking"></div>
    <p id="hint"></p>
  </div>

</div>

<script>
(function(){
"use strict";
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;

const menuScreen = document.getElementById('menuScreen');
const gameScreen = document.getElementById('gameScreen');
const gameTitleEl = document.getElementById('gameTitle');
const turnInfoEl = document.getElementById('turnInfo');
const overlayEl = document.getElementById('overlay');
const overlayTextEl = document.getElementById('overlayText');
const hintEl = document.getElementById('hint');
const controlsEl = document.getElementById('controls');
const aiThinkingEl = document.getElementById('aiThinking');

function roundRect(ctx,x,y,w,h,r){
  ctx.beginPath(); ctx.moveTo(x+r,y);
  ctx.arcTo(x+w,y,x+w,y+h,r); ctx.arcTo(x+w,y+h,x,y+h,r);
  ctx.arcTo(x,y+h,x,y,r); ctx.arcTo(x,y,x+w,y,r); ctx.closePath();
}
function drawWoodFrame(){
  const fg = ctx.createLinearGradient(0,0,0,H);
  fg.addColorStop(0,'#8a5a2e'); fg.addColorStop(1,'#5a3616');
  ctx.fillStyle = fg; roundRect(ctx,4,4,W-8,H-8,16); ctx.fill();
}
function drawStoneCircle(x,y,r,color,alpha){
  ctx.save(); ctx.globalAlpha = alpha===undefined?1:alpha;
  ctx.beginPath(); ctx.ellipse(x+1.2,y+2.5,r*0.9,r*0.75,0,0,Math.PI*2);
  ctx.fillStyle='rgba(0,0,0,.25)'; ctx.fill();
  const grad = ctx.createRadialGradient(x-r*0.35,y-r*0.4,r*0.2,x,y,r);
  if(color==='black'){ grad.addColorStop(0,'#585858'); grad.addColorStop(.55,'#242424'); grad.addColorStop(1,'#0a0a0a'); }
  else { grad.addColorStop(0,'#ffffff'); grad.addColorStop(.55,'#f0e8d4'); grad.addColorStop(1,'#c7bb99'); }
  ctx.beginPath(); ctx.arc(x,y,r,0,Math.PI*2); ctx.fillStyle=grad; ctx.fill();
  ctx.lineWidth=1; ctx.strokeStyle= color==='black' ? 'rgba(0,0,0,.6)' : 'rgba(120,105,75,.4)'; ctx.stroke();
  ctx.restore();
}
function rand(a,b){ return a+Math.random()*(b-a); }
function clamp(v,a,b){ return Math.max(a,Math.min(b,v)); }

let current = null;
function setTurnInfo(text, who){
  turnInfoEl.textContent = text;
  turnInfoEl.className = who==='you' ? 'you' : (who==='ai' ? 'ai' : '');
}
function showOverlay(text){ overlayTextEl.textContent = text; overlayEl.classList.add('show'); }
function hideOverlay(){ overlayEl.classList.remove('show'); }

function launch(key){
  menuScreen.style.display='none';
  gameScreen.style.display='flex';
  hideOverlay(); aiThinkingEl.textContent=''; controlsEl.innerHTML='';
  current = makeGame(key);
  current.init();
}
function backToMenu(){
  current = null;
  gameScreen.style.display='none';
  menuScreen.style.display='flex';
}
document.querySelectorAll('.card').forEach(c=>{
  c.addEventListener('click', ()=> launch(c.getAttribute('data-game')));
});
document.getElementById('backBtn').addEventListener('click', backToMenu);
document.getElementById('menuBtn2').addEventListener('click', backToMenu);
document.getElementById('retryBtn').addEventListener('click', ()=>{ if(current && current.key) launch(current.key); });

function getPos(evt){
  const rect = canvas.getBoundingClientRect();
  const sx=W/rect.width, sy=H/rect.height;
  const cx=(evt.touches?evt.touches[0].clientX:evt.clientX)-rect.left;
  const cy=(evt.touches?evt.touches[0].clientY:evt.clientY)-rect.top;
  return {x:cx*sx, y:cy*sy};
}
function onDown(e){ if(current && current.onDown){ current.onDown(getPos(e)); e.preventDefault(); } }
function onMove(e){ if(current && current.onMove){ current.onMove(getPos(e)); e.preventDefault(); } }
function onUp(e){ if(current && current.onUp){ current.onUp(getPos(e)); e.preventDefault(); } }
canvas.addEventListener('mousedown',onDown);
window.addEventListener('mousemove',onMove);
window.addEventListener('mouseup',onUp);
canvas.addEventListener('touchstart',onDown,{passive:false});
canvas.addEventListener('touchmove',onMove,{passive:false});
canvas.addEventListener('touchend',onUp,{passive:false});

function mainLoop(){
  if(current){ if(current.step) current.step(); ctx.clearRect(0,0,W,H); if(current.render) current.render(); }
  requestAnimationFrame(mainLoop);
}
mainLoop();

/* =========================================================
   GAME FACTORY
   ========================================================= */
function makeGame(key){
  if(key==='alkkagi') return AlkkagiGame();
  if(key==='baduk') return BadukGame();
  if(key==='omok') return OmokGame();
  if(key==='janggi') return JanggiGame();
}

/* =========================================================
   1) ALKKAGI  (vs AI)
   ========================================================= */
function AlkkagiGame(){
  const G = { key:'alkkagi' };
  const PAD=66, DIV=12, BX=PAD, BY=PAD, BS=W-PAD*2, CELL=BS/DIV;
  const STONE_R=15, OUT_MARGIN=26, FRICTION=0.983, STOP_SPEED=0.045;
  const MAX_PULL=120, POWER=0.14, MAX_SPEED=25;
  let stones=[], turn='white', moving=false, gameOver=false, drag=null, aiTimer=null;

  function gridPt(cx,cy){ return {x:BX+cx*CELL, y:BY+cy*CELL}; }

  G.init = function(){
    gameTitleEl.textContent='알까기';
    hintEl.textContent='내 돌(백)을 당겼다가 놓으면 튕겨 나갑니다. 상대 돌을 판 밖으로 밀어내세요.';
    stones=[];
    const wp=[[4,3],[6,2],[8,3],[5,4],[7,4],[6,5]];
    const bp=[[4,9],[6,10],[8,9],[5,8],[7,8],[6,7]];
    wp.forEach((p,i)=>{ const g=gridPt(p[0],p[1]); stones.push({x:g.x,y:g.y,vx:0,vy:0,color:'white',alive:true,fallT:0}); });
    bp.forEach((p,i)=>{ const g=gridPt(p[0],p[1]); stones.push({x:g.x,y:g.y,vx:0,vy:0,color:'black',alive:true,fallT:0}); });
    turn='white'; moving=false; gameOver=false; drag=null;
    setTurnInfo('내 차례 (백)','you');
  };

  function anyMoving(){ return stones.some(s=>s.alive && (Math.abs(s.vx)>STOP_SPEED||Math.abs(s.vy)>STOP_SPEED)); }
  function offBoard(s){ return s.x<BX-OUT_MARGIN||s.x>BX+BS+OUT_MARGIN||s.y<BY-OUT_MARGIN||s.y>BY+BS+OUT_MARGIN; }

  function physicsStep(){
    for(const s of stones){
      if(!s.alive) continue;
      if(s.fallT>0){ s.fallT+=0.06; continue; }
      s.x+=s.vx; s.y+=s.vy; s.vx*=FRICTION; s.vy*=FRICTION;
      if(Math.abs(s.vx)<STOP_SPEED) s.vx=0;
      if(Math.abs(s.vy)<STOP_SPEED) s.vy=0;
      if(offBoard(s)) s.fallT=0.001;
    }
    for(const s of stones){ if(s.fallT>1){ s.alive=false; s.fallT=0; } }
    const active = stones.filter(s=>s.alive && s.fallT===0);
    for(let i=0;i<active.length;i++) for(let j=i+1;j<active.length;j++){
      const a=active[i], b=active[j];
      const dx=b.x-a.x, dy=b.y-a.y, dist=Math.hypot(dx,dy)||0.001, minDist=STONE_R*2;
      if(dist<minDist){
        const nx=dx/dist, ny=dy/dist, overlap=(minDist-dist)/2;
        a.x-=nx*overlap; a.y-=ny*overlap; b.x+=nx*overlap; b.y+=ny*overlap;
        const rvx=a.vx-b.vx, rvy=a.vy-b.vy, rel=rvx*nx+rvy*ny;
        if(rel>0){ a.vx-=rel*nx; a.vy-=rel*ny; b.vx+=rel*nx; b.vy+=rel*ny; }
      }
    }
    if(moving && !anyMoving()){
      moving=false;
      for(const s of stones){ if(s.fallT>0){ s.alive=false; s.fallT=0; } }
      afterSettle();
    }
  }

  function afterSettle(){
    const wc=stones.filter(s=>s.color==='white'&&s.alive).length;
    const bc=stones.filter(s=>s.color==='black'&&s.alive).length;
    if(wc===0||bc===0){
      gameOver=true;
      showOverlay((wc===0?'상대(흑)':'나(백)')+' 승리!');
      setTurnInfo('게임 종료','');
      return;
    }
    turn = turn==='white' ? 'black':'white';
    if(turn==='black'){ setTurnInfo('AI 차례 (흑)','ai'); aiTimer=setTimeout(aiMove, 650); }
    else setTurnInfo('내 차례 (백)','you');
  }

  function aiMove(){
    if(gameOver) return;
    const mine = stones.filter(s=>s.color==='black'&&s.alive);
    const enemy = stones.filter(s=>s.color==='white'&&s.alive);
    if(mine.length===0||enemy.length===0) return;
    let best=null, bestScore=-Infinity;
    for(const b of mine) for(const w of enemy){
      const dist = Math.hypot(w.x-b.x,w.y-b.y);
      const edgeDist = Math.min(w.x-BX, BX+BS-w.x, w.y-BY, BY+BS-w.y);
      const score = 1200/(dist+40) + 260/(edgeDist+30) + rand(0,15);
      if(score>bestScore){ bestScore=score; best={b,w,dist}; }
    }
    const ang = Math.atan2(best.w.y-best.b.y, best.w.x-best.b.x) + rand(-0.05,0.05);
    const speed = clamp(best.dist*0.085*rand(0.9,1.1), 9, MAX_SPEED);
    best.b.vx = Math.cos(ang)*speed; best.b.vy = Math.sin(ang)*speed;
    moving=true;
  }

  function drawBoard(){
    drawWoodFrame();
    const tg = ctx.createLinearGradient(BX,BY,BX,BY+BS);
    tg.addColorStop(0,'#f2d9a8'); tg.addColorStop(1,'#dcb877');
    ctx.fillStyle=tg; ctx.fillRect(BX,BY,BS,BS);
    ctx.strokeStyle='rgba(120,80,30,.65)'; ctx.lineWidth=1;
    for(let i=0;i<=DIV;i++){ const x=BX+i*CELL,y=BY+i*CELL;
      ctx.beginPath();ctx.moveTo(x,BY);ctx.lineTo(x,BY+BS);ctx.stroke();
      ctx.beginPath();ctx.moveTo(BX,y);ctx.lineTo(BX+BS,y);ctx.stroke(); }
    ctx.strokeStyle='rgba(90,54,22,.9)'; ctx.lineWidth=3; ctx.strokeRect(BX,BY,BS,BS);
  }
  function drawAim(){
    if(!drag) return;
    const s=drag.stone;
    let dx=drag.curX-drag.startX, dy=drag.curY-drag.startY;
    const dist=Math.min(Math.hypot(dx,dy),MAX_PULL); const ang=Math.atan2(dy,dx);
    dx=Math.cos(ang)*dist; dy=Math.sin(ang)*dist;
    const px=s.x+dx, py=s.y+dy;
    ctx.beginPath();ctx.moveTo(s.x,s.y);ctx.lineTo(px,py);
    ctx.strokeStyle='rgba(230,60,60,.85)';ctx.lineWidth=3;ctx.setLineDash([6,6]);ctx.stroke();ctx.setLineDash([]);
    const lx=s.x-dx*1.5, ly=s.y-dy*1.5;
    ctx.beginPath();ctx.moveTo(s.x,s.y);ctx.lineTo(lx,ly);
    ctx.strokeStyle='rgba(255,255,255,.6)';ctx.lineWidth=2;ctx.stroke();
    ctx.save();ctx.globalAlpha=.55;ctx.beginPath();ctx.arc(px,py,STONE_R,0,Math.PI*2);
    ctx.fillStyle= s.color==='black'?'#222':'#eee6d2'; ctx.fill(); ctx.restore();
  }

  G.step = function(){ if(moving) physicsStep(); };
  G.render = function(){
    drawBoard();
    for(const s of stones){
      if(!s.alive) continue;
      let scale=1, alpha=1;
      if(s.fallT>0){ scale=Math.max(0,1-s.fallT); alpha=scale; }
      if(scale<=0) continue;
      drawStoneCircle(s.x,s.y,STONE_R*scale,s.color,alpha);
    }
    drawAim();
  };
  G.onDown = function(p){
    if(gameOver||moving||turn!=='white') return;
    for(const s of stones){
      if(!s.alive||s.fallT>0||s.color!=='white') continue;
      if(Math.hypot(s.x-p.x,s.y-p.y)<=STONE_R*1.5){ drag={stone:s,startX:p.x,startY:p.y,curX:p.x,curY:p.y}; return; }
    }
  };
  G.onMove = function(p){ if(drag){ drag.curX=p.x; drag.curY=p.y; } };
  G.onUp = function(){
    if(!drag) return;
    const dx=drag.curX-drag.startX, dy=drag.curY-drag.startY;
    const dist=Math.min(Math.hypot(dx,dy),MAX_PULL);
    if(dist>6){
      const ang=Math.atan2(dy,dx); const speed=Math.min(dist*POWER,MAX_SPEED);
      drag.stone.vx=-Math.cos(ang)*speed; drag.stone.vy=-Math.sin(ang)*speed; moving=true;
    }
    drag=null;
  };
  return G;
}

/* =========================================================
   2) BADUK (Go, 9x9, vs AI)
   ========================================================= */
function BadukGame(){
  const G = { key:'baduk' };
  const N=18, PAD=32, BS=W-PAD*2, CELL=BS/(N-1), BX=PAD, BY=PAD;
  const STONE_R = CELL*0.46;
  let board, turn, history, passes, gameOver, locked;

  function boardStr(b){ return b.map(row=>row.join('')).join('/'); }
  function cloneBoard(b){ return b.map(r=>r.slice()); }
  function inB(r,c){ return r>=0&&r<N&&c>=0&&c<N; }
  function neigh(r,c){ return [[r-1,c],[r+1,c],[r,c-1],[r,c+1]].filter(p=>inB(p[0],p[1])); }
  function getGroup(b,r,c){
    const color=b[r][c]; if(color===0) return null;
    const seen=new Set(); const stack=[[r,c]]; const stones=[]; const libs=new Set();
    seen.add(r*N+c);
    while(stack.length){
      const [cr,cc]=stack.pop(); stones.push([cr,cc]);
      for(const [nr,nc] of neigh(cr,cc)){
        if(b[nr][nc]===0) libs.add(nr*N+nc);
        else if(b[nr][nc]===color && !seen.has(nr*N+nc)){ seen.add(nr*N+nc); stack.push([nr,nc]); }
      }
    }
    return {stones, liberties:libs};
  }
  function tryMove(b, r, c, color){
    if(b[r][c]!==0) return {legal:false};
    const nb = cloneBoard(b); nb[r][c]=color;
    let captured=0; const opp = color===1?2:1;
    for(const [nr,nc] of neigh(r,c)){
      if(nb[nr][nc]===opp){
        const grp = getGroup(nb,nr,nc);
        if(grp.liberties.size===0){ for(const [gr,gc] of grp.stones) nb[gr][gc]=0; captured+=grp.stones.length; }
      }
    }
    const ownGrp = getGroup(nb,r,c);
    if(ownGrp.liberties.size===0) return {legal:false};
    return {legal:true, board:nb, captured, ownLibs:ownGrp.liberties.size};
  }

  G.init = function(){
    gameTitleEl.textContent='바둑 (18×18)';
    hintEl.textContent='교차점을 눌러 돌을 놓으세요. 상대 돌을 완전히 에워싸면 잡을 수 있습니다.';
    board = Array.from({length:N},()=>new Array(N).fill(0));
    turn=1; history=[boardStr(board)]; passes=0; gameOver=false; locked=false;
    setTurnInfo('내 차례 (흑)','you');
    controlsEl.innerHTML='';
    const passBtn=document.createElement('button'); passBtn.className='ctlBtn'; passBtn.textContent='패스';
    passBtn.addEventListener('click', ()=>doPass(1));
    controlsEl.appendChild(passBtn);
  };

  function switchTurn(){ turn = turn===1?2:1; }

  function doPass(color){
    if(gameOver||locked||turn!==color) return;
    passes++;
    if(passes>=2){ finishGame(); return; }
    switchTurn();
    if(turn===2){ setTurnInfo('AI 차례 (백)','ai'); locked=true; setTimeout(aiMove,600); }
    else setTurnInfo('내 차례 (흑)','you');
  }

  function placeAt(r,c,color){
    const res = tryMove(board,r,c,color);
    if(!res.legal) return false;
    const str = boardStr(res.board);
    if(history.length>=2 && str===history[history.length-2]) return false; // simple ko
    board = res.board; history.push(str); passes=0;
    return true;
  }

  function scoreAndFinish(){
    const region = Array.from({length:N},()=>new Array(N).fill(false));
    let terr=[0,0]; // index0 black,1 white
    let stoneCount=[0,0];
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){ if(board[r][c]===1) stoneCount[0]++; else if(board[r][c]===2) stoneCount[1]++; }
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      if(board[r][c]!==0 || region[r][c]) continue;
      const stack=[[r,c]]; region[r][c]=true; const cells=[[r,c]]; const borders=new Set();
      while(stack.length){
        const [cr,cc]=stack.pop();
        for(const [nr,nc] of neigh(cr,cc)){
          if(board[nr][nc]===0){ if(!region[nr][nc]){ region[nr][nc]=true; stack.push([nr,nc]); cells.push([nr,nc]); } }
          else borders.add(board[nr][nc]);
        }
      }
      if(borders.size===1){ const who=[...borders][0]; terr[who-1]+=cells.length; }
    }
    const blackTotal = stoneCount[0]+terr[0];
    const whiteTotal = stoneCount[1]+terr[1]+6.5;
    gameOver=true;
    const winner = blackTotal>whiteTotal ? '나(흑)' : 'AI(백)';
    showOverlay(winner+' 승리!\n흑 '+blackTotal.toFixed(1)+' : 백 '+whiteTotal.toFixed(1));
    setTurnInfo('게임 종료','');
  }
  function finishGame(){ scoreAndFinish(); }

  function aiMove(){
    if(gameOver) return;
    let best=null, bestScore=-Infinity;
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      const res=tryMove(board,r,c,2);
      if(!res.legal) continue;
      const str=boardStr(res.board);
      if(history.length>=2 && str===history[history.length-2]) continue;
      let score = res.captured*18 + res.ownLibs*2 + rand(0,4);
      if(res.ownLibs===1 && res.captured===0) score-=25;
      const centerDist = Math.hypot(r-8.5,c-8.5); score += (10-centerDist)*0.4;
      if(score>bestScore){ bestScore=score; best={r,c}; }
    }
    locked=false;
    if(!best || bestScore< -5){ doPass(2); return; }
    placeAt(best.r,best.c,2);
    switchTurn();
    setTurnInfo('내 차례 (흑)','you');
    if(gameOver) return;
  }

  G.step=function(){};
  G.render=function(){
    drawWoodFrame();
    const tg=ctx.createLinearGradient(BX,BY,BX,BY+BS);
    tg.addColorStop(0,'#f2d9a8'); tg.addColorStop(1,'#dcb877');
    ctx.fillStyle=tg; ctx.fillRect(BX-14,BY-14,BS+28,BS+28);
    ctx.strokeStyle='rgba(90,54,22,.85)'; ctx.lineWidth=1.4;
    for(let i=0;i<N;i++){
      const x=BX+i*CELL, y=BY+i*CELL;
      ctx.beginPath(); ctx.moveTo(x,BY); ctx.lineTo(x,BY+(N-1)*CELL); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(BX,y); ctx.lineTo(BX+(N-1)*CELL,y); ctx.stroke();
    }
    [[3,3],[3,14],[14,3],[14,14],[3,9],[9,3],[14,9],[9,14]].forEach(p=>{
      const x=BX+p[1]*CELL, y=BY+p[0]*CELL;
      ctx.beginPath(); ctx.arc(x,y,3.5,0,Math.PI*2); ctx.fillStyle='rgba(90,54,22,.6)'; ctx.fill();
    });
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      if(board[r][c]===0) continue;
      const x=BX+c*CELL, y=BY+r*CELL;
      drawStoneCircle(x,y,STONE_R, board[r][c]===1?'black':'white');
    }
  };
  G.onDown=function(p){
    if(gameOver||locked||turn!==1) return;
    const c = Math.round((p.x-BX)/CELL), r = Math.round((p.y-BY)/CELL);
    if(!inB(r,c)) return;
    const dist = Math.hypot(BX+c*CELL-p.x, BY+r*CELL-p.y);
    if(dist>CELL*0.5) return;
    if(placeAt(r,c,1)){
      switchTurn();
      if(!gameOver){ setTurnInfo('AI 차례 (백)','ai'); locked=true; setTimeout(aiMove,600); }
    }
  };
  return G;
}

/* =========================================================
   3) OMOK (Gomoku, 15x15, vs AI)
   ========================================================= */
function OmokGame(){
  const G = { key:'omok' };
  const N=18, PAD=28, BS=W-PAD*2, CELL=BS/(N-1), BX=PAD, BY=PAD;
  const STONE_R = CELL*0.44;
  let board, turn, gameOver, locked, lastMove;

  function inB(r,c){ return r>=0&&r<N&&c>=0&&c<N; }
  function checkWin(r,c,color){
    const dirs=[[1,0],[0,1],[1,1],[1,-1]];
    for(const [dx,dy] of dirs){
      let cnt=1;
      let x=r+dx,y=c+dy; while(inB(x,y)&&board[x][y]===color){cnt++;x+=dx;y+=dy;}
      x=r-dx;y=c-dy; while(inB(x,y)&&board[x][y]===color){cnt++;x-=dx;y-=dy;}
      if(cnt>=5) return true;
    }
    return false;
  }
  function countLine(r,c,dx,dy,color){
    let cnt=0, x=r+dx,y=c+dy;
    while(inB(x,y)&&board[x][y]===color){ cnt++; x+=dx; y+=dy; }
    const open = inB(x,y)&&board[x][y]===0;
    return {cnt, open};
  }
  function patternValue(total,openEnds){
    if(total>=5) return 1000000;
    if(total===4) return openEnds>=2?100000:(openEnds===1?10000:0);
    if(total===3) return openEnds===2?5000:(openEnds===1?800:0);
    if(total===2) return openEnds===2?200:(openEnds===1?50:0);
    if(total===1) return openEnds===2?10:2;
    return 0;
  }
  function scorePoint(r,c,color){
    if(board[r][c]!==0) return -1;
    let total=0;
    const axes=[[1,0],[0,1],[1,1],[1,-1]];
    for(const [dx,dy] of axes){
      const a=countLine(r,c,dx,dy,color), b=countLine(r,c,-dx,-dy,color);
      const cnt=1+a.cnt+b.cnt; const openEnds=(a.open?1:0)+(b.open?1:0);
      total += patternValue(cnt,openEnds);
    }
    return total;
  }

  G.init=function(){
    gameTitleEl.textContent='오목 (18×18)';
    hintEl.textContent='교차점을 눌러 돌을 놓으세요. 가로·세로·대각선 중 하나로 5개를 먼저 연결하면 승리!';
    board=Array.from({length:N},()=>new Array(N).fill(0));
    turn=1; gameOver=false; locked=false; lastMove=null;
    setTurnInfo('내 차례 (흑)','you');
    controlsEl.innerHTML='';
  };

  function candidates(){
    const set=new Set(); let any=false;
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      if(board[r][c]===0) continue;
      any=true;
      for(let dr=-2;dr<=2;dr++) for(let dc=-2;dc<=2;dc++){
        const nr=r+dr,nc=c+dc;
        if(inB(nr,nc)&&board[nr][nc]===0) set.add(nr*N+nc);
      }
    }
    if(!any) set.add(Math.floor(N/2)*N+Math.floor(N/2));
    return [...set].map(v=>({r:Math.floor(v/N), c:v%N}));
  }

  function aiMove(){
    if(gameOver) return;
    const cands = candidates();
    let best=null, bestScore=-Infinity;
    for(const {r,c} of cands){
      const off = scorePoint(r,c,2);
      const def = scorePoint(r,c,1);
      const score = off + def*0.9 + rand(0,5);
      if(score>bestScore){ bestScore=score; best={r,c}; }
    }
    locked=false;
    if(!best) return;
    board[best.r][best.c]=2; lastMove={r:best.r,c:best.c};
    if(checkWin(best.r,best.c,2)){ gameOver=true; showOverlay('AI(백) 승리!'); setTurnInfo('게임 종료',''); return; }
    if(cands.length<=1){ gameOver=true; showOverlay('무승부'); return; }
    turn=1; setTurnInfo('내 차례 (흑)','you');
  }

  G.step=function(){};
  G.render=function(){
    drawWoodFrame();
    const tg=ctx.createLinearGradient(BX,BY,BX,BY+BS);
    tg.addColorStop(0,'#f2d9a8'); tg.addColorStop(1,'#dcb877');
    ctx.fillStyle=tg; ctx.fillRect(BX-10,BY-10,BS+20,BS+20);
    ctx.strokeStyle='rgba(90,54,22,.8)'; ctx.lineWidth=1;
    for(let i=0;i<N;i++){
      const x=BX+i*CELL, y=BY+i*CELL;
      ctx.beginPath(); ctx.moveTo(x,BY); ctx.lineTo(x,BY+(N-1)*CELL); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(BX,y); ctx.lineTo(BX+(N-1)*CELL,y); ctx.stroke();
    }
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      if(board[r][c]===0) continue;
      const x=BX+c*CELL, y=BY+r*CELL;
      drawStoneCircle(x,y,STONE_R, board[r][c]===1?'black':'white');
      if(lastMove && lastMove.r===r && lastMove.c===c){
        ctx.beginPath(); ctx.arc(x,y,STONE_R*0.4,0,Math.PI*2);
        ctx.strokeStyle='rgba(230,60,60,.9)'; ctx.lineWidth=2; ctx.stroke();
      }
    }
  };
  G.onDown=function(p){
    if(gameOver||locked||turn!==1) return;
    const c=Math.round((p.x-BX)/CELL), r=Math.round((p.y-BY)/CELL);
    if(!inB(r,c) || board[r][c]!==0) return;
    const dist=Math.hypot(BX+c*CELL-p.x, BY+r*CELL-p.y);
    if(dist>CELL*0.5) return;
    board[r][c]=1; lastMove={r,c};
    if(checkWin(r,c,1)){ gameOver=true; showOverlay('나(흑) 승리!'); setTurnInfo('게임 종료',''); return; }
    turn=2; setTurnInfo('AI 차례 (백)','ai'); locked=true; setTimeout(aiMove,550);
  };
  return G;
}

/* =========================================================
   4) JANGGI (Korean Chess, vs AI) - simplified rule set
      (no check/checkmate enforcement or flying-general rule;
       game ends when a general is captured)
   ========================================================= */
function JanggiGame(){
  const G = { key:'janggi' };
  const COLS=9, ROWS=10;
  const PAD=44; const cw=(W-PAD*2)/(COLS-1), ch=(H-PAD*2)/(ROWS-1);
  const BX=PAD, BY=PAD;
  const PR = Math.min(cw,ch)*0.42;
  let board, turn, gameOver, locked, sel, legalDests;

  const HAN_LINES=[[[0,3],[1,4],[2,5]],[[0,5],[1,4],[2,3]]];
  const CHO_LINES=[[[7,3],[8,4],[9,5]],[[7,5],[8,4],[9,3]]];
  function palaceLines(side){ return side==='han'?HAN_LINES:CHO_LINES; }
  function inPalace(r,c,side){
    if(side==='han') return r>=0&&r<=2&&c>=3&&c<=5;
    return r>=7&&r<=9&&c>=3&&c<=5;
  }
  function findLinePoint(r,c,side){
    for(const line of palaceLines(side)){
      const idx = line.findIndex(p=>p[0]===r&&p[1]===c);
      if(idx>=0) return {line, idx};
    }
    return null;
  }
  function inB(r,c){ return r>=0&&r<ROWS&&c>=0&&c<COLS; }
  function pieceAt(r,c){ return board[r][c]; }
  function otherSide(s){ return s==='han'?'cho':'han'; }

  function initBoard(){
    const b = Array.from({length:ROWS},()=>new Array(COLS).fill(null));
    function put(r,c,type,side){ b[r][c]={type,side}; }
    // Han (top, red) side='han'
    put(0,0,'cha','han'); put(0,1,'sang','han'); put(0,2,'ma','han'); put(0,3,'sa','han');
    put(0,5,'sa','han'); put(0,6,'ma','han'); put(0,7,'sang','han'); put(0,8,'cha','han');
    put(1,4,'gung','han');
    put(2,1,'po','han'); put(2,7,'po','han');
    [0,2,4,6,8].forEach(c=>put(3,c,'jol','han'));
    // Cho (bottom, blue) side='cho'
    put(9,0,'cha','cho'); put(9,1,'sang','cho'); put(9,2,'ma','cho'); put(9,3,'sa','cho');
    put(9,5,'sa','cho'); put(9,6,'ma','cho'); put(9,7,'sang','cho'); put(9,8,'cha','cho');
    put(8,4,'gung','cho');
    put(7,1,'po','cho'); put(7,7,'po','cho');
    [0,2,4,6,8].forEach(c=>put(6,c,'jol','cho'));
    return b;
  }

  function slideOrth(r,c,side){
    const moves=[]; const dirs=[[-1,0],[1,0],[0,-1],[0,1]];
    for(const [dr,dc] of dirs){
      let nr=r+dr, nc=c+dc;
      while(inB(nr,nc)){
        const p=pieceAt(nr,nc);
        if(!p){ moves.push([nr,nc]); }
        else { if(p.side!==side) moves.push([nr,nc]); break; }
        nr+=dr; nc+=dc;
      }
    }
    return moves;
  }
  function slidePalaceLines(r,c,side){
    const moves=[];
    for(const pside of ['han','cho']){
      const found = findLinePoint(r,c,pside);
      if(!found) continue;
      const {line, idx} = found;
      for(const dir of [-1,1]){
        let i=idx+dir;
        while(i>=0 && i<line.length){
          const [nr,nc]=line[i];
          const p=pieceAt(nr,nc);
          if(!p){ moves.push([nr,nc]); }
          else { if(p.side!==side) moves.push([nr,nc]); break; }
          i+=dir;
        }
      }
    }
    return moves;
  }
  function genCha(r,c,side){ return slideOrth(r,c,side).concat(slidePalaceLines(r,c,side)); }

  function genPoDir(r,c,side,cells){
    const moves=[];
    let screenIdx=-1;
    for(let i=0;i<cells.length;i++){ if(pieceAt(cells[i][0],cells[i][1])){ screenIdx=i; break; } }
    if(screenIdx===-1) return moves;
    const screen = pieceAt(cells[screenIdx][0],cells[screenIdx][1]);
    if(screen.type==='po') return moves;
    for(let i=screenIdx+1;i<cells.length;i++){
      const p=pieceAt(cells[i][0],cells[i][1]);
      if(!p){ moves.push(cells[i]); continue; }
      if(p.side!==side && p.type!=='po') moves.push(cells[i]);
      break;
    }
    return moves;
  }
  function genPo(r,c,side){
    let moves=[];
    const dirs=[[-1,0],[1,0],[0,-1],[0,1]];
    for(const [dr,dc] of dirs){
      const cells=[]; let nr=r+dr,nc=c+dc;
      while(inB(nr,nc)){ cells.push([nr,nc]); nr+=dr; nc+=dc; }
      moves = moves.concat(genPoDir(r,c,side,cells));
    }
    for(const pside of ['han','cho']){
      const found=findLinePoint(r,c,pside);
      if(!found) continue;
      const {line, idx}=found;
      for(const dir of [-1,1]){
        const cells=[]; let i=idx+dir;
        while(i>=0&&i<line.length){ cells.push(line[i]); i+=dir; }
        if(cells.length===2) moves = moves.concat(genPoDir(r,c,side,cells));
      }
    }
    return moves;
  }
  function genSang(r,c,side){
    const T=[
      [[-1,0],[-2,-1],[-3,-2]], [[-1,0],[-2,1],[-3,2]],
      [[1,0],[2,-1],[3,-2]], [[1,0],[2,1],[3,2]],
      [[0,-1],[-1,-2],[-2,-3]], [[0,-1],[1,-2],[2,-3]],
      [[0,1],[-1,2],[-2,3]], [[0,1],[1,2],[2,3]]
    ];
    const moves=[];
    for(const [m1,m2,d] of T){
      const p1=[r+m1[0],c+m1[1]], p2=[r+m2[0],c+m2[1]], pd=[r+d[0],c+d[1]];
      if(!inB(p1[0],p1[1])||pieceAt(p1[0],p1[1])) continue;
      if(!inB(p2[0],p2[1])||pieceAt(p2[0],p2[1])) continue;
      if(!inB(pd[0],pd[1])) continue;
      const dest=pieceAt(pd[0],pd[1]);
      if(!dest || dest.side!==side) moves.push(pd);
    }
    return moves;
  }
  function genMa(r,c,side){
    const G2=[
      [[-1,0],[[-2,-1],[-2,1]]], [[1,0],[[2,-1],[2,1]]],
      [[0,-1],[[-1,-2],[1,-2]]], [[0,1],[[-1,2],[1,2]]]
    ];
    const moves=[];
    for(const [mid,dests] of G2){
      const pm=[r+mid[0],c+mid[1]];
      if(!inB(pm[0],pm[1])||pieceAt(pm[0],pm[1])) continue;
      for(const d of dests){
        const pd=[r+d[0],c+d[1]];
        if(!inB(pd[0],pd[1])) continue;
        const dest=pieceAt(pd[0],pd[1]);
        if(!dest||dest.side!==side) moves.push(pd);
      }
    }
    return moves;
  }
  function genSaGung(r,c,side){
    const moves=[];
    const dirs=[[-1,0],[1,0],[0,-1],[0,1]];
    for(const [dr,dc] of dirs){
      const nr=r+dr,nc=c+dc;
      if(inPalace(nr,nc,side)){ const p=pieceAt(nr,nc); if(!p||p.side!==side) moves.push([nr,nc]); }
    }
    const found=findLinePoint(r,c,side);
    if(found){
      const {line,idx}=found;
      for(const dir of [-1,1]){
        const i=idx+dir;
        if(i>=0&&i<line.length){
          const [nr,nc]=line[i];
          const p=pieceAt(nr,nc);
          if(!p||p.side!==side) moves.push([nr,nc]);
        }
      }
    }
    return moves;
  }
  function genJol(r,c,side){
    const fwd = side==='cho' ? -1 : 1;
    const moves=[];
    [[fwd,0],[0,-1],[0,1]].forEach(([dr,dc])=>{
      const nr=r+dr,nc=c+dc;
      if(inB(nr,nc)){ const p=pieceAt(nr,nc); if(!p||p.side!==side) moves.push([nr,nc]); }
    });
    const oppSide = otherSide(side);
    const found=findLinePoint(r,c,oppSide);
    if(found){
      const {line,idx}=found;
      for(const i of [idx-1, idx+1]){
        if(i>=0&&i<line.length){
          const [nr,nc]=line[i];
          const p=pieceAt(nr,nc);
          if(!p||p.side!==side) moves.push([nr,nc]);
        }
      }
    }
    return moves;
  }
  function genMoves(r,c){
    const p=pieceAt(r,c); if(!p) return [];
    switch(p.type){
      case 'cha': return genCha(r,c,p.side);
      case 'po': return genPo(r,c,p.side);
      case 'sang': return genSang(r,c,p.side);
      case 'ma': return genMa(r,c,p.side);
      case 'sa': case 'gung': return genSaGung(r,c,p.side);
      case 'jol': return genJol(r,c,p.side);
    }
    return [];
  }

  const GLYPH = { cha:{han:'車',cho:'車'}, po:{han:'包',cho:'包'}, ma:{han:'馬',cho:'馬'},
    sang:{han:'象',cho:'象'}, sa:{han:'士',cho:'士'}, gung:{han:'漢',cho:'楚'}, jol:{han:'兵',cho:'卒'} };
  const VALUE = {gung:0, cha:13, po:7, ma:5, sang:3, sa:3, jol:2};

  G.init=function(){
    gameTitleEl.textContent='장기';
    hintEl.textContent='말을 눌러 선택한 뒤, 표시된 칸으로 이동하세요. 상대 궁을 잡으면 승리! (장군/외통 판정은 단순화되어 있습니다)';
    board = initBoard();
    turn='cho'; gameOver=false; locked=false; sel=null; legalDests=[];
    setTurnInfo('내 차례 (초/파랑)','you');
    controlsEl.innerHTML='';
  };

  function applyMove(from,to){
    const p = board[from[0]][from[1]];
    const captured = board[to[0]][to[1]];
    board[to[0]][to[1]] = p; board[from[0]][from[1]] = null;
    if(captured && captured.type==='gung'){
      gameOver=true;
      showOverlay((p.side==='cho'?'나(초)':'AI(한)')+' 승리!');
      setTurnInfo('게임 종료','');
      return true;
    }
    return false;
  }

  function aiMove(){
    if(gameOver) return;
    let best=null, bestScore=-Infinity;
    for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++){
      const p=board[r][c]; if(!p||p.side!=='han') continue;
      const dests=genMoves(r,c);
      for(const d of dests){
        const target=board[d[0]][d[1]];
        let score = target ? VALUE[target.type]*10 : 0;
        score += rand(0,3);
        if(p.type==='jol') score += (9-d[0])*0.3;
        if(score>bestScore){ bestScore=score; best={from:[r,c], to:d}; }
      }
    }
    locked=false;
    if(!best){ return; }
    const ended = applyMove(best.from,best.to);
    if(!ended){ turn='cho'; setTurnInfo('내 차례 (초/파랑)','you'); }
  }

  G.step=function(){};
  G.render=function(){
    drawWoodFrame();
    const tg=ctx.createLinearGradient(BX,BY,BX,BY+ (ROWS-1)*ch);
    tg.addColorStop(0,'#f2d9a8'); tg.addColorStop(1,'#dcb877');
    ctx.fillStyle=tg; ctx.fillRect(BX-16,BY-16,(COLS-1)*cw+32,(ROWS-1)*ch+32);
    ctx.strokeStyle='rgba(90,54,22,.85)'; ctx.lineWidth=1.3;
    for(let c=0;c<COLS;c++){ ctx.beginPath(); ctx.moveTo(BX+c*cw,BY); ctx.lineTo(BX+c*cw,BY+(ROWS-1)*ch); ctx.stroke(); }
    for(let r=0;r<ROWS;r++){ ctx.beginPath(); ctx.moveTo(BX,BY+r*ch); ctx.lineTo(BX+(COLS-1)*cw,BY+r*ch); ctx.stroke(); }
    [HAN_LINES,CHO_LINES].forEach(lines=>{
      lines.forEach(line=>{
        ctx.beginPath();
        ctx.moveTo(BX+line[0][1]*cw, BY+line[0][0]*ch);
        ctx.lineTo(BX+line[2][1]*cw, BY+line[2][0]*ch);
        ctx.strokeStyle='rgba(90,54,22,.85)'; ctx.lineWidth=1.3; ctx.stroke();
      });
    });
    if(sel){
      const x=BX+sel[1]*cw, y=BY+sel[0]*ch;
      ctx.beginPath(); ctx.arc(x,y,PR*1.25,0,Math.PI*2); ctx.strokeStyle='rgba(232,163,61,.9)'; ctx.lineWidth=3; ctx.stroke();
    }
    for(const d of legalDests){
      const x=BX+d[1]*cw, y=BY+d[0]*ch;
      ctx.beginPath(); ctx.arc(x,y,7,0,Math.PI*2); ctx.fillStyle='rgba(95,191,106,.85)'; ctx.fill();
    }
    for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++){
      const p=board[r][c]; if(!p) continue;
      const x=BX+c*cw, y=BY+r*ch;
      ctx.beginPath(); ctx.arc(x,y,PR,0,Math.PI*2);
      ctx.fillStyle = '#f2e6cf'; ctx.fill();
      ctx.lineWidth=2.4; ctx.strokeStyle = p.side==='han' ? 'var(--red)' : 'var(--blue)';
      ctx.strokeStyle = p.side==='han' ? '#c0392b' : '#1a4fa0';
      ctx.stroke();
      ctx.fillStyle = p.side==='han' ? '#c0392b' : '#1a4fa0';
      ctx.font = (PR*1.15)+'px "Apple SD Gothic Neo","Malgun Gothic",sans-serif';
      ctx.textAlign='center'; ctx.textBaseline='middle';
      ctx.fillText(GLYPH[p.type][p.side], x, y+1);
    }
  };
  G.onDown=function(p){
    if(gameOver||locked||turn!=='cho') return;
    const c=Math.round((p.x-BX)/cw), r=Math.round((p.y-BY)/ch);
    if(!inB(r,c)) return;
    const dist=Math.hypot(BX+c*cw-p.x, BY+r*ch-p.y);
    if(dist>Math.min(cw,ch)*0.5) return;
    const piece = board[r][c];
    if(sel){
      const isDest = legalDests.some(d=>d[0]===r&&d[1]===c);
      if(isDest){
        const ended = applyMove(sel,[r,c]);
        sel=null; legalDests=[];
        if(!ended){ turn='han'; setTurnInfo('AI 차례 (한/빨강)','ai'); locked=true; setTimeout(aiMove,600); }
        return;
      }
    }
    if(piece && piece.side==='cho'){ sel=[r,c]; legalDests=genMoves(r,c); }
    else { sel=null; legalDests=[]; }
  };
  return G;
}

})();
</script>
</body>
</html>
