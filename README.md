<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1, user-scalable=no">
<title>보드게임 아케이드</title>
<script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
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
  #app{ display:flex; flex-direction:column; align-items:center; height:100%; padding:8px 6px; gap:6px; box-sizing:border-box; }

  /* ---- Menu ---- */
  #menuScreen{
    display:flex; flex-direction:column; align-items:center;
    gap:14px; width:100%; max-width:640px; margin-top:2vh; flex:1;
  }
  #menuScreen h1{ margin:0; font-size:clamp(24px,6vw,32px); letter-spacing:3px; font-weight:800; color:var(--accent); }
  #menuScreen p.sub{ margin:0; font-size:14px; color:var(--ink-dim); }
  #cardGrid{
    display:grid; grid-template-columns:1fr 1fr; gap:10px;
    width:100%; padding:0 4px; flex:1; align-content:stretch;
  }
  .card{
    background:linear-gradient(180deg,var(--panel),#241a10);
    border:1px solid rgba(255,255,255,.1);
    border-radius:16px; padding:20px 12px; text-align:center; cursor:pointer;
    box-shadow:0 4px 14px rgba(0,0,0,.4);
    transition:transform .15s, box-shadow .15s;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    min-height:120px;
  }
  .card:active{ transform:scale(.97); }
  .card:hover{ box-shadow:0 0 0 2px var(--accent), 0 8px 18px rgba(0,0,0,.45); }
  .card .emoji{ font-size:clamp(36px,10vw,48px); display:block; margin-bottom:8px; line-height:1; }
  .card .name{ font-size:clamp(16px,4vw,19px); font-weight:800; color:var(--ink); }
  .card .desc{ font-size:clamp(12px,3vw,13px); color:var(--ink-dim); margin-top:6px; line-height:1.45; }

  /* Mode select — big full-width cards */
  #modeScreen{
    display:none; flex-direction:column; align-items:stretch;
    gap:0; width:100%; max-width:640px; margin-top:2vh;
    padding:0 6px; box-sizing:border-box; flex:1;
  }
  #modeScreen h2{
    margin:0 0 12px; font-size:clamp(18px,5vw,22px); color:var(--accent); font-weight:800;
    text-align:center; letter-spacing:1px;
  }
  #modeScreen .modeBtns{
    display:flex; flex-direction:column; gap:10px; width:100%; flex:1;
  }
  .modeBtn{
    font-family:inherit; font-size:16px; font-weight:700; color:var(--ink);
    background:linear-gradient(165deg, #3d2c1c 0%, #241a10 100%);
    border:2px solid rgba(255,255,255,.12); border-radius:16px;
    padding:0; cursor:pointer; text-align:left;
    box-shadow:0 4px 14px rgba(0,0,0,.4), inset 0 1px 0 rgba(255,255,255,.06);
    display:flex; align-items:center; gap:14px;
    min-height:100px; width:100%;
    flex:1;
    transition:transform .12s, box-shadow .12s, border-color .12s;
  }
  .modeBtn:active{ transform:scale(.98); }
  .modeBtn:hover{
    border-color:rgba(232,163,61,.55);
    box-shadow:0 0 0 2px rgba(232,163,61,.3), 0 6px 16px rgba(0,0,0,.45);
  }
  .modeBtn .mIcon{
    flex-shrink:0; width:68px; height:68px; margin-left:14px;
    border-radius:14px; display:flex; align-items:center; justify-content:center;
    font-size:34px; background:rgba(0,0,0,.35);
  }
  .modeBtn .mText{ flex:1; padding:14px 16px 14px 0; min-width:0; }
  .modeBtn .mTitle{ display:block; font-size:clamp(18px,4.5vw,22px); font-weight:800; margin-bottom:4px; line-height:1.2; }
  .modeBtn .mDesc{ display:block; font-size:clamp(13px,3.2vw,15px); color:var(--ink-dim); font-weight:500; line-height:1.35; }
  .modeBtn.ai .mIcon{ background:rgba(224,87,76,.22); }
  .modeBtn.ai .mTitle{ color:#f07870; }
  .modeBtn.p2 .mIcon{ background:rgba(95,191,106,.22); }
  .modeBtn.p2 .mTitle{ color:#7fd48a; }
  .modeBtn.online .mIcon{ background:rgba(110,200,255,.2); }
  .modeBtn.online .mTitle{ color:#7ec8ff; }
  #modeBack{
    font-family:inherit; font-size:15px; font-weight:700; color:var(--ink-dim);
    background:rgba(255,255,255,.08); border:1px solid rgba(255,255,255,.16);
    border-radius:999px; padding:14px 20px; cursor:pointer;
    margin-top:14px; width:100%; text-align:center;
  }
  #modeBack:active{ background:rgba(255,255,255,.14); }

  /* Online lobby */
  #onlineScreen{ display:none; flex-direction:column; align-items:center; gap:14px; width:100%; max-width:520px; margin-top:3vh; padding:0 10px; flex:1; }
  #onlineScreen h2{ margin:0; font-size:clamp(18px,5vw,22px); color:var(--accent); font-weight:800; }
  #onlineStatus{ font-size:14px; color:var(--ink-dim); min-height:20px; text-align:center; }
  #onlineStatus.ok{ color:var(--good); }
  #onlineStatus.err{ color:var(--bad); }
  #roomCodeBox{
    font-size:clamp(26px,8vw,34px); font-weight:900; letter-spacing:4px; color:var(--accent);
    background:rgba(0,0,0,.35); border:1px dashed rgba(232,163,61,.5);
    border-radius:14px; padding:16px 24px; min-width:180px; text-align:center;
  }
  #joinInput{
    font-family:inherit; font-size:18px; font-weight:700; letter-spacing:2px;
    text-align:center; width:100%; max-width:280px; padding:14px 14px;
    border-radius:12px; border:1px solid rgba(255,255,255,.2);
    background:rgba(0,0,0,.3); color:var(--ink); outline:none;
  }
  .onlineActions{ display:flex; flex-wrap:wrap; gap:10px; justify-content:center; width:100%; }
  .onlineActions .ctlBtn{ flex:1; min-width:120px; padding:12px 16px; font-size:14px; }
  #netBadge{
    position:absolute; top:8px; right:8px; font-size:10px; font-weight:700;
    padding:3px 8px; border-radius:999px; background:rgba(0,0,0,.5);
    color:var(--ink-dim); display:none; z-index:5;
  }
  #netBadge.on{ display:block; color:var(--good); }
  #netBadge.wait{ display:block; color:var(--accent); }

  /* ---- Game screen ---- */
  #gameScreen{ display:none; flex-direction:column; align-items:center; width:100%; height:100%; gap:8px; }
  #topbar{ display:flex; align-items:center; justify-content:space-between; width:100%; max-width:560px; }
  #backBtn{
    font-family:inherit; font-size:13px; font-weight:700; color:var(--ink);
    background:rgba(255,255,255,.08); border:1px solid rgba(255,255,255,.15);
    border-radius:999px; padding:8px 14px; cursor:pointer;
  }
  #gameTitle{ font-size:16px; font-weight:800; color:var(--accent); }
  #turnInfo{ font-size:13px; font-weight:700; padding:6px 14px; border-radius:999px; background:rgba(255,255,255,.08); min-width:110px; text-align:center; }
  #turnInfo.you{ color:var(--good); }
  #turnInfo.ai{ color:var(--bad); }

  #boardWrap{ position:relative; width:min(88vmin,560px); height:min(88vmin,560px); flex-shrink:0; }
  canvas{ width:100%; height:100%; display:block; touch-action:none; cursor:pointer; border-radius:10px; }

  #controls{ display:flex; gap:10px; }
  .ctlBtn{
    font-family:inherit; font-size:14px; font-weight:700; color:#4a2f0c;
    background:linear-gradient(180deg,#ffe373,#f6b93b); border:2px solid #b9781a;
    border-radius:999px; padding:10px 20px; cursor:pointer; box-shadow:0 2px 0 rgba(0,0,0,.25);
  }
  .ctlBtn:active{ transform:translateY(1px); box-shadow:none; }
  .ctlBtn.secondary{ background:linear-gradient(180deg,#d9d9d9,#aaa); border-color:#777; color:#222; }

  #overlay{ position:absolute; inset:0; display:none; align-items:center; justify-content:center; flex-direction:column; gap:14px; background:rgba(10,8,6,.72); border-radius:10px; text-align:center; padding:16px; }
  #overlay.show{ display:flex; }
  #overlayText{ font-size:22px; font-weight:900; color:var(--accent); text-shadow:0 2px 6px rgba(0,0,0,.6); white-space:pre-line; }
  #overlayBtns{ display:flex; gap:10px; flex-wrap:wrap; justify-content:center; }
  #hint{ font-size:11px; color:var(--ink-dim); text-align:center; max-width:520px; margin:0; }
  #aiThinking{ font-size:11px; color:var(--bad); min-height:14px; }
  @media (max-width:420px){
    .card{ min-height:110px; padding:16px 8px; }
    .modeBtn{ min-height:88px; }
    .modeBtn .mIcon{ width:56px; height:56px; font-size:28px; margin-left:10px; }
  }
</style>
</head>
<body>
<div id="app">

  <div id="menuScreen">
    <h1>보드게임 아케이드</h1>
    <p class="sub">게임을 선택하세요</p>
    <div id="cardGrid">
      <div class="card" data-game="alkkagi"><span class="emoji">⚪</span><div class="name">알까기</div><div class="desc">돌을 튕겨 상대 돌을<br>판 밖으로 밀어내기</div></div>
      <div class="card" data-game="baduk"><span class="emoji">⚫</span><div class="name">바둑</div><div class="desc">9×9 · 상대 돌을 에워싸<br>더 넓은 집을 차지하기</div></div>
      <div class="card" data-game="janggi"><span class="emoji">🀄</span><div class="name">장기</div><div class="desc">상대 궁(장군)을<br>먼저 잡기</div></div>
      <div class="card" data-game="omok"><span class="emoji">⚫⚪</span><div class="name">오목</div><div class="desc">15×15 · 가로·세로·대각선<br>5개를 먼저 잇기</div></div>
      <div class="card" data-game="chess"><span class="emoji">♞</span><div class="name">체스</div><div class="desc">기물을 움직여 상대<br>킹을 먼저 잡기</div></div>
      <div class="card" data-game="connect4"><span class="emoji">🔴🟡</span><div class="name">커넥트4</div><div class="desc">말을 떨어뜨려 4개를<br>먼저 연결하기</div></div>
      <div class="card" data-game="rhythm"><span class="emoji">🎵</span><div class="name">리듬게임</div><div class="desc">판정선에 닿는 노트를<br>맞춰 콤보 쌓기 · 1인용</div></div>
    </div>
  </div>

  <div id="modeScreen">
    <h2 id="modeGameName">게임</h2>
    <div class="modeBtns">
      <button class="modeBtn ai" data-mode="ai">
        <span class="mIcon">🤖</span>
        <span class="mText">
          <span class="mTitle">컴퓨터와 대결</span>
          <span class="mDesc">AI와 1:1로 플레이</span>
        </span>
      </button>
      <button class="modeBtn p2" data-mode="p2">
        <span class="mIcon">👥</span>
        <span class="mText">
          <span class="mTitle">2인 플레이</span>
          <span class="mDesc">같은 기기 · 번갈아 두기</span>
        </span>
      </button>
      <button class="modeBtn online" data-mode="online">
        <span class="mIcon">🌐</span>
        <span class="mText">
          <span class="mTitle">온라인 멀티</span>
          <span class="mDesc">방 코드로 친구와 실시간</span>
        </span>
      </button>
    </div>
    <button id="modeBack">← 게임 선택으로</button>
  </div>

  <div id="onlineScreen">
    <h2 id="onlineGameName">온라인</h2>
    <div id="onlineStatus">연결 준비 중…</div>
    <div id="hostPanel" style="display:none; flex-direction:column; align-items:center; gap:10px; width:100%;">
      <p style="margin:0; font-size:12px; color:var(--ink-dim);">방 코드를 친구에게 공유하세요</p>
      <div id="roomCodeBox">----</div>
      <button class="ctlBtn" id="copyCodeBtn">코드 복사</button>
      <p style="margin:0; font-size:11px; color:var(--ink-dim);">상대가 참가하면 자동으로 시작합니다</p>
    </div>
    <div id="joinPanel" style="display:none; flex-direction:column; align-items:center; gap:10px; width:100%;">
      <p style="margin:0; font-size:12px; color:var(--ink-dim);">방 코드를 입력하세요</p>
      <input id="joinInput" type="text" maxlength="20" placeholder="방 코드" autocomplete="off" spellcheck="false">
      <button class="ctlBtn" id="joinBtn">참가하기</button>
    </div>
    <div class="onlineActions">
      <button class="ctlBtn" id="createRoomBtn">방 만들기</button>
      <button class="ctlBtn secondary" id="showJoinBtn">방 참가</button>
    </div>
    <button id="onlineBack" class="modeBtn" style="margin-top:8px; text-align:center; padding:8px;">← 모드 선택으로</button>
  </div>

  <div id="gameScreen">
    <div id="topbar">
      <button id="backBtn">← 뒤로</button>
      <div id="gameTitle"></div>
      <div id="turnInfo">-</div>
    </div>
    <div id="boardWrap">
      <div id="netBadge">온라인</div>
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
const modeScreen = document.getElementById('modeScreen');
const onlineScreen = document.getElementById('onlineScreen');
const gameScreen = document.getElementById('gameScreen');
const gameTitleEl = document.getElementById('gameTitle');
const turnInfoEl = document.getElementById('turnInfo');
const overlayEl = document.getElementById('overlay');
const overlayTextEl = document.getElementById('overlayText');
const hintEl = document.getElementById('hint');
const controlsEl = document.getElementById('controls');
const aiThinkingEl = document.getElementById('aiThinking');
const modeGameNameEl = document.getElementById('modeGameName');
const onlineGameNameEl = document.getElementById('onlineGameName');
const onlineStatusEl = document.getElementById('onlineStatus');
const roomCodeBox = document.getElementById('roomCodeBox');
const netBadge = document.getElementById('netBadge');
const hostPanel = document.getElementById('hostPanel');
const joinPanel = document.getElementById('joinPanel');

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
let selectedGame = null;
let playMode = 'ai'; // 'ai' | 'p2' | 'online'

const GAME_NAMES = { alkkagi:'알까기', baduk:'바둑', janggi:'장기', omok:'오목', chess:'체스', connect4:'커넥트4', rhythm:'리듬게임' };
const SOLO_ONLY = new Set(['rhythm']);

function setTurnInfo(text, who){
  turnInfoEl.textContent = text;
  turnInfoEl.className = who==='you' ? 'you' : (who==='ai' || who==='p2' || who==='opp' ? 'ai' : '');
}
function showOverlay(text){ overlayTextEl.textContent = text; overlayEl.classList.add('show'); }
function hideOverlay(){ overlayEl.classList.remove('show'); }

/* ========== Networking (PeerJS / WebRTC DataChannel) ========== */
const Net = {
  peer: null,
  conn: null,
  isHost: false,
  roomId: null,
  ready: false,
  handlers: {},
  setStatus(msg, cls){
    onlineStatusEl.textContent = msg;
    onlineStatusEl.className = cls || '';
  },
  destroy(){
    try{ if(this.conn) this.conn.close(); }catch(e){}
    try{ if(this.peer) this.peer.destroy(); }catch(e){}
    this.peer = null; this.conn = null; this.ready = false; this.isHost = false; this.roomId = null;
    this.handlers = {};
    netBadge.className = '';
  },
  send(obj){
    if(this.conn && this.conn.open){
      try{ this.conn.send(obj); }catch(e){ console.warn(e); }
    }
  },
  on(type, fn){ this.handlers[type] = fn; },
  _dispatch(data){
    if(!data || !data.type) return;
    const fn = this.handlers[data.type];
    if(fn) fn(data);
  },
  _wireConn(c){
    this.conn = c;
    c.on('open', ()=>{
      this.ready = true;
      this.setStatus('상대와 연결됨! 곧 시작합니다…', 'ok');
      netBadge.className = 'on';
      netBadge.textContent = '온라인 연결됨';
      // host starts the game
      if(this.isHost){
        setTimeout(()=>{
          this.send({ type:'start', game: selectedGame });
          launch(selectedGame, 'online');
        }, 400);
      }
    });
    c.on('data', (data)=> this._dispatch(data));
    c.on('close', ()=>{
      this.ready = false;
      this.setStatus('연결이 끊겼습니다', 'err');
      netBadge.className = 'wait';
      netBadge.textContent = '연결 끊김';
      if(current && playMode==='online'){
        showOverlay('상대와 연결이 끊겼습니다');
      }
    });
    c.on('error', (err)=>{
      this.setStatus('연결 오류: '+(err&&err.type||'unknown'), 'err');
    });
  },
  createRoom(){
    this.destroy();
    this.isHost = true;
    this.setStatus('방 만드는 중…');
    hostPanel.style.display = 'flex';
    joinPanel.style.display = 'none';
    // short-ish id
    const id = 'bg'+Math.random().toString(36).slice(2,8);
    this.peer = new Peer(id, { debug: 1 });
    this.peer.on('open', (rid)=>{
      this.roomId = rid;
      roomCodeBox.textContent = rid;
      this.setStatus('방 생성됨 · 상대를 기다리는 중…', 'ok');
      netBadge.className = 'wait';
      netBadge.textContent = '대기 중';
    });
    this.peer.on('connection', (c)=> this._wireConn(c));
    this.peer.on('error', (err)=>{
      this.setStatus('Peer 오류: '+(err&&err.type||err), 'err');
    });
  },
  joinRoom(code){
    code = (code||'').trim();
    if(!code){ this.setStatus('방 코드를 입력하세요', 'err'); return; }
    this.destroy();
    this.isHost = false;
    this.setStatus('연결 중…');
    hostPanel.style.display = 'none';
    joinPanel.style.display = 'flex';
    this.peer = new Peer({ debug: 1 });
    this.peer.on('open', ()=>{
      const c = this.peer.connect(code, { reliable: true });
      this._wireConn(c);
    });
    this.peer.on('error', (err)=>{
      this.setStatus('참가 실패: '+(err&&err.type||'방 코드를 확인하세요'), 'err');
    });
  }
};

Net.on('start', (msg)=>{
  if(!Net.isHost && msg.game){
    launch(msg.game, 'online');
  }
});

function showModeSelect(key){
  selectedGame = key;
  modeGameNameEl.textContent = GAME_NAMES[key] || key;
  menuScreen.style.display = 'none';
  modeScreen.style.display = 'flex';
  onlineScreen.style.display = 'none';
  gameScreen.style.display = 'none';
}
function showOnlineLobby(){
  onlineGameNameEl.textContent = (GAME_NAMES[selectedGame]||'') + ' · 온라인';
  onlineStatusEl.textContent = '방 만들기 또는 참가를 선택하세요';
  onlineStatusEl.className = '';
  hostPanel.style.display = 'none';
  joinPanel.style.display = 'none';
  document.getElementById('joinInput').value = '';
  menuScreen.style.display = 'none';
  modeScreen.style.display = 'none';
  onlineScreen.style.display = 'flex';
  gameScreen.style.display = 'none';
}
function launch(key, mode){
  selectedGame = key;
  playMode = mode || 'ai';
  menuScreen.style.display = 'none';
  modeScreen.style.display = 'none';
  onlineScreen.style.display = 'none';
  gameScreen.style.display = 'flex';
  hideOverlay(); aiThinkingEl.textContent = ''; controlsEl.innerHTML = '';
  if(playMode !== 'online') Net.destroy();
  current = makeGame(key, playMode);
  current.init();
}
function backToMenu(){
  current = null;
  selectedGame = null;
  Net.destroy();
  gameScreen.style.display = 'none';
  modeScreen.style.display = 'none';
  onlineScreen.style.display = 'none';
  menuScreen.style.display = 'flex';
}
function backToMode(){
  current = null;
  if(playMode === 'online') Net.destroy();
  gameScreen.style.display = 'none';
  onlineScreen.style.display = 'none';
  modeScreen.style.display = 'flex';
}

document.querySelectorAll('.card').forEach(c=>{
  c.addEventListener('click', ()=> {
    const key = c.getAttribute('data-game');
    if(SOLO_ONLY.has(key)) launch(key, 'solo');
    else showModeSelect(key);
  });
});
document.querySelectorAll('#modeScreen .modeBtn').forEach(b=>{
  b.addEventListener('click', ()=> {
    if(!selectedGame) return;
    const m = b.getAttribute('data-mode');
    if(m === 'online') showOnlineLobby();
    else launch(selectedGame, m);
  });
});
document.getElementById('modeBack').addEventListener('click', backToMenu);
document.getElementById('onlineBack').addEventListener('click', ()=>{
  Net.destroy();
  onlineScreen.style.display = 'none';
  modeScreen.style.display = 'flex';
});
document.getElementById('createRoomBtn').addEventListener('click', ()=> Net.createRoom());
document.getElementById('showJoinBtn').addEventListener('click', ()=>{
  hostPanel.style.display = 'none';
  joinPanel.style.display = 'flex';
  onlineStatusEl.textContent = '방 코드를 입력하세요';
  onlineStatusEl.className = '';
});
document.getElementById('joinBtn').addEventListener('click', ()=>{
  Net.joinRoom(document.getElementById('joinInput').value);
});
document.getElementById('joinInput').addEventListener('keydown', (e)=>{
  if(e.key === 'Enter') Net.joinRoom(document.getElementById('joinInput').value);
});
document.getElementById('copyCodeBtn').addEventListener('click', ()=>{
  const t = roomCodeBox.textContent;
  if(navigator.clipboard) navigator.clipboard.writeText(t).then(()=> Net.setStatus('코드가 복사되었습니다', 'ok'));
  else Net.setStatus('코드를 직접 복사하세요: '+t, 'ok');
});
document.getElementById('backBtn').addEventListener('click', ()=>{
  if(onlineScreen.style.display === 'flex'){
    Net.destroy();
    onlineScreen.style.display = 'none';
    modeScreen.style.display = 'flex';
  } else if(modeScreen.style.display === 'flex') backToMenu();
  else if(current && SOLO_ONLY.has(current.key)) backToMenu();
  else backToMode();
});
document.getElementById('menuBtn2').addEventListener('click', backToMenu);
document.getElementById('retryBtn').addEventListener('click', ()=>{
  if(playMode === 'online'){
    showOverlay('온라인 게임은 다시 방을 만들어 주세요');
    return;
  }
  if(current && current.key) launch(current.key, playMode);
});

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
function onKeyDown(e){
  if(current && current.onKeyDown){
    const handled = current.onKeyDown(e.key);
    if(handled) e.preventDefault();
  }
}
window.addEventListener('keydown', onKeyDown);
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
function makeGame(key, mode){
  if(key==='alkkagi') return AlkkagiGame(mode);
  if(key==='baduk') return BadukGame(mode);
  if(key==='omok') return OmokGame(mode);
  if(key==='janggi') return JanggiGame(mode);
  if(key==='chess') return ChessGame(mode);
  if(key==='connect4') return Connect4Game(mode);
  if(key==='rhythm') return RhythmGame(mode);
}

/* =========================================================
   1) ALKKAGI
   ========================================================= */
function AlkkagiGame(mode){
  const isP2 = mode === 'p2';
  const isOnline = mode === 'online';
  const amHost = isOnline ? Net.isHost : true;
  const myColor = isOnline ? (amHost ? 'white' : 'black') : 'white';
  const G = { key:'alkkagi' };
  const PAD=66, DIV=12, BX=PAD, BY=PAD, BS=W-PAD*2, CELL=BS/DIV;
  const STONE_R=15, OUT_MARGIN=26, FRICTION=0.983, STOP_SPEED=0.045;
  const MAX_PULL=120, POWER=0.14, MAX_SPEED=25;
  let stones=[], turn='white', moving=false, gameOver=false, drag=null, aiTimer=null;

  function gridPt(cx,cy){ return {x:BX+cx*CELL, y:BY+cy*CELL}; }

  function updateTurnUI(){
    if(isOnline){
      if(turn===myColor) setTurnInfo('내 차례 ('+(myColor==='white'?'백':'흑')+')','you');
      else setTurnInfo('상대 차례','opp');
    } else if(isP2){
      if(turn==='white') setTurnInfo('P1 차례 (백)','you');
      else setTurnInfo('P2 차례 (흑)','p2');
    } else setTurnInfo(turn==='white'?'내 차례 (백)':'AI 차례 (흑)', turn==='white'?'you':'ai');
  }
  G.init = function(){
    if(isOnline) gameTitleEl.textContent = '알까기 · 온라인';
    else if(isP2) gameTitleEl.textContent = '알까기 · 2인';
    else gameTitleEl.textContent = '알까기';
    if(isOnline) hintEl.textContent = (amHost?'당신=백':'당신=흑')+' · 자기 돌을 당겼다가 놓으면 튕깁니다. 상대 돌을 판 밖으로 밀어내세요.';
    else if(isP2) hintEl.textContent = '플레이어1(백) / 플레이어2(흑). 자기 돌을 당겼다가 놓으면 튕겨 나갑니다. 상대 돌을 판 밖으로 밀어내세요.';
    else hintEl.textContent = '내 돌(백)을 당겼다가 놓으면 튕겨 나갑니다. 상대 돌을 판 밖으로 밀어내세요.';
    stones=[];
    const wp=[[4,3],[6,2],[8,3],[5,4],[7,4],[6,5]];
    const bp=[[4,9],[6,10],[8,9],[5,8],[7,8],[6,7]];
    wp.forEach((p)=>{ const g=gridPt(p[0],p[1]); stones.push({x:g.x,y:g.y,vx:0,vy:0,color:'white',alive:true,fallT:0}); });
    bp.forEach((p)=>{ const g=gridPt(p[0],p[1]); stones.push({x:g.x,y:g.y,vx:0,vy:0,color:'black',alive:true,fallT:0}); });
    turn='white'; moving=false; gameOver=false; drag=null;
    updateTurnUI();
    if(isOnline){
      Net.on('shot', (msg)=>{
        if(gameOver||moving) return;
        const s = stones[msg.idx];
        if(!s || !s.alive || s.color!==turn) return;
        s.vx = msg.vx; s.vy = msg.vy;
        moving = true;
      });
    }
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
      if(isOnline){
        const winnerColor = wc===0 ? 'black' : 'white';
        showOverlay((winnerColor===myColor?'나':'상대')+' 승리!');
      } else if(isP2){
        showOverlay((wc===0 ? '플레이어2 (흑)' : '플레이어1 (백)') + ' 승리!');
      } else {
        showOverlay((wc===0?'상대(흑)':'나(백)')+' 승리!');
      }
      setTurnInfo('게임 종료','');
      return;
    }
    turn = turn==='white' ? 'black' : 'white';
    updateTurnUI();
    if(!isOnline && !isP2 && turn==='black') aiTimer=setTimeout(aiMove, 650);
  }

  function aiMove(){
    if(gameOver || isP2 || isOnline) return;
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
    if(gameOver||moving) return;
    if(isOnline){ if(turn!==myColor) return; }
    else if(!isP2 && turn!=='white') return;
    const allowed = (isP2 || isOnline) ? turn : 'white';
    for(let i=0;i<stones.length;i++){
      const s = stones[i];
      if(!s.alive||s.fallT>0||s.color!==allowed) continue;
      if(Math.hypot(s.x-p.x,s.y-p.y)<=STONE_R*1.5){ drag={stone:s, idx:i, startX:p.x,startY:p.y,curX:p.x,curY:p.y}; return; }
    }
  };
  G.onMove = function(p){ if(drag){ drag.curX=p.x; drag.curY=p.y; } };
  G.onUp = function(){
    if(!drag) return;
    const dx=drag.curX-drag.startX, dy=drag.curY-drag.startY;
    const dist=Math.min(Math.hypot(dx,dy),MAX_PULL);
    if(dist>6){
      const ang=Math.atan2(dy,dx); const speed=Math.min(dist*POWER,MAX_SPEED);
      const vx=-Math.cos(ang)*speed, vy=-Math.sin(ang)*speed;
      drag.stone.vx=vx; drag.stone.vy=vy; moving=true;
      if(isOnline) Net.send({ type:'shot', idx:drag.idx, vx, vy });
    }
    drag=null;
  };
  return G;
}

/* =========================================================
   2) BADUK (Go, 9x9)
   ========================================================= */
function BadukGame(mode){
  const isP2 = mode === 'p2';
  const isOnline = mode === 'online';
  const amHost = isOnline ? Net.isHost : true;
  const myColor = isOnline ? (amHost ? 1 : 2) : 1;
  const G = { key:'baduk' };
  const N=9, PAD=40, BS=W-PAD*2, CELL=BS/(N-1), BX=PAD, BY=PAD;
  const STONE_R = CELL*0.44;
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

  function updateTurnUI(){
    if(isOnline){
      if(turn===myColor) setTurnInfo('내 차례 ('+(myColor===1?'흑':'백')+')','you');
      else setTurnInfo('상대 차례','opp');
    } else if(isP2){
      if(turn===1) setTurnInfo('P1 차례 (흑)','you');
      else setTurnInfo('P2 차례 (백)','p2');
    } else setTurnInfo(turn===1?'내 차례 (흑)':'AI 차례 (백)', turn===1?'you':'ai');
  }
  G.init = function(){
    if(isOnline) gameTitleEl.textContent = '바둑 (9×9) · 온라인';
    else if(isP2) gameTitleEl.textContent = '바둑 (9×9) · 2인';
    else gameTitleEl.textContent = '바둑 (9×9)';
    if(isOnline) hintEl.textContent = (amHost?'당신=흑(선공)':'당신=백')+' · 교차점 클릭 / 패스. 두 번 연속 패스 시 집계산.';
    else if(isP2) hintEl.textContent = '교차점을 눌러 돌을 놓으세요. P1(흑) / P2(백). 두 번 연속 패스하면 집계산으로 종료됩니다.';
    else hintEl.textContent = '교차점을 눌러 돌을 놓으세요. 상대 돌을 완전히 에워싸면 잡을 수 있습니다. 두 번 연속 패스하면 집계산으로 종료됩니다.';
    board = Array.from({length:N},()=>new Array(N).fill(0));
    turn=1; history=[boardStr(board)]; passes=0; gameOver=false; locked=false;
    updateTurnUI();
    controlsEl.innerHTML='';
    const passBtn=document.createElement('button'); passBtn.className='ctlBtn'; passBtn.textContent='패스';
    passBtn.addEventListener('click', ()=>{
      if(isOnline && turn!==myColor) return;
      doPass(turn, false);
    });
    controlsEl.appendChild(passBtn);
    if(isOnline){
      Net.on('move', (msg)=>{
        if(gameOver) return;
        if(msg.pass){ doPass(msg.color, true); return; }
        if(msg.color===turn && placeAt(msg.r,msg.c,msg.color)){
          switchTurn(); updateTurnUI();
        }
      });
    }
  };

  function switchTurn(){ turn = turn===1?2:1; }

  function doPass(color, fromNet){
    if(gameOver||locked||turn!==color) return;
    passes++;
    if(passes>=2){ finishGame(); return; }
    switchTurn();
    updateTurnUI();
    if(!fromNet && isOnline) Net.send({ type:'move', pass:true, color });
    if(!isOnline && !isP2 && turn===2){ locked=true; setTimeout(aiMove,600); }
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
    let winner;
    if(isOnline) winner = blackTotal>whiteTotal ? (myColor===1?'나':'상대')+' (흑)' : (myColor===2?'나':'상대')+' (백)';
    else if(isP2) winner = blackTotal>whiteTotal ? '플레이어1 (흑)' : '플레이어2 (백)';
    else winner = blackTotal>whiteTotal ? '나(흑)' : 'AI(백)';
    showOverlay(winner+' 승리!\n흑 '+blackTotal.toFixed(1)+' : 백 '+whiteTotal.toFixed(1));
    setTurnInfo('게임 종료','');
  }
  function finishGame(){ scoreAndFinish(); }

  function aiMove(){
    if(gameOver || isP2 || isOnline) return;
    let best=null, bestScore=-Infinity;
    const mid = (N-1)/2;
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      const res=tryMove(board,r,c,2);
      if(!res.legal) continue;
      const str=boardStr(res.board);
      if(history.length>=2 && str===history[history.length-2]) continue;
      let score = res.captured*20 + res.ownLibs*2.5 + rand(0,4);
      if(res.ownLibs===1 && res.captured===0) score-=30;
      const centerDist = Math.hypot(r-mid,c-mid); score += (mid+1-centerDist)*0.6;
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
    [[2,2],[2,6],[6,2],[6,6],[4,4]].forEach(p=>{
      const x=BX+p[1]*CELL, y=BY+p[0]*CELL;
      ctx.beginPath(); ctx.arc(x,y,3.2,0,Math.PI*2); ctx.fillStyle='rgba(90,54,22,.65)'; ctx.fill();
    });
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      if(board[r][c]===0) continue;
      const x=BX+c*CELL, y=BY+r*CELL;
      drawStoneCircle(x,y,STONE_R, board[r][c]===1?'black':'white');
    }
  };
  G.onDown=function(p){
    if(gameOver||locked) return;
    if(isOnline){ if(turn!==myColor) return; }
    else if(!isP2 && turn!==1) return;
    const c = Math.round((p.x-BX)/CELL), r = Math.round((p.y-BY)/CELL);
    if(!inB(r,c)) return;
    const dist = Math.hypot(BX+c*CELL-p.x, BY+r*CELL-p.y);
    if(dist>CELL*0.5) return;
    const color = turn;
    if(placeAt(r,c,color)){
      switchTurn();
      updateTurnUI();
      if(isOnline) Net.send({ type:'move', r, c, color });
      if(!isOnline && !isP2 && turn===2){ locked=true; setTimeout(aiMove,600); }
    }
  };
  return G;
}

/* =========================================================
   3) OMOK (Gomoku, 15x15)
   ========================================================= */
function OmokGame(mode){
  const isP2 = mode === 'p2';
  const isOnline = mode === 'online';
  const amHost = isOnline ? Net.isHost : true; // host = black = P1
  const myColor = isOnline ? (amHost ? 1 : 2) : 1;
  const G = { key:'omok' };
  const N=15, PAD=32, BS=W-PAD*2, CELL=BS/(N-1), BX=PAD, BY=PAD;
  const STONE_R = CELL*0.43;
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
    if(isOnline) gameTitleEl.textContent = '오목 (15×15) · 온라인';
    else if(isP2) gameTitleEl.textContent = '오목 (15×15) · 2인';
    else gameTitleEl.textContent = '오목 (15×15)';
    if(isOnline) hintEl.textContent = (amHost?'당신=흑(선공)':'당신=백')+' · 교차점을 눌러 돌을 놓으세요. 5개 연결 시 승리!';
    else if(isP2) hintEl.textContent = '교차점을 눌러 돌을 놓으세요. P1(흑) / P2(백). 5개를 먼저 연결하면 승리!';
    else hintEl.textContent = '교차점을 눌러 돌을 놓으세요. 가로·세로·대각선 중 하나로 5개를 먼저 연결하면 승리!';
    board=Array.from({length:N},()=>new Array(N).fill(0));
    turn=1; gameOver=false; locked=false; lastMove=null;
    updateTurnUI();
    controlsEl.innerHTML='';
    if(isOnline){
      Net.on('move', (msg)=>{
        if(gameOver || msg.color !== turn) return;
        applyPlace(msg.r, msg.c, msg.color, true);
      });
    }
  };
  function updateTurnUI(){
    if(isOnline){
      if(turn === myColor) setTurnInfo('내 차례 ('+(myColor===1?'흑':'백')+')','you');
      else setTurnInfo('상대 차례','opp');
    } else if(isP2){
      if(turn===1) setTurnInfo('P1 차례 (흑)','you');
      else setTurnInfo('P2 차례 (백)','p2');
    } else setTurnInfo('내 차례 (흑)','you');
  }
  function applyPlace(r,c,color, fromNet){
    if(board[r][c]!==0) return false;
    board[r][c]=color; lastMove={r,c};
    if(checkWin(r,c,color)){
      gameOver=true;
      if(isOnline) showOverlay((color===myColor?'나':'상대')+' 승리!');
      else if(isP2) showOverlay((color===1?'플레이어1 (흑)':'플레이어2 (백)')+' 승리!');
      else showOverlay(color===1?'나(흑) 승리!':'AI(백) 승리!');
      setTurnInfo('게임 종료','');
      return true;
    }
    let empty=0;
    for(let rr=0;rr<N;rr++) for(let cc=0;cc<N;cc++) if(board[rr][cc]===0) empty++;
    if(empty===0){ gameOver=true; showOverlay('무승부'); setTurnInfo('게임 종료',''); return true; }
    turn = turn===1 ? 2 : 1;
    updateTurnUI();
    if(!fromNet && isOnline) Net.send({ type:'move', r, c, color });
    if(!isOnline && !isP2 && turn===2){ locked=true; setTimeout(aiMove,550); }
    return true;
  }

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
    if(gameOver || isP2 || isOnline) return;
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
    applyPlace(best.r, best.c, 2, true);
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
    if(gameOver||locked) return;
    if(isOnline){
      if(turn !== myColor) return;
    } else if(!isP2 && turn!==1) return;
    const c=Math.round((p.x-BX)/CELL), r=Math.round((p.y-BY)/CELL);
    if(!inB(r,c) || board[r][c]!==0) return;
    const dist=Math.hypot(BX+c*CELL-p.x, BY+r*CELL-p.y);
    if(dist>CELL*0.5) return;
    applyPlace(r, c, turn, false);
  };
  return G;
}

