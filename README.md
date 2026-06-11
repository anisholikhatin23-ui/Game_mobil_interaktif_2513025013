<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>TURBO RUSH 3D</title>
<link rel="preconnect" href="https://fonts.googleapis.com"/>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Rajdhani:wght@400;600;700&display=swap" rel="stylesheet"/>
<style>
*{margin:0;padding:0;box-sizing:border-box}
body{background:#000;overflow:hidden;font-family:'Rajdhani',sans-serif;user-select:none;-webkit-user-select:none}
canvas{display:block}

/* HUD */
#hud{position:fixed;inset:0;pointer-events:none;z-index:20}

#top-left{position:absolute;top:16px;left:16px;display:flex;flex-direction:column;gap:8px}
#top-right{position:absolute;top:16px;right:16px;display:flex;flex-direction:column;align-items:flex-end;gap:8px}

.stat-box{
  background:rgba(0,0,0,0.55);
  border:1px solid rgba(255,255,255,0.08);
  border-radius:10px;
  padding:6px 14px 8px;
  backdrop-filter:blur(10px);
  min-width:80px;
}
.stat-label{
  font-size:9px;font-weight:700;letter-spacing:3px;
  text-transform:uppercase;color:rgba(255,255,255,0.4);
  margin-bottom:1px;
}
.stat-value{
  font-family:'Orbitron',sans-serif;
  font-size:26px;font-weight:900;line-height:1;
  color:#fff;
}
.stat-box.spd .stat-value{color:#00e5ff;text-shadow:0 0 12px rgba(0,229,255,0.5)}
.stat-box.tmr .stat-value{color:#ff4d4d;text-shadow:0 0 12px rgba(255,77,77,0.5)}
.stat-box.scr .stat-value{color:#ffd700;text-shadow:0 0 12px rgba(255,215,0,0.5)}
.stat-box.lvl .stat-value{color:#7fff7f;text-shadow:0 0 12px rgba(127,255,127,0.5)}
.stat-box.hp  .stat-value{color:#ff6b6b;text-shadow:0 0 12px rgba(255,107,107,0.5)}

.hearts{display:flex;gap:4px;align-items:center;margin-top:2px}
.heart{width:14px;height:14px;color:#ff4d4d;filter:drop-shadow(0 0 4px rgba(255,77,77,0.7))}
.heart.dead{color:rgba(255,255,255,0.15);filter:none}

#turbo-wrap{
  position:absolute;bottom:70px;left:50%;transform:translateX(-50%);
  width:240px;pointer-events:none;text-align:center;
}
#turbo-label{font-size:9px;font-weight:700;letter-spacing:3px;color:#00e5ff;text-transform:uppercase;margin-bottom:5px;opacity:.7}
#turbo-track{
  height:6px;background:rgba(255,255,255,0.08);
  border-radius:3px;overflow:hidden;
  border:1px solid rgba(0,229,255,0.2);
}
#turbo-fill{
  height:100%;width:100%;border-radius:3px;
  background:linear-gradient(90deg,#00b4d8,#00e5ff);
  transition:width .1s;
  box-shadow:0 0 8px rgba(0,229,255,0.6);
}

#prog-wrap{
  position:absolute;bottom:90px;left:50%;transform:translateX(-50%);
  width:220px;pointer-events:none;text-align:center;
}
#prog-label{font-size:9px;font-weight:700;letter-spacing:3px;color:rgba(255,255,255,0.35);text-transform:uppercase;margin-bottom:4px}
#prog-track{height:4px;background:rgba(255,255,255,0.07);border-radius:2px;overflow:hidden}
#prog-fill{height:100%;width:0%;border-radius:2px;background:linear-gradient(90deg,#ffd700,#ff6000);transition:width .2s}

#btn-bar{
  position:fixed;bottom:16px;left:50%;transform:translateX(-50%);
  display:flex;gap:10px;z-index:25;pointer-events:all;
}
.btn{
  font-family:'Rajdhani',sans-serif;font-weight:700;font-size:11px;
  letter-spacing:1.5px;padding:9px 18px;border-radius:8px;
  border:1px solid rgba(255,255,255,0.1);
  background:rgba(255,255,255,0.05);
  color:rgba(255,255,255,0.55);
  cursor:pointer;transition:all .15s;text-transform:uppercase;
  backdrop-filter:blur(6px);
}
.btn:hover{background:rgba(255,255,255,0.1);color:#fff;border-color:rgba(255,255,255,0.25)}
.btn.danger{border-color:rgba(255,77,77,0.25);color:rgba(255,100,100,0.6)}
.btn.danger:hover{background:rgba(255,77,77,0.12);color:#ff6b6b;border-color:rgba(255,77,77,0.5)}

#hints{
  position:fixed;left:16px;bottom:16px;
  background:rgba(0,0,0,0.5);border:1px solid rgba(255,255,255,0.06);
  border-radius:10px;padding:10px 14px;font-size:11px;line-height:2;
  z-index:20;color:rgba(255,255,255,0.45);backdrop-filter:blur(6px);
}
#hints kbd{
  background:rgba(255,255,255,0.07);border:1px solid rgba(255,255,255,0.12);
  border-radius:3px;padding:1px 5px;font-size:10px;
  color:rgba(255,255,255,0.6);margin:0 2px;
}

#slines{position:fixed;inset:0;pointer-events:none;z-index:6;overflow:hidden;opacity:0;transition:opacity .2s}
#slines .sl{position:absolute;top:0;width:1px;background:linear-gradient(to bottom,transparent,rgba(0,229,255,.25),transparent);animation:sl .12s linear infinite}
@keyframes sl{from{top:-100%}to{top:100%}}

.overlay{
  position:fixed;inset:0;display:flex;align-items:center;justify-content:center;
  background:rgba(0,0,0,0.8);z-index:100;backdrop-filter:blur(6px);
}
.overlay.hidden{display:none}
.card{
  background:rgba(8,12,20,0.9);
  border:1px solid rgba(255,255,255,0.07);
  border-radius:20px;padding:44px 56px;
  display:flex;flex-direction:column;align-items:center;
  max-width:520px;width:90%;text-align:center;
  box-shadow:0 0 60px rgba(0,0,0,0.8);
}
.card-title{
  font-family:'Orbitron',sans-serif;font-size:clamp(36px,6vw,64px);
  font-weight:900;letter-spacing:6px;margin-bottom:8px;
  color:#fff;text-shadow:0 0 40px rgba(255,255,255,0.2);
}
.card-title.accent{color:#ffd700;text-shadow:0 0 30px rgba(255,215,0,0.5)}
.card-sub{
  font-size:13px;letter-spacing:2px;color:rgba(255,255,255,0.35);
  text-transform:uppercase;margin-bottom:6px;font-weight:600;
}
.card-score{
  font-family:'Orbitron',sans-serif;font-size:72px;font-weight:900;
  color:#00e5ff;text-shadow:0 0 30px rgba(0,229,255,0.5);
  line-height:1;margin:16px 0;
}
.card-btn{
  font-family:'Rajdhani',sans-serif;font-weight:700;font-size:14px;
  letter-spacing:3px;padding:14px 44px;border-radius:10px;
  border:1px solid rgba(255,215,0,0.4);
  background:rgba(255,215,0,0.06);
  color:#ffd700;cursor:pointer;text-transform:uppercase;
  pointer-events:all;transition:all .2s;margin-top:18px;
}
.card-btn:hover{background:rgba(255,215,0,0.14);border-color:#ffd700;box-shadow:0 0 30px rgba(255,215,0,0.25);transform:scale(1.02)}
.card-btn.danger{border-color:rgba(255,77,77,0.4);color:#ff6b6b;background:rgba(255,77,77,0.06)}
.card-btn.danger:hover{background:rgba(255,77,77,0.14);border-color:#ff6b6b;box-shadow:0 0 30px rgba(255,77,77,0.25)}

#level-banner{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  font-family:'Orbitron',sans-serif;font-size:clamp(36px,6vw,72px);
  font-weight:900;text-align:center;pointer-events:none;z-index:90;
  opacity:0;transition:opacity .3s;color:#fff;
  text-shadow:0 0 40px rgba(255,255,255,0.6);letter-spacing:4px;
}
#portal-flash{position:fixed;inset:0;background:#fff;opacity:0;pointer-events:none;z-index:80;transition:opacity .06s}
</style>
</head>
<body>

<!-- HUD -->
<div id="hud">
  <div id="top-left">
    <div class="stat-box lvl">
      <div class="stat-label">Level</div>
      <div class="stat-value" id="v-lvl">1</div>
    </div>
    <div class="stat-box scr">
      <div class="stat-label">Score</div>
      <div class="stat-value" id="v-score">0</div>
    </div>
    <div class="stat-box hp">
      <div class="stat-label">Lives</div>
      <div class="hearts" id="v-hearts">
        <svg class="heart" viewBox="0 0 24 24" fill="currentColor"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>
        <svg class="heart" viewBox="0 0 24 24" fill="currentColor"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>
        <svg class="heart" viewBox="0 0 24 24" fill="currentColor"><path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/></svg>
      </div>
    </div>
  </div>

  <div id="top-right" style="flex-direction:row;align-items:flex-start;gap:10px">
    <div class="stat-box spd">
      <div class="stat-label">Speed</div>
      <div class="stat-value" id="v-spd">0</div>
      <div id="v-spd-label" style="font-size:8px;letter-spacing:2px;color:#00e5ff;opacity:0.6;text-transform:uppercase;margin-top:2px;font-family:'Rajdhani',sans-serif;font-weight:700"></div>
    </div>
    <div class="stat-box tmr">
      <div class="stat-label">Time</div>
      <div class="stat-value" id="v-tmr">90</div>
    </div>
  </div>

  <div id="prog-wrap">
    <div id="prog-label">Level Progress</div>
    <div id="prog-track"><div id="prog-fill"></div></div>
  </div>

  <div id="turbo-wrap">
    <div id="turbo-label">⚡ Turbo</div>
    <div id="turbo-track"><div id="turbo-fill"></div></div>
  </div>
</div>

<!-- Speed lines -->
<div id="slines">
  <div class="sl" style="left:8%;height:65%;animation-delay:0s"></div>
  <div class="sl" style="left:18%;height:50%;animation-delay:.04s"></div>
  <div class="sl" style="left:31%;height:75%;animation-delay:.09s"></div>
  <div class="sl" style="left:47%;height:60%;animation-delay:.02s"></div>
  <div class="sl" style="left:60%;height:70%;animation-delay:.06s"></div>
  <div class="sl" style="left:74%;height:45%;animation-delay:.03s"></div>
  <div class="sl" style="left:87%;height:55%;animation-delay:.07s"></div>
</div>

<div id="portal-flash"></div>
<div id="level-banner"></div>

<!-- Bottom buttons -->
<div id="btn-bar">
  <button class="btn" onclick="G.camToggle()">📷 Cam</button>
  <button class="btn" onclick="G.toggleWire()">⬡ Wire</button>
  <button class="btn danger" onclick="G.restart()">↺ Restart</button>
</div>

<div id="hints">
  <kbd>A</kbd><kbd>D</kbd> / <kbd>←</kbd><kbd>→</kbd> Lane<br>
  <kbd>W</kbd><kbd>S</kbd> Speed &nbsp;<kbd>Space</kbd> Turbo<br>
  <kbd>R</kbd> Restart
</div>

<!-- Overlays -->
<div class="overlay" id="ov-start">
  <div class="card">
    <div class="card-title accent">TURBO RUSH</div>
    <div class="card-sub" style="margin-bottom:24px">Lewati 4 Dunia — Raih Finish!</div>
    <div style="display:flex;gap:16px;flex-wrap:wrap;justify-content:center;margin-bottom:24px">
      <span style="font-size:22px;filter:drop-shadow(0 0 6px rgba(255,255,255,0.3))">🌅</span>
      <span style="font-size:22px;filter:drop-shadow(0 0 6px rgba(255,255,255,0.3))">🏜️</span>
      <span style="font-size:22px;filter:drop-shadow(0 0 6px rgba(255,255,255,0.3))">🌊</span>
      <span style="font-size:22px;filter:drop-shadow(0 0 6px rgba(255,255,255,0.3))">🏙️</span>
    </div>
    <button class="card-btn" onclick="G.start()">▶ MULAI BALAPAN</button>
  </div>
</div>

<div class="overlay hidden" id="ov-gameover">
  <div class="card">
    <div class="card-title danger" style="color:#ff6b6b;text-shadow:0 0 30px rgba(255,77,77,0.4)">GAME OVER</div>
    <div class="card-sub">Skor Akhir</div>
    <div class="card-score" id="go-score">0</div>
    <div class="card-sub" id="go-msg" style="margin-top:-8px"></div>
    <button class="card-btn danger" onclick="G.restart()">↺ COBA LAGI</button>
  </div>
</div>

<div class="overlay hidden" id="ov-win">
  <div class="card">
    <div class="card-title accent">🏆 SELAMAT!</div>
    <div class="card-sub">Semua rintangan terlewati</div>
    <div class="card-score" id="win-score">0</div>
    <div class="card-sub" id="win-rank" style="margin-top:-8px"></div>
    <button class="card-btn" onclick="G.restart()">↺ MAIN LAGI</button>
  </div>
</div>

<script type="module">
import * as THREE from 'https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.module.js';

// ── CONSTANTS ─────────────────────────────────────────────────────────────────
const ROAD_W   = 12;
const LANE_W   = ROAD_W / 3;
const SEG_LEN  = 10;
const NUM_SEGS = 50; // dikurangi dari 70 → 50, cukup untuk tampilan + hemat draw call
const PORTAL_EVERY = 160;

const LEVELS = [
  // speedBase = kecepatan default, speedMin = batas bawah S, speedMax = batas atas W
  { name:'Pedesaan',  emoji:'🌅', sky:0x6fb3d2, fog:0x9ecfe0, fogD:0.008, ground:0x3a7a30, road:0x2e2e2e, ambCol:0xfff5e0, sunCol:0xfff0d0, sunPos:[-60,55,-180], night:false, trees:true,  desert:false, beach:false, city:false, timeLimit:90,  speedBase:0.10, speedMin:0.06, speedMax:0.18, label:'' },
  { name:'Gurun',     emoji:'🏜️', sky:0xd4804a, fog:0xdfbf9a, fogD:0.009, ground:0xd8b86a, road:0x3d3c34, ambCol:0xffe8c0, sunCol:0xffdd88, sunPos:[0,90,-180],  night:false, trees:false, desert:true,  beach:false, city:false, timeLimit:90,  speedBase:0.18, speedMin:0.12, speedMax:0.26, label:'' },
  { name:'Pantai',    emoji:'🌊', sky:0xd46040, fog:0xde9070, fogD:0.010, ground:0xd8c898, road:0x4a4640, ambCol:0xffe0b3, sunCol:0xff8833, sunPos:[70,25,-180],  night:false, trees:false, desert:false, beach:true,  city:false, timeLimit:95,  speedBase:0.26, speedMin:0.18, speedMax:0.36, label:'' },
  { name:'Perkotaan', emoji:'🏙️', sky:0x030810, fog:0x030810, fogD:0.014, ground:0x101018, road:0x181820, ambCol:0x1c2240, sunCol:0xffffff, sunPos:[-40,65,-180], night:true,  trees:false, desert:false, beach:false, city:true,  timeLimit:120, speedBase:0.36, speedMin:0.26, speedMax:0.50, label:'' },
];

// ── RENDERER ──────────────────────────────────────────────────────────────────
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const scene  = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(65, innerWidth / innerHeight, 0.1, 300);

// ── LIGHTS ────────────────────────────────────────────────────────────────────
const ambientLight = new THREE.AmbientLight(0xffffff, 0.9);
const sunLight     = new THREE.DirectionalLight(0xffffff, 1.6);
sunLight.castShadow = true;
scene.add(ambientLight, sunLight);

const neonL = new THREE.PointLight(0xff00ff, 0, 30);
const neonR = new THREE.PointLight(0x00ffff, 0, 30);
scene.add(neonL, neonR);

// ── SKY / GROUND ──────────────────────────────────────────────────────────────
const skySphere = new THREE.Mesh(
  new THREE.SphereGeometry(250, 16, 8),
  new THREE.MeshBasicMaterial({ color: 0x87ceeb, side: THREE.BackSide })
);
scene.add(skySphere);

const groundMesh = new THREE.Mesh(
  new THREE.PlaneGeometry(400, 800),
  new THREE.MeshStandardMaterial({ color: 0x4caf50, roughness: 1 })
);
groundMesh.rotation.x = -Math.PI / 2;
groundMesh.position.y = -0.05;
scene.add(groundMesh);

const sunDisc = new THREE.Mesh(
  new THREE.CircleGeometry(10, 16),
  new THREE.MeshBasicMaterial({ color: 0xffee88 })
);
sunDisc.position.set(-60, 80, -150);
sunDisc.lookAt(0, 0, 0);
scene.add(sunDisc);

// ── STARS & MOON ─────────────────────────────────────────────────────────────
let starField = null, moonMesh = null;

function initStars() {
  const geo = new THREE.BufferGeometry();
  const cnt = 400, pos = new Float32Array(cnt * 3);
  for (let i = 0; i < cnt; i++) {
    const th = Math.random() * Math.PI * 2, ph = Math.acos(2 * Math.random() - 1), r = 240;
    pos[i*3]   = r * Math.sin(ph) * Math.cos(th);
    pos[i*3+1] = Math.abs(r * Math.cos(ph)) + 10;
    pos[i*3+2] = r * Math.sin(ph) * Math.sin(th);
  }
  geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
  starField = new THREE.Points(geo, new THREE.PointsMaterial({ color: 0xffffff, size: 1.2, sizeAttenuation: false }));
  starField.visible = false;
  scene.add(starField);
}

function initMoon() {
  const cv = document.createElement('canvas');
  cv.width = cv.height = 128;
  const cx = cv.getContext('2d');
  cx.fillStyle = '#f0f4ff';
  cx.beginPath(); cx.arc(64, 64, 42, 0, Math.PI * 2); cx.fill();
  cx.globalCompositeOperation = 'destination-out';
  cx.beginPath(); cx.arc(80, 54, 42, 0, Math.PI * 2); cx.fill();
  moonMesh = new THREE.Mesh(
    new THREE.PlaneGeometry(16, 16),
    new THREE.MeshBasicMaterial({ map: new THREE.CanvasTexture(cv), transparent: true, side: THREE.DoubleSide })
  );
  moonMesh.position.set(-60, 80, -150);
  moonMesh.lookAt(0, 0, 0);
  moonMesh.visible = false;
  scene.add(moonMesh);
}

// ── BUILDING MATERIALS ────────────────────────────────────────────────────────
let bldMatDay, bldMatNight;
function initBuildingMats() {
  const cd = document.createElement('canvas'); cd.width = 128; cd.height = 256;
  const cxd = cd.getContext('2d');
  cxd.fillStyle = '#2d3a4a'; cxd.fillRect(0, 0, 128, 256);
  cxd.fillStyle = '#7a9ab8';
  for (let r = 0; r < 14; r++) for (let c = 0; c < 4; c++) cxd.fillRect(14+c*26, 18+r*17, 12, 10);
  bldMatDay = new THREE.MeshStandardMaterial({ map: new THREE.CanvasTexture(cd), roughness: 0.3, metalness: 0.7 });

  const cn = document.createElement('canvas'); cn.width = 128; cn.height = 256;
  const cxn = cn.getContext('2d');
  cxn.fillStyle = '#0a0f1a'; cxn.fillRect(0, 0, 128, 256);
  cxn.fillStyle = '#ffe87a';
  for (let r = 0; r < 14; r++) for (let c = 0; c < 4; c++) if (Math.random() > 0.3) cxn.fillRect(14+c*26, 18+r*17, 12, 10);
  const nightTex = new THREE.CanvasTexture(cn);
  bldMatNight = new THREE.MeshStandardMaterial({ map: nightTex, emissiveMap: nightTex, emissive: new THREE.Color(0xffe87a), emissiveIntensity: 0.75, roughness: 0.4, metalness: 0.5 });
}

// ── ROAD ──────────────────────────────────────────────────────────────────────
const roadMat  = new THREE.MeshStandardMaterial({ color: 0x2e2e2e, roughness: 0.95 });
const roadSegs = [];

function makeRoadSeg(z) {
  const g = new THREE.Group();
  const surf = new THREE.Mesh(new THREE.PlaneGeometry(ROAD_W, SEG_LEN), roadMat.clone());
  surf.rotation.x = -Math.PI / 2;
  surf.receiveShadow = true;
  g.add(surf);

  for (let l = 1; l < 3; l++) {
    const lx = -ROAD_W/2 + LANE_W * l;
    const div = new THREE.Mesh(
      new THREE.PlaneGeometry(0.1, SEG_LEN * 0.55),
      new THREE.MeshBasicMaterial({ color: 0xddddaa })
    );
    div.rotation.x = -Math.PI / 2;
    div.position.set(lx, 0.01, 0);
    g.add(div);
  }

  const cy = new THREE.Mesh(
    new THREE.PlaneGeometry(0.18, SEG_LEN * 0.55),
    new THREE.MeshBasicMaterial({ color: 0xffcc00 })
  );
  cy.rotation.x = -Math.PI / 2;
  cy.position.set(0, 0.01, 0);
  g.add(cy);

  const kCol = (Math.floor(z / SEG_LEN) % 2 === 0) ? 0xcc2222 : 0xffffff;
  for (const kx of [-(ROAD_W/2 + 0.3), (ROAD_W/2 + 0.3)]) {
    const k = new THREE.Mesh(
      new THREE.BoxGeometry(0.55, 0.1, SEG_LEN),
      new THREE.MeshStandardMaterial({ color: kCol, roughness: 0.85 })
    );
    k.position.set(kx, 0.05, 0);
    g.add(k);
  }

  g.position.set(0, 0, z + SEG_LEN / 2);
  scene.add(g);
  return g;
}

function rebuildRoad() {
  roadSegs.forEach(s => scene.remove(s));
  roadSegs.length = 0;
  for (let i = 0; i < NUM_SEGS; i++) roadSegs.push(makeRoadSeg(-i * SEG_LEN));
}

// ── SCENERY POOL ──────────────────────────────────────────────────────────────
const sceneryPool = [];
function clearScenery() { sceneryPool.forEach(s => scene.remove(s)); sceneryPool.length = 0; }

const SIDE_L = -(ROAD_W/2);
const SIDE_R =  (ROAD_W/2);

function makeTree(x, z) {
  const g = new THREE.Group();
  const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.22, 0.3, 2, 6), new THREE.MeshStandardMaterial({ color: 0x7b4f2e, roughness: 1 }));
  trunk.position.y = 1;
  const top = new THREE.Mesh(new THREE.ConeGeometry(1.5, 3.2, 7), new THREE.MeshStandardMaterial({ color: 0x2d6a30, roughness: 0.9 }));
  top.position.y = 3.6;
  g.add(trunk, top);
  g.position.set(x, 0, z);
  scene.add(g); sceneryPool.push(g);
}

function makePalm(x, z) {
  const g = new THREE.Group();
  const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.25, 3.5, 6), new THREE.MeshStandardMaterial({ color: 0x9a7a3a, roughness: 1 }));
  trunk.position.y = 1.75;
  g.add(trunk);
  for (let d = 0; d < 360; d += 60) {
    const leaf = new THREE.Mesh(new THREE.BoxGeometry(0.18, 2.2, 0.1), new THREE.MeshStandardMaterial({ color: 0x33aa44, roughness: 0.9 }));
    leaf.position.set(Math.cos(d*Math.PI/180)*.85, 3.8+Math.sin(d*Math.PI/180)*.3, Math.sin(d*Math.PI/180)*.85);
    leaf.rotation.z = Math.cos(d*Math.PI/180)*0.5;
    leaf.rotation.x = Math.sin(d*Math.PI/180)*0.5;
    g.add(leaf);
  }
  g.position.set(x, 0, z);
  scene.add(g); sceneryPool.push(g);
}

function makeCactus(x, z) {
  const g = new THREE.Group();
  const mat = new THREE.MeshStandardMaterial({ color: 0x2e5a1c, roughness: 0.9, flatShading: true });
  const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.25, 0.28, 3.2, 8), mat);
  trunk.position.y = 1.6; trunk.castShadow = true;
  g.add(trunk);
  const al = new THREE.Mesh(new THREE.CylinderGeometry(0.16, 0.16, 0.7, 8), mat);
  al.rotation.z = Math.PI/2; al.position.set(-0.44, 2.0, 0); g.add(al);
  const al2 = new THREE.Mesh(new THREE.CylinderGeometry(0.16, 0.16, 1.2, 8), mat);
  al2.position.set(-0.8, 2.5, 0); g.add(al2);
  const ar = new THREE.Mesh(new THREE.CylinderGeometry(0.16, 0.16, 0.7, 8), mat);
  ar.rotation.z = Math.PI/2; ar.position.set(0.44, 1.4, 0); g.add(ar);
  const ar2 = new THREE.Mesh(new THREE.CylinderGeometry(0.16, 0.16, 1.2, 8), mat);
  ar2.position.set(0.8, 1.9, 0); g.add(ar2);
  g.position.set(x, 0, z);
  scene.add(g); sceneryPool.push(g);
}

