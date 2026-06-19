<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Hand Battle</title>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
<style>
* { margin:0; padding:0; box-sizing:border-box; }
body { background:#0a0a0a; font-family:'Arial Black',Arial,sans-serif; color:#fff; min-height:100vh; overflow-x:hidden; }
.page { display:none; min-height:100vh; }
.page.active { display:flex; flex-direction:column; align-items:center; justify-content:center; }

/* HOME */
#page-home { background:radial-gradient(ellipse at 50% 30%, #1a0000 0%, #0a0a0a 70%); text-align:center; padding:40px 20px; }
.home-title { font-size:clamp(40px,10vw,90px); font-weight:900; background:linear-gradient(135deg,#ff3333,#ff8800); -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text; line-height:1; }
.home-subtitle { font-size:clamp(14px,3vw,20px); color:rgba(255,255,255,0.5); margin:12px 0 48px; letter-spacing:0.3em; text-transform:uppercase; }
.btn-primary { background:linear-gradient(135deg,#ff3333,#cc0000); border:none; color:#fff; border-radius:50px; padding:18px 60px; font-size:22px; font-weight:900; cursor:pointer; letter-spacing:0.05em; transition:transform 0.1s,box-shadow 0.1s; box-shadow:0 4px 30px rgba(255,50,50,0.4); }
.btn-primary:hover { transform:scale(1.05); box-shadow:0 6px 40px rgba(255,50,50,0.6); }
.btn-primary:active { transform:scale(0.97); }

/* TEAM SELECT */
#page-teams { padding:30px 20px; background:#0a0a0a; }
.page-title { font-size:clamp(22px,5vw,36px); font-weight:900; text-align:center; margin-bottom:8px; }
.page-sub { color:rgba(255,255,255,0.45); text-align:center; font-size:14px; letter-spacing:0.2em; margin-bottom:30px; }
.teams-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(260px,1fr)); gap:16px; width:100%; max-width:860px; }
.team-card { background:#141414; border:2px solid rgba(255,255,255,0.08); border-radius:16px; padding:20px; cursor:pointer; transition:transform 0.15s,border-color 0.15s,box-shadow 0.15s; position:relative; overflow:hidden; }
.team-card:hover { transform:translateY(-3px); border-color:rgba(255,255,255,0.25); }
.team-card.selected { border-color:var(--tc); box-shadow:0 0 20px var(--tcs); }
.team-card.full { opacity:0.4; cursor:not-allowed; }
.team-emoji { font-size:40px; margin-bottom:10px; display:block; }
.team-name { font-size:18px; font-weight:900; margin-bottom:4px; }
.team-members { font-size:13px; color:rgba(255,255,255,0.45); }
.team-slots { position:absolute; top:14px; right:14px; display:flex; gap:6px; }
.slot { width:14px; height:14px; border-radius:50%; border:2px solid rgba(255,255,255,0.2); background:transparent; }
.slot.filled { background:var(--tc); border-color:var(--tc); }
.name-input-wrap { margin-top:24px; display:flex; flex-direction:column; align-items:center; gap:12px; width:100%; max-width:400px; }
.name-input { background:#1a1a1a; border:2px solid rgba(255,255,255,0.15); border-radius:50px; color:#fff; font-size:16px; padding:12px 24px; width:100%; text-align:center; outline:none; }
.name-input:focus { border-color:rgba(255,50,50,0.6); }
.btn-join { background:linear-gradient(135deg,#ff3333,#cc0000); border:none; color:#fff; border-radius:50px; padding:14px 48px; font-size:17px; font-weight:900; cursor:pointer; opacity:0.4; pointer-events:none; transition:opacity 0.2s; }
.btn-join.ready { opacity:1; pointer-events:auto; }

/* LOBBY */
#page-lobby { padding:30px 20px; background:#0a0a0a; }
.lobby-teams { display:grid; grid-template-columns:repeat(auto-fit,minmax(220px,1fr)); gap:12px; width:100%; max-width:860px; margin-bottom:30px; }
.lobby-team { background:#141414; border:2px solid rgba(255,255,255,0.08); border-radius:14px; padding:16px; }
.lteam-name { font-size:15px; font-weight:900; margin-bottom:10px; display:flex; align-items:center; gap:8px; }
.lteam-dot { width:10px; height:10px; border-radius:50%; }
.lteam-players { display:flex; flex-direction:column; gap:6px; }
.player-slot { background:#1a1a1a; border-radius:8px; padding:8px 12px; font-size:13px; color:rgba(255,255,255,0.4); display:flex; align-items:center; gap:8px; }
.player-slot.filled { color:#fff; }
.player-avatar { width:26px; height:26px; border-radius:50%; background:#333; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:900; flex-shrink:0; }
.lobby-status { text-align:center; margin-bottom:20px; }
.status-badge { display:inline-block; background:#1a1a1a; border:1px solid rgba(255,255,255,0.12); border-radius:50px; padding:10px 28px; font-size:14px; color:rgba(255,255,255,0.6); letter-spacing:0.1em; }
.btn-launch { background:linear-gradient(135deg,#ff3333,#cc0000); border:none; color:#fff; border-radius:50px; padding:16px 56px; font-size:19px; font-weight:900; cursor:pointer; opacity:0.35; pointer-events:none; }
.btn-launch.ready { opacity:1; pointer-events:auto; }
.my-badge { background:rgba(255,50,50,0.2); color:#ff6666; border-radius:50px; padding:2px 8px; font-size:11px; font-weight:900; }

/* ADMIN */
#admin-gear { position:fixed; top:18px; right:18px; z-index:100; background:rgba(255,255,255,0.08); border:1px solid rgba(255,255,255,0.15); border-radius:50%; width:42px; height:42px; display:flex; align-items:center; justify-content:center; cursor:pointer; transition:background 0.15s,transform 0.2s; display:none; }
#admin-gear:hover { background:rgba(255,255,255,0.16); transform:rotate(30deg); }
#admin-gear svg { width:20px; height:20px; fill:rgba(255,255,255,0.6); }
#admin-modal-bg { display:none; position:fixed; inset:0; background:rgba(0,0,0,0.75); z-index:200; align-items:center; justify-content:center; }
#admin-modal-bg.open { display:flex; }
#admin-modal { background:#141414; border:1px solid rgba(255,255,255,0.12); border-radius:20px; padding:30px 28px; width:340px; max-width:90vw; }
#admin-modal h2 { font-size:20px; font-weight:900; margin-bottom:6px; display:flex; align-items:center; gap:10px; }
#admin-modal .admin-sub { font-size:12px; color:rgba(255,255,255,0.35); letter-spacing:0.15em; text-transform:uppercase; margin-bottom:24px; }
.admin-stat { background:#1a1a1a; border-radius:10px; padding:12px 16px; margin-bottom:12px; display:flex; justify-content:space-between; align-items:center; font-size:14px; color:rgba(255,255,255,0.5); }
.admin-stat span { font-size:18px; font-weight:900; color:#fff; }
.admin-actions { display:flex; flex-direction:column; gap:10px; margin-top:20px; }
.btn-admin-launch { background:linear-gradient(135deg,#ff3333,#cc0000); border:none; color:#fff; border-radius:50px; padding:14px; font-size:16px; font-weight:900; cursor:pointer; width:100%; }
.btn-admin-launch:hover { opacity:0.9; }
.btn-admin-reset { background:transparent; border:1.5px solid rgba(255,255,255,0.2); color:rgba(255,255,255,0.6); border-radius:50px; padding:12px; font-size:15px; font-weight:700; cursor:pointer; width:100%; }
.btn-admin-reset:hover { border-color:rgba(255,80,80,0.5); color:#ff6666; }
.btn-admin-close { background:transparent; border:none; color:rgba(255,255,255,0.3); font-size:13px; cursor:pointer; margin-top:10px; width:100%; padding:8px; }
.btn-admin-close:hover { color:rgba(255,255,255,0.6); }

/* GAME */
#page-game { position:relative; overflow:hidden; }
#game-video { position:absolute; top:0; left:0; width:100%; height:100%; object-fit:cover; transform:scaleX(-1); }
#game-canvas { position:absolute; top:0; left:0; width:100%; height:100%; transform:scaleX(-1); }
.game-overlay { position:absolute; top:0; left:0; width:100%; height:100%; pointer-events:none; }
#timer-bar-wrap { position:absolute; top:0; left:0; width:100%; height:5px; background:rgba(255,255,255,0.1); }
#timer-bar { height:100%; background:linear-gradient(90deg,#ff3333,#ff8800); width:100%; transition:width 1s linear; }
#my-score-display { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); text-align:center; }
#my-score-num { font-size:clamp(80px,20vw,160px); font-weight:900; color:#fff; text-shadow:0 0 30px rgba(255,50,50,0.9),3px 3px 0 #c00; line-height:1; }
#my-score-label { font-size:13px; color:rgba(255,255,255,0.5); letter-spacing:0.3em; text-transform:uppercase; }
#plus5 { position:absolute; font-size:clamp(24px,6vw,50px); font-weight:900; color:#ff3333; opacity:0; pointer-events:none; z-index:20; }
#live-ranking { position:absolute; right:16px; top:50%; transform:translateY(-50%); background:rgba(0,0,0,0.6); backdrop-filter:blur(10px); border:1px solid rgba(255,255,255,0.1); border-radius:14px; padding:14px; min-width:200px; }
#live-ranking h3 { font-size:11px; color:rgba(255,255,255,0.4); letter-spacing:0.2em; text-transform:uppercase; margin-bottom:10px; }
.rank-row { display:flex; justify-content:space-between; align-items:center; padding:6px 0; border-bottom:1px solid rgba(255,255,255,0.06); font-size:13px; }
.rank-row:last-child { border-bottom:none; }
.rank-row.me { color:#ff6666; font-weight:700; }
.rank-num { color:rgba(255,255,255,0.35); width:18px; font-size:11px; }
.rank-name { flex:1; padding:0 6px; overflow:hidden; text-overflow:ellipsis; white-space:nowrap; }
.rank-score { font-weight:900; }
#game-timer-display { position:absolute; top:16px; left:50%; transform:translateX(-50%); font-size:clamp(20px,5vw,36px); font-weight:900; color:#fff; text-shadow:0 0 10px rgba(255,50,50,0.7); }
.countdown-overlay { position:absolute; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.8); display:flex; flex-direction:column; align-items:center; justify-content:center; z-index:30; }
.countdown-num { font-size:clamp(120px,30vw,220px); font-weight:900; color:#ff3333; text-shadow:0 0 40px rgba(255,50,50,0.6); line-height:1; animation:cdown 1s ease-out; }
@keyframes cdown { 0%{transform:scale(1.5);opacity:0} 20%{opacity:1;transform:scale(1)} 80%{opacity:1;transform:scale(1)} 100%{opacity:0;transform:scale(0.7)} }
.countdown-go { font-size:clamp(60px,15vw,120px); font-weight:900; color:#fff; text-shadow:0 0 40px rgba(255,255,255,0.4); animation:cdown 0.8s ease-out; }

/* RESULTS */
#page-results { padding:30px 20px; background:radial-gradient(ellipse at 50% 20%, #1a0000 0%, #0a0a0a 70%); }
.results-title { font-size:clamp(28px,7vw,56px); font-weight:900; text-align:center; background:linear-gradient(135deg,#ffd700,#ff8800); -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text; margin-bottom:6px; }
.results-subtitle { text-align:center; color:rgba(255,255,255,0.4); font-size:13px; letter-spacing:0.2em; text-transform:uppercase; margin-bottom:32px; }
.results-teams { width:100%; max-width:860px; display:flex; flex-direction:column; gap:12px; }
.result-team-card { background:#141414; border-radius:14px; padding:18px 20px; display:flex; align-items:center; gap:16px; border:2px solid rgba(255,255,255,0.06); }
.result-team-card.winner { border-color:rgba(255,215,0,0.4); background:#1a1500; }
.result-rank { font-size:32px; font-weight:900; color:rgba(255,255,255,0.2); min-width:48px; text-align:center; }
.result-rank.gold { color:#ffd700; }
.result-rank.silver { color:#c0c0c0; }
.result-rank.bronze { color:#cd7f32; }
.result-info { flex:1; }
.result-tname { font-size:18px; font-weight:900; margin-bottom:4px; }
.result-players { font-size:12px; color:rgba(255,255,255,0.4); }
.result-total { font-size:28px; font-weight:900; color:#ff3333; }
.result-total.winner-score { color:#ffd700; }
.btn-restart { margin-top:32px; background:transparent; border:2px solid rgba(255,255,255,0.2); color:#fff; border-radius:50px; padding:14px 48px; font-size:17px; font-weight:900; cursor:pointer; letter-spacing:0.05em; }
.btn-restart:hover { border-color:rgba(255,255,255,0.5); }
</style>
</head>
<body>

<!-- HOME -->
<div id="page-home" class="page active">
  <div class="home-title">HAND<br>BATTLE</div>
  <div class="home-subtitle">6 équipes · 2 joueurs · 1 gagnant</div>
  <button class="btn-primary" onclick="goTo('teams')">▶ START</button>
</div>

<!-- TEAM SELECT -->
<div id="page-teams" class="page">
  <div class="page-title">Choisis ton équipe</div>
  <div class="page-sub">2 joueurs par équipe</div>
  <div class="teams-grid" id="teams-grid"></div>
  <div class="name-input-wrap" style="margin-top:24px;">
    <input class="name-input" id="player-name" placeholder="Ton pseudo..." maxlength="20" oninput="checkJoinReady()">
    <button class="btn-join" id="btn-join" onclick="joinTeam()">Rejoindre l'équipe</button>
  </div>
</div>

<!-- LOBBY -->
<div id="page-lobby" class="page">
  <div id="admin-gear" onclick="openAdmin()" title="Mode admin">
    <svg viewBox="0 0 24 24"><path d="M19.14,12.94c0.04-0.3,0.06-0.61,0.06-0.94c0-0.32-0.02-0.64-0.07-0.94l2.03-1.58c0.18-0.14,0.23-0.41,0.12-0.61 l-1.92-3.32c-0.12-0.22-0.37-0.29-0.59-0.22l-2.39,0.96c-0.5-0.38-1.03-0.7-1.62-0.94L14.4,2.81c-0.04-0.24-0.24-0.41-0.48-0.41 h-3.84c-0.24,0-0.43,0.17-0.47,0.41L9.25,5.35C8.66,5.59,8.12,5.92,7.63,6.29L5.24,5.33c-0.22-0.08-0.47,0-0.59,0.22L2.74,8.87 C2.62,9.08,2.66,9.34,2.86,9.48l2.03,1.58C4.84,11.36,4.8,11.69,4.8,12s0.02,0.64,0.07,0.94l-2.03,1.58 c-0.18,0.14-0.23,0.41-0.12,0.61l1.92,3.32c0.12,0.22,0.37,0.29,0.59,0.22l2.39-0.96c0.5,0.38,1.03,0.7,1.62,0.94l0.36,2.54 c0.05,0.24,0.24,0.41,0.48,0.41h3.84c0.24,0,0.44-0.17,0.47-0.41l0.36-2.54c0.59-0.24,1.13-0.56,1.62-0.94l2.39,0.96 c0.22,0.08,0.47,0,0.59-0.22l1.92-3.32c0.12-0.22,0.07-0.47-0.12-0.61L19.14,12.94z M12,15.6c-1.98,0-3.6-1.62-3.6-3.6 s1.62-3.6,3.6-3.6s3.6,1.62,3.6,3.6S13.98,15.6,12,15.6z"/></svg>
  </div>
  <div class="page-title">Lobby</div>
  <div class="page-sub" id="lobby-sub">En attente des joueurs…</div>
  <div class="lobby-teams" id="lobby-teams"></div>
  <div class="lobby-status"><div class="status-badge" id="lobby-badge">0 / 12 joueurs connectés</div></div>
  <button class="btn-launch" id="btn-launch" onclick="launchGame()">🚀 Lancer la partie</button>

  <div id="admin-modal-bg" onclick="if(event.target===this) closeAdmin()">
    <div id="admin-modal">
      <h2>⚙️ Mode Admin</h2>
      <div class="admin-sub">Contrôles superviseur</div>
      <div class="admin-stat">Joueurs connectés <span id="admin-count">0 / 12</span></div>
      <div class="admin-actions">
        <button class="btn-admin-launch" onclick="adminForceLaunch()">🚀 Lancer la partie (forcer)</button>
        <button class="btn-admin-reset" onclick="adminReset()">↺ Réinitialiser les équipes</button>
        <button class="btn-admin-close" onclick="closeAdmin()">Fermer</button>
      </div>
    </div>
  </div>
</div>

<!-- GAME -->
<div id="page-game" class="page">
  <video id="game-video" autoplay playsinline muted></video>
  <canvas id="game-canvas"></canvas>
  <div class="game-overlay">
    <div id="timer-bar-wrap"><div id="timer-bar"></div></div>
    <div id="game-timer-display">30</div>
    <div id="my-score-display">
      <div id="my-score-num">0</div>
      <div id="my-score-label">MON SCORE</div>
    </div>
    <div id="plus5"></div>
    <div id="live-ranking">
      <h3>Classement</h3>
      <div id="ranking-list"></div>
    </div>
  </div>
  <div class="countdown-overlay" id="countdown-overlay"></div>
</div>

<!-- RESULTS -->
<div id="page-results" class="page">
  <div class="results-title">🏆 RÉSULTATS</div>
  <div class="results-subtitle">Scores finaux par équipe</div>
  <div class="results-teams" id="results-teams"></div>
  <button class="btn-restart" onclick="restartGame()">↺ Nouvelle partie</button>
</div>

<script>
const TEAMS = [
  { id:'mercegraisse',       name:'Mercegraisse',       emoji:'🍔', color:'#e74c3c', colorShadow:'rgba(231,76,60,0.4)' },
  { id:'mcnuggets',          name:'Mc Nuggets',         emoji:'🍗', color:'#e67e22', colorShadow:'rgba(230,126,34,0.4)' },
  { id:'bouzelouf',          name:'Bouzelouf',          emoji:'🧆', color:'#9b59b6', colorShadow:'rgba(155,89,182,0.4)' },
  { id:'mangeur_nuggets',    name:'Mangeur de Nuggets', emoji:'🦅', color:'#3498db', colorShadow:'rgba(52,152,219,0.4)' },
  { id:'french_tacos',       name:'French Tacos',       emoji:'🌯', color:'#2ecc71', colorShadow:'rgba(46,204,113,0.4)' },
  { id:'ferragrille',        name:'Ferragrille',        emoji:'🔥', color:'#f39c12', colorShadow:'rgba(243,156,18,0.4)' }
];

const GAME_DURATION = 120;
const STORAGE_KEY = 'handbattle_v3';

let state = load();
let myTeamId = null;
let myName = null;
let myPlayerId = null;
let selectedTeamId = null;
let gameScore = 0;
let gameTimer = null;
let cameraStream = null;
let handsDetector = null;
let mpCamera = null;
let crossDetected = false;
let lastCrossTime = 0;
let lastHandTime = 0;
let gameRunning = false;
let passiveActiveUntil = 0;
let lastPalmPos = null;
const COOLDOWN = 400;
const HAND_INTERVAL = 500;
const PASSIVE_WINDOW = 4000;
const MOVE_THRESHOLD = 0.015;

function defaultState() {
  return { players: {}, launched: false, finished: false };
}

function load() {
  try { const s = JSON.parse(localStorage.getItem(STORAGE_KEY)); return s || defaultState(); }
  catch(e) { return defaultState(); }
}

function save() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function goTo(page) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById('page-'+page).classList.add('active');
  document.getElementById('admin-gear').style.display = (page === 'lobby') ? 'flex' : 'none';
  if(page === 'teams') renderTeams();
  if(page === 'lobby') { renderLobby(); startLobbyPolling(); }
  if(page === 'results') renderResults();
}

/* ─── ADMIN ─── */
function openAdmin() {
  state = load();
  const total = Object.keys(state.players).length;
  document.getElementById('admin-count').textContent = total + ' / 12';
  document.getElementById('admin-modal-bg').classList.add('open');
}

function closeAdmin() {
  document.getElementById('admin-modal-bg').classList.remove('open');
}

function adminForceLaunch() {
  state = load();
  if(Object.keys(state.players).length === 0) {
    alert("Aucun joueur n'a encore rejoint une équipe.");
    return;
  }
  state.launched = true;
  save();
  closeAdmin();
  clearInterval(lobbyPoll);
  startGameFlow();
}

function adminReset() {
  if(!confirm("Réinitialiser toutes les équipes et déconnecter tous les joueurs ?")) return;
  localStorage.removeItem(STORAGE_KEY);
  state = defaultState();
  save();
  closeAdmin();
  renderLobby();
}

/* ─── TEAMS PAGE ─── */
function getTeamPlayers(teamId) {
  return Object.values(state.players).filter(p => p.teamId === teamId);
}

function renderTeams() {
  state = load();
  const grid = document.getElementById('teams-grid');
  grid.innerHTML = '';
  TEAMS.forEach(t => {
    const members = getTeamPlayers(t.id);
    const full = members.length >= 2;
    const card = document.createElement('div');
    card.className = 'team-card' + (full ? ' full' : '') + (selectedTeamId === t.id ? ' selected' : '');
    card.style.setProperty('--tc', t.color);
    card.style.setProperty('--tcs', t.colorShadow);
    card.innerHTML = `
      <div class="team-slots">
        <div class="slot ${members.length>=1?'filled':''}" style="--tc:${t.color}"></div>
        <div class="slot ${members.length>=2?'filled':''}" style="--tc:${t.color}"></div>
      </div>
      <span class="team-emoji">${t.emoji}</span>
      <div class="team-name" style="color:${t.color}">${t.name}</div>
      <div class="team-members">${members.length}/2 joueurs${members.length>0?' — '+members.map(p=>p.name).join(', '):''}</div>
    `;
    if(!full) {
      card.onclick = () => {
        selectedTeamId = t.id;
        document.querySelectorAll('.team-card').forEach(c => c.classList.remove('selected'));
        card.classList.add('selected');
        checkJoinReady();
      };
    }
    grid.appendChild(card);
  });
}

function checkJoinReady() {
  const btn = document.getElementById('btn-join');
  const name = document.getElementById('player-name').value.trim();
  if(selectedTeamId && name.length >= 2) btn.classList.add('ready');
  else btn.classList.remove('ready');
}

function joinTeam() {
  const name = document.getElementById('player-name').value.trim();
  if(!name || !selectedTeamId) return;
  state = load();
  const members = getTeamPlayers(selectedTeamId);
  if(members.length >= 2) { alert("Cette équipe est déjà complète !"); renderTeams(); return; }
  myName = name;
  myTeamId = selectedTeamId;
  myPlayerId = 'p_' + Date.now() + '_' + Math.random().toString(36).slice(2,7);
  state.players[myPlayerId] = { name: myName, teamId: myTeamId, score: 0 };
  save();
  goTo('lobby');
}

/* ─── LOBBY PAGE ─── */
let lobbyPoll = null;

function startLobbyPolling() {
  if(lobbyPoll) clearInterval(lobbyPoll);
  lobbyPoll = setInterval(() => {
    state = load();
    renderLobby();
    if(state.launched && !state.finished) {
      clearInterval(lobbyPoll);
      startGameFlow();
    }
  }, 1000);
  renderLobby();
}

function renderLobby() {
  const container = document.getElementById('lobby-teams');
  const total = Object.keys(state.players).length;
  document.getElementById('lobby-badge').textContent = total + ' / 12 joueurs connectés';
  const adminCountEl = document.getElementById('admin-count');
  if(adminCountEl) adminCountEl.textContent = total + ' / 12';
  const sub = document.getElementById('lobby-sub');
  sub.textContent = total >= 12 ? 'Tout le monde est là !' : 'En attente des joueurs…';
  const btn = document.getElementById('btn-launch');
  if(total >= 2) btn.classList.add('ready'); else btn.classList.remove('ready');
  container.innerHTML = '';
  TEAMS.forEach(t => {
    const members = getTeamPlayers(t.id);
    const div = document.createElement('div');
    div.className = 'lobby-team';
    div.innerHTML = `
      <div class="lteam-name">
        <div class="lteam-dot" style="background:${t.color}"></div>
        ${t.emoji} ${t.name}
      </div>
      <div class="lteam-players">
        ${[0,1].map(i => {
          const p = members[i];
          const isMe = p && p.name === myName && p.teamId === myTeamId;
          return `<div class="player-slot ${p?'filled':''}">
            <div class="player-avatar" style="${p?'background:'+t.color+';color:#fff':''}">
              ${p ? p.name[0].toUpperCase() : '?'}
            </div>
            ${p ? p.name + (isMe?' <span class="my-badge">Moi</span>':'') : 'En attente…'}
          </div>`;
        }).join('')}
      </div>
    `;
    container.appendChild(div);
  });
}

function launchGame() {
  state = load();
  state.launched = true;
  save();
  clearInterval(lobbyPoll);
  startGameFlow();
}

/* ─── GAME ─── */
async function startGameFlow() {
  goTo('game');
  const overlay = document.getElementById('countdown-overlay');
  overlay.style.display = 'flex';

  try {
    cameraStream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode:'user', width:{ideal:1280}, height:{ideal:720} },
      audio: false
    });
    const vid = document.getElementById('game-video');
    vid.srcObject = cameraStream;
  } catch(e) {
    overlay.innerHTML = `<div style="text-align:center;padding:40px"><div style="font-size:24px;color:#ff3333;margin-bottom:16px">⚠️ Caméra refusée</div><div style="color:rgba(255,255,255,0.5);font-size:14px">Le jeu nécessite l'accès à la caméra</div></div>`;
    return;
  }

  const canvas = document.getElementById('game-canvas');
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  handsDetector = new Hands({ locateFile: f => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${f}` });
  handsDetector.setOptions({ maxNumHands:2, modelComplexity:1, minDetectionConfidence:0.7, minTrackingConfidence:0.6 });
  handsDetector.onResults(onHandResults);

  mpCamera = new Camera(document.getElementById('game-video'), {
    onFrame: async () => { await handsDetector.send({ image: document.getElementById('game-video') }); },
    width: 1280, height: 720
  });
  mpCamera.start();

  for(let i=3; i>=1; i--) {
    overlay.innerHTML = `<div class="countdown-num">${i}</div>`;
    await sleep(1000);
  }
  overlay.innerHTML = `<div class="countdown-go">GO !</div>`;
  await sleep(800);
  overlay.style.display = 'none';

  gameScore = 0;
  gameRunning = true;
  crossDetected = false;
  lastCrossTime = 0;
  lastHandTime = 0;
  passiveActiveUntil = 0;
  lastPalmPos = null;
  document.getElementById('my-score-num').textContent = '0';
  updateRanking();

  let timeLeft = GAME_DURATION;
  document.getElementById('game-timer-display').textContent = timeLeft;
  document.getElementById('timer-bar').style.width = '100%';

  gameTimer = setInterval(() => {
    timeLeft--;
    document.getElementById('game-timer-display').textContent = timeLeft;
    document.getElementById('timer-bar').style.width = (timeLeft/GAME_DURATION*100)+'%';
    if(timeLeft <= 0) {
      clearInterval(gameTimer);
      gameRunning = false;
      endGame();
    }
  }, 1000);

  setInterval(() => {
    if(!gameRunning) return;
    state = load();
    if(state.players[myPlayerId]) {
      state.players[myPlayerId].score = gameScore;
      save();
    }
    updateRanking();
  }, 500);
}

function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

function getPalmCenter(lm) {
  return { x: lm[0].x*0.35 + lm[9].x*0.65, y: lm[0].y*0.35 + lm[9].y*0.65 };
}

function onHandResults(results) {
  const canvas = document.getElementById('game-canvas');
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  if(!gameRunning || !results.multiHandLandmarks || results.multiHandLandmarks.length === 0) {
    crossDetected = false; lastPalmPos = null; return;
  }
  const W = canvas.width, H = canvas.height;
  const palmCenters = [];
  results.multiHandLandmarks.forEach(lm => {
    const p = getPalmCenter(lm);
    const px = p.x*W, py = p.y*H;
    palmCenters.push({x:px,y:py, nx:p.x, ny:p.y});
    const g = ctx.createRadialGradient(px,py,4,px,py,32);
    g.addColorStop(0,'rgba(255,30,30,0.9)'); g.addColorStop(1,'rgba(255,30,30,0)');
    ctx.beginPath(); ctx.arc(px,py,32,0,Math.PI*2); ctx.fillStyle=g; ctx.fill();
    ctx.beginPath(); ctx.arc(px,py,10,0,Math.PI*2); ctx.fillStyle='#ff2020';
    ctx.shadowColor='#ff0000'; ctx.shadowBlur=15; ctx.fill(); ctx.shadowBlur=0;
  });
  const now = Date.now();

  // Détection de mouvement des mains -> prolonge la fenêtre de points passifs
  const avgPos = {
    x: palmCenters.reduce((s,p)=>s+p.nx,0) / palmCenters.length,
    y: palmCenters.reduce((s,p)=>s+p.ny,0) / palmCenters.length
  };
  if(lastPalmPos) {
    const dist = Math.hypot(avgPos.x - lastPalmPos.x, avgPos.y - lastPalmPos.y);
    if(dist > MOVE_THRESHOLD && passiveActiveUntil > now) {
      passiveActiveUntil = now + PASSIVE_WINDOW;
    }
  }
  lastPalmPos = avgPos;

  // Points passifs : +2 toutes les 0.5s tant que la fenêtre active est ouverte
  if(passiveActiveUntil > now && now - lastHandTime > HAND_INTERVAL) {
    gameScore += 2;
    document.getElementById('my-score-num').textContent = gameScore;
    lastHandTime = now;
  }

  // Croisement des mains : +5 et relance la fenêtre de 4s
  if(palmCenters.length === 2) {
    const [a,b] = palmCenters;
    const isCrossing = Math.abs(a.x-b.x) < W*0.35 && Math.abs(a.y-b.y) > H*0.04;
    if(isCrossing && !crossDetected && now-lastCrossTime > COOLDOWN) {
      crossDetected = true; lastCrossTime = now;
      gameScore += 5;
      passiveActiveUntil = now + PASSIVE_WINDOW;
      document.getElementById('my-score-num').textContent = gameScore;
      pulseScore();
      showPlus5((a.x+b.x)/2, (a.y+b.y)/2);
    } else if(!isCrossing) crossDetected = false;
  } else crossDetected = false;
}

function pulseScore() {
  const el = document.getElementById('my-score-num');
  el.style.transform = 'scale(1.3)'; setTimeout(() => el.style.transform='', 150);
}

function showPlus5(x, y) {
  const el = document.getElementById('plus5');
  el.style.left=x+'px'; el.style.top=y+'px'; el.style.transform='translate(-50%,-50%)';
  el.style.opacity='1'; el.style.transition='none';
  requestAnimationFrame(() => requestAnimationFrame(() => {
    el.style.transition='opacity 0.5s,top 0.5s';
    el.style.top=(y-80)+'px'; el.style.opacity='0';
  }));
}

function updateRanking() {
  const players = Object.entries(state.players).map(([id,p]) => ({...p,id}));
  const teams = TEAMS.map(t => {
    const tp = players.filter(p=>p.teamId===t.id);
    return { name:t.name, score:tp.reduce((s,p)=>s+p.score,0), emoji:t.emoji, members:tp };
  });
  teams.sort((a,b)=>b.score-a.score);
  const list = document.getElementById('ranking-list');
  list.innerHTML = teams.slice(0,6).map((t,i) => {
    const isMe = t.members.some(p=>p.id===myPlayerId);
    return `<div class="rank-row ${isMe?'me':''}">
      <span class="rank-num">${i+1}</span>
      <span class="rank-name">${t.emoji} ${t.name}</span>
      <span class="rank-score">${t.score}</span>
    </div>`;
  }).join('');
}

async function endGame() {
  if(mpCamera) mpCamera.stop();
  if(cameraStream) cameraStream.getTracks().forEach(t=>t.stop());
  state = load();
  if(state.players[myPlayerId]) { state.players[myPlayerId].score = gameScore; }
  state.finished = true;
  save();
  await sleep(1000);
  goTo('results');
}

/* ─── RESULTS ─── */
function renderResults() {
  state = load();
  const players = Object.entries(state.players).map(([id,p]) => ({...p,id}));
  const teams = TEAMS.map(t => {
    const tp = players.filter(p=>p.teamId===t.id);
    return { ...t, total:tp.reduce((s,p)=>s+p.score,0), members:tp };
  });
  teams.sort((a,b)=>b.total-a.total);
  const ranks = ['🥇','🥈','🥉'];
  const rankClass = ['gold','silver','bronze'];
  document.getElementById('results-teams').innerHTML = teams.map((t,i) => `
    <div class="result-team-card ${i===0?'winner':''}">
      <div class="result-rank ${rankClass[i]||''}">${ranks[i]||i+1}</div>
      <div class="result-info">
        <div class="result-tname" style="color:${t.color}">${t.emoji} ${t.name}</div>
        <div class="result-players">${t.members.length>0?t.members.map(p=>p.name+' ('+p.score+')').join(' + '):'Aucun joueur'}</div>
      </div>
      <div class="result-total ${i===0?'winner-score':''}">${t.total}</div>
    </div>
  `).join('');
}

function restartGame() {
  localStorage.removeItem(STORAGE_KEY);
  state = defaultState();
  myTeamId = null; myName = null; myPlayerId = null; selectedTeamId = null;
  gameScore = 0;
  goTo('home');
}

window.addEventListener('storage', () => { state = load(); });
</script>
</body>
</html>