/* =========================================================
   4) JANGGI (Korean Chess) - simplified rule set
      (no check/checkmate enforcement or flying-general rule;
       game ends when a general is captured)
   ========================================================= */
function JanggiGame(mode){
  const isP2 = mode === 'p2';
  const isOnline = mode === 'online';
  const amHost = isOnline ? Net.isHost : true; // host = cho
  const mySide = isOnline ? (amHost ? 'cho' : 'han') : 'cho';
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

  function updateTurnUI(){
    if(isOnline){
      if(turn===mySide) setTurnInfo('내 차례 ('+(mySide==='cho'?'초':'한')+')','you');
      else setTurnInfo('상대 차례','opp');
    } else if(isP2){
      if(turn==='cho') setTurnInfo('P1 차례 (초)','you');
      else setTurnInfo('P2 차례 (한)','p2');
    } else setTurnInfo(turn==='cho'?'내 차례 (초/파랑)':'AI 차례 (한/빨강)', turn==='cho'?'you':'ai');
  }
  G.init=function(){
    if(isOnline) gameTitleEl.textContent = '장기 · 온라인';
    else if(isP2) gameTitleEl.textContent = '장기 · 2인';
    else gameTitleEl.textContent = '장기';
    if(isOnline) hintEl.textContent = (amHost?'당신=초(파랑)':'당신=한(빨강)')+' · 말을 선택 후 이동. 상대 궁을 잡으면 승리!';
    else if(isP2) hintEl.textContent = '말을 눌러 선택한 뒤 이동하세요. P1(초/파랑) / P2(한/빨강). 상대 궁을 잡으면 승리!';
    else hintEl.textContent = '말을 눌러 선택한 뒤, 표시된 칸으로 이동하세요. 상대 궁을 잡으면 승리! (장군/외통 판정은 단순화되어 있습니다)';
    board = initBoard();
    turn='cho'; gameOver=false; locked=false; sel=null; legalDests=[];
    updateTurnUI();
    controlsEl.innerHTML='';
    if(isOnline){
      Net.on('move', (msg)=>{
        if(gameOver) return;
        const ended = applyMove(msg.from, msg.to, true);
        if(!ended){ turn = turn==='cho'?'han':'cho'; updateTurnUI(); }
      });
    }
  };

  function applyMove(from,to, fromNet){
    const p = board[from[0]][from[1]];
    if(!p) return false;
    const captured = board[to[0]][to[1]];
    board[to[0]][to[1]] = p; board[from[0]][from[1]] = null;
    if(captured && captured.type==='gung'){
      gameOver=true;
      if(isOnline) showOverlay((p.side===mySide?'나':'상대')+' 승리!');
      else if(isP2) showOverlay((p.side==='cho'?'플레이어1 (초)':'플레이어2 (한)')+' 승리!');
      else showOverlay((p.side==='cho'?'나(초)':'AI(한)')+' 승리!');
      setTurnInfo('게임 종료','');
      return true;
    }
    return false;
  }

  function aiMove(){
    if(gameOver || isP2 || isOnline) return;
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
    const ended = applyMove(best.from,best.to, true);
    if(!ended){ turn='cho'; updateTurnUI(); }
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
    if(gameOver||locked) return;
    if(isOnline){ if(turn!==mySide) return; }
    else if(!isP2 && turn!=='cho') return;
    const c=Math.round((p.x-BX)/cw), r=Math.round((p.y-BY)/ch);
    if(!inB(r,c)) return;
    const dist=Math.hypot(BX+c*cw-p.x, BY+r*ch-p.y);
    if(dist>Math.min(cw,ch)*0.5) return;
    const piece = board[r][c];
    if(sel){
      const isDest = legalDests.some(d=>d[0]===r&&d[1]===c);
      if(isDest){
        const from = sel.slice();
        const ended = applyMove(from,[r,c], false);
        sel=null; legalDests=[];
        if(!ended){
          turn = turn==='cho' ? 'han' : 'cho';
          updateTurnUI();
          if(isOnline) Net.send({ type:'move', from, to:[r,c] });
          else if(!isP2){ locked=true; setTimeout(aiMove,600); }
        } else if(isOnline){
          Net.send({ type:'move', from, to:[r,c] });
        }
        return;
      }
    }
    if(piece && piece.side===turn){ sel=[r,c]; legalDests=genMoves(r,c); }
    else { sel=null; legalDests=[]; }
  };
  return G;
}