function makeDune(x, z) {
  const m = new THREE.Mesh(new THREE.SphereGeometry(3 + Math.random()*3, 8, 6), new THREE.MeshStandardMaterial({ color: 0xe2c175, roughness: 1 }));
  m.scale.y = 0.32; m.position.set(x, -0.5, z);
  scene.add(m); sceneryPool.push(m);
}

function makeRock(x, z) {
  const g = new THREE.Group();
  const mat = new THREE.MeshStandardMaterial({ color: 0x5a5d64, roughness: 0.9, flatShading: true });
  for (let i = 0; i < 2; i++) {
    const s = 4 + Math.random()*5;
    const m = new THREE.Mesh(new THREE.DodecahedronGeometry(s), mat);
    m.position.set((Math.random()-.5)*3, s*.4-1, (Math.random()-.5)*3);
    m.rotation.set(Math.random()*Math.PI, Math.random()*Math.PI, 0);
    m.castShadow = true;
    g.add(m);
  }
  g.position.set(x, 0, z);
  scene.add(g); sceneryPool.push(g);
}

let oceanMesh = null;
function addOcean() {
  if (oceanMesh) { scene.remove(oceanMesh); oceanMesh = null; }
  oceanMesh = new THREE.Mesh(
    new THREE.PlaneGeometry(180, 800),
    new THREE.MeshStandardMaterial({ color: 0x0099bb, transparent: true, opacity: 0.85, roughness: 0.2 })
  );
  oceanMesh.rotation.x = -Math.PI/2;
  oceanMesh.position.set(100, 0.05, 0);
  scene.add(oceanMesh);
  sceneryPool.push(oceanMesh);
}