/* =========================================================
   5) CHESS
   ========================================================= */
function ChessGame(mode){
  const isP2 = mode === 'p2';
  const isOnline = mode === 'online';
  const amHost = isOnline ? Net.isHost : true;
  const mySide = isOnline ? (amHost ? 'white' : 'black') : 'white';
  const G = { key:'chess' };
  const N=8, PAD=40, BS=W-PAD*2, CELL=BS/N, BX=PAD, BY=PAD;
  let board, turn, gameOver, locked, sel, legalDests;

  function inB(r,c){ return r>=0&&r<N&&c>=0&&c<N; }
  function pieceAt(r,c){ return board[r][c]; }

  function initBoard(){
    const b=Array.from({length:N},()=>new Array(N).fill(null));
    const back=['rook','knight','bishop','queen','king','bishop','knight','rook'];
    for(let c=0;c<N;c++){
      b[0][c] = {type:back[c], side:'black'};
      b[1][c] = {type:'pawn', side:'black'};
      b[6][c] = {type:'pawn', side:'white'};
      b[7][c] = {type:back[c], side:'white'};
    }
    return b;
  }

  function slide(r,c,side,dirs){
    const moves=[];
    for(const [dr,dc] of dirs){
      let nr=r+dr,nc=c+dc;
      while(inB(nr,nc)){
        const p=pieceAt(nr,nc);
        if(!p){ moves.push([nr,nc]); }
        else { if(p.side!==side) moves.push([nr,nc]); break; }
        nr+=dr; nc+=dc;
      }
    }
    return moves;
  }
  function genRook(r,c,side){ return slide(r,c,side,[[1,0],[-1,0],[0,1],[0,-1]]); }
  function genBishop(r,c,side){ return slide(r,c,side,[[1,1],[1,-1],[-1,1],[-1,-1]]); }
  function genQueen(r,c,side){ return genRook(r,c,side).concat(genBishop(r,c,side)); }
  function genKnight(r,c,side){
    const deltas=[[1,2],[2,1],[-1,2],[-2,1],[1,-2],[2,-1],[-1,-2],[-2,-1]];
    const moves=[];
    for(const [dr,dc] of deltas){
      const nr=r+dr,nc=c+dc; if(!inB(nr,nc)) continue;
      const p=pieceAt(nr,nc); if(!p||p.side!==side) moves.push([nr,nc]);
    }
    return moves;
  }
  function genKing(r,c,side){
    const moves=[];
    for(let dr=-1;dr<=1;dr++) for(let dc=-1;dc<=1;dc++){
      if(dr===0&&dc===0) continue;
      const nr=r+dr,nc=c+dc; if(!inB(nr,nc)) continue;
      const p=pieceAt(nr,nc); if(!p||p.side!==side) moves.push([nr,nc]);
    }
    return moves;
  }
  function genPawn(r,c,side){
    const moves=[];
    const fwd = side==='white' ? -1 : 1;
    const startRow = side==='white' ? 6 : 1;
    const nr1=r+fwd;
    if(inB(nr1,c) && !pieceAt(nr1,c)){
      moves.push([nr1,c]);
      const nr2=r+fwd*2;
      if(r===startRow && !pieceAt(nr2,c)) moves.push([nr2,c]);
    }
    for(const dc of [-1,1]){
      const nr=r+fwd, nc=c+dc;
      if(inB(nr,nc)){ const p=pieceAt(nr,nc); if(p && p.side!==side) moves.push([nr,nc]); }
    }
    return moves;
  }
  function genMoves(r,c){
    const p=pieceAt(r,c); if(!p) return [];
    switch(p.type){
      case 'rook': return genRook(r,c,p.side);
      case 'bishop': return genBishop(r,c,p.side);
      case 'queen': return genQueen(r,c,p.side);
      case 'knight': return genKnight(r,c,p.side);
      case 'king': return genKing(r,c,p.side);
      case 'pawn': return genPawn(r,c,p.side);
    }
    return [];
  }

  const GLYPH = {
    white:{king:'♔',queen:'♕',rook:'♖',bishop:'♗',knight:'♘',pawn:'♙'},
    black:{king:'♚',queen:'♛',rook:'♜',bishop:'♝',knight:'♞',pawn:'♟'}
  };
  const VALUE = {king:0,queen:9,rook:5,bishop:3,knight:3,pawn:1};

  function updateTurnUI(){
    if(isOnline){
      if(turn===mySide) setTurnInfo('내 차례 ('+(mySide==='white'?'백':'흑')+')','you');
      else setTurnInfo('상대 차례','opp');
    } else if(isP2){
      if(turn==='white') setTurnInfo('P1 차례 (백)','you');
      else setTurnInfo('P2 차례 (흑)','p2');
    } else setTurnInfo(turn==='white'?'내 차례 (백)':'AI 차례 (흑)', turn==='white'?'you':'ai');
  }

  G.init=function(){
    if(isOnline) gameTitleEl.textContent='체스 · 온라인';
    else if(isP2) gameTitleEl.textContent='체스 · 2인';
    else gameTitleEl.textContent='체스';
    if(isOnline) hintEl.textContent=(amHost?'당신=백':'당신=흑')+' · 기물을 선택 후 이동. 상대 킹을 잡으면 승리! (체크/체크메이트 판정은 단순화)';
    else if(isP2) hintEl.textContent='기물을 눌러 선택한 뒤 이동하세요. P1(백)/P2(흑). 상대 킹을 잡으면 승리!';
    else hintEl.textContent='기물을 눌러 선택한 뒤, 표시된 칸으로 이동하세요. 상대 킹을 잡으면 승리! (체크/체크메이트 판정은 단순화되어 있습니다)';
    board=initBoard();
    turn='white'; gameOver=false; locked=false; sel=null; legalDests=[];
    updateTurnUI();
    controlsEl.innerHTML='';
    if(isOnline){
      Net.on('move', (msg)=>{
        if(gameOver) return;
        const ended=applyMove(msg.from,msg.to);
        if(!ended){ turn=turn==='white'?'black':'white'; updateTurnUI(); }
      });
    }
  };

  function applyMove(from,to){
    const p=board[from[0]][from[1]];
    if(!p) return false;
    const captured=board[to[0]][to[1]];
    board[to[0]][to[1]]=p; board[from[0]][from[1]]=null;
    if(p.type==='pawn'){
      if((p.side==='white'&&to[0]===0)||(p.side==='black'&&to[0]===N-1)) p.type='queen';
    }
    if(captured && captured.type==='king'){
      gameOver=true;
      if(isOnline) showOverlay((p.side===mySide?'나':'상대')+' 승리!');
      else if(isP2) showOverlay((p.side==='white'?'플레이어1 (백)':'플레이어2 (흑)')+' 승리!');
      else showOverlay((p.side==='white'?'나(백)':'AI(흑)')+' 승리!');
      setTurnInfo('게임 종료','');
      return true;
    }
    return false;
  }

  function aiMove(){
    if(gameOver||isP2||isOnline) return;
    let best=null,bestScore=-Infinity;
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      const p=board[r][c]; if(!p||p.side!=='black') continue;
      const dests=genMoves(r,c);
      for(const d of dests){
        const target=board[d[0]][d[1]];
        let score = target?VALUE[target.type]*10:0;
        score += rand(0,3);
        if(p.type==='pawn') score += d[0]*0.3;
        if(score>bestScore){ bestScore=score; best={from:[r,c],to:d}; }
      }
    }
    locked=false;
    if(!best) return;
    const ended=applyMove(best.from,best.to);
    if(!ended){ turn='white'; updateTurnUI(); }
  }

  G.step=function(){};
  G.render=function(){
    drawWoodFrame();
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      const x=BX+c*CELL, y=BY+r*CELL;
      ctx.fillStyle = (r+c)%2===0 ? '#f2d9a8' : '#c99a5c';
      ctx.fillRect(x,y,CELL,CELL);
    }
    ctx.strokeStyle='rgba(90,54,22,.9)'; ctx.lineWidth=3; ctx.strokeRect(BX,BY,N*CELL,N*CELL);
    if(sel){
      const x=BX+sel[1]*CELL, y=BY+sel[0]*CELL;
      ctx.fillStyle='rgba(232,163,61,.35)'; ctx.fillRect(x,y,CELL,CELL);
    }
    for(const d of legalDests){
      const x=BX+d[1]*CELL+CELL/2, y=BY+d[0]*CELL+CELL/2;
      ctx.beginPath(); ctx.arc(x,y,7,0,Math.PI*2); ctx.fillStyle='rgba(95,191,106,.85)'; ctx.fill();
    }
    for(let r=0;r<N;r++) for(let c=0;c<N;c++){
      const p=board[r][c]; if(!p) continue;
      const x=BX+c*CELL+CELL/2, y=BY+r*CELL+CELL/2;
      ctx.font=(CELL*0.68)+'px "Apple SD Gothic Neo",sans-serif';
      ctx.textAlign='center'; ctx.textBaseline='middle';
      ctx.fillStyle = p.side==='white' ? '#f7f2e6' : '#1c1c1c';
      ctx.strokeStyle = p.side==='white' ? 'rgba(0,0,0,.5)' : 'rgba(255,255,255,.35)';
      ctx.lineWidth=1.4;
      ctx.fillText(GLYPH[p.side][p.type], x, y+2);
      ctx.strokeText(GLYPH[p.side][p.type], x, y+2);
    }
  };
  G.onDown=function(p){
    if(gameOver||locked) return;
    if(isOnline){ if(turn!==mySide) return; }
    else if(!isP2 && turn!=='white') return;
    const c=Math.floor((p.x-BX)/CELL), r=Math.floor((p.y-BY)/CELL);
    if(!inB(r,c)) return;
    const piece=board[r][c];
    if(sel){
      const isDest=legalDests.some(d=>d[0]===r&&d[1]===c);
      if(isDest){
        const from=sel.slice();
        const ended=applyMove(from,[r,c]);
        sel=null; legalDests=[];
        if(!ended){
          turn = turn==='white'?'black':'white';
          updateTurnUI();
          if(isOnline) Net.send({type:'move',from,to:[r,c]});
          else if(!isP2){ locked=true; setTimeout(aiMove,600); }
        } else if(isOnline){
          Net.send({type:'move',from,to:[r,c]});
        }
        return;
      }
    }
    if(piece && piece.side===turn){ sel=[r,c]; legalDests=genMoves(r,c); }
    else { sel=null; legalDests=[]; }
  };
  return G;
}

/* =========================================================
   6) CONNECT 4
   ========================================================= */
function Connect4Game(mode){
  const isP2 = mode === 'p2';
  const isOnline = mode === 'online';
  const amHost = isOnline ? Net.isHost : true;
  const myColor = isOnline ? (amHost ? 1 : 2) : 1;
  const G = { key:'connect4' };
  const COLS=7, ROWS=6, PAD=30;
  const availW=W-PAD*2, availH=H-PAD*2;
  const CELL=Math.min(availW/COLS, availH/ROWS);
  const BW=CELL*COLS, BH=CELL*ROWS;
  const BX=(W-BW)/2, BY=PAD+(availH-BH)/2;
  const RAD=CELL*0.38;
  let board, turn, gameOver, locked, drop;

  function emptyBoard(){ return Array.from({length:ROWS},()=>new Array(COLS).fill(0)); }
  function lowestEmptyRow(b,col){
    for(let r=ROWS-1;r>=0;r--) if(b[r][col]===0) return r;
    return -1;
  }
  function checkWinAt(b,r,c,color){
    const dirs=[[0,1],[1,0],[1,1],[1,-1]];
    for(const [dr,dc] of dirs){
      let cnt=1;
      let rr=r+dr,cc=c+dc;
      while(rr>=0&&rr<ROWS&&cc>=0&&cc<COLS&&b[rr][cc]===color){cnt++;rr+=dr;cc+=dc;}
      rr=r-dr;cc=c-dc;
      while(rr>=0&&rr<ROWS&&cc>=0&&cc<COLS&&b[rr][cc]===color){cnt++;rr-=dr;cc-=dc;}
      if(cnt>=4) return true;
    }
    return false;
  }
  function boardFull(b){ for(let c=0;c<COLS;c++) if(b[0][c]===0) return false; return true; }

  function updateTurnUI(){
    if(isOnline){
      if(turn===myColor) setTurnInfo('내 차례 ('+(myColor===1?'빨강':'노랑')+')','you');
      else setTurnInfo('상대 차례','opp');
    } else if(isP2){
      if(turn===1) setTurnInfo('P1 차례 (빨강)','you');
      else setTurnInfo('P2 차례 (노랑)','p2');
    } else setTurnInfo(turn===1?'내 차례 (빨강)':'AI 차례 (노랑)', turn===1?'you':'ai');
  }

  G.init=function(){
    if(isOnline) gameTitleEl.textContent='커넥트4 · 온라인';
    else if(isP2) gameTitleEl.textContent='커넥트4 · 2인';
    else gameTitleEl.textContent='커넥트4';
    if(isOnline) hintEl.textContent=(amHost?'당신=빨강':'당신=노랑')+' · 열을 눌러 말을 떨어뜨리세요. 4개를 먼저 연결하면 승리!';
    else if(isP2) hintEl.textContent='열을 눌러 말을 떨어뜨리세요. P1(빨강)/P2(노랑). 4개를 먼저 연결하면 승리!';
    else hintEl.textContent='열을 눌러 말을 떨어뜨리세요. 가로·세로·대각선 중 하나로 4개를 먼저 연결하면 승리!';
    board=emptyBoard(); turn=1; gameOver=false; locked=false; drop=null;
    updateTurnUI();
    controlsEl.innerHTML='';
    if(isOnline){
      Net.on('move',(msg)=>{
        if(gameOver) return;
        startDrop(msg.col, msg.color, true);
      });
    }
  };

  function startDrop(col,color,fromNet){
    if(gameOver||locked) return false;
    const row=lowestEmptyRow(board,col);
    if(row<0) return false;
    locked=true;
    drop={col,row,color,y:BY-CELL*0.5, targetY:BY+row*CELL+CELL/2};
    if(!fromNet && isOnline) Net.send({type:'move',col,color});
    return true;
  }

  function finishDrop(){
    const {col,row,color}=drop;
    board[row][col]=color;
    drop=null; locked=false;
    if(checkWinAt(board,row,col,color)){
      gameOver=true;
      if(isOnline) showOverlay((color===myColor?'나':'상대')+' 승리!');
      else if(isP2) showOverlay((color===1?'플레이어1 (빨강)':'플레이어2 (노랑)')+' 승리!');
      else showOverlay((color===1?'나(빨강)':'AI(노랑)')+' 승리!');
      setTurnInfo('게임 종료','');
      return;
    }
    if(boardFull(board)){ gameOver=true; showOverlay('무승부'); setTurnInfo('게임 종료',''); return; }
    turn = turn===1?2:1;
    updateTurnUI();
    if(!isOnline && !isP2 && turn===2) setTimeout(aiMove,550);
  }

  function aiMove(){
    if(gameOver||isP2||isOnline) return;
    let best=null,bestScore=-Infinity;
    for(let c=0;c<COLS;c++){
      const row=lowestEmptyRow(board,c); if(row<0) continue;
      let score=rand(0,3);
      board[row][c]=2;
      if(checkWinAt(board,row,c,2)) score+=1000;
      board[row][c]=0;
      board[row][c]=1;
      if(checkWinAt(board,row,c,1)) score+=500;
      board[row][c]=0;
      const centerDist=Math.abs(c-3); score += (3-centerDist)*3;
      if(score>bestScore){ bestScore=score; best=c; }
    }
    if(best===null) return;
    startDrop(best,2);
  }

  G.step=function(){
    if(drop){
      drop.y += 14;
      if(drop.y>=drop.targetY){ drop.y=drop.targetY; finishDrop(); }
    }
  };
  G.render=function(){
    drawWoodFrame();
    ctx.fillStyle='#1a4fa0'; roundRect(ctx,BX-10,BY-10,BW+20,BH+20,14); ctx.fill();
    for(let r=0;r<ROWS;r++) for(let c=0;c<COLS;c++){
      const x=BX+c*CELL+CELL/2, y=BY+r*CELL+CELL/2;
      ctx.beginPath(); ctx.arc(x,y,RAD,0,Math.PI*2);
      const v=board[r][c];
      ctx.fillStyle = v===0 ? 'rgba(255,255,255,.12)' : (v===1?'#c0392b':'#e8c23d');
      ctx.fill();
      if(v!==0){ ctx.lineWidth=2; ctx.strokeStyle='rgba(0,0,0,.25)'; ctx.stroke(); }
    }
    if(drop){
      const x=BX+drop.col*CELL+CELL/2;
      ctx.beginPath(); ctx.arc(x,drop.y,RAD,0,Math.PI*2);
      ctx.fillStyle = drop.color===1?'#c0392b':'#e8c23d'; ctx.fill();
    }
  };
  G.onDown=function(p){
    if(gameOver||locked) return;
    if(isOnline){ if(turn!==myColor) return; }
    else if(!isP2 && turn!==1) return;
    if(p.x<BX||p.x>BX+BW) return;
    const col=Math.floor((p.x-BX)/CELL);
    if(col<0||col>=COLS) return;
    startDrop(col, turn);
  };
  return G;
}