function makeBuilding(x, z, night) {
  const h = 8 + Math.random()*16, w = 4 + Math.random()*4, d = 4 + Math.random()*4;
  const m = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), night ? bldMatNight : bldMatDay);
  m.position.set(x, h/2, z);
  m.castShadow = true; m.receiveShadow = true;
  scene.add(m); sceneryPool.push(m);
}

function makeStreetLight(x, z) {
  const g = new THREE.Group();
  const mm = new THREE.MeshStandardMaterial({ color: 0x1a1a1a, metalness: 0.8, roughness: 0.3 });
  const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.1, 5, 7), mm);
  pole.position.y = 2.5; g.add(pole);
  const dir = x > 0 ? -1 : 1;
  const arm = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 1.4, 7), mm);
  arm.rotation.z = Math.PI/2; arm.position.set(dir*.65, 5, 0); g.add(arm);
  const bulb = new THREE.Mesh(new THREE.SphereGeometry(0.18, 8, 8), new THREE.MeshBasicMaterial({ color: 0xffea77 }));
  bulb.position.set(dir*1.3, 4.9, 0); g.add(bulb);
  const pl = new THREE.PointLight(0xffea77, 2.0, 12, 1.5);
  pl.position.set(dir*1.3, 4.7, 0); g.add(pl);
  g.position.set(x, 0, z);
  scene.add(g); sceneryPool.push(g);
}

function createGantry(text, color) {
  const g = new THREE.Group();
  const mm = new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.8, roughness: 0.25 });
  for (const sx of [-(ROAD_W/2+0.5), (ROAD_W/2+0.5)]) {
    const p = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.24, 6, 8), mm);
    p.position.set(sx, 3, 0); p.castShadow = true; g.add(p);
  }
  const bar = new THREE.Mesh(new THREE.BoxGeometry(ROAD_W+1.2, 1.1, 0.4), mm);
  bar.position.set(0, 5.8, 0); bar.castShadow = true; g.add(bar);
  const board = new THREE.Mesh(new THREE.BoxGeometry(ROAD_W-2, 0.75, 0.45), new THREE.MeshStandardMaterial({ color, roughness: 0.6 }));
  board.position.set(0, 5.8, 0.06); g.add(board);
  const cv = document.createElement('canvas'); cv.width = 512; cv.height = 128;
  const cx = cv.getContext('2d');
  cx.fillStyle = '#0f172a'; cx.fillRect(0,0,512,128);
  cx.strokeStyle = '#e2e8f0'; cx.lineWidth = 8; cx.strokeRect(4,4,504,120);
  cx.font = 'bold 68px "Orbitron", sans-serif';
  cx.fillStyle = '#ffffff'; cx.textAlign = 'center'; cx.textBaseline = 'middle';
  cx.fillText(text, 256, 64);
  const tm = new THREE.Mesh(new THREE.PlaneGeometry(ROAD_W-2.2, 0.65), new THREE.MeshBasicMaterial({ map: new THREE.CanvasTexture(cv), transparent: true }));
  tm.position.set(0, 5.8, 0.27); g.add(tm);
  return g;
}