/* =========================================================
   7) RHYTHM GAME (solo)
   ========================================================= */
function RhythmGame(mode){
  const G = { key:'rhythm' };
  const LANES=4, PAD=20;
  const boardW=W-PAD*2, laneW=boardW/LANES;
  const BX=PAD, BY=20;
  const JUDGE_Y=H-140;
  const SPEED=4.6, SPAWN_GAP=48, TOTAL_NOTES=28;
  const HIT_WINDOW=42, GOOD_WINDOW=70;
  const LANE_COLORS=['#e0574c','#5fbf6a','#e8a33d','#6ec8ff'];
  const LANE_KEYS=['D','F','J','K'];
  const KEY_MAP={ d:0, f:1, j:2, k:3 };
  let notes, frame, spawned, score, combo, maxCombo, hits, misses, gameOver, flashLane, flashT;

  function updateHud(){ setTurnInfo('점수 '+score+' · 콤보 '+combo, combo>0?'you':''); }

  G.init=function(){
    gameTitleEl.textContent='리듬게임';
    hintEl.textContent='키보드 D · F · J · K 를 눌러 노트가 판정선에 닿는 순간 맞추세요! (탭으로도 플레이 가능)';
    notes=[]; frame=0; spawned=0; score=0; combo=0; maxCombo=0; hits=0; misses=0; gameOver=false;
    flashLane=-1; flashT=0;
    controlsEl.innerHTML='';
    updateHud();
  };

  function spawnNote(){
    const lane=Math.floor(Math.random()*LANES);
    notes.push({lane, y:BY, judged:false});
    spawned++;
  }

  function endGame(){
    gameOver=true;
    const acc = TOTAL_NOTES ? hits/TOTAL_NOTES : 0;
    let grade='D';
    if(acc>=0.95) grade='S'; else if(acc>=0.8) grade='A'; else if(acc>=0.6) grade='B'; else if(acc>=0.4) grade='C';
    showOverlay('결과: '+grade+'\n점수 '+score+' · 최대 콤보 '+maxCombo+'\n히트 '+hits+' / 미스 '+misses);
    setTurnInfo('게임 종료','');
  }

  G.step=function(){
    if(gameOver) return;
    frame++;
    if(spawned<TOTAL_NOTES && frame % SPAWN_GAP===0) spawnNote();
    for(const n of notes){
      if(n.judged) continue;
      n.y += SPEED;
      if(n.y > JUDGE_Y + GOOD_WINDOW){ n.judged=true; misses++; combo=0; }
    }
    notes = notes.filter(n=>!n.judged);
    if(flashT>0) flashT--; else flashLane=-1;
    updateHud();
    if(spawned>=TOTAL_NOTES && notes.length===0) endGame();
  };

  G.render=function(){
    drawWoodFrame();
    for(let i=0;i<LANES;i++){
      const x=BX+i*laneW;
      ctx.fillStyle = i%2===0 ? 'rgba(255,255,255,.04)' : 'rgba(0,0,0,.12)';
      ctx.fillRect(x,BY,laneW,H-BY-40);
      if(flashLane===i && flashT>0){
        ctx.fillStyle='rgba(255,255,255,.18)';
        ctx.fillRect(x,BY,laneW,H-BY-40);
      }
    }
    ctx.strokeStyle='rgba(232,163,61,.9)'; ctx.lineWidth=4;
    ctx.beginPath(); ctx.moveTo(BX,JUDGE_Y); ctx.lineTo(BX+boardW,JUDGE_Y); ctx.stroke();
    for(let i=0;i<LANES;i++){
      const x=BX+i*laneW+laneW/2;
      ctx.beginPath(); ctx.arc(x,JUDGE_Y+34,17,0,Math.PI*2);
      ctx.fillStyle = (flashLane===i&&flashT>0) ? 'rgba(232,163,61,.9)' : 'rgba(255,255,255,.12)';
      ctx.fill();
      ctx.strokeStyle='rgba(255,255,255,.35)'; ctx.lineWidth=1.5; ctx.stroke();
      ctx.font='bold 16px "Apple SD Gothic Neo",sans-serif';
      ctx.textAlign='center'; ctx.textBaseline='middle';
      ctx.fillStyle='#f3e8d6';
      ctx.fillText(LANE_KEYS[i], x, JUDGE_Y+35);
    }
    for(const n of notes){
      const x=BX+n.lane*laneW+laneW/2;
      roundRect(ctx, x-laneW*0.36, n.y-14, laneW*0.72, 28, 8);
      ctx.fillStyle=LANE_COLORS[n.lane]; ctx.fill();
    }
    ctx.font='bold 22px "Apple SD Gothic Neo",sans-serif';
    ctx.textBaseline='alphabetic';
    ctx.fillStyle='#f3e8d6';
    ctx.textAlign='left'; ctx.fillText('점수 '+score, BX, 44);
    ctx.textAlign='right'; ctx.fillText('콤보 '+combo, BX+boardW, 44);
  };

  function hitLane(lane){
    if(gameOver || lane<0 || lane>=LANES) return;
    flashLane=lane; flashT=8;
    let target=null,bestDist=Infinity;
    for(const n of notes){
      if(n.judged||n.lane!==lane) continue;
      const d=Math.abs(n.y-JUDGE_Y);
      if(d<bestDist){ bestDist=d; target=n; }
    }
    if(target && bestDist<=GOOD_WINDOW){
      target.judged=true;
      hits++;
      score += bestDist<=HIT_WINDOW ? 100 : 50;
      combo++;
      if(combo>maxCombo) maxCombo=combo;
    }
  }
  G.onDown=function(p){
    if(gameOver) return;
    if(p.x<BX||p.x>BX+boardW) return;
    hitLane(Math.floor((p.x-BX)/laneW));
  };
  G.onKeyDown=function(key){
    const lane=KEY_MAP[(key||'').toLowerCase()];
    if(lane===undefined) return false;
    hitLane(lane);
    return true;
  };
  return G;
}

})();
</script>
</body>
</html>