// ── CAR BUILDER ───────────────────────────────────────────────────────────────
function buildCar(bodyColor, roofColor) {
  const g = new THREE.Group();
  const bm = new THREE.MeshStandardMaterial({ color: bodyColor, metalness: 0.7, roughness: 0.3 });
  const rm = new THREE.MeshStandardMaterial({ color: roofColor, metalness: 0.5, roughness: 0.4 });
  const body  = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.55, 3.8), bm); body.position.y = 0.52;
  const cabin = new THREE.Mesh(new THREE.BoxGeometry(1.5, 0.5, 2), rm); cabin.position.set(0, 1.02, -0.1);
  const glF = new THREE.Mesh(new THREE.BoxGeometry(1.44, 0.4, 0.08), new THREE.MeshStandardMaterial({ color: 0x88ccff, transparent: true, opacity: 0.55, metalness: 0.3 }));
  glF.position.set(0, 0.98, 0.9);
  const glB = glF.clone(); glB.position.set(0, 0.98, -1.1);
  const bumper = new THREE.Mesh(new THREE.BoxGeometry(1.7, 0.22, 0.18), new THREE.MeshStandardMaterial({ color: 0xcccccc, metalness: 0.9, roughness: 0.2 }));
  bumper.position.set(0, 0.35, 1.98);
  g.add(body, cabin, glF, glB, bumper);
  const tireMat = new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.9 });
  const rimMat  = new THREE.MeshStandardMaterial({ color: 0xdddddd, metalness: 0.8, roughness: 0.2 });
  for (const [wx, wz] of [[0.95,1.4],[-0.95,1.4],[0.95,-1.4],[-0.95,-1.4]]) {
    const wg = new THREE.Group();
    const t = new THREE.Mesh(new THREE.CylinderGeometry(0.38, 0.38, 0.28, 16), tireMat); t.rotation.z = Math.PI/2;
    const r = new THREE.Mesh(new THREE.CylinderGeometry(0.2,  0.2,  0.3,  8),  rimMat);  r.rotation.z = Math.PI/2;
    wg.add(t, r);
    wg.position.set(wx, 0.38, wz);
    wg.userData.isWheel = true;
    g.add(wg);
  }
  // SpotLight di belakang model (z=-2) — setelah diputar 180°, ini jadi arah belakang dunia (ke kamera)
  // Efek: cahaya menyinari jalan di belakang mobil (seperti lampu rem/mundur yang kuat)
  const spot = new THREE.SpotLight(0xffffcc, 2, 18, 0.4, 0.5);
  spot.position.set(0, 1, -2);
  const st = new THREE.Object3D(); st.position.set(0, 0, -8);
  g.add(spot, st); spot.target = st;
  g.traverse(o => { if (o.isMesh) { o.castShadow = true; o.receiveShadow = true; } });
  return g;
}

const playerCar = buildCar(0xff2020, 0xcc1515);
playerCar.position.set(0, 0, 5);
playerCar.rotation.y = Math.PI; // putar 180°: alpot di depan, lampu di belakang
scene.add(playerCar);

// ── COINS / PICKUPS ───────────────────────────────────────────────────────────
const coinPool = [];
function laneX(l) { return (l - 1) * LANE_W; }

function spawnCoin(lane, z) {
  const g = new THREE.Group();
  g.add(
    new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.4, 0.12, 16), new THREE.MeshStandardMaterial({ color: 0xffd700, emissive: 0xffaa00, emissiveIntensity: 0.8, metalness: 0.6 })),
    new THREE.Mesh(new THREE.BoxGeometry(0.22, 0.14, 0.22), new THREE.MeshStandardMaterial({ color: 0xff8800, emissive: 0xff6600, emissiveIntensity: 1 }))
  );
  g.position.set(laneX(lane), 0.7, z);
  g.userData = { rotSpeed: 0.04, bobPhase: Math.random()*Math.PI*2, collected: false, isTurbo: false };
  scene.add(g); coinPool.push(g);
}

function spawnTurbo(lane, z) {
  const g = new THREE.Group();
  const m = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.5), new THREE.MeshStandardMaterial({ color: 0x00e5ff, emissive: 0x00e5ff, emissiveIntensity: 1.5 }));
  m.rotation.y = Math.PI/4;
  g.add(m, new THREE.PointLight(0x00e5ff, 2, 4));
  g.position.set(laneX(lane), 0.9, z);
  g.userData = { rotSpeed: 0.07, bobPhase: Math.random()*Math.PI*2, collected: false, isTurbo: true };
  scene.add(g); coinPool.push(g);
}

// ── OPPONENTS ──────────────────────────────────────────────────────────────────
const oppPool  = [];
const OPP_COLS = [0x2255cc, 0x22aa44, 0xaa8800, 0x883300, 0xcc44aa, 0x226688];

function spawnOpp(lane, z) {
  const col = OPP_COLS[Math.floor(Math.random()*OPP_COLS.length)];
  const car = buildCar(col, new THREE.Color(col).multiplyScalar(0.65).getHex());
  car.position.set(laneX(lane), 0, z);
  car.rotation.y = Math.PI;
  car.userData = { speed: 0.12 + Math.random()*0.12, lane };
  scene.add(car); oppPool.push(car);
}

// ── PORTAL / FINISH ────────────────────────────────────────────────────────────
let portalMesh = null;

function spawnPortal(z) {
  if (portalMesh) scene.remove(portalMesh);
  const g = new THREE.Group();
  const ring = new THREE.Mesh(new THREE.TorusGeometry(2.2, 0.3, 12, 30), new THREE.MeshStandardMaterial({ color: 0x00ffcc, emissive: 0x00ffcc, emissiveIntensity: 2, metalness: 0.8 }));
  ring.rotation.y = Math.PI/2;
  const disc = new THREE.Mesh(new THREE.CircleGeometry(2, 24), new THREE.MeshBasicMaterial({ color: 0x00ffcc, transparent: true, opacity: 0.35, side: THREE.DoubleSide }));
  disc.rotation.y = Math.PI/2;
  g.add(ring, disc, new THREE.PointLight(0x00ffcc, 3, 10));
  g.position.set(0, 2.2, z);
  scene.add(g); portalMesh = g;
}

function spawnFinish(z) {
  if (portalMesh) scene.remove(portalMesh);
  portalMesh = createGantry('FINISH', 0x15803d);
  portalMesh.position.set(0, 0, z);
  scene.add(portalMesh);
}

// ── PARTICLES ─────────────────────────────────────────────────────────────────
const particles = [];
const pGeo = new THREE.SphereGeometry(0.07, 4, 4);

function burst(pos, col = 0xffd700) {
  if (particles.length > 80) return; // cap: jangan spawn jika sudah terlalu banyak
  for (let i = 0; i < 10; i++) { // dikurangi dari 14 → 10
    const m = new THREE.Mesh(pGeo, new THREE.MeshBasicMaterial({ color: col, transparent: true }));
    m.position.copy(pos);
    m.userData.vel  = new THREE.Vector3((Math.random()-.5)*.55, Math.random()*.38, (Math.random()-.5)*.55);
    m.userData.life = 1;
    scene.add(m); particles.push(m);
  }
}

const exhaustGeo = new THREE.PlaneGeometry(0.22, 0.12);
function spawnExhaustFlame(pos, intensity = 1) {
  if (particles.length > 80) return; // cap bersama burst
  const flameCount = Math.round(3 * intensity); // dikurangi dari 4 → 3
  for (let i = 0; i < flameCount; i++) {
    const col = (Math.random() < 0.5) ? 0xffa000 : 0xff3300;
    const mat = new THREE.MeshBasicMaterial({ color: col, transparent: true, opacity: 0.95, depthWrite: false });
    const m = new THREE.Mesh(exhaustGeo, mat);
    m.position.copy(pos);
    m.rotation.y = (Math.random() - 0.5) * 0.6;
    m.rotation.z = (Math.random() - 0.5) * 0.6;
    m.userData.vel = new THREE.Vector3(
      (Math.random() - 0.5) * 0.05,
      Math.random() * 0.03,
      +0.35 + Math.random() * 0.55  // +Z: menyembur ke depan (arah alpot)
    );
    m.userData.life = 0.55 + Math.random() * 0.35;
    m.userData.scale = 0.7 + Math.random() * 0.8;
    m.scale.setScalar(m.userData.scale);
    scene.add(m);
    particles.push(m);
  }
}

// ── GAME STATE ────────────────────────────────────────────────────────────────
const G = {
  running: false, inMenu: true,
  score: 0, timeLeft: 90, lives: 3,
  currentLevel: 0,
  distance: 0,
  nextPortal: PORTAL_EVERY,
  portalActive: false,
  playerLane: 1, laneTarget: 0, laneLerp: 1,
  camMode: 0,
  wireframe: false,
  turbo: 100, turboActive: false,
  speed: 0.15,
  spawnZ: -40,
  transitioning: false,
  invulnerable: false,
  keys: {},
};

window.G = G;

// ── HUD HELPERS ───────────────────────────────────────────────────────────────
function updateHearts() {
  const hearts = document.querySelectorAll('.heart');
  hearts.forEach((h, i) => h.classList.toggle('dead', i >= G.lives));
}

let hudAccum = 0;
function tickHUD(dt) {
  hudAccum += dt;
  if (hudAccum < 0.06) return;
  hudAccum = 0;
  document.getElementById('v-score').textContent = G.score;
  document.getElementById('v-lvl').textContent   = G.currentLevel + 1;
  document.getElementById('v-tmr').textContent   = G.timeLeft;
  document.getElementById('v-spd').textContent   = Math.round(G.speed * 360);
  // tampilkan label kecepatan level di bawah angka speed
  const lvlLabel = document.getElementById('v-spd-label');
  if (lvlLabel) lvlLabel.textContent = '';
  document.getElementById('turbo-fill').style.width = G.turbo + '%';
  document.getElementById('prog-fill').style.width  = Math.min(100, (G.distance / G.nextPortal) * 100) + '%';
  document.getElementById('slines').style.opacity   = (G.turboActive && G.turbo > 0) ? '1' : '0';
}

// ── APPLY LEVEL VISUALS ───────────────────────────────────────────────────────
function applyLevel(idx) {
  const L = LEVELS[idx];
  renderer.setClearColor(L.sky);
  scene.background = new THREE.Color(L.sky);
  scene.fog = new THREE.FogExp2(L.fog, L.fogD);
  skySphere.material.color.setHex(L.sky);
  groundMesh.material.color.setHex(L.ground);
  roadMat.color.setHex(L.road);
  roadSegs.forEach(s => s.children.forEach(c => { if (c.material && c.material.color) c.material.color.setHex(L.road); }));
  ambientLight.color.setHex(L.ambCol);
  sunLight.color.setHex(L.sunCol);
  sunLight.position.set(...L.sunPos);

  if (L.night) {
    ambientLight.intensity = 0.32; sunLight.intensity = 0.18;
    neonL.intensity = 3.2; neonR.intensity = 3.2;
    starField.visible = true; moonMesh.visible = true; sunDisc.visible = false;
  } else {
    ambientLight.intensity = 0.9; sunLight.intensity = 1.6;
    neonL.intensity = 0; neonR.intensity = 0;
    starField.visible = false; moonMesh.visible = false; sunDisc.visible = true;
    sunDisc.material.color.setHex(L.sunCol);
    sunDisc.position.set(...L.sunPos); sunDisc.lookAt(0,0,0);
  }

  clearScenery();
  if (oceanMesh) { scene.remove(oceanMesh); oceanMesh = null; }

  const pz = playerCar.position.z;
  for (let dz = 14; dz < 160; dz += 18) { // lebih jarang dari sebelumnya (dulu 12..220 step 14)
    const z = pz - dz;
    if (L.trees) {
      makeTree(SIDE_R + 3 + Math.random()*5, z);
      makeTree(SIDE_L - 3 - Math.random()*5, z);
    }
    if (L.desert) {
      if (Math.random() > .35) makeCactus(SIDE_R + 4 + Math.random()*6, z);
      if (Math.random() > .35) makeCactus(SIDE_L - 4 - Math.random()*6, z);
      makeDune(SIDE_R + 20 + Math.random()*15, z);
      makeDune(SIDE_L - 20 - Math.random()*15, z);
    }
    if (L.beach) {
      makePalm(SIDE_L - 3.5 - Math.random()*3, z);
      makeRock(SIDE_R + 7 + Math.random()*10, z);
      if (dz === 12) addOcean();
    }
    if (L.city) {
      if (Math.random() > .35) makeBuilding(SIDE_R + 7 + Math.random()*7, z, true);
      if (Math.random() > .35) makeBuilding(SIDE_L - 7 - Math.random()*7, z, true);
      if (Math.floor(z/SEG_LEN) % 4 === 0) {
        makeStreetLight(SIDE_R + 0.6, z);
        makeStreetLight(SIDE_L - 0.6, z);
      }
    }
  }

  rebuildRoad();
  G.timeLeft = L.timeLimit;
  // Reset kecepatan ke speedBase level baru
  G.speed = L.speedBase;
  if (G.running) {
    const el = document.getElementById('level-banner');
    el.textContent = `${L.emoji} ${L.name}  ·  ${L.label}`;
    el.style.opacity = 1;
    setTimeout(() => el.style.opacity = 0, 2200);
  }
}

// ── SPAWNING ───────────────────────────────────────────────────────────────────
function spawnObjects() {
  const isNightHeavy = (G.currentLevel === 3);
  const SPAWN_MAX_ITER = isNightHeavy ? 1 : 2;
  const COIN_CAP = isNightHeavy ? 14 : 18; // dikurangi dari 22
  let iter = 0, spawnedThisCall = 0;

  while (coinPool.length < COIN_CAP && iter < SPAWN_MAX_ITER && spawnedThisCall < 1) {
    iter++;
    const z = G.spawnZ;
    G.spawnZ -= 9 + Math.random()*7;
    const lane = Math.floor(Math.random()*3);
    if (Math.random() < 0.11) { spawnTurbo(lane, z); }
    else { spawnCoin(lane, z); }
    spawnedThisCall++;
    if (oppPool.length < 5 && Math.random() < 0.28) {
      spawnOpp(Math.floor(Math.random()*3), z - 5);
    }
  }

  // Hapus scenery yang sudah jauh di belakang pemain (>80 unit) untuk hemat draw calls
  for (let i = sceneryPool.length - 1; i >= 0; i--) {
    if (sceneryPool[i].position.z > playerCar.position.z + 80) {
      scene.remove(sceneryPool[i]);
      sceneryPool.splice(i, 1);
    }
  }

  if (!G.portalActive && !portalMesh && G.distance >= G.nextPortal) {
    const pz = playerCar.position.z - 90;
    if (G.currentLevel === 3) spawnFinish(pz);
    else spawnPortal(pz);
    G.portalActive = true;
  }
}

// ── COLLISION ──────────────────────────────────────────────────────────────────
const _ba = new THREE.Box3(), _bb = new THREE.Box3();
function collide(a, b) {
  _ba.setFromObject(a); _bb.setFromObject(b);
  return _ba.intersectsBox(_bb);
}

// ── GAME EVENTS ────────────────────────────────────────────────────────────────
function loseLife() {
  if (G.invulnerable) return;
  G.lives = Math.max(0, G.lives - 1);
  G.invulnerable = true;
  updateHearts();
  setTimeout(() => G.invulnerable = false, 900);
  if (G.lives <= 0) endGame();
}

function triggerPortal() {
  if (G.transitioning) return;
  G.transitioning = true;
  const flash = document.getElementById('portal-flash');
  flash.style.opacity = 1;
  setTimeout(() => {
    scene.remove(portalMesh); portalMesh = null; G.portalActive = false;
    G.currentLevel++;
    G.nextPortal = G.distance + PORTAL_EVERY;
    G.spawnZ = playerCar.position.z - 30;
    [...coinPool].forEach(c => scene.remove(c)); coinPool.length = 0;
    [...oppPool].forEach(o => scene.remove(o)); oppPool.length = 0;
    if (G.currentLevel >= LEVELS.length) { triggerWin(); return; }
    applyLevel(G.currentLevel);
    spawnObjects();
    setTimeout(() => { flash.style.opacity = 0; G.transitioning = false; }, 300);
  }, 250);
}

function triggerWin() {
  G.running = false;
  document.getElementById('win-score').textContent = G.score;
  const dist = G.distance;
  let rank;
  if (dist >= PORTAL_EVERY * 3.0) rank = 'Ketahanan Legenda!';
  else if (dist >= PORTAL_EVERY * 2.1) rank = 'Tangguh Abis!';
  else if (dist >= PORTAL_EVERY * 1.2) rank = 'Bertahan Bagus!';
  else rank = 'Good Run!';
  document.getElementById('win-rank').textContent = rank;
  document.getElementById('ov-win').classList.remove('hidden');
}

function endGame() {
  G.running = false;
  document.getElementById('go-score').textContent = G.score;
  document.getElementById('go-msg').textContent   = 'Nyawa habis. Coba lagi!';
  document.getElementById('ov-gameover').classList.remove('hidden');
}

// ── START / RESTART ────────────────────────────────────────────────────────────
function startGame() {
  ['ov-start','ov-gameover','ov-win'].forEach(id => document.getElementById(id).classList.add('hidden'));
  Object.assign(G, {
    running: true, inMenu: false,
    score: 0, lives: 3, timeLeft: 90,
    currentLevel: 0, distance: 0,
    nextPortal: PORTAL_EVERY,
    portalActive: false,
    playerLane: 1, laneTarget: 0, laneLerp: 1,
    turbo: 100, turboActive: false,
    speed: LEVELS[0].speedBase, spawnZ: -40,
    transitioning: false, invulnerable: false,
  });
  G.laneTarget = laneX(1);
  playerCar.position.set(0, 0, 5);
  playerCar.rotation.set(0, Math.PI, 0); // tetap 180° saat restart
  [...coinPool].forEach(c => scene.remove(c)); coinPool.length = 0;
  [...oppPool].forEach(o => scene.remove(o));  oppPool.length = 0;
  if (portalMesh) { scene.remove(portalMesh); portalMesh = null; }
  clearScenery();
  applyLevel(0);
  for (const [lane, z] of [[0,-12],[2,-18]]) spawnOpp(lane, z);
  const sg = createGantry('START', 0xdc2626);
  sg.position.set(0, 0, -2);
  scene.add(sg); sceneryPool.push(sg);
  spawnObjects();
  updateHearts();
  startTimer();
}

G.start   = startGame;
G.restart = startGame;
G.camToggle  = () => { G.camMode = (G.camMode + 1) % 3; };
G.toggleWire = () => {
  G.wireframe = !G.wireframe;
  scene.traverse(o => { if (o.isMesh && o.material) o.material.wireframe = G.wireframe; });
};

// ── TIMER ─────────────────────────────────────────────────────────────────────
let timerInt = null;
function startTimer() {
  clearInterval(timerInt);
  timerInt = setInterval(() => {
    if (!G.running || G.transitioning) return;
    G.timeLeft--;
    if (G.timeLeft <= 0) endGame();
  }, 1000);
}

// ── INPUT ─────────────────────────────────────────────────────────────────────
window.addEventListener('keydown', e => {
  G.keys[e.code] = true;
  if (e.code === 'KeyR') startGame();
  if (e.code === 'Space') { G.turboActive = true; e.preventDefault(); }
  if (!G.running || G.transitioning) return;
  if ((e.code === 'KeyA' || e.code === 'ArrowLeft')  && G.playerLane > 0) { G.playerLane--; G.laneLerp = 0; G.laneTarget = laneX(G.playerLane); }
  if ((e.code === 'KeyD' || e.code === 'ArrowRight') && G.playerLane < 2) { G.playerLane++; G.laneLerp = 0; G.laneTarget = laneX(G.playerLane); }
});
window.addEventListener('keyup', e => {
  G.keys[e.code] = false;
  if (e.code === 'Space') G.turboActive = false;
});

// ── AUDIO ─────────────────────────────────────────────────────────────────────
// PERBAIKAN: audio object sekarang di-assign dengan benar ke variabel global.
// Sebelumnya: ensureAudio() membuat variabel lokal saja, sehingga audio tetap null.
// Sekarang: semua property (ctx, master, rumbleGain, rumbleFilter, bg) disimpan
// ke satu object `audio` yang bisa diakses di seluruh kode.

let audio = null;

function ensureAudio() {
  if (audio) return; // sudah diinisialisasi, skip

  const AudioCtx = window.AudioContext || window.webkitAudioContext;
  if (!AudioCtx) return;

  const ctx = new AudioCtx();

  // Master gain node
  const master = ctx.createGain();
  master.gain.value = 0.22;
  master.connect(ctx.destination);

  // Engine rumble: white noise + lowpass filter
  const bufferSize = ctx.sampleRate * 1.5;
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
  const data = buffer.getChannelData(0);
  for (let i = 0; i < bufferSize; i++) data[i] = (Math.random() * 2 - 1);

  const rumbleSrc = ctx.createBufferSource();
  rumbleSrc.buffer = buffer;
  rumbleSrc.loop = true;

  const rumbleFilter = ctx.createBiquadFilter();
  rumbleFilter.type = 'lowpass';
  rumbleFilter.frequency.value = 600;

  const rumbleGain = ctx.createGain();
  rumbleGain.gain.value = 0.0;

  rumbleSrc.connect(rumbleFilter);
  rumbleFilter.connect(rumbleGain);
  rumbleGain.connect(master);
  rumbleSrc.start();

  // Background music
  // File "son game.mpeg" harus ada di folder yang sama dengan TURBO_RUSH_3D.html
  let bg = null;
  try {
    bg = new Audio('son game.mpeg');
    bg.loop = true;
    bg.volume = 0.35;
    bg.preload = 'auto';
  } catch (e) {
    bg = null;
  }

  // Simpan semua ke object global — INI KUNCI PERBAIKANNYA
  audio = { ctx, master, rumbleGain, rumbleFilter, bg };
}

function setAudioOn(on) {
  ensureAudio();
  if (!audio) return;

  // Resume AudioContext jika suspended (wajib setelah gesture pertama)
  if (audio.ctx.state === 'suspended') audio.ctx.resume();

  if (audio.bg) {
    try {
      if (on) audio.bg.play().catch(() => {});
      else audio.bg.pause();
    } catch (e) {}
  }

  const now = audio.ctx.currentTime;
  audio.master.gain.cancelScheduledValues(now);
  audio.master.gain.setTargetAtTime(on ? 0.22 : 0.0, now, 0.04);
}

function playTickSFX(type) {
  if (!audio) return;
  const ctx = audio.ctx;
  const now = ctx.currentTime;

  const g = ctx.createGain();
  g.gain.value = 0.0001;
  g.connect(audio.master);

  if (type === 'coin') {
    const o = ctx.createOscillator(); o.type = 'sine';
    o.frequency.setValueAtTime(660, now);
    o.frequency.exponentialRampToValueAtTime(990, now + 0.05);
    g.gain.setValueAtTime(0.0001, now);
    g.gain.exponentialRampToValueAtTime(0.22, now + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, now + 0.12);
    o.connect(g); o.start(now); o.stop(now + 0.14);
    return;
  }
  if (type === 'hit') {
    const n = ctx.createOscillator(); n.type = 'square';
    n.frequency.setValueAtTime(120, now);
    n.frequency.exponentialRampToValueAtTime(60, now + 0.12);
    g.gain.setValueAtTime(0.0001, now);
    g.gain.exponentialRampToValueAtTime(0.35, now + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, now + 0.18);
    n.connect(g); n.start(now); n.stop(now + 0.2);
    return;
  }
}

// Aktifkan audio saat user pertama kali klik/tap (wajib untuk autoplay browser)
window.addEventListener('pointerdown', () => {
  ensureAudio();
  if (audio && audio.ctx.state === 'suspended') audio.ctx.resume();
  setAudioOn(true);
}, { once: true });

// ── MAIN LOOP ──────────────────────────────────────────────────────────────────
const clock = new THREE.Clock();
let elapsed = 0;

function animate() {
  requestAnimationFrame(animate);
  const dt = Math.min(clock.getDelta(), 0.05);
  elapsed += dt;

  if (G.inMenu) {
    camera.position.set(
      Math.sin(elapsed * 0.35) * 8.5,
      3.8 + Math.sin(elapsed * 0.15) * 0.6,
      playerCar.position.z + Math.cos(elapsed * 0.35) * 8.5
    );
    camera.lookAt(0, 0.9, playerCar.position.z - 2.5);
    renderer.render(scene, camera); return;
  }

  if (!G.running) { renderer.render(scene, camera); return; }

  const L = LEVELS[G.currentLevel];
  const base = L.speedBase;
  if (G.keys['KeyW'] || G.keys['ArrowUp'])        G.speed = Math.min(L.speedMax, G.speed + 0.006);
  else if (G.keys['KeyS'] || G.keys['ArrowDown']) G.speed = Math.max(L.speedMin, G.speed - 0.008);
  else G.speed += (base - G.speed) * 0.04; // drift ke kecepatan default level

  const spd = (G.turboActive && G.turbo > 0) ? G.speed * 1.5 : G.speed;
  const turboOn = (G.turboActive && G.turbo > 0);

  // Update engine rumble berdasarkan kecepatan
  if (audio) {
    const t = Math.min(1, spd / 0.3);
    const now = audio.ctx.currentTime;
    audio.rumbleGain.gain.setTargetAtTime(0.03 + t * 0.11, now, 0.08);
    audio.rumbleFilter.frequency.setTargetAtTime(350 + t * 1300, now, 0.08);
  }

  if (turboOn) {
    G.turbo = Math.max(0, G.turbo - 1.2);
    // Alpot di depan (world +Z) karena mobil diputar 180°
    const frontZ = playerCar.position.z + 2.0;
    const baseY = playerCar.position.y + 0.35;
    spawnExhaustFlame(new THREE.Vector3(playerCar.position.x - 0.65, baseY, frontZ), 1.0);
    spawnExhaustFlame(new THREE.Vector3(playerCar.position.x + 0.65, baseY, frontZ), 0.85);
  } else {
    G.turbo = Math.min(100, G.turbo + 0.25);
  }

  tickHUD(dt);

  G.laneLerp = Math.min(1, G.laneLerp + dt * 6);
  playerCar.position.x = THREE.MathUtils.lerp(playerCar.position.x, G.laneTarget, G.laneLerp);

  neonL.position.set(playerCar.position.x - 5, 5, playerCar.position.z - 5);
  neonR.position.set(playerCar.position.x + 5, 5, playerCar.position.z - 5);

  playerCar.children.forEach(c => {
    if (c.userData.isWheel) c.children.forEach(w => w.rotation.x += spd * 3);
  });

  G.distance += spd;

  for (let i = 0; i < roadSegs.length; i++) {
    roadSegs[i].position.z += spd;
    if (roadSegs[i].position.z > playerCar.position.z + SEG_LEN * 2) {
      scene.remove(roadSegs[i]);
      const newZ = roadSegs[i].position.z - NUM_SEGS * SEG_LEN - SEG_LEN;
      roadSegs[i] = makeRoadSeg(newZ);
    }
  }

  sceneryPool.forEach(s => s.position.z += spd);

  for (let i = coinPool.length - 1; i >= 0; i--) {
    const c = coinPool[i];
    c.position.z += spd;
    c.rotation.y += c.userData.rotSpeed;
    c.position.y = 0.7 + Math.sin(elapsed*3 + c.userData.bobPhase) * 0.18;
    if (!c.userData.collected && collide(playerCar, c)) {
      c.userData.collected = true;
      scene.remove(c); coinPool.splice(i, 1);
      if (c.userData.isTurbo) G.turbo = Math.min(100, G.turbo + 38);
      else { G.score++; burst(c.position.clone(), 0xffd700); playTickSFX('coin'); }
      continue;
    }
    if (c.position.z > playerCar.position.z + 15) { scene.remove(c); coinPool.splice(i, 1); }
  }

  for (let i = oppPool.length - 1; i >= 0; i--) {
    const o = oppPool[i];
    o.position.z += spd + o.userData.speed * 0.3;
    o.position.x  = laneX(o.userData.lane);
    if (collide(playerCar, o)) {
      scene.remove(o); oppPool.splice(i, 1);
      loseLife(); burst(playerCar.position.clone(), 0xff4400); playTickSFX('hit'); continue;
    }
    if (o.position.z > playerCar.position.z + 18) { scene.remove(o); oppPool.splice(i, 1); }
  }

  if (portalMesh) {
    portalMesh.position.z += spd;
    if (portalMesh.rotation) portalMesh.rotation.y += 0.02;
    if (collide(playerCar, portalMesh)) {
      triggerPortal();
    } else if (portalMesh.position.z > playerCar.position.z + 8) {
      scene.remove(portalMesh); portalMesh = null; G.portalActive = false;
      G.nextPortal = G.distance + PORTAL_EVERY;
    }
  }

  G.spawnZ += spd;
  spawnObjects();

  for (let i = particles.length - 1; i >= 0; i--) {
    const p = particles[i];
    p.userData.life -= dt * 2;
    p.position.addScaledVector(p.userData.vel, 1);
    p.userData.vel.y -= 0.01;
    p.material.opacity = p.userData.life;
    if (p.userData.life <= 0) { scene.remove(p); particles.splice(i, 1); }
  }

  const px = playerCar.position.x, pz = playerCar.position.z;
  if (G.camMode === 0) {
    camera.position.lerp(new THREE.Vector3(px*.3, 5.5, pz + 11), 0.1);
    camera.lookAt(px*.15, 0.5, pz - 12);
  } else if (G.camMode === 1) {
    camera.position.lerp(new THREE.Vector3(px, 22, pz + 4), 0.06);
    camera.lookAt(px, 0, pz - 6);
  } else {
    camera.position.lerp(new THREE.Vector3(px + 18, 6, pz), 0.06);
    camera.lookAt(px, 1, pz);
  }

  renderer.render(scene, camera);
}

// ── INIT ──────────────────────────────────────────────────────────────────────
initStars();
initMoon();
initBuildingMats();
applyLevel(0);
animate();

window.addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>
