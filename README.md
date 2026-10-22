[Uploading index (18).html…]()
<!doctype html>
<html lang="et">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">
<meta http-equiv="Pragma" content="no-cache">
<meta http-equiv="Expires" content="0">
<title>Kiire Kokk VR</title>
<script>
window.GAME_VERSION = 'kk2';
(function(){
  try {
    fetch(location.pathname + '?nocache=' + Date.now(), { cache: 'no-store' }).then(function(r){ return r.text(); }).then(function(t){
      var m = t.match(/window\.GAME_VERSION = '([^']+)'/);
      if (m && m[1] !== window.GAME_VERSION) { var k = 'kk_reload_' + m[1]; if (!sessionStorage.getItem(k)) { sessionStorage.setItem(k, '1'); location.replace(location.pathname + '?v=' + m[1] + '_' + Date.now()); } }
    }).catch(function(){});
  } catch (e) {}
})();
</script>
<style>
  html,body{margin:0;height:100%;background:#2b1d2e;overflow:hidden;font-family:system-ui,"Segoe UI",sans-serif}
  #info{position:fixed;top:10px;left:10px;color:#fff;background:rgba(40,20,40,.78);border:2px solid #ff8fab;padding:10px 14px;border-radius:14px;font-size:14px;line-height:1.5;max-width:430px;z-index:5}
  #info b{color:#ffd166}
</style>
<script type="importmap">{"imports":{"three":"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js","three/addons/":"https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"}}</script>
</head>
<body>
<div id="info">
  <b>🍔 KIIRE KOKK VR</b> <span style="opacity:.6">(kk2)</span><br>
  Questis: vajuta all <b>ENTER VR</b> – mängi pultide või paljaste kätega.<br>
  Arvutis: <b>klõps</b> = võta / pane maha, <b>parem klõps</b> = viska, <b>A/D</b> = pööra, <b>W/S</b> = vaata üles/alla, <b>Esc</b> = paus.
</div>
<script type="module">
import * as THREE from 'three';
import { VRButton } from 'three/addons/webxr/VRButton.js';
import { XRHandModelFactory } from 'three/addons/webxr/XRHandModelFactory.js';

/* =========================================================
   ABI
   ========================================================= */
const V3 = (x = 0, y = 0, z = 0) => new THREE.Vector3(x, y, z);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const rand = (a = 0, b = 1) => a + Math.random() * (b - a);
const pick = a => a[Math.floor(Math.random() * a.length)];
const store = (k, v) => { try { if (v === undefined) return JSON.parse(localStorage.getItem(k)); localStorage.setItem(k, JSON.stringify(v)); } catch (e) { return null; } };
const FONT = '"Segoe UI", system-ui, sans-serif';
const tmpV = V3(), tmpV2 = V3(), tmpD = V3(), tmpQ = new THREE.Quaternion(), head = V3();
function roundRect(c, x, y, w, h, r){ c.beginPath(); c.moveTo(x + r, y); c.arcTo(x + w, y, x + w, y + h, r); c.arcTo(x + w, y + h, x, y + h, r); c.arcTo(x, y + h, x, y, r); c.arcTo(x, y, x + w, y, r); c.closePath(); }
function canvasTex(w, h, fn, rep){ const cv = document.createElement('canvas'); cv.width = w; cv.height = h; fn(cv.getContext('2d'), w, h); const t = new THREE.CanvasTexture(cv); t.colorSpace = THREE.SRGBColorSpace; if (rep) { t.wrapS = t.wrapT = THREE.RepeatWrapping; t.repeat.set(rep[0], rep[1]); } t.anisotropy = 4; return t; }

/* =========================================================
   RENDERER, VALGUS, STSEEN
   ========================================================= */
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2)); renderer.setSize(innerWidth, innerHeight);
renderer.xr.enabled = true; renderer.xr.setReferenceSpaceType('local-floor');
renderer.shadowMap.enabled = true; renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping; renderer.toneMappingExposure = 1.05;
try { renderer.xr.setFoveation(0.3); } catch (e) {}
document.body.appendChild(renderer.domElement);
const vrButton = VRButton.createButton(renderer); document.body.appendChild(vrButton);
vrButton.addEventListener('click', () => audioInit());
const scene = new THREE.Scene(); scene.background = new THREE.Color(0x2b1d2e);
scene.fog = new THREE.Fog(0x2b1d2e, 9, 22);
const camera = new THREE.PerspectiveCamera(70, innerWidth / innerHeight, 0.02, 100);
const rig = new THREE.Group(); rig.add(camera); scene.add(rig);
camera.position.set(0, 1.6, 0.2);
let deskYaw = 0, deskPitch = -0.38;
addEventListener('resize', () => { camera.aspect = innerWidth / innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); });
const hemi = new THREE.HemisphereLight(0xfff4e6, 0x6b4a5a, 1.3); scene.add(hemi);
const sun = new THREE.DirectionalLight(0xfff0dd, 1.6); sun.position.set(1.5, 4, 1.5); sun.castShadow = true;
sun.shadow.mapSize.set(1024, 1024); Object.assign(sun.shadow.camera, { left: -2.2, right: 2.2, top: 2.2, bottom: -2.2, near: 0.5, far: 9 }); sun.shadow.bias = -0.0008; scene.add(sun); scene.add(sun.target); sun.target.position.set(0, 0.9, -0.6);
const discoLights = [0xff4fa3, 0x4fd1ff, 0xffe14f].map((c, i) => { const l = new THREE.PointLight(c, 0, 6, 1.5); l.position.set(0, 2.4, -2); scene.add(l); return l; });

const M = {};
const std = (color, o = {}) => new THREE.MeshStandardMaterial(Object.assign({ color, roughness: 0.6, metalness: 0 }, o));
Object.assign(M, {
  steel: std(0xc9d1d9, { roughness: 0.3, metalness: 0.75 }), darkSteel: std(0x4a5560, { roughness: 0.4, metalness: 0.7 }),
  counter: std(0xf7f3ee, { roughness: 0.35 }), wood: std(0xb07a4f, { roughness: 0.7 }), pink: std(0xff8fab), teal: std(0x35c2b3), yellow: std(0xffd166),
  black: std(0x222222, { roughness: 0.5 }), white: std(0xffffff, { roughness: 0.4 }), red: std(0xe63946, { roughness: 0.4 }), glass: std(0xbfe9ff, { transparent: true, opacity: 0.35, roughness: 0.1 })
});
const mesh = (geo, mat, x = 0, y = 0, z = 0, parent = scene, shadow = true) => { const m = new THREE.Mesh(geo, mat); m.position.set(x, y, z); if (shadow) { m.castShadow = true; m.receiveShadow = true; } parent.add(m); return m; };
const box = (w, h, d) => new THREE.BoxGeometry(w, h, d);
const cyl = (r1, r2, h, s = 24) => new THREE.CylinderGeometry(r1, r2, h, s);

/* =========================================================
   KÖÖK (kõrgus kohandub mängija pikkusega)
   ========================================================= */
const K = new THREE.Group(); scene.add(K);
const CT = 0.92;           // tööpinna kõrgus
const PASS_Y = 1.06;       // väljaandmisleti kõrgus
// põrand, seinad
{
  const floorTex = canvasTex(256, 256, (c, W, H) => { for (let i = 0; i < 8; i++) for (let j = 0; j < 8; j++) { c.fillStyle = (i + j) % 2 ? '#f4e9dc' : '#3d2c3e'; c.fillRect(i * 32, j * 32, 32, 32); } }, [10, 10]);
  mesh(new THREE.PlaneGeometry(14, 14).rotateX(-Math.PI / 2), std(0xffffff, { map: floorTex, roughness: 0.5 }), 0, 0, -2, scene, false).receiveShadow = true;
  const tile = canvasTex(256, 256, (c, W, H) => { c.fillStyle = '#fdf6ec'; c.fillRect(0, 0, W, H); c.strokeStyle = '#e8d8c6'; c.lineWidth = 4; for (let i = 0; i <= 256; i += 32) { c.beginPath(); c.moveTo(i, 0); c.lineTo(i, H); c.stroke(); c.beginPath(); c.moveTo(0, i); c.lineTo(W, i); c.stroke(); } }, [6, 2]);
  const wallM = std(0xffffff, { map: tile, roughness: 0.6 });
  for (const [x, ry] of [[-2.2, Math.PI / 2], [2.2, -Math.PI / 2]]) mesh(new THREE.PlaneGeometry(5, 3.2), wallM, x, 1.6, -0.2, scene, false).rotation.y = ry;
  mesh(new THREE.PlaneGeometry(4.4, 3.2), wallM, 0, 1.6, 1.6, scene, false).rotation.y = Math.PI;
  // söögisaal leti taga
  const hallTex = canvasTex(512, 256, (c, W, H) => { const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#ff8fab'); g.addColorStop(1, '#c95b84'); c.fillStyle = g; c.fillRect(0, 0, W, H); c.fillStyle = 'rgba(255,255,255,0.12)'; for (let x = 0; x < W; x += 40) c.fillRect(x, 0, 18, H); });
  mesh(new THREE.PlaneGeometry(12, 4), std(0xffffff, { map: hallTex }), 0, 2, -5.5, scene, false);
  // aknad ja öine linn
  const win = canvasTex(512, 256, (c, W, H) => { const g = c.createLinearGradient(0, 0, 0, H); g.addColorStop(0, '#1b1b4b'); g.addColorStop(1, '#6a3d8f'); c.fillStyle = g; c.fillRect(0, 0, W, H); for (let i = 0; i < 18; i++) { const x = i * 30, h = 60 + Math.random() * 120; c.fillStyle = '#251a3a'; c.fillRect(x, H - h, 26, h); c.fillStyle = '#ffd166'; for (let k = 0; k < 8; k++) if (Math.random() < 0.5) c.fillRect(x + 4 + (k % 2) * 10, H - h + 8 + Math.floor(k / 2) * 18, 6, 8); } c.fillStyle = '#fff8'; c.beginPath(); c.arc(420, 50, 22, 0, 7); c.fill(); });
  for (const x of [-3.2, 3.2]) { const w = mesh(new THREE.PlaneGeometry(2.2, 1.2), new THREE.MeshBasicMaterial({ map: win }), x, 2.1, -5.48, scene, false); mesh(box(2.3, 0.08, 0.06), M.white, x, 1.48, -5.46, scene, false); }
  // laudad saalis
  for (const [x, z] of [[-2.8, -3.6], [2.8, -3.6], [-1.2, -4.6], [1.4, -4.7]]) { mesh(cyl(0.45, 0.45, 0.04), M.white, x, 0.75, z); mesh(cyl(0.05, 0.05, 0.75), M.darkSteel, x, 0.375, z); for (const a of [0, 2.1, 4.2]) mesh(cyl(0.16, 0.16, 0.05), M.pink, x + Math.cos(a) * 0.6, 0.45, z + Math.sin(a) * 0.6); }
  // neoonsilt
  const sign = canvasTex(1024, 256, (c, W, H) => { c.fillStyle = 'rgba(0,0,0,0)'; c.fillRect(0, 0, W, H); c.font = `900 ${H * 0.55}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.shadowColor = '#ff4fa3'; c.shadowBlur = 40; c.fillStyle = '#ffd6e7'; c.fillText('🍔 KIIRE KOKK', W / 2, H / 2); c.shadowBlur = 20; c.fillText('🍔 KIIRE KOKK', W / 2, H / 2); });
  mesh(new THREE.PlaneGeometry(2.6, 0.65), new THREE.MeshBasicMaterial({ map: sign, transparent: true }), 0, 3.0, -5.4, scene, false);
  // laelambid
  for (const x of [-1, 0, 1]) { mesh(cyl(0.004, 0.004, 0.8), M.black, x, 2.85, -0.6, scene, false); const sh = mesh(new THREE.ConeGeometry(0.18, 0.16, 20, 1, true), std(0xff8fab, { side: THREE.DoubleSide }), x, 2.42, -0.6, scene, false); mesh(new THREE.SphereGeometry(0.05, 12, 8), new THREE.MeshBasicMaterial({ color: 0xfff1c9 }), x, 2.36, -0.6, scene, false); }
}
const SURF = [];   // füüsika pinnad (K-kohalikud): [x0,x1,z0,z1,y]
function counterBox(x0, x1, z0, z1, y, mat = M.counter, base = M.teal){
  const w = x1 - x0, d = z1 - z0, cx = (x0 + x1) / 2, cz = (z0 + z1) / 2;
  mesh(box(w, 0.05, d), mat, cx, y - 0.025, cz, K);
  mesh(box(w - 0.04, y - 0.05, d - 0.04), base, cx, (y - 0.05) / 2, cz, K);
  SURF.push([x0, x1, z0, z1, y]);
}
counterBox(-0.9, 0.9, -0.95, -0.32, CT);                 // põhilett
counterBox(-1.5, -0.9, -0.95, 0.55, CT);                 // vasak tiib
counterBox(0.9, 1.5, -0.95, 0.55, CT);                   // parem tiib
counterBox(-0.95, 0.95, -1.32, -0.98, PASS_Y, M.steel, M.pink);   // väljaandmislett
counterBox(-1.05, 1.05, 0.95, 1.45, CT);                // tagumine lett (avatavad toidud)
SURF.push([-3, 3, -3, 2, 0]);                            // põrand
// grill
const grill = new THREE.Group(); grill.position.set(-0.52, CT, -0.66); K.add(grill);
mesh(box(0.52, 0.05, 0.5), M.darkSteel, 0, 0.025, 0, grill);
for (let i = 0; i < 9; i++) mesh(box(0.5, 0.008, 0.012), M.black, 0, 0.054, -0.22 + i * 0.055, grill);
const grillGlow = mesh(box(0.48, 0.002, 0.46), new THREE.MeshBasicMaterial({ color: 0xff5722, transparent: true, opacity: 0.25 }), 0, 0.05, 0, grill, false);
SURF.push([-0.78, -0.26, -0.91, -0.41, CT + 0.058]);
// fritüür
const fryer = new THREE.Group(); fryer.position.set(0.6, CT, -0.66); K.add(fryer);
mesh(box(0.5, 0.12, 0.42), M.steel, 0, 0.06, 0, fryer);
for (const x of [-0.12, 0.12]) { mesh(box(0.2, 0.01, 0.3), new THREE.MeshStandardMaterial({ color: 0xd9a400, roughness: 0.15, metalness: 0.2 }), x, 0.115, 0, fryer, false); }
// limonaadimasin (parem tiib)
const soda = new THREE.Group(); soda.position.set(1.36, CT, -0.45); K.add(soda);
mesh(box(0.22, 0.55, 0.62), M.red, 0.02, 0.32, 0, soda);
mesh(box(0.02, 0.3, 0.55), std(0xffffff, { map: canvasTex(256, 512, (c, W, H) => { c.fillStyle = '#ffeef3'; c.fillRect(0, 0, W, H); c.font = `bold ${W * 0.3}px ${FONT}`; c.textAlign = 'center'; c.fillStyle = '#e63946'; c.save(); c.translate(W / 2, H / 2); c.rotate(-Math.PI / 2); c.fillText('LIMPS', 0, 30); c.restore(); }) }), -0.095, 0.42, 0, soda);
const FLAVORS = [{ n: 'Koola', c: 0x4e2a1e, e: '🟤' }, { n: 'Apelsin', c: 0xff8c1a, e: '🟠' }, { n: 'Laim', c: 0x7ed957, e: '🟢' }];
const taps = FLAVORS.map((f, i) => { const z = -0.2 + i * 0.2; mesh(cyl(0.012, 0.012, 0.06), M.steel, -0.13, 0.3, z - 0.0, soda); const b = mesh(box(0.05, 0.05, 0.03), std(f.c, { emissive: f.c, emissiveIntensity: 0.3 }), -0.11, 0.42, z, soda); b.rotation.z = 0.3; return { z, btn: b, f, i }; });
mesh(box(0.14, 0.01, 0.6), M.darkSteel, -0.13, 0.005, 0, soda);
// prügikast
const trash = new THREE.Group(); trash.position.set(-0.55, 0, 0.42); K.add(trash);
mesh(cyl(0.2, 0.17, 0.62, 20), M.teal, 0, 0.31, 0, trash); mesh(cyl(0.21, 0.21, 0.03, 20), M.darkSteel, 0, 0.62, 0, trash);
mesh(new THREE.PlaneGeometry(0.2, 0.08), new THREE.MeshBasicMaterial({ map: canvasTex(256, 100, (c, W, H) => { c.fillStyle = '#fff'; c.font = `bold ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('🗑️ PRÜGI', W / 2, H / 2); }), transparent: true }), 0, 0.45, 0.205, trash, false);
// väljaandmismatid
const MAT_X = [-0.6, 0, 0.6], MAT_Z = -1.15;
const mats = MAT_X.map((x, i) => { const m = mesh(box(0.42, 0.006, 0.26), std(0xffd166, { roughness: 0.8 }), x, PASS_Y + 0.003, MAT_Z, K, false); m.receiveShadow = true; return { i, x, z: MAT_Z, mesh: m, items: [], cust: null }; });
// kell
const bell = new THREE.Group(); bell.position.set(0.8, CT, -0.85); K.add(bell);
mesh(cyl(0.06, 0.06, 0.012), M.wood, 0, 0.006, 0, bell); mesh(new THREE.SphereGeometry(0.05, 20, 10, 0, Math.PI * 2, 0, Math.PI / 2), std(0xffd700, { metalness: 0.9, roughness: 0.25 }), 0, 0.012, 0, bell); mesh(new THREE.SphereGeometry(0.012, 10, 8), M.black, 0, 0.068, 0, bell);

/* =========================================================
   AUDIO
   ========================================================= */
let actx = null, master, sfxBus, musicBus, noiseBuf, sizzleG, fryG;
function audioInit(){
  if (actx) { if (actx.state === 'suspended' && mode !== 'pause') actx.resume(); return; }
  actx = new (window.AudioContext || window.webkitAudioContext)();
  master = actx.createGain(); master.gain.value = 0.9; const comp = actx.createDynamicsCompressor(); master.connect(comp); comp.connect(actx.destination);
  sfxBus = actx.createGain(); sfxBus.gain.value = 0.8; sfxBus.connect(master); musicBus = actx.createGain(); musicBus.gain.value = 0.35; musicBus.connect(master);
  noiseBuf = actx.createBuffer(1, actx.sampleRate * 2, actx.sampleRate); const d = noiseBuf.getChannelData(0); for (let i = 0; i < d.length; i++) d[i] = Math.random() * 2 - 1;
  const loop = (freq, q) => { const s = actx.createBufferSource(); s.buffer = noiseBuf; s.loop = true; const f = actx.createBiquadFilter(); f.type = 'bandpass'; f.frequency.value = freq; f.Q.value = q; const g = actx.createGain(); g.gain.value = 0; s.connect(f); f.connect(g); g.connect(sfxBus); s.start(); return g; };
  sizzleG = loop(5200, 0.8); fryG = loop(2600, 0.6);
}
const mtof = m => 440 * Math.pow(2, (m - 69) / 12);
function tone(f, dur, type = 'sine', pk = 0.15, delay = 0, bus){ if (!actx) return; const t = actx.currentTime + delay, o = actx.createOscillator(), g = actx.createGain(); o.type = type; o.frequency.setValueAtTime(f, t); g.gain.setValueAtTime(0.0001, t); g.gain.linearRampToValueAtTime(pk, t + 0.008); g.gain.exponentialRampToValueAtTime(0.0001, t + dur); o.connect(g); g.connect(bus || sfxBus); o.start(t); o.stop(t + dur + 0.05); return o; }
function sweep(f0, f1, dur, type = 'sine', pk = 0.12){ const o = tone(f0, dur, type, pk); if (o) o.frequency.exponentialRampToValueAtTime(f1, actx.currentTime + dur); }
function noise(dur, freq, pk = 0.2, type = 'bandpass', delay = 0){ if (!actx) return; const t = actx.currentTime + delay, s = actx.createBufferSource(); s.buffer = noiseBuf; const f = actx.createBiquadFilter(); f.type = type; f.frequency.value = freq; const g = actx.createGain(); g.gain.setValueAtTime(pk, t); g.gain.exponentialRampToValueAtTime(0.0001, t + dur); s.connect(f); f.connect(g); g.connect(sfxBus); s.start(t, Math.random()); s.stop(t + dur + 0.05); }
const sfx = {
  pick: () => tone(660, 0.07, 'triangle', 0.1), drop: () => noise(0.08, 900, 0.15), plop: () => sweep(500, 200, 0.12, 'sine', 0.15),
  stack: () => { tone(880, 0.06, 'triangle', 0.12); tone(1320, 0.08, 'triangle', 0.08, 0.04); },
  ding: () => { tone(1568, 1.2, 'sine', 0.25); tone(3136, 0.8, 'sine', 0.06); }, coin: () => { tone(988, 0.08, 'square', 0.08); tone(1319, 0.25, 'square', 0.08, 0.08); },
  happy: () => [523, 659, 784, 1047].forEach((f, i) => tone(f, 0.18, 'triangle', 0.12, i * 0.07)), angry: () => { tone(200, 0.3, 'sawtooth', 0.1); tone(150, 0.4, 'sawtooth', 0.1, 0.15); },
  wrong: () => { tone(330, 0.15, 'square', 0.08); tone(262, 0.25, 'square', 0.08, 0.12); }, squeak: () => sweep(1800, 900, 0.18, 'square', 0.08),
  pour: () => noise(1.6, 1200, 0.12, 'lowpass'), trash: () => { noise(0.25, 300, 0.3, 'lowpass'); tone(90, 0.2, 'sine', 0.2); }, poof: () => noise(0.5, 700, 0.3, 'lowpass'),
  fire: () => noise(0.8, 400, 0.25, 'lowpass'), spray: () => noise(0.2, 3000, 0.1, 'highpass'), meow: () => { const o = tone(700, 0.45, 'triangle', 0.12); if (o) { o.frequency.linearRampToValueAtTime(1100, actx.currentTime + 0.15); o.frequency.linearRampToValueAtTime(600, actx.currentTime + 0.45); } },
  purr: () => { for (let i = 0; i < 6; i++) noise(0.12, 120, 0.25, 'lowpass', i * 0.14); }, click: () => tone(1200, 0.04, 'square', 0.06)
};
// muusika: mõnus köögi-bossa (disco uuendusega rütmikam)
let musicStep = 0, musicNext = 0;
const PROG = [[60, 64, 67, 71], [57, 60, 64, 67], [62, 65, 69, 72], [55, 59, 62, 65]];
function music(){
  if (!actx || mode !== 'play') return;
  const bpm = up.disco ? 118 : 104, st = 60 / bpm / 2;
  if (musicNext < actx.currentTime) musicNext = actx.currentTime + 0.05;
  while (musicNext < actx.currentTime + 0.2) {
    const s = musicStep++, t = musicNext - actx.currentTime, bar = Math.floor(s / 8) % 4, pos = s % 8, ch = PROG[bar];
    if (pos === 0) ch.forEach(m => tone(mtof(m), st * 7, 'triangle', 0.035, t, musicBus));
    if (pos % 2 === 0) tone(mtof(ch[0] - 24 + (pos === 4 ? 7 : pos === 6 ? 12 : 0)), st * 1.6, 'sine', 0.18, t, musicBus);
    if (pos % 2 === 1) noise(0.05, 8000, 0.05, 'highpass', t);
    if (pos === 3 || pos === 6) tone(mtof(ch[(pos / 3 | 0) + 1] + 12), st * 0.9, 'sine', 0.05, t, musicBus);
    if (up.disco && pos % 2 === 0) { const o = tone(110, 0.25, 'sine', 0.35, t, musicBus); if (o) o.frequency.exponentialRampToValueAtTime(45, actx.currentTime + t + 0.12); }
    if (up.disco && (pos === 2 || pos === 6)) noise(0.12, 2000, 0.12, 'bandpass', t);
    musicNext += st;
  }
}

/* =========================================================
   OSAKESED (suits, tuli, sädemed, südamed, pihusti)
   ========================================================= */
const PN = 500, pPos = new Float32Array(PN * 3), pCol = new Float32Array(PN * 3), pSize = new Float32Array(PN), pAlpha = new Float32Array(PN);
const pVel = Array.from({ length: PN }, () => V3()), pLife = new Float32Array(PN), pMax = new Float32Array(PN), pGrav = new Float32Array(PN); let pI = 0;
const pGeo = new THREE.BufferGeometry();
pGeo.setAttribute('position', new THREE.BufferAttribute(pPos, 3)); pGeo.setAttribute('pcolor', new THREE.BufferAttribute(pCol, 3)); pGeo.setAttribute('psize', new THREE.BufferAttribute(pSize, 1)); pGeo.setAttribute('palpha', new THREE.BufferAttribute(pAlpha, 1));
const pMat = new THREE.ShaderMaterial({ uniforms: { uScale: { value: 600 } }, transparent: true, depthWrite: false,
  vertexShader: `attribute vec3 pcolor; attribute float psize; attribute float palpha; uniform float uScale; varying vec3 vC; varying float vA; void main(){ vC = pcolor; vA = palpha; vec4 mv = modelViewMatrix * vec4(position, 1.0); gl_PointSize = psize * uScale / max(0.05, -mv.z); gl_Position = projectionMatrix * mv; }`,
  fragmentShader: `varying vec3 vC; varying float vA; void main(){ float d = length(gl_PointCoord - 0.5); float a = smoothstep(0.5, 0.15, d); gl_FragColor = vec4(vC, a * vA); }` });
const pts = new THREE.Points(pGeo, pMat); pts.frustumCulled = false; pts.renderOrder = 5; scene.add(pts);
const tmpC = new THREE.Color();
function emit(p, v, col, size, life, grav = 0){ const i = pI = (pI + 1) % PN; pPos[i * 3] = p.x; pPos[i * 3 + 1] = p.y; pPos[i * 3 + 2] = p.z; pVel[i].copy(v); tmpC.set(col); pCol[i * 3] = tmpC.r; pCol[i * 3 + 1] = tmpC.g; pCol[i * 3 + 2] = tmpC.b; pSize[i] = size; pLife[i] = life; pMax[i] = life; pGrav[i] = grav; }
function burst(p, col, n = 20, sp = 1.2, size = 0.03){ for (let i = 0; i < n; i++) emit(p, V3(rand(-1, 1), rand(0, 1.5), rand(-1, 1)).multiplyScalar(sp), col, size * rand(0.6, 1.3), rand(0.4, 0.9), -3); }
function updateParticles(dt){ for (let i = 0; i < PN; i++) { if (pLife[i] <= 0) { pAlpha[i] = 0; continue; } pLife[i] -= dt; pVel[i].y += pGrav[i] * dt; pPos[i * 3] += pVel[i].x * dt; pPos[i * 3 + 1] += pVel[i].y * dt; pPos[i * 3 + 2] += pVel[i].z * dt; pAlpha[i] = Math.max(0, pLife[i] / pMax[i]); } for (const k of ['position', 'pcolor', 'psize', 'palpha']) pGeo.attributes[k].needsUpdate = true; }

/* =========================================================
   ESEMED
   ========================================================= */
const LAYER_H = { bunB: 0.03, patty: 0.022, cheese: 0.007, lettuce: 0.012, tomato: 0.012, bunT: 0.04 };
const NAMES = { bunB: 'Põhjakukkel', bunT: 'Kaanekukkel', patty: 'Kotlet', cheese: 'Juust', lettuce: 'Salat', tomato: 'Tomat' };
const items = [];
const SAUS = { raw: new THREE.Color(0xf48fb1), done: new THREE.Color(0xb5442a), burnt: new THREE.Color(0x2b1a12) };
const COLR = { raw: new THREE.Color(0xe57373), done: new THREE.Color(0x7a4a2e), burnt: new THREE.Color(0x1d1d1d), fryRaw: new THREE.Color(0xfff3c4), fryDone: new THREE.Color(0xf5b82e), fryBurnt: new THREE.Color(0x5a3a12) };
function makeItemMesh(type, extra = {}){
  const g = new THREE.Group(); let r = 0.06, hh = 0.015;
  if (type === 'bunB') { mesh(cyl(0.056, 0.058, 0.03), std(0xe0a65a), 0, 0, 0, g); mesh(cyl(0.054, 0.054, 0.002), std(0xf5deb3), 0, 0.016, 0, g); hh = 0.015; }
  else if (type === 'bunT') { const b = mesh(new THREE.SphereGeometry(0.058, 20, 10, 0, Math.PI * 2, 0, Math.PI / 2), std(0xd99549), 0, -0.012, 0, g); b.scale.y = 0.75; for (let i = 0; i < 9; i++) { const a = i * 2.4, rr = 0.015 + (i % 3) * 0.013; const s = mesh(new THREE.SphereGeometry(0.004, 6, 4), M.white, Math.cos(a) * rr, 0.024 + (0.03 - rr) * 0.3, Math.sin(a) * rr, g, false); s.scale.y = 0.5; } hh = 0.02; }
  else if (type === 'patty') { g.userData.m = mesh(cyl(0.052, 0.052, 0.022, 20), std(0xe57373, { roughness: 0.85 }), 0, 0, 0, g); hh = 0.011; }
  else if (type === 'cheese') { const c = mesh(box(0.1, 0.007, 0.1), std(0xffc93c), 0, 0, 0, g); c.rotation.y = Math.PI / 4; hh = 0.004; r = 0.065; }
  else if (type === 'lettuce') { const l = mesh(cyl(0.064, 0.06, 0.012, 12), std(0x6abe45), 0, 0, 0, g); l.scale.set(1, 1, 0.92); hh = 0.006; }
  else if (type === 'tomato') { for (const x of [-0.026, 0.026]) { mesh(cyl(0.03, 0.03, 0.012, 16), std(0xe63946), x, 0, 0, g); mesh(cyl(0.022, 0.022, 0.013, 12), std(0xff8a80), x, 0, 0, g, false); } hh = 0.006; }
  else if (type === 'plate') { mesh(cyl(0.11, 0.09, 0.016, 32), M.white, 0, 0, 0, g); mesh(cyl(0.085, 0.085, 0.002, 32), std(0xf0f6ff), 0, 0.008, 0, g, false); hh = 0.008; r = 0.1; }
  else if (type === 'friesRaw') { mesh(box(0.08, 0.1, 0.04), std(0x9fd3ff), 0, 0, 0, g); mesh(new THREE.PlaneGeometry(0.06, 0.06), new THREE.MeshBasicMaterial({ map: canvasTex(64, 64, (c) => { c.font = '48px sans-serif'; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillText('❄️', 32, 34); }), transparent: true }), 0, 0, 0.021, g, false); hh = 0.05; r = 0.05; }
  else if (type === 'fries') { mesh(cyl(0.045, 0.03, 0.08, 4), std(0xe63946), 0, 0, 0, g).rotation.y = Math.PI / 4; g.userData.sticks = []; for (let i = 0; i < 11; i++) { const s = mesh(box(0.009, 0.08, 0.009), std(0xfff3c4, { roughness: 0.7 }), rand(-0.022, 0.022), 0.04 + rand(0, 0.02), rand(-0.022, 0.022), g, false); s.rotation.set(rand(-0.25, 0.25), 0, rand(-0.25, 0.25)); g.userData.sticks.push(s); } hh = 0.04; r = 0.05; }
  else if (type === 'cup') { mesh(cyl(0.037, 0.03, 0.11, 20, ), std(0xffffff), 0, 0, 0, g); g.userData.band = mesh(cyl(0.0375, 0.035, 0.03, 20), std(0xff8fab), 0, 0.01, 0, g, false); g.userData.lid = mesh(cyl(0.039, 0.039, 0.008, 20), std(0xffffff, { transparent: true, opacity: 0.9 }), 0, 0.058, 0, g); g.userData.lid.visible = false; g.userData.straw = mesh(cyl(0.004, 0.004, 0.09), std(0xe63946), 0.01, 0.09, 0, g, false); g.userData.straw.visible = false; hh = 0.055; r = 0.045; }
  else if (type === 'ext') { mesh(cyl(0.04, 0.04, 0.26, 18), M.red, 0, 0, 0, g); mesh(cyl(0.012, 0.012, 0.05), M.black, 0, 0.15, 0, g); const n = mesh(cyl(0.008, 0.012, 0.09), M.black, 0, 0.17, -0.04, g); n.rotation.x = Math.PI / 2; g.userData.nozzle = new THREE.Object3D(); g.userData.nozzle.position.set(0, 0.2, 0); g.add(g.userData.nozzle); hh = 0.13; r = 0.05; }
  else if (type === 'hdBun') { const b = mesh(new THREE.CapsuleGeometry(0.032, 0.12, 6, 12).rotateZ(Math.PI / 2), std(0xe0a65a), 0, 0, 0, g); b.scale.set(1, 0.75, 1); mesh(box(0.13, 0.012, 0.02), std(0xf5deb3), 0, 0.022, 0, g, false); hh = 0.025; r = 0.09; }
  else if (type === 'sausage') { g.userData.m = mesh(new THREE.CapsuleGeometry(0.016, 0.13, 6, 10).rotateZ(Math.PI / 2), std(0xf48fb1, { roughness: 0.5 }), 0, 0, 0, g); hh = 0.016; r = 0.08; }
  else if (type === 'cone') { mesh(new THREE.ConeGeometry(0.035, 0.11, 16).rotateX(Math.PI), std(0xd9a55b), 0, 0, 0, g); g.userData.scoop = mesh(new THREE.SphereGeometry(0.038, 16, 12), std(0xffffff), 0, 0.065, 0, g); g.userData.scoop.visible = false; hh = 0.055; r = 0.04; }
  else if (type === 'donut') { const col = extra.flavor ? extra.flavor.c : 0xff8fab; mesh(new THREE.TorusGeometry(0.035, 0.018, 12, 24).rotateX(Math.PI / 2), std(0xd99549), 0, 0, 0, g); const ic = mesh(new THREE.TorusGeometry(0.035, 0.016, 10, 24).rotateX(Math.PI / 2), std(col), 0, 0.006, 0, g); ic.scale.y = 0.6; for (let i = 0; i < 8; i++) { const a = i * 0.8; mesh(box(0.008, 0.003, 0.003), std(pick([0xffffff, 0x4fd1ff, 0xffe14f])), Math.cos(a) * 0.035, 0.016, Math.sin(a) * 0.035, g, false).rotation.y = a; } hh = 0.018; r = 0.055; }
  else if (type === 'duck') { mesh(new THREE.SphereGeometry(0.045, 16, 12), std(0xffe14f), 0, 0, 0, g).scale.set(1.2, 0.85, 1); mesh(new THREE.SphereGeometry(0.03, 14, 10), std(0xffe14f), 0.035, 0.04, 0, g); mesh(new THREE.ConeGeometry(0.012, 0.03, 8), std(0xff8c1a), 0.07, 0.04, 0, g).rotation.z = -Math.PI / 2; for (const z of [-0.012, 0.012]) mesh(new THREE.SphereGeometry(0.005, 6, 4), M.black, 0.055, 0.05, z, g, false); hh = 0.04; }
  g.traverse(o => { if (o.isMesh) { o.castShadow = true; } });
  return { g, r, hh };
}
function newItem(type, pos, extra = {}){
  const { g, r, hh } = makeItemMesh(type, extra); scene.add(g); g.position.copy(pos);
  const it = Object.assign({ type, g, r, hh, vel: V3(), held: null, rest: false, slot: null, cook: 0, state: type === 'patty' || type === 'friesRaw' || type === 'sausage' ? 'raw' : 'ok', stack: [], age: 0, flavor: null, onPlate: null }, extra);
  g.userData.item = it; items.push(it); return it;
}
function removeItem(it){ if (it.slot) freeSlot(it); it.g.removeFromParent(); const i = items.indexOf(it); if (i >= 0) items.splice(i, 1); for (const s of it.stack) removeItem(s); }
// küpsetamise välimus
function setCookLook(it){
  if (it.type === 'sausage') { const m = it.g.userData.m.material; if (it.state === 'raw') m.color.copy(SAUS.raw).lerp(SAUS.done, clamp(it.cook / cookT(), 0, 1)); else if (it.state === 'done') m.color.copy(SAUS.done).lerp(SAUS.burnt, clamp((it.cook - cookT()) / (burnT() - cookT()), 0, 1) * 0.5); else m.color.copy(SAUS.burnt); }
  if (it.type === 'patty') { const m = it.g.userData.m.material; const k = it.cook; if (it.state === 'raw') m.color.copy(COLR.raw).lerp(COLR.done, clamp(k / cookT(), 0, 1)); else if (it.state === 'done') m.color.copy(COLR.done).lerp(COLR.burnt, clamp((k - cookT()) / (burnT() - cookT()), 0, 1) * 0.5); else m.color.copy(COLR.burnt); }
  if (it.type === 'fries') { const c = it.state === 'burnt' ? COLR.fryBurnt : COLR.fryDone; it.g.userData.sticks.forEach(s => s.material.color.copy(c)); }
}
// jaoturid (lõputud kastid)
const DISP = [];
function dispenser(type, x, z, label, col, rotY = 0, lock = null, extra = {}){
  const g = new THREE.Group(); g.position.set(x, CT, z); g.rotation.y = rotY; K.add(g);
  mesh(box(0.2, 0.08, 0.2), std(col), 0, 0.04, 0, g); mesh(box(0.18, 0.005, 0.18), std(0x333333), 0, 0.075, 0, g, false);
  const show = makeItemMesh(type, extra).g; show.position.set(0, 0.095, 0); show.scale.setScalar(0.9); g.add(show);
  if (type !== 'plate' && type !== 'cup' && type !== 'cone') { const s2 = makeItemMesh(type, extra).g; s2.position.set(0.03, 0.1 + 0.015, -0.02); s2.rotation.y = 1; s2.scale.setScalar(0.85); g.add(s2); }
  const lab = mesh(new THREE.PlaneGeometry(0.2, 0.05), new THREE.MeshBasicMaterial({ map: canvasTex(320, 80, (c, W, H) => { roundRect(c, 2, 2, W - 4, H - 4, 18); c.fillStyle = '#fff'; c.fill(); c.font = `bold ${H * 0.5}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillStyle = '#333'; c.fillText(label, W / 2, H / 2 + 2); }), transparent: true }), 0, 0.04, 0.101, g, false);
  const d = { type, g, lock, extra, pos: () => g.getWorldPosition(V3()).add(V3(0, 0.1, 0)) }; DISP.push(d); return d;
}
dispenser('bunB', -1.2, 0.3, 'Põhjakukkel', 0xe8c39e); dispenser('bunT', -1.2, 0.05, 'Kaanekukkel', 0xe8c39e);
dispenser('patty', -1.2, -0.21, '🥩 Kotlet', 0xffb3b3); dispenser('cheese', -1.2, -0.45, '🧀 Juust', 0xffe29a);
dispenser('lettuce', -1.2, -0.68, '🥬 Salat', 0xbfe8a8); dispenser('tomato', -1.2, -0.88, '🍅 Tomat', 0xffb3b3);
dispenser('plate', 0.0, -0.84, '🍽️ Taldrik', 0xd7e9f7);
dispenser('friesRaw', 1.2, 0.3, '❄️ Friikad', 0xbfe3ff); dispenser('cup', 1.2, 0.05, '🥤 Topsid', 0xffd6e2);
// tagumine lett: avatavad toidud
const DONUTS = [{ n: 'Roosa', c: 0xff8fab, e: '🩷' }, { n: 'Šokolaad', c: 0x5d3a1a, e: '🟤' }];
const ICES = [{ n: 'Vanill', c: 0xfff3c4, e: '🤍' }, { n: 'Maasikas', c: 0xff8fab, e: '🩷' }, { n: 'Šokolaad', c: 0x6d4c41, e: '🟤' }];
dispenser('hdBun', -0.85, 1.2, '🌭 Kukkel', 0xe8c39e, Math.PI, 'hotdog'); dispenser('sausage', -0.6, 1.2, '🌭 Vorst', 0xffb3b3, Math.PI, 'hotdog');
dispenser('donut', -0.3, 1.2, '🍩 Roosa', 0xffd6e2, Math.PI, 'donut', { flavor: DONUTS[0] }); dispenser('donut', -0.06, 1.2, '🍩 Šokolaad', 0xe0c7b0, Math.PI, 'donut', { flavor: DONUTS[1] });
dispenser('cone', 0.18, 1.2, '🍦 Vahvlid', 0xf3dcb0, Math.PI, 'ice');
const iceM = new THREE.Group(); iceM.position.set(0.62, CT, 1.25); iceM.rotation.y = Math.PI; K.add(iceM);
mesh(box(0.62, 0.5, 0.2), std(0x9be7ff), 0, 0.42, 0.06, iceM); mesh(box(0.62, 0.06, 0.24), std(0xffffff), 0, 0.68, 0.04, iceM);
mesh(new THREE.PlaneGeometry(0.4, 0.1), new THREE.MeshBasicMaterial({ map: canvasTex(400, 100, (c, W, H) => { c.font = `bold ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillStyle = '#e63946'; c.fillText('🍦 JÄÄTIS', W / 2, H / 2); }), transparent: true }), 0, 0.6, -0.042, iceM, false);
mesh(box(0.6, 0.01, 0.16), M.darkSteel, 0, 0.005, -0.12, iceM);
const iceTaps = ICES.map((f, i) => { const x = -0.2 + i * 0.2; mesh(cyl(0.02, 0.012, 0.06), M.steel, x, 0.16, -0.08, iceM); const b = mesh(box(0.06, 0.05, 0.03), std(f.c, { emissive: f.c, emissiveIntensity: 0.25 }), x, 0.3, -0.055, iceM); return { ice: true, x, btn: b, f, i }; });
const lockSign = mesh(new THREE.PlaneGeometry(0.9, 0.18), new THREE.MeshBasicMaterial({ map: canvasTex(900, 180, (c, W, H) => { roundRect(c, 4, 4, W - 8, H - 8, 40); c.fillStyle = '#fffaf3'; c.fill(); c.lineWidth = 8; c.strokeStyle = '#ff8fab'; c.stroke(); c.font = `bold ${H * 0.36}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.fillStyle = '#3d2c3e'; c.fillText('🔒 Uued toidud – ava poes! 🌭🍦🍩', W / 2, H / 2); }), transparent: true }), 0, CT + 0.35, 1.25, K, false); lockSign.rotation.y = Math.PI;
// tööriistad: tulekustuti, kummipart
let extItem = null, duckItem = null;

/* ---------- pesad: grill, fritüür, kraanid ---------- */
const SLOTS = [];
function addSlot(kind, x, y, z, accepts){ const s = { kind, pos: V3(x, y, z), item: null, accepts, fire: 0 }; SLOTS.push(s); return s; }
for (const sx of [-0.66, -0.4]) for (const sz of [-0.55, -0.77]) addSlot('grill', sx, CT + 0.058, sz, it => it.type === 'patty' || it.type === 'sausage');
for (const sx of [0.48, 0.72]) addSlot('fryer', sx, CT + 0.12, -0.66, it => it.type === 'friesRaw');
iceTaps.forEach(t => { t.slot = addSlot('icetap', 0.62 - t.x, CT + 0.06, 1.25 + 0.12, it => it.type === 'cone' && !it.flavor); t.slot.tap = t; });
taps.forEach(t => { t.slot = addSlot('tap', 1.36 - 0.13, CT + 0.012, -0.45 + t.z, it => it.type === 'cup' && !it.flavor); t.slot.tap = t; });
function slotWorld(s){ return K.localToWorld(s.pos.clone()); }
function putInSlot(it, s){ s.item = it; it.slot = s; it.rest = true; it.vel.set(0, 0, 0); it.g.position.copy(slotWorld(s)).add(V3(0, it.hh, 0)); it.g.rotation.set(0, 0, 0); if (s.kind === 'fryer') { it.g.position.y -= 0.03; } }
function freeSlot(it){ if (it.slot) { it.slot.item = null; it.slot = null; } }
const up = Object.assign({ grill: false, fryer: false, calm: false, disco: false, decor: false, cat: false, hotdog: false, ice: false, donut: false }, (store('kk_save') || {}).up || {});
const cookT = () => (diff().id === 0 ? 7 : 6) * (up.grill ? 0.6 : 1), burnT = () => diff().burn ? cookT() + (diff().id === 1 ? 14 : 9) : Infinity, fireT = () => burnT() + 8;
const fryT = () => 5 * (up.fryer ? 0.6 : 1);

/* =========================================================
   KLIENDID
   ========================================================= */
const HATS = ['none', 'cap', 'beanie', 'top', 'party', 'chef'];
const SKINS = [0xffdbb5, 0xf1c27d, 0xc68642, 0x8d5524, 0xffe0bd];
const custs = [];
function makeCustomer(kind){
  const g = new THREE.Group(); K.add(g);
  const bodyCol = kind === 'robot' ? 0xb0bec5 : pick([0xff6b6b, 0x4ecdc4, 0x5f6caf, 0xffb347, 0x9b5de5, 0x00bbf9, 0xf15bb5]);
  const skin = kind === 'robot' ? 0xcfd8dc : kind === 'cat' ? 0xffa64d : pick(SKINS);
  const body = mesh(new THREE.CapsuleGeometry(0.2, 0.45, 6, 16), std(bodyCol), 0, 0.75, 0, g);
  const headG = new THREE.Group(); headG.position.set(0, 1.42, 0); g.add(headG);
  const hm = kind === 'robot' ? mesh(box(0.3, 0.26, 0.26), std(skin, { metalness: 0.6, roughness: 0.3 }), 0, 0, 0, headG) : mesh(new THREE.SphereGeometry(0.16, 24, 16), std(skin), 0, 0, 0, headG);
  for (const x of [-0.055, 0.055]) { mesh(new THREE.SphereGeometry(0.035, 12, 10), std(kind === 'robot' ? 0x00e5ff : 0xffffff, kind === 'robot' ? { emissive: 0x00e5ff, emissiveIntensity: 1 } : {}), x, 0.03, 0.14, headG, false); if (kind !== 'robot') mesh(new THREE.SphereGeometry(0.018, 10, 8), M.black, x, 0.03, 0.17, headG, false); }
  const mouth = mesh(new THREE.TorusGeometry(0.045, 0.01, 6, 16, Math.PI), M.black, 0, -0.05, 0.145, headG, false); mouth.rotation.z = Math.PI;
  const blush = []; if (kind !== 'robot') for (const x of [-0.1, 0.1]) blush.push(mesh(new THREE.CircleGeometry(0.025, 12), new THREE.MeshBasicMaterial({ color: 0xff8fab, transparent: true, opacity: 0.6 }), x, -0.02, 0.15, headG, false));
  let hat = kind === 'vip' ? 'crown' : kind === 'robot' ? 'antenna' : kind === 'cat' ? 'ears' : pick(HATS);
  if (hat === 'cap') { mesh(new THREE.SphereGeometry(0.165, 20, 10, 0, Math.PI * 2, 0, Math.PI / 2), std(pick([0xe63946, 0x1d3557, 0x2a9d8f])), 0, 0.03, 0, headG); mesh(box(0.16, 0.012, 0.12), M.black, 0, 0.04, 0.15, headG); }
  if (hat === 'beanie') { mesh(new THREE.SphereGeometry(0.168, 20, 10, 0, Math.PI * 2, 0, Math.PI / 2), std(0x9b5de5), 0, 0.02, 0, headG); mesh(new THREE.SphereGeometry(0.04, 10, 8), M.white, 0, 0.19, 0, headG); }
  if (hat === 'top') { mesh(cyl(0.2, 0.2, 0.015), M.black, 0, 0.12, 0, headG); mesh(cyl(0.11, 0.11, 0.18), M.black, 0, 0.21, 0, headG); mesh(cyl(0.112, 0.112, 0.03), M.red, 0, 0.14, 0, headG); }
  if (hat === 'party') { mesh(new THREE.ConeGeometry(0.08, 0.2, 16), std(0xf15bb5), 0, 0.22, 0, headG); mesh(new THREE.SphereGeometry(0.025, 8, 6), M.yellow, 0, 0.33, 0, headG); }
  if (hat === 'chef') { mesh(cyl(0.13, 0.13, 0.12), M.white, 0, 0.17, 0, headG); mesh(new THREE.SphereGeometry(0.15, 16, 10), M.white, 0, 0.26, 0, headG).scale.y = 0.6; }
  if (hat === 'crown') { mesh(cyl(0.11, 0.11, 0.07, 6, ), std(0xffd700, { metalness: 0.9, roughness: 0.2 }), 0, 0.17, 0, headG); for (let i = 0; i < 6; i++) { const a = i / 6 * Math.PI * 2; mesh(new THREE.ConeGeometry(0.02, 0.06, 6), std(0xffd700, { metalness: 0.9, roughness: 0.2 }), Math.cos(a) * 0.1, 0.23, Math.sin(a) * 0.1, headG); } mesh(new THREE.SphereGeometry(0.018, 8, 6), std(0xe63946), 0, 0.17, 0.11, headG); }
  if (hat === 'antenna') { mesh(cyl(0.006, 0.006, 0.12), M.darkSteel, 0, 0.19, 0, headG); mesh(new THREE.SphereGeometry(0.025, 10, 8), std(0xff1744, { emissive: 0xff1744, emissiveIntensity: 1 }), 0, 0.26, 0, headG); }
  if (hat === 'ears') { for (const x of [-0.09, 0.09]) mesh(new THREE.ConeGeometry(0.05, 0.1, 4), std(skin), x, 0.15, 0, headG); for (const x of [-0.08, 0.08]) mesh(box(0.08, 0.004, 0.004), M.black, x * 1.6, -0.03, 0.15, headG, false); }
  for (const x of [-0.24, 0.24]) { const arm = mesh(new THREE.CapsuleGeometry(0.05, 0.3, 4, 8), std(bodyCol), x, 0.85, 0.03, g); arm.rotation.z = x > 0 ? 0.25 : -0.25; }
  // tellimuse mull ja kannatusriba
  const bubble = new Panel(0.62, 0.42, 420); bubble.mesh.position.set(0, 1.95, 0.05); g.add(bubble.mesh);
  return { g, headG, mouth, bubble, kind, hat, mood: 1 };
}

/* =========================================================
   UI PANEELID
   ========================================================= */
class Panel {
  constructor(w, h, ppm = 500){ this.w = w; this.h = h; this.cv = document.createElement('canvas'); this.cv.width = Math.round(w * ppm); this.cv.height = Math.round(h * ppm); this.ctx = this.cv.getContext('2d'); this.tex = new THREE.CanvasTexture(this.cv); this.tex.colorSpace = THREE.SRGBColorSpace; this.tex.anisotropy = 4; this.mesh = new THREE.Mesh(new THREE.PlaneGeometry(w, h), new THREE.MeshBasicMaterial({ map: this.tex, transparent: true, depthWrite: false, depthTest: false, toneMapped: false })); this.mesh.renderOrder = 30; }
  draw(fn){ const c = this.ctx; c.clearRect(0, 0, this.cv.width, this.cv.height); fn(c, this.cv.width, this.cv.height); this.tex.needsUpdate = true; }
}
function cardBg(c, W, H, col = '#ff8fab'){ roundRect(c, 8, 8, W - 16, H - 16, Math.min(W, H) * 0.08); c.fillStyle = '#fffaf3'; c.fill(); c.lineWidth = 10; c.strokeStyle = col; c.stroke(); }
function txt(c, s, x, y, size, col = '#3d2c3e', weight = 'bold', align = 'center'){ c.font = `${weight} ${size}px ${FONT}`; c.textAlign = align; c.textBaseline = 'middle'; c.fillStyle = col; c.fillText(s, x, y); }
const buttons = [];
function makeButton(parent, w, h, x, y, label, onClick, col = '#ff8fab'){
  const p = new Panel(w, h, 600); p.mesh.position.set(x, y, 0.01); p.mesh.renderOrder = 31; parent.add(p.mesh);
  const b = { p, w, h, label, col, onClick, sel: false, hover: false, sub: '' };
  b.draw = () => p.draw((c, W, H) => {
    const m = Math.min(W, H) * 0.1; roundRect(c, m, m + (b.hover ? 0 : 4), W - m * 2, H - m * 2, Math.min(W, H) * 0.3);
    c.fillStyle = 'rgba(0,0,0,0.18)'; c.fill(); roundRect(c, m, m - (b.hover ? 4 : 0), W - m * 2, H - m * 2 - 4, Math.min(W, H) * 0.3); c.fillStyle = b.sel ? b.col : b.hover ? '#ffffff' : '#fff3e6'; c.fill(); c.lineWidth = b.sel ? 10 : 6; c.strokeStyle = b.col; c.stroke();
    let fs = H * (b.sub ? 0.3 : 0.4); c.font = `bold ${fs}px ${FONT}`; while (c.measureText(b.label).width > W * 0.84 && fs > 10) { fs--; c.font = `bold ${fs}px ${FONT}`; }
    txt(c, b.label, W / 2, b.sub ? H * 0.38 : H * 0.5, fs, b.sel ? '#fff' : '#3d2c3e');
    if (b.sub) { let f2 = H * 0.2; c.font = `${f2}px ${FONT}`; while (c.measureText(b.sub).width > W * 0.84 && f2 > 8) { f2--; c.font = `${f2}px ${FONT}`; } txt(c, b.sub, W / 2, H * 0.7, f2, b.sel ? '#fff' : '#6b5a6e', 'normal'); }
  });
  b.draw(); p.mesh.userData.btn = b; buttons.push(b); return b;
}
function isShown(o){ while (o) { if (!o.visible) return false; o = o.parent; } return true; }
function clickBtn(b, h){ if (!isShown(b.p.mesh)) return; sfx.click(); if (h && h.kind === 'xr') pulse(h, 0.4, 30); b.onClick(); }
function placeInFront(g, dist, yOff = 0){ if (!renderer.xr.isPresenting) dist *= 1.45; camera.getWorldPosition(head); camera.getWorldDirection(tmpD); tmpD.y = 0; if (tmpD.lengthSq() < 1e-4) tmpD.set(0, 0, -1); tmpD.normalize(); g.position.copy(head).addScaledVector(tmpD, dist); g.position.y = head.y + yOff; g.rotation.set(0, Math.atan2(-tmpD.x, -tmpD.z), 0); }

/* ---------- seaded ja salvestus ---------- */
const DIFFS = [
  { id: 0, name: '🧒 Laps', patience: 170, maxCust: 1, extras: 1, fries: 0.3, drink: 0.3, burn: false, gap: 14, dayLen: 150, goal: 0.6, color: '#7ed957' },
  { id: 1, name: '🙂 Kerge', patience: 110, maxCust: 2, extras: 2, fries: 0.45, drink: 0.45, burn: true, gap: 11, dayLen: 170, goal: 1, color: '#4ecdc4' },
  { id: 2, name: '😎 Tavaline', patience: 80, maxCust: 3, extras: 3, fries: 0.6, drink: 0.6, burn: true, gap: 9, dayLen: 180, goal: 1.3, color: '#ffb347' },
  { id: 3, name: '🔥 Raske', patience: 58, maxCust: 3, extras: 4, fries: 0.75, drink: 0.75, burn: true, gap: 6.5, dayLen: 190, goal: 1.7, color: '#e63946' }
];
const save = Object.assign({ coins: 0, diff: 1, day: 1, best: {}, up: {} }, store('kk_save') || {});
const persist = () => { save.up = up; store('kk_save', save); };
const diff = () => DIFFS[save.diff];

/* ---------- menüü ---------- */
const menuG = new THREE.Group(); scene.add(menuG);
const menuP = new Panel(1.4, 1.0); menuG.add(menuP.mesh);
function drawMenu(){
  menuP.draw((c, W, H) => {
    cardBg(c, W, H, '#ff8fab');
    txt(c, '🍔 KIIRE KOKK', W / 2, H * 0.12, H * 0.1, '#e63946', '900');
    txt(c, 'Küpseta • lao burgereid • prae friikaid • vala limpsi • tee kliendid õnnelikuks!', W / 2, H * 0.22, H * 0.032, '#6b5a6e', 'normal');
    txt(c, `🪙 ${save.coins} münti   •   📅 Päev ${save.day}${save.best[save.diff] ? `   •   🏆 rekord ${save.best[save.diff]} 🪙/päev` : ''}`, W / 2, H * 0.31, H * 0.036, '#3d2c3e');
    txt(c, 'RASKUS', W / 2, H * 0.375, H * 0.03, '#ff8fab');
    const tips = ['🥩 Kotlet grillile → oota pruuniks → pane kuklile', '🍔 Põhjakukkel + kotlet + … + kaanekukkel', '🍽️ Pane taldrik/friikad/jook kliendi ette kollasele matile', '🔥 Kõrbenud? Viska prügikasti! Tuli? Võta kustuti!'];
    tips.forEach((t, i) => txt(c, t, W / 2, H * (0.76 + i * 0.05), H * 0.028, '#6b5a6e', 'normal'));
  });
}
const diffBtns = DIFFS.map((d, i) => makeButton(menuG, 0.31, 0.11, -0.48 + i * 0.32, 0.05, d.name, () => { save.diff = i; persist(); refreshMenu(); }, d.color));
const startBtn = makeButton(menuG, 0.46, 0.13, -0.26, -0.13, '▶ ALUSTA PÄEVA', () => startDay(), '#e63946');
const resetBtn = makeButton(menuG, 0.4, 0.13, 0.27, -0.13, '🔄 Uus restoran', () => { if (resetBtn.label.includes('?')) { save.coins = 0; save.day = 1; for (const k in up) up[k] = false; persist(); applyUpgrades(); resetBtn.label = '🔄 Uus restoran'; refreshMenu(); } else { resetBtn.label = '❓ Kindel? Vajuta veel'; resetBtn.draw(); } }, '#9b5de5');
function refreshMenu(){ diffBtns.forEach((b, i) => { b.sel = save.diff === i; b.draw(); }); resetBtn.draw(); drawMenu(); }
/* ---------- HUD ---------- */
const hud = new Panel(1.4, 0.24, 420); hud.mesh.position.set(0, 2.5, -1.5); hud.mesh.rotation.x = 0.15; K.add(hud.mesh); hud.mesh.visible = false;
const toast = new Panel(1.4, 0.2, 400); toast.mesh.renderOrder = 40; scene.add(toast.mesh); toast.mesh.visible = false; let toastT = 0;
function showToast(s, col = '#3d2c3e', dur = 2){ toast.draw((c, W, H) => { roundRect(c, 6, 6, W - 12, H - 12, H * 0.4); c.fillStyle = 'rgba(255,250,243,0.95)'; c.fill(); c.lineWidth = 6; c.strokeStyle = col; c.stroke(); let fs = H * 0.42; c.font = `bold ${fs}px ${FONT}`; while (c.measureText(s).width > W * 0.92 && fs > 12) { fs--; c.font = `bold ${fs}px ${FONT}`; } txt(c, s, W / 2, H / 2, fs, col); }); toast.mesh.visible = true; toastT = dur; placeInFront(toast.mesh, 1.2, 0.32); }
// hüpikud (+5 🪙)
const pops = Array.from({ length: 8 }, () => { const p = new Panel(0.4, 0.13, 400); p.mesh.renderOrder = 35; p.mesh.visible = false; scene.add(p.mesh); return { p, t: 0 }; }); let popI = 0;
function pop(pos, s, col = '#e63946'){ const o = pops[popI = (popI + 1) % pops.length]; o.p.draw((c, W, H) => { c.font = `900 ${H * 0.6}px ${FONT}`; c.textAlign = 'center'; c.textBaseline = 'middle'; c.lineWidth = 10; c.strokeStyle = '#fff'; c.strokeText(s, W / 2, H / 2); c.fillStyle = col; c.fillText(s, W / 2, H / 2); }); o.p.mesh.position.copy(pos); o.p.mesh.visible = true; o.t = 1.2; camera.getWorldPosition(head); o.p.mesh.lookAt(head); }
/* ---------- päeva lõpp + pood ---------- */
const endG = new THREE.Group(); endG.visible = false; scene.add(endG);
const endP = new Panel(1.4, 1.05); endG.add(endP.mesh);
const SHOP = [
  { k: 'hotdog', n: '🌭 Hotdog', d: 'vorst grillile → kuklisse', p: 20 },
  { k: 'ice', n: '🍦 Jäätis', d: 'jäätisemasin, 3 maitset', p: 25 },
  { k: 'donut', n: '🍩 Sõõrikud', d: 'võta lauast ja serveeri', p: 15 },
  { k: 'grill', n: '🔥 Turbogrill', d: 'kotletid valmivad 40% kiiremini', p: 30 },
  { k: 'fryer', n: '🍟 Kiirfritüür', d: 'friikad valmivad 40% kiiremini', p: 30 },
  { k: 'calm', n: '😊 Rahulik muusika', d: 'kliendid on 35% kannatlikumad', p: 40 },
  { k: 'disco', n: '🪩 Discokuul', d: 'tuled + funky muusika + rohkem jootraha', p: 35 },
  { k: 'cat', n: '🐱 Köögikass', d: 'silita teda – ta nurrub 😻', p: 25 },
  { k: 'decor', n: '🌴 Kaunistused', d: 'taimed, lilled, tuled', p: 15 }
];
const shopBtns = SHOP.map((s, i) => makeButton(endG, 0.42, 0.12, -0.44 + (i % 3) * 0.44, 0.03 - Math.floor(i / 3) * 0.135, s.n, () => buy(s), '#4ecdc4'));
const nextBtn = makeButton(endG, 0.46, 0.11, -0.24, -0.44, '▶ Järgmine päev', () => startDay(), '#e63946');
makeButton(endG, 0.38, 0.11, 0.25, -0.44, '🏠 Menüü', () => goMenu(), '#9b5de5');
let lastSummary = null;
function buy(s){ if (up[s.k]) { showToast('See on juba olemas ✔', '#4ecdc4', 1.5); return; } if (save.coins < s.p) { sfx.wrong(); showToast(`Vaja ${s.p} 🪙 – sul on ${save.coins}`, '#e63946', 1.6); return; } save.coins -= s.p; up[s.k] = true; persist(); applyUpgrades(); sfx.coin(); sfx.happy(); showToast(`Ostetud: ${s.n}! 🎉`, '#4ecdc4', 2); drawEnd(); }
function drawEnd(){
  const r = lastSummary; if (!r) return;
  endP.draw((c, W, H) => {
    cardBg(c, W, H, '#ffd166');
    txt(c, r.ok ? `🎉 PÄEV ${r.day} LÄBI!` : `📅 PÄEV ${r.day} LÄBI`, W / 2, H * 0.09, H * 0.075, '#e63946', '900');
    txt(c, '⭐'.repeat(r.stars) + '☆'.repeat(3 - r.stars), W / 2, H * 0.19, H * 0.07, '#ffb347');
    txt(c, `🪙 Teenisid ${r.earned} (eesmärk ${r.goal})   •   😀 ${r.served} rahul   •   😠 ${r.angry} pahane`, W / 2, H * 0.28, H * 0.034);
    txt(c, `Rahakotis 🪙 ${save.coins}   •   🛒 Pood – uued toidud ja uuendused:`, W / 2, H * 0.355, H * 0.032, '#6b5a6e', 'normal');
  });
  shopBtns.forEach((b, i) => { const s = SHOP[i]; b.sel = !!up[s.k]; b.sub = up[s.k] ? '✔ olemas' : `🪙 ${s.p} • ${s.d}`; b.col = up[s.k] ? '#4ecdc4' : save.coins >= s.p ? '#ff8fab' : '#bbbbbb'; b.draw(); });
}
/* ---------- paus ---------- */
const pauseG = new THREE.Group(); pauseG.visible = false; scene.add(pauseG);
const pauseP = new Panel(0.8, 0.55); pauseG.add(pauseP.mesh); pauseP.draw((c, W, H) => { cardBg(c, W, H, '#4ecdc4'); txt(c, '⏸ PAUS', W / 2, H * 0.15, H * 0.12, '#3d2c3e', '900'); });
makeButton(pauseG, 0.55, 0.12, 0, 0.04, '▶ Jätka', () => resume(), '#4ecdc4');
makeButton(pauseG, 0.55, 0.12, 0, -0.12, '🏠 Menüü (päev katkeb)', () => goMenu(), '#e63946');
const wristG = new THREE.Group(); wristG.visible = false; scene.add(wristG);
const wristBtn = makeButton(wristG, 0.07, 0.07, 0, 0, '⏸', () => togglePause(), '#4ecdc4');

/* =========================================================
   KASS JA KAUNISTUSED
   ========================================================= */
const decorG = new THREE.Group(); K.add(decorG);
for (const [x, z] of [[-1.85, 0.9], [1.85, 0.9], [-1.85, -1.6], [1.85, -1.6]]) { mesh(cyl(0.14, 0.11, 0.28), std(0xc1785a), x, 0.14, z, decorG); for (let i = 0; i < 7; i++) { const l = mesh(new THREE.SphereGeometry(0.12, 10, 8), std(0x3fa34d), x + rand(-0.1, 0.1), 0.4 + i * 0.07, z + rand(-0.1, 0.1), decorG); l.scale.set(1, 1.4, 0.5); l.rotation.y = rand(0, 3); } }
for (let i = 0; i < 14; i++) { const b = mesh(new THREE.SphereGeometry(0.03, 8, 6), new THREE.MeshBasicMaterial({ color: [0xff4fa3, 0xffe14f, 0x4fd1ff][i % 3] }), -1.8 + i * 0.28, 2.75 + Math.sin(i) * 0.05, -1.4, decorG, false); }
const disco = new THREE.Group(); disco.position.set(0, 2.7, -2.4); scene.add(disco);
{ const ball = mesh(new THREE.IcosahedronGeometry(0.22, 2), std(0xdddddd, { metalness: 1, roughness: 0.15, flatShading: true }), 0, 0, 0, disco, false); mesh(cyl(0.005, 0.005, 0.4), M.black, 0, 0.3, 0, disco, false); }
const cat = new THREE.Group(); K.add(cat);
{ const fur = std(0xff9f43), dark = std(0x5a3a1a);
  const b = mesh(new THREE.CapsuleGeometry(0.06, 0.16, 6, 12), fur, 0, 0.08, 0, cat); b.rotation.z = Math.PI / 2;
  const hd = new THREE.Group(); hd.position.set(0.15, 0.14, 0); cat.add(hd); mesh(new THREE.SphereGeometry(0.065, 16, 12), fur, 0, 0, 0, hd);
  for (const z of [-0.035, 0.035]) { mesh(new THREE.ConeGeometry(0.025, 0.05, 4), fur, 0, 0.065, z, hd); mesh(new THREE.SphereGeometry(0.01, 8, 6), M.black, 0.055, 0.015, z * 0.8, hd, false); }
  mesh(new THREE.SphereGeometry(0.008, 6, 4), std(0xff8fab), 0.065, -0.005, 0, hd, false);
  const tail = mesh(cyl(0.012, 0.008, 0.16), fur, -0.15, 0.15, 0, cat); tail.rotation.z = 0.6; cat.userData = { tail, hd, t: 0, x: 0, dir: 1, petT: 0 };
  for (const [x, z] of [[0.07, 0.035], [0.07, -0.035], [-0.07, 0.035], [-0.07, -0.035]]) mesh(cyl(0.015, 0.015, 0.06), fur, x, 0.03, z, cat); }
function applyUpgrades(){ DISP.forEach(d => { if (d.lock) d.g.visible = !!up[d.lock]; }); iceM.visible = !!up.ice; lockSign.visible = !(up.hotdog || up.ice || up.donut); decorG.visible = !!up.decor; disco.visible = !!up.disco; cat.visible = !!up.cat; discoLights.forEach(l => l.intensity = 0); }
applyUpgrades();

/* =========================================================
   KÄED
   ========================================================= */
const handFactory = new XRHandModelFactory();
const raycaster = new THREE.Raycaster();
const hands = [0, 1].map(i => {
  const ctrl = renderer.xr.getController(i), grip = renderer.xr.getControllerGrip(i), hand = renderer.xr.getHand(i);
  rig.add(ctrl); rig.add(grip); rig.add(hand);
  const hm = handFactory.createHandModel(hand, 'spheres'); hand.add(hm);
  // pultide asemel kokakindad
  const glove = new THREE.Group(); grip.add(glove); glove.rotation.x = -0.4;
  mesh(new THREE.SphereGeometry(0.045, 16, 12), M.white, 0, -0.01, 0.03, glove, false).scale.set(1, 0.7, 1.3);
  for (let f = 0; f < 4; f++) mesh(new THREE.CapsuleGeometry(0.011, 0.04, 4, 8), M.white, -0.025 + f * 0.017, -0.01, -0.03, glove, false).rotation.x = Math.PI / 2;
  mesh(new THREE.CapsuleGeometry(0.012, 0.03, 4, 8), M.white, -0.045, -0.01, 0.01, glove, false).rotation.set(Math.PI / 2, 0, 0.8);
  const holder = new THREE.Group(); holder.position.set(0, -0.02, -0.06); grip.add(holder);
  const ray = new THREE.Mesh(new THREE.CylinderGeometry(0.002, 0.002, 1, 6).translate(0, -0.5, 0).rotateX(Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0xff8fab, transparent: true, opacity: 0.7 }));
  ray.visible = false; ctrl.add(ray);
  const handHolder = new THREE.Group(); scene.add(handHolder);
  const marker = new THREE.Mesh(new THREE.TorusGeometry(0.07, 0.006, 8, 32).rotateX(Math.PI / 2), new THREE.MeshBasicMaterial({ color: 0xffd166, transparent: true, opacity: 0.85, depthTest: false })); marker.renderOrder = 20; marker.visible = false; scene.add(marker);
  const h = { i, kind: 'xr', ctrl, grip, hand, hm, glove, holder, handHolder, marker, gClosed: false, ray, side: null, isHand: false, connected: false, held: null, vel: V3(), prev: V3(), hasPrev: false, hover: null, poke: null, gp: null, colored: false };
  ctrl.addEventListener('connected', e => { h.connected = true; h.side = e.data.handedness; h.isHand = !!e.data.hand; h.gp = e.data.gamepad; glove.visible = !h.isHand; h.colored = false; });
  ctrl.addEventListener('disconnected', () => { if (h.held) release(h); h.connected = false; });
  ctrl.addEventListener('selectstart', () => { audioInit(); grabStart(h); });
  ctrl.addEventListener('selectend', () => { if (h.held && !h.isHand && h.bySelect) release(h); });
  ctrl.addEventListener('squeezestart', () => { audioInit(); if (!h.held) { h.bySelect = false; grabStart(h, true); } });
  ctrl.addEventListener('squeezeend', () => { if (h.held && !h.bySelect) release(h); });
  return h;
});
function handPoint(h, out){
  if (h.kind === 'mouse') return h.holder.getWorldPosition(out);
  if (h.isHand && h.hand.joints) return h.handHolder.getWorldPosition(out);
  return h.holder.getWorldPosition(out);
}
function pulse(h, s = 0.5, ms = 50){ try { const gp = h.gp; if (gp && gp.hapticActuators && gp.hapticActuators[0]) gp.hapticActuators[0].pulse(s, ms); } catch (e) {} }
// hiir
const mouseHolder = new THREE.Group(); mouseHolder.position.set(0.12, -0.12, -0.38); camera.add(mouseHolder);
const mouseHand = { kind: 'mouse', holder: mouseHolder, held: null, vel: V3(), prev: V3(), hasPrev: false, hover: null };
const mouseNDC = new THREE.Vector2();
addEventListener('mousemove', e => mouseNDC.set(e.clientX / innerWidth * 2 - 1, -(e.clientY / innerHeight) * 2 + 1));
renderer.domElement.addEventListener('contextmenu', e => e.preventDefault());
renderer.domElement.addEventListener('mousedown', e => {
  audioInit();
  if (mouseHand.hover) { clickBtn(mouseHand.hover, mouseHand); return; }
  if (mode !== 'play') return;
  if (e.button === 2) { if (mouseHand.held) { const it = mouseHand.held; camera.getWorldDirection(tmpD); mouseHand.vel.copy(tmpD).multiplyScalar(4).add(V3(0, 1.5, 0)); release(mouseHand); } return; }
  if (mouseHand.held) { deskPlace(); return; }
  grabStart(mouseHand);
});
const keys = {};
addEventListener('keydown', e => { keys[e.code] = true; if (e.code === 'Escape') togglePause(); });
addEventListener('keyup', e => keys[e.code] = false);

/* ---------- võtmine ---------- */
function grabbables(){ return items.filter(it => !it.held && !it.onPlate && !(it.mat)); }
function nearestItem(p, r){ let best = null, bd = r; for (const it of grabbables()) { const d = it.g.getWorldPosition(tmpV).distanceTo(p); if (d < bd) { bd = d; best = it; } } return best; }
function nearestDisp(p, r){ let best = null, bd = r; for (const d of DISP) { if (!d.g.visible) continue; const dd = d.pos().distanceTo(p); if (dd < bd) { bd = dd; best = d; } } return best; }
function rayPick(h){
  if (h.kind === 'mouse') raycaster.setFromCamera(mouseNDC, camera);
  else { h.ctrl.getWorldPosition(tmpV); h.ctrl.getWorldQuaternion(tmpQ); raycaster.set(tmpV, tmpD.set(0, 0, -1).applyQuaternion(tmpQ)); }
  raycaster.far = 4;
  const objs = []; for (const it of grabbables()) objs.push(it.g); for (const d of DISP) if (d.g.visible) objs.push(d.g); objs.push(bell); for (const t of taps) objs.push(t.btn); if (up.ice) for (const t of iceTaps) objs.push(t.btn); if (cat.visible) objs.push(cat);
  const hit = raycaster.intersectObjects(objs, true)[0]; if (!hit) return null;
  let o = hit.object; while (o) { if (o.userData.item) return { item: o.userData.item }; const d = DISP.find(d => d.g === o); if (d) return { disp: d }; if (o === bell) return { bell: true }; const t = taps.find(t => t.btn === o) || iceTaps.find(t => t.btn === o); if (t) return { tap: t }; if (o === cat) return { cat: true }; o = o.parent; }
  return null;
}
function grabStart(h, bySqueeze = false){
  if (h.hover) { clickBtn(h.hover, h); return; }
  if (mode !== 'play' || h.held) return;
  h.bySelect = !bySqueeze;
  const p = handPoint(h, V3());
  // lähedalt (päris käed haaravad žestiga, vt handGesture)
  if (h.kind === 'xr' && h.isHand) { if (nearTarget(p)) return; }
  else if (h.kind === 'xr') {
    if (pressNear(p, h)) return;
    const it = nearestItem(p, 0.13); if (it) { grab(h, it); return; }
    const d = nearestDisp(p, 0.16); if (d) { grab(h, spawnFrom(d, p)); return; }
  }
  // kiirega kaugelt
  const r = rayPick(h); if (!r) return;
  if (r.item) grab(h, r.item); else if (r.disp) grab(h, spawnFrom(r.disp, p)); else if (r.bell) ringBell(); else if (r.tap) pressTap(r.tap); else if (r.cat) petCat();
}
function nearTarget(p){ return nearestItem(p, 0.14) || nearestDisp(p, 0.17); }
// päris käe žest: rusikas või näpistus = haara, käsi lahti = lase lahti
const J = (h, n) => h.hand.joints && h.hand.joints[n];
function handGesture(h){
  const w = J(h, 'wrist'), palm = J(h, 'middle-finger-metacarpal'), prox = J(h, 'middle-finger-phalanx-proximal'), it = J(h, 'index-finger-tip'), th = J(h, 'thumb-tip'), mt = J(h, 'middle-finger-tip'), rt = J(h, 'ring-finger-tip');
  if (!w || !palm || !it || !th || !mt || !rt || !w.visible) return;
  const P = palm.getWorldPosition(V3()), I = it.getWorldPosition(V3()), T = th.getWorldPosition(V3());
  const curl = (I.distanceTo(P) + mt.getWorldPosition(V3()).distanceTo(P) + rt.getWorldPosition(V3()).distanceTo(P)) / 3, pinch = I.distanceTo(T);
  // haarde punkt: näpistuses sõrmede vahel, muidu peopesa ees
  const gp = pinch < 0.045 ? I.clone().add(T).multiplyScalar(0.5) : (prox ? prox.getWorldPosition(V3()) : P).lerp(P, 0.3);
  h.handHolder.position.copy(gp); w.getWorldQuaternion(h.handHolder.quaternion);
  const closed = curl < 0.068 || pinch < 0.026, open = curl > 0.085 && pinch > 0.045;
  if (!h.gClosed && closed) { h.gClosed = true; if (mode === 'play' && !h.held && !h.hover) { if (pressNear(gp, h)) return; const item = nearestItem(gp, 0.14); if (item) grab(h, item); else { const d = nearestDisp(gp, 0.17); if (d) grab(h, spawnFrom(d, gp)); } } }
  else if (h.gClosed && open) { h.gClosed = false; if (h.held) release(h); }
  // näita, mida saaks haarata
  const tgt = !h.held && mode === 'play' ? (nearestItem(gp, 0.14) || nearestDisp(gp, 0.17)) : null;
  h.marker.visible = !!tgt; if (tgt) { (tgt.g.getWorldPosition ? tgt.g.getWorldPosition(h.marker.position) : h.marker.position.copy(tgt.pos())); if (tgt.pos) h.marker.position.y -= 0.05; h.marker.scale.setScalar(1 + Math.sin(performance.now() / 150) * 0.1); }
}
function pressNear(p, h){
  const bp = bell.getWorldPosition(V3()).add(V3(0, 0.05, 0)); if (bp.distanceTo(p) < 0.09) { ringBell(); pulse(h, 0.6, 60); return true; }
  for (const t of [...taps, ...(up.ice ? iceTaps : [])]) if (t.btn.getWorldPosition(V3()).distanceTo(p) < 0.07) { pressTap(t); pulse(h, 0.4, 40); return true; }
  return false;
}
function spawnFrom(d, p){ sfx.pick(); return newItem(d.type, p.clone(), d.extra && d.extra.flavor ? { flavor: d.extra.flavor } : {}); }
function grab(h, it){
  for (const o of [...hands, mouseHand]) if (o.held === it) o.held = null;
  if (it.slot) { if (it.slot.kind === 'tap' && it.filling) return; freeSlot(it); }
  if (it.mat) { const m = it.mat; m.items.splice(m.items.indexOf(it), 1); it.mat = null; }
  h.held = it; it.held = h; it.rest = false; it.vel.set(0, 0, 0);
  (h.isHand ? h.handHolder : h.holder).attach(it.g); it.g.position.set(0, 0, 0); it.g.rotation.set(it.type === 'ext' ? -0.3 : 0, 0, 0);
  if (it.type === 'ext') it.g.rotation.set(-(Math.PI / 2 - 0.3), 0, 0);
  sfx.pick(); if (h.kind === 'xr') pulse(h, 0.3, 30);
  if (it.type === 'duck') sfx.squeak();
}
function release(h){
  const it = h.held; if (!it) return; h.held = null; it.held = null;
  scene.attach(it.g); it.g.rotation.set(0, it.g.rotation.y, 0);
  it.vel.copy(h.vel).multiplyScalar(h.kind === 'mouse' ? 1 : 1.1); it.rest = false;
  if (it.vel.length() < 0.6 && trySnap(it)) return;
  if (it.vel.length() < 0.6) it.vel.set(0, 0, 0);
}
function deskPlace(){
  const it = mouseHand.held; raycaster.setFromCamera(mouseNDC, camera); raycaster.far = 4;
  const objs = [K, ...items.filter(x => x !== it).map(x => x.g)]; const hit = raycaster.intersectObjects(objs, true)[0];
  mouseHand.held = null; it.held = null; scene.attach(it.g); it.g.rotation.set(0, 0, 0); it.vel.set(0, 0, 0); it.rest = false;
  if (hit) it.g.position.copy(hit.point).add(V3(0, 0.08, 0));
  if (!trySnap(it)) {}
}

/* ---------- paigutamine: taldrik, grill, fritüür, kraan, matt, prügi ---------- */
function trySnap(it){
  const p = it.g.getWorldPosition(V3());
  // prügikast
  const tp = trash.getWorldPosition(V3()).add(V3(0, 0.62, 0)); if (Math.hypot(p.x - tp.x, p.z - tp.z) < 0.24 && p.y > tp.y - 0.25 && p.y < tp.y + 0.6) { trashIt(it); return true; }
  // pesad
  for (const s of SLOTS) { if (s.item || !s.accepts(it)) continue; const w = slotWorld(s); if (Math.hypot(p.x - w.x, p.z - w.z) < 0.11 && Math.abs(p.y - w.y) < 0.3) { putInSlot(it, s); if (s.kind === 'grill') { sfx.drop(); emit(w, V3(0, 0.3, 0), 0xffffff, 0.04, 0.6); } else sfx.plop(); return true; } }
  // vorst kuklisse
  if (it.type === 'sausage') for (const b of items) { if (b.type !== 'hdBun' || b.held || b.sausage || b.mat) continue; const q = b.g.getWorldPosition(V3()); if (Math.hypot(p.x - q.x, p.z - q.z) < 0.09 && p.y > q.y - 0.05 && p.y < q.y + 0.4) { freeSlot(it); it.onPlate = b; it.rest = true; it.vel.set(0, 0, 0); b.g.attach(it.g); it.g.position.set(0, 0.022, 0); it.g.rotation.set(0, 0, 0); b.sausage = it; b.stack.push(it); sfx.stack(); pop(q.clone().add(V3(0, 0.12, 0)), '🌭 Valmis!', '#2a9d8f'); return true; } }
  // taldrikule
  if (LAYER_H[it.type]) for (const pl of items) { if (pl.type !== 'plate' || pl === it || pl.held || pl.mat) continue; const q = pl.g.getWorldPosition(V3()); if (Math.hypot(p.x - q.x, p.z - q.z) < 0.1 && p.y > q.y - 0.05 && p.y < q.y + 0.4) { addToPlate(pl, it); return true; } }
  // matile (kliendile)
  for (const m of mats) { const q = m.mesh.getWorldPosition(V3()); if (Math.abs(p.x - q.x) < 0.24 && Math.abs(p.z - q.z) < 0.2 && p.y > q.y - 0.1 && p.y < q.y + 0.5) { serve(m, it); return true; } }
  return false;
}
function trashIt(it){ const p = it.g.getWorldPosition(V3()); burst(p, 0x888888, 12, 0.6); sfx.trash(); if (it.type === 'ext' || it.type === 'duck') { respawnTool(it); return; } removeItem(it); }
function respawnTool(it){ it.vel.set(0, 0, 0); it.rest = false; it.g.position.copy(K.localToWorld(it.type === 'ext' ? V3(1.2, CT + 0.2, 0.42) : V3(0.35, CT + 0.1, -0.86))); }
function addToPlate(pl, it){
  if (pl.stack.length && pl.stack[pl.stack.length - 1].type === 'bunT') { sfx.wrong(); showToast('Burger on juba valmis – kaanekukkel on peal!', '#e63946', 1.5); bounceBack(it); return; }
  if (!pl.stack.length && it.type !== 'bunB') showToast('💡 Burger algab põhjakukliga!', '#ffb347', 1.6);
  freeSlot(it); it.onPlate = pl; it.rest = true; it.vel.set(0, 0, 0);
  const h = 0.008 + pl.stack.reduce((a, s) => a + LAYER_H[s.type], 0);
  pl.g.attach(it.g); it.g.position.set(rand(-0.004, 0.004), h + LAYER_H[it.type] / 2, rand(-0.004, 0.004)); it.g.rotation.set(0, rand(0, 6.28), 0);
  pl.stack.push(it); sfx.stack(); burst(it.g.getWorldPosition(V3()), 0xfff1c9, 6, 0.4, 0.02);
  if (it.type === 'bunT') { pop(it.g.getWorldPosition(V3()).add(V3(0, 0.12, 0)), '🍔 Valmis!', '#2a9d8f'); sfx.happy(); }
}
function bounceBack(it){ it.vel.set(rand(-0.3, 0.3), 1.5, 1.2); it.rest = false; if (it.mat) { it.mat.items.splice(it.mat.items.indexOf(it), 1); it.mat = null; } }

/* =========================================================
   FÜÜSIKA
   ========================================================= */
function surfaceBelow(p, r){ let best = -1; for (const s of SURF) { const y = s[4] + K.position.y; if (p.x > s[0] - r * 0.5 && p.x < s[1] + r * 0.5 && p.z > s[2] - r * 0.5 && p.z < s[3] + r * 0.5 && y <= p.y + 0.02 && y > best) best = y; } return best; }
function updatePhysics(dt){
  for (const it of items) {
    if (it.held || it.onPlate || it.slot || it.mat) continue;
    if (it.rest) { it.age += dt; if (it.age > 40 && it.g.position.y < 0.2 && it.type !== 'ext' && it.type !== 'duck') { removeItem(it); } continue; }
    const p = it.g.position; it.vel.y -= 9.8 * dt; const prevY = p.y; p.addScaledVector(it.vel, dt);
    // vastu kliente (visatud toit)
    for (const c of custs) { if (!c.here || c.hitCd > 0) continue; const hp = c.m.headG.getWorldPosition(V3()); if (hp.distanceTo(p) < 0.2 || (Math.hypot(p.x - hp.x, p.z - hp.z) < 0.22 && p.y < hp.y && p.y > hp.y - 0.9)) { custHit(c, it); it.vel.set(-it.vel.x * 0.3, 1, -it.vel.z * 0.3 + 1); break; } }
    const sy = surfaceBelow(p, it.r);
    if (sy > -1 && p.y - it.hh <= sy && prevY - it.hh >= sy - 0.05 && it.vel.y <= 0) {
      p.y = sy + it.hh;
      if (trySnap(it)) continue;
      if (Math.abs(it.vel.y) > 1.2) { it.vel.y *= -0.3; it.vel.x *= 0.6; it.vel.z *= 0.6; if (it.type === 'duck') sfx.squeak(); else sfx.drop(); }
      else { it.vel.set(0, 0, 0); it.rest = true; it.age = 0; it.g.rotation.x = 0; it.g.rotation.z = 0; }
    }
    if (p.y < -1) { respawnTool(it); if (it.type !== 'ext' && it.type !== 'duck') removeItem(it); }
    if (!it.rest) it.g.rotation.x += dt * it.vel.length() * 2;
  }
}

/* =========================================================
   KÜPSETAMINE, KRAANID, TULI
   ========================================================= */
function ringBell(){ sfx.ding(); bell.scale.setScalar(1.15); setTimeout(() => bell.scale.setScalar(1), 120); if (mode === 'play') { let n = 0; for (const c of custs) if (c.here && c.patience > 0) { c.patience = Math.min(c.maxP, c.patience + c.maxP * 0.05); n++; } if (n) showToast('🔔 Ding! Kliendid naeratavad', '#ffb347', 1.2); } }
function pressTap(t){
  const s = t.slot, cup = s.item; if (t.ice) { t.btn.scale.setScalar(0.85); setTimeout(() => t.btn.scale.setScalar(1), 150); } else { t.btn.position.x = -0.1; setTimeout(() => t.btn.position.x = -0.11, 150); }
  if (!cup) { sfx.click(); emit(slotWorld(s).add(V3(0, 0.2, 0)), V3(0, -1, 0), t.f.c, 0.02, 0.4, -4); showToast(t.ice ? 'Pane vahvel masina alla! 🍦' : 'Pane tops kraani alla! 🥤', '#ffb347', 1.3); return; }
  if (cup.flavor || cup.filling) return;
  cup.filling = 1.4; cup.fillFlavor = t.f; sfx.pour();
}
function updateCooking(dt){
  let sizz = 0, fry = 0;
  for (const s of SLOTS) {
    const it = s.item, w = slotWorld(s);
    if (s.kind === 'grill') {
      if (s.fire > 0) { for (let k = 0; k < 3; k++) emit(w.clone().add(V3(rand(-0.05, 0.05), 0.02, rand(-0.05, 0.05))), V3(rand(-0.1, 0.1), rand(0.6, 1.1), rand(-0.1, 0.1)), pick([0xff5722, 0xffc107, 0xff9800]), 0.06, 0.5); if (Math.random() < dt * 2) emit(w.clone().add(V3(0, 0.2, 0)), V3(0, 0.4, 0), 0x555555, 0.12, 1.5); }
      if (!it) continue;
      sizz++; it.cook += dt; const ct = cookT(), bt = burnT();
      if (it.state === 'raw' && it.cook >= ct) { it.state = 'done'; pop(w.clone().add(V3(0, 0.12, 0)), '✔ Valmis!', '#2a9d8f'); tone(880, 0.12, 'triangle', 0.1); }
      if (it.state === 'done' && it.cook >= bt) { it.state = 'burnt'; pop(w.clone().add(V3(0, 0.12, 0)), '💨 Kõrbes!', '#555'); }
      if (it.state === 'burnt' && Math.random() < dt * 6) emit(w.clone().add(V3(rand(-0.03, 0.03), 0.03, rand(-0.03, 0.03))), V3(rand(-0.05, 0.05), 0.35, rand(-0.05, 0.05)), 0x444444, 0.07, 1.4);
      if (it.state === 'done' && Math.random() < dt * 3) emit(w.clone().add(V3(rand(-0.03, 0.03), 0.03, 0)), V3(0, 0.25, 0), 0xeeeeee, 0.03, 0.8);
      if (it.state === 'burnt' && it.cook >= fireT() && s.fire <= 0) { s.fire = 1; sfx.fire(); showToast('🔥 GRILL PÕLEB! Võta tulekustuti (paremal)!', '#e63946', 3); }
      setCookLook(it);
    } else if (s.kind === 'fryer') {
      if (!it) continue; fry++; it.cook += dt;
      if (Math.random() < dt * 10) emit(w.clone().add(V3(rand(-0.06, 0.06), 0.01, rand(-0.06, 0.06))), V3(0, 0.15, 0), 0xfff8e1, 0.02, 0.4);
      if (it.type === 'friesRaw' && it.cook >= fryT()) { // muutub friikateks
        const fr = newItem('fries', w.clone().add(V3(0, 0.06, 0))); fr.state = 'ok'; s.item = null; it.slot = null; removeItem(it); putInSlot(fr, s); fr.g.position.y = w.y + fr.hh; fr.fryCook = 0; setCookLook(fr); pop(w.clone().add(V3(0, 0.15, 0)), '🍟 Valmis!', '#ffb347'); tone(988, 0.12, 'triangle', 0.1);
      }
    }
    if (s.kind === 'fryer' && it && it.type === 'fries') { it.fryCook += dt; if (diff().burn && it.state !== 'burnt' && it.fryCook > (diff().id === 1 ? 18 : 12)) { it.state = 'burnt'; setCookLook(it); pop(w.clone().add(V3(0, 0.15, 0)), '💨 Kõrbes!', '#555'); } }
  }
  // tops kraani all
  for (const it of items) if (it.filling > 0) {
    it.filling -= dt; const w = it.g.getWorldPosition(V3()); emit(w.clone().add(V3(0, 0.15, 0)), V3(rand(-0.02, 0.02), -1.2, rand(-0.02, 0.02)), it.fillFlavor.c, 0.025, 0.12);
    if (it.filling <= 0 && it.type === 'cone') { it.filling = 0; it.flavor = it.fillFlavor; it.g.userData.scoop.material = std(it.flavor.c); it.g.userData.scoop.visible = true; tone(1175, 0.1, 'triangle', 0.1); pop(w.clone().add(V3(0, 0.15, 0)), `🍦 ${it.flavor.n}`, '#2a9d8f'); continue; }
    if (it.filling <= 0) { it.filling = 0; it.flavor = it.fillFlavor; it.g.userData.band.material = std(it.flavor.c); it.g.userData.lid.visible = true; it.g.userData.straw.visible = true; tone(1175, 0.1, 'triangle', 0.1); pop(w.clone().add(V3(0, 0.15, 0)), `${it.flavor.e} ${it.flavor.n}`, '#2a9d8f'); }
  }
  if (sizzleG) { sizzleG.gain.setTargetAtTime(mode === 'play' ? Math.min(0.25, sizz * 0.07) : 0, actx.currentTime, 0.1); fryG.gain.setTargetAtTime(mode === 'play' ? Math.min(0.2, fry * 0.08) : 0, actx.currentTime, 0.1); }
  grillGlow.material.opacity = 0.18 + Math.sin(performance.now() / 300) * 0.05 + SLOTS.filter(s => s.fire > 0).length * 0.2;
  // tulekustuti
  if (extItem && extItem.held) {
    const nz = extItem.g.userData.nozzle, np = nz.getWorldPosition(V3()), dir = V3(0, 1, 0).applyQuaternion(extItem.g.getWorldQuaternion(new THREE.Quaternion())).normalize();
    for (let k = 0; k < 4; k++) emit(np, dir.clone().multiplyScalar(rand(2, 3.2)).add(V3(rand(-0.3, 0.3), rand(-0.3, 0.3), rand(-0.3, 0.3))), 0xffffff, 0.06, 0.6, -1);
    if (Math.random() < dt * 5) sfx.spray();
    for (const s of SLOTS) if (s.fire > 0) { const w = slotWorld(s), to = w.clone().sub(np), d = to.length(); if (d < 2 && to.normalize().dot(dir) > 0.85) { s.fire -= dt * 0.8; if (s.fire <= 0) { s.fire = 0; sfx.poof(); burst(w, 0xffffff, 30, 1); showToast('🧯 Tuli kustus! Tubli!', '#2a9d8f', 2); if (s.item) { removeItem(s.item); } } } }
  }
}

/* =========================================================
   KLIENTIDE LOOGIKA
   ========================================================= */
let day = null;
function allowedExtras(){ const d = save.day, D = diff(); const a = ['cheese']; if (d >= 2 || D.id === 0) a.push('lettuce'); if (d >= 2) a.push('tomato'); return a; }
function genOrder(kind){
  const D = diff(), dd = save.day, ex = allowedExtras(), reqs = [], maxItems = D.id === 0 || kind === 'kid' ? 2 : D.id === 1 ? 3 : 4;
  const mains = ['burger']; if (up.hotdog) mains.push('hotdog', 'hotdog');
  const main = kind === 'kid' && (up.ice || up.donut) && Math.random() < 0.3 ? null : pick(mains);
  if (main === 'burger') { const n = Math.min(D.extras, Math.floor(rand(0, Math.min(D.extras, 1 + dd * 0.5) + 1))); const mid = ['patty']; for (let i = 0; i < n; i++) mid.push(pick(ex)); if (D.id === 3 && Math.random() < 0.25) mid.push('patty'); if (kind === 'kid' || D.id === 0) mid.splice(1 + Math.floor(Math.random() * 2)); reqs.push({ k: 'burger', mid }); }
  if (main === 'hotdog') reqs.push({ k: 'hotdog' });
  const extras = [];
  if (Math.random() < D.fries) extras.push({ k: 'fries' });
  if (Math.random() < D.drink) extras.push({ k: 'drink', f: pick(FLAVORS) });
  if (up.ice && Math.random() < (main ? 0.4 : 1)) extras.push({ k: 'ice', f: pick(ICES) });
  if (up.donut && Math.random() < 0.4) extras.push({ k: 'donut', f: pick(DONUTS) });
  for (const e of extras.sort(() => Math.random() - 0.5)) if (reqs.length < maxItems) reqs.push(e);
  if (!reqs.length) reqs.push({ k: 'fries' });
  return { reqs };
}
function spawnCustomer(slotI){
  const r = Math.random(), kind = r < 0.1 ? 'vip' : r < 0.2 ? 'robot' : r < 0.28 ? 'cat' : r < 0.4 ? 'kid' : 'normal';
  const m = makeCustomer(kind); const side = Math.random() < 0.5 ? -1 : 1;
  if (kind === 'kid') m.g.scale.setScalar(0.75);
  const maxP = diff().patience * (up.calm ? 1.35 : 1) * (kind === 'vip' ? 0.8 : 1);
  const c = { m, kind, slot: slotI, x: side * 3.2, tx: MAT_X[slotI], z: -1.75, state: 'in', order: genOrder(kind), patience: maxP, maxP, here: false, hitCd: 0, t: 0 };
  m.g.position.set(c.x, 0, c.z); mats[slotI].cust = c; custs.push(c); drawBubble(c); return c;
}
function needs(c){ return c.order.reqs.filter(r => !r.done); }
const LAYER_COL = { bunB: '#e0a65a', bunT: '#d99549', patty: '#7a4a2e', cheese: '#ffc93c', lettuce: '#6abe45', tomato: '#e63946' };
function drawBubble(c){
  const reqs = c.order.reqs, pr = clamp(c.patience / c.maxP, 0, 1);
  c.m.bubble.draw((ctx, W, H) => {
    roundRect(ctx, 6, 6, W - 12, H * 0.78, 40); ctx.fillStyle = '#fffaf3'; ctx.fill(); ctx.lineWidth = 8; ctx.strokeStyle = c.kind === 'vip' ? '#ffd700' : '#ff8fab'; ctx.stroke();
    ctx.beginPath(); ctx.moveTo(W * 0.45, H * 0.78); ctx.lineTo(W * 0.5, H * 0.92); ctx.lineTo(W * 0.55, H * 0.78); ctx.fillStyle = '#fffaf3'; ctx.fill();
    const cw = (W - 40) / Math.max(2, reqs.length);
    reqs.forEach((r, i) => {
      const cx = 20 + cw * (i + 0.5);
      if (r.k === 'burger') { const layers = ['bunB', ...r.mid, 'bunT']; let y = H * 0.66; const sc = Math.min(1, cw / 130); layers.forEach(L => { const h = (L === 'bunT' ? 30 : L === 'bunB' ? 20 : L === 'patty' ? 16 : 9) * sc, w = (L === 'cheese' ? 112 : 100) * sc; ctx.fillStyle = LAYER_COL[L]; roundRect(ctx, cx - w / 2, y - h, w, h, L === 'bunT' ? 14 * sc : 4); ctx.fill(); y -= h + 2; }); }
      else { const em = { fries: '🍟', drink: '🥤', hotdog: '🌭', ice: '🍦', donut: '🍩' }[r.k]; txt(ctx, em, cx, H * 0.36, H * 0.22); if (r.f) { ctx.fillStyle = '#' + new THREE.Color(r.f.c).getHexString(); ctx.beginPath(); ctx.arc(cx, H * 0.62, H * 0.055, 0, 7); ctx.fill(); ctx.lineWidth = 3; ctx.strokeStyle = '#3d2c3e'; ctx.stroke(); } }
      if (r.done) { ctx.fillStyle = 'rgba(42,157,143,0.25)'; roundRect(ctx, cx - cw / 2 + 4, 16, cw - 8, H * 0.6, 20); ctx.fill(); txt(ctx, '✔', cx, H * 0.38, H * 0.25, '#2a9d8f', '900'); }
    });
    if (c.kind === 'vip') txt(ctx, '👑', W * 0.93, H * 0.12, H * 0.11);
    ctx.fillStyle = '#e0d6cc'; roundRect(ctx, W * 0.1, H * 0.86, W * 0.8, H * 0.1, 12); ctx.fill();
    ctx.fillStyle = pr > 0.5 ? '#2a9d8f' : pr > 0.25 ? '#ffb347' : '#e63946'; roundRect(ctx, W * 0.1, H * 0.86, Math.max(24, W * 0.8 * pr), H * 0.1, 12); ctx.fill();
  });
}
function burgerSpec(pl){ const s = pl.stack.map(x => x.type); return s; }
function matchBurger(pl, order){
  const s = pl.stack; if (s.length < 2) return 'Burger pole valmis!';
  if (s[0].type !== 'bunB') return 'Põhjakukkel puudub!'; if (s[s.length - 1].type !== 'bunT') return 'Kaanekukkel puudub!';
  const mid = s.slice(1, -1); if (mid.some(x => x.type === 'patty' && x.state === 'raw')) return '🤢 Toores kotlet!'; if (mid.some(x => x.type === 'patty' && x.state === 'burnt')) return '🤢 Kõrbenud kotlet!';
  const a = mid.map(x => x.type).sort().join(), b = order.mid.slice().sort().join(); if (a !== b) return 'Vale burger! Vaata tellimust.';
  return null;
}
function serve(m, it){
  const c = m.cust; const q = m.mesh.getWorldPosition(V3());
  if (!c || c.state !== 'wait') { sfx.wrong(); showToast('Siin pole klienti 🙂', '#ffb347', 1.2); bounceBack(it); return; }
  const n = needs(c), kindOf = { plate: 'burger', fries: 'fries', cup: 'drink', hdBun: 'hotdog', cone: 'ice', donut: 'donut' }[it.type];
  const NAMEK = { burger: 'Burgerit', fries: 'Friikaid', drink: 'Jooki', hotdog: 'Hotdogi', ice: 'Jäätist', donut: 'Sõõrikut' };
  let err = null, req = null;
  if (!kindOf) err = LAYER_H[it.type] ? 'Pane see taldrikule! 🍽️' : it.type === 'sausage' ? 'Pane vorst kuklisse! 🌭' : 'Mida see on? 😅';
  else {
    const cand = n.filter(r => r.k === kindOf);
    if (!cand.length) err = `${NAMEK[kindOf]} ma ei tellinud!`;
    else if (kindOf === 'burger') { err = matchBurger(it, cand[0]); req = cand[0]; }
    else if (kindOf === 'fries') { if (it.state === 'burnt') err = '🤢 Kõrbenud friikad!'; else req = cand[0]; }
    else if (kindOf === 'hotdog') { if (!it.sausage) err = 'Vorst puudub! 🌭'; else if (it.sausage.state === 'raw') err = '🤢 Toores vorst!'; else if (it.sausage.state === 'burnt') err = '🤢 Kõrbenud vorst!'; else req = cand[0]; }
    else { if (!it.flavor) err = kindOf === 'drink' ? 'Tops on tühi!' : 'Jäätis puudub!'; else { req = cand.find(r => r.f === it.flavor); if (!req) err = `Ma tahtsin ${cand[0].f.n}!`; } }
  }
  if (err) { sfx.wrong(); showToast(`${c.kind === 'robot' ? '🤖 ' : ''}${err}`, '#e63946', 2); c.patience -= c.maxP * 0.06; bounceBack(it); emote(c, 'sad'); return; }
  req.done = true; it.mat = m; m.items.push(it); it.rest = true; it.vel.set(0, 0, 0);
  const k = m.items.length - 1; it.g.position.set(q.x - 0.15 + (k % 4) * 0.1, q.y + it.hh + 0.004, q.z + (k >= 4 ? 0.08 : 0)); it.g.rotation.set(0, 0, 0);
  sfx.stack(); emote(c, 'happy'); drawBubble(c);
  if (!needs(c).length) finishCustomer(c, true);
}
function emote(c, kind){ c.emote = kind; c.emoteT = 1; }
function finishCustomer(c, happy){
  c.state = 'out'; c.m.bubble.mesh.visible = false; const hp = c.m.headG.getWorldPosition(V3()).add(V3(0, 0.3, 0));
  if (happy) {
    const pr = clamp(c.patience / c.maxP, 0, 1), o = c.order;
    const PRICE = { fries: 3, drink: 2, hotdog: 8, ice: 5, donut: 3 }; let pay = o.reqs.reduce((a, r) => a + (r.k === 'burger' ? 5 + r.mid.length * 2 : PRICE[r.k]), 0); const tip = Math.round(pr * 5 * (up.disco ? 1.4 : 1)); pay += tip; if (c.kind === 'vip') pay *= 2;
    day.earned += pay; save.coins += pay; day.served++; persist();
    pop(hp, `+${pay} 🪙`, '#e6a100'); sfx.coin(); sfx.happy(); burst(hp, 0xff4fa3, 16, 0.8, 0.05); c.hearts = 1.5;
    if (c.kind === 'vip') showToast('👑 VIP oli väga rahul! Topelt raha!', '#e6a100', 2);
  } else { day.angry++; sfx.angry(); pop(hp, '😠', '#e63946'); emote(c, 'angry'); }
  for (const it of [...mats[c.slot].items]) { removeItem(it); } mats[c.slot].items = []; mats[c.slot].cust = null;
  c.tx = c.x > 0 ? -3.4 : 3.4;
}
function custHit(c, it){
  c.hitCd = 1; const hp = c.m.headG.getWorldPosition(V3());
  if (it.type === 'duck') { sfx.squeak(); pop(hp.add(V3(0, 0.3, 0)), '😂 Haha!', '#2a9d8f'); c.patience = Math.min(c.maxP, c.patience + 5); return; }
  sfx.angry(); pop(hp.add(V3(0, 0.3, 0)), pick(['Hei! 😠', 'Ai! 😵', 'Mis see oli?!']), '#e63946'); c.patience -= c.maxP * 0.1; emote(c, 'angry');
}
function updateCustomers(dt){
  if (!day) return;
  const D = diff();
  // uued kliendid
  day.spawnT -= dt; const active = custs.filter(c => c.state !== 'out').length;
  if (day.t < D.dayLen && day.spawnT <= 0 && active < D.maxCust) { const free = mats.filter(m => !m.cust).map(m => m.i); if (free.length) { spawnCustomer(pick(free)); day.spawnT = D.gap * rand(0.7, 1.3) * (save.day > 3 ? 0.85 : 1); } }
  for (let i = custs.length - 1; i >= 0; i--) {
    const c = custs[i], g = c.m.g; c.t += dt; c.hitCd -= dt;
    const dx = c.tx - c.x, sp = 1.3 * dt; c.x += Math.sign(dx) * Math.min(Math.abs(dx), sp); g.position.x = c.x;
    const walking = Math.abs(dx) > 0.01; g.position.y = walking ? Math.abs(Math.sin(c.t * 9)) * 0.04 : 0; g.rotation.y = walking ? Math.sign(dx) * Math.PI / 2 * 0.8 : 0;
    if (c.state === 'in' && !walking) { c.state = 'wait'; c.here = true; c.m.bubble.mesh.visible = true; drawBubble(c); tone(660, 0.1, 'sine', 0.08); tone(880, 0.12, 'sine', 0.08, 0.08); }
    c.m.bubble.mesh.visible = c.state === 'wait';
    if (c.state === 'wait') {
      c.patience -= dt * (SLOTS.some(s => s.fire > 0) ? 1.6 : 1); if (Math.floor(c.patience * 2) !== c.lastP) { c.lastP = Math.floor(c.patience * 2); drawBubble(c); }
      if (c.patience <= 0) finishCustomer(c, false);
      camera.getWorldPosition(head); c.m.headG.lookAt(head.x, c.m.headG.getWorldPosition(V3()).y, head.z);
      c.m.headG.rotation.z = Math.sin(c.t * 2) * 0.05;
    }
    // tuju
    const pr = clamp(c.patience / c.maxP, 0, 1), mood = c.emote && c.emoteT > 0 ? (c.emote === 'happy' ? 1 : -1) : pr > 0.5 ? 1 : pr > 0.25 ? 0 : -1;
    c.m.mouth.rotation.z = mood > 0 ? Math.PI : 0; c.m.mouth.position.y = mood > 0 ? -0.05 : -0.08; c.m.mouth.scale.x = mood === 0 ? 0.01 + 1 : 1; c.m.mouth.scale.y = mood === 0 ? 0.15 : 1;
    if (c.emoteT > 0) { c.emoteT -= dt; if (c.emote === 'happy') g.position.y = Math.abs(Math.sin(c.t * 14)) * 0.08; }
    if (c.hearts > 0) { c.hearts -= dt; if (Math.random() < dt * 8) emit(c.m.headG.getWorldPosition(V3()).add(V3(rand(-0.15, 0.15), 0.2, 0)), V3(0, 0.5, 0), 0xff4fa3, 0.05, 1); }
    if (c.state === 'out' && !walking) { g.removeFromParent(); custs.splice(i, 1); }
  }
}
/* ---------- kass ---------- */
function petCat(){ const u = cat.userData; if (u.petT > 0) return; u.petT = 2; sfx.purr(); burst(cat.getWorldPosition(V3()).add(V3(0, 0.25, 0)), 0xff4fa3, 8, 0.4, 0.05); pop(cat.getWorldPosition(V3()).add(V3(0, 0.35, 0)), '😻 Nurr!', '#ff4fa3'); }
function updateCat(dt){
  if (!cat.visible) return; const u = cat.userData; u.t += dt;
  if (u.petT > 0) { u.petT -= dt; if (Math.random() < dt * 3) emit(cat.getWorldPosition(V3()).add(V3(0, 0.25, 0)), V3(0, 0.4, 0), 0xff4fa3, 0.04, 1); }
  else { u.x += u.dir * dt * 0.15; if (Math.abs(u.x) > 0.75) { u.dir *= -1; if (Math.random() < 0.5) sfx.meow(); } }
  cat.position.set(u.x, PASS_Y, -1.27); cat.rotation.y = u.dir > 0 ? 0 : Math.PI; u.tail.rotation.x = Math.sin(u.t * 3) * 0.4; u.hd.rotation.z = Math.sin(u.t * 1.3) * 0.1;
  for (const h of hands) if (h.connected) { const p = handPoint(h, V3()); if (p.distanceTo(cat.getWorldPosition(V3()).add(V3(0, 0.1, 0))) < 0.15) petCat(); }
}

/* =========================================================
   MÄNGU OLEK
   ========================================================= */
let mode = 'menu';
function setupKitchen(H){ K.position.y = clamp(H * 0.575 - CT, -0.32, 0.12); }
function clearAll(){ for (const it of [...items]) removeItem(it); for (const c of custs) c.m.g.removeFromParent(); custs.length = 0; mats.forEach(m => { m.items = []; m.cust = null; }); SLOTS.forEach(s => { s.item = null; s.fire = 0; }); for (const h of [...hands, mouseHand]) h.held = null; }
function spawnTools(){ extItem = newItem('ext', K.localToWorld(V3(1.2, CT + 0.14, 0.42))); duckItem = newItem('duck', K.localToWorld(V3(0.35, CT + 0.04, -0.86))); for (const it of [extItem, duckItem]) { it.rest = true; } extItem.g.position.y = K.localToWorld(V3(0, CT, 0)).y + extItem.hh; duckItem.g.position.y = K.localToWorld(V3(0, CT, 0)).y + duckItem.hh; newItem('plate', K.localToWorld(V3(-0.12, CT + 0.008, -0.55))).rest = true; newItem('plate', K.localToWorld(V3(0.15, CT + 0.008, -0.55))).rest = true; }
function hideUI(){ menuG.visible = endG.visible = pauseG.visible = false; }
function startDay(){
  audioInit(); if (actx.state === 'suspended') actx.resume();
  hideUI(); clearAll();
  camera.getWorldPosition(head); const H = renderer.xr.isPresenting ? clamp(head.y, 1.0, 2.0) : 1.6;
  if (renderer.xr.isPresenting) { rig.position.x -= head.x; rig.position.z -= head.z - 0.05; }
  setupKitchen(H); spawnTools();
  day = { t: 0, earned: 0, served: 0, angry: 0, spawnT: 1.5, goal: Math.round((20 + save.day * 12) * diff().goal) };
  mode = 'play'; hud.mesh.visible = true; musicStep = 0;
  showToast(`📅 Päev ${save.day} – eesmärk ${day.goal} 🪙! ${save.day === 1 ? 'Kliendid tulevad leti taha 👀' : save.day === 2 ? 'UUS: 🍟 friikad ja 🥬🍅!' : save.day === 3 ? 'UUS: 🥤 joogid!' : ''}`, '#e63946', 3.5);
}
function endDay(){
  mode = 'end'; toast.mesh.visible = false; toastT = 0; for (const c of custs) if (c.state === 'wait') { c.state = 'out'; c.m.bubble.mesh.visible = false; c.tx = c.x > 0 ? 3.4 : -3.4; mats[c.slot].cust = null; }
  const D = diff(), ok = day.earned >= day.goal, stars = day.earned >= day.goal * 1.5 ? 3 : ok ? 2 : day.served > 0 ? 1 : 0;
  lastSummary = { day: save.day, earned: day.earned, goal: day.goal, served: day.served, angry: day.angry, ok, stars };
  if (!save.best[save.diff] || day.earned > save.best[save.diff]) save.best[save.diff] = day.earned;
  if (ok) save.day++; persist();
  nextBtn.label = ok ? '▶ Järgmine päev' : '🔁 Proovi päeva uuesti'; nextBtn.draw();
  drawEnd(); placeInFront(endG, 1.15, -0.1); endG.visible = true; hud.mesh.visible = false;
  if (ok) { sfx.happy(); for (let i = 0; i < 5; i++) setTimeout(() => { camera.getWorldPosition(head); burst(head.clone().add(V3(rand(-1, 1), 0.6, -1.5)), pick([0xff4fa3, 0xffd166, 0x4ecdc4]), 40, 1.8, 0.06); }, i * 250); } else sfx.wrong();
}
function goMenu(){ hideUI(); clearAll(); day = null; mode = 'menu'; hud.mesh.visible = false; refreshMenu(); placeInFront(menuG, 1.1, -0.15); menuG.visible = true; if (actx && actx.state === 'suspended') actx.resume(); }
function togglePause(){ if (mode === 'play') { mode = 'pause'; if (actx) actx.suspend(); placeInFront(pauseG, 0.8, -0.15); pauseG.visible = true; } else if (mode === 'pause') resume(); }
function resume(){ pauseG.visible = false; mode = 'play'; if (actx) actx.resume(); }
let hudKey = '';
function drawHud(){
  const D = diff(), left = Math.max(0, D.dayLen - day.t), key = `${Math.ceil(left)}|${day.earned}|${save.day}`;
  if (key === hudKey) return; hudKey = key;
  hud.draw((c, W, H) => { roundRect(c, 6, 6, W - 12, H - 12, H * 0.35); c.fillStyle = 'rgba(255,250,243,0.92)'; c.fill(); c.lineWidth = 6; c.strokeStyle = '#ff8fab'; c.stroke();
    txt(c, `📅 Päev ${save.day}`, W * 0.15, H / 2, H * 0.38); txt(c, `⏱ ${Math.floor(left / 60)}:${String(Math.floor(left % 60)).padStart(2, '0')}`, W * 0.4, H / 2, H * 0.38, left < 20 ? '#e63946' : '#3d2c3e');
    txt(c, `🪙 ${day.earned} / ${day.goal}`, W * 0.72, H / 2, H * 0.38, day.earned >= day.goal ? '#2a9d8f' : '#3d2c3e'); });
}

/* =========================================================
   KÄTE UUENDAMINE (hõljumine, kiired, sõrmega vajutus)
   ========================================================= */
function updateHands(dt){
  const btnMeshes = buttons.filter(b => isShown(b.p.mesh)).map(b => b.p.mesh);
  const setHover = (h, b) => { if (h.hover === b) return; if (h.hover) { h.hover.hover = false; h.hover.draw(); } h.hover = b; if (b) { b.hover = true; b.draw(); } };
  const ui = mode !== 'play';
  for (const h of hands) {
    h.ray.visible = false; if (!h.connected) { setHover(h, null); h.marker.visible = false; continue; }
    const p = handPoint(h, V3()); if (h.hasPrev) h.vel.lerp(tmpD.copy(p).sub(h.prev).divideScalar(Math.max(dt, 1e-3)), 0.5); h.prev.copy(p); h.hasPrev = true;
    if (h.isHand) handGesture(h); else h.marker.visible = false;
    if (h.isHand && !h.colored) { const mc = h.hm.motionController, m = mc && mc.handMesh; if (m && m.material) { m.material.color.set(0xffffff); if (m.material.emissive) m.material.emissive.set(0x553344); h.colored = true; } }
    if (btnMeshes.length) {
      h.ctrl.getWorldPosition(tmpV); h.ctrl.getWorldQuaternion(tmpQ); raycaster.set(tmpV, tmpD.set(0, 0, -1).applyQuaternion(tmpQ)); raycaster.far = 5;
      const hit = raycaster.intersectObjects(btnMeshes, false)[0]; setHover(h, hit && (ui || hit.object === wristBtn.p.mesh) ? hit.object.userData.btn : null);
      if (ui) { h.ray.visible = true; h.ray.scale.set(1, 1, hit ? hit.distance : 1.5); }
    }
    if (!ui && !h.held && !h.isHand) { const r = rayPick(h); h.ray.visible = !!r; h.ray.scale.set(1, 1, 1.2); }
    if (h.isHand && h.hand.joints && h.hand.joints['index-finger-tip']) {
      const tip = h.hand.joints['index-finger-tip'].getWorldPosition(V3()); let touched = null;
      for (const m of btnMeshes) { if (m === wristBtn.p.mesh && h.side === 'left') continue; const lp = m.worldToLocal(tip.clone()), b = m.userData.btn; if (Math.abs(lp.x) < b.w / 2 && Math.abs(lp.y) < b.h / 2 && Math.abs(lp.z) < 0.025) { touched = b; break; } }
      if (!touched && mode === 'play') { if (bell.getWorldPosition(V3()).add(V3(0, 0.05, 0)).distanceTo(tip) < 0.06) touched = 'bell'; for (const t of [...taps, ...(up.ice ? iceTaps : [])]) if (t.btn.getWorldPosition(V3()).distanceTo(tip) < 0.045) touched = t; }
      if (touched && h.poke !== touched) { if (touched === 'bell') ringBell(); else if (touched.slot) pressTap(touched); else clickBtn(touched, h); pulse(h, 0.4, 30); }
      h.poke = touched;
    }
  }
  if (!renderer.xr.isPresenting) {
    raycaster.setFromCamera(mouseNDC, camera); raycaster.far = 5;
    const hit = raycaster.intersectObjects(btnMeshes, false)[0]; setHover(mouseHand, hit ? hit.object.userData.btn : null);
    const p = mouseHolder.getWorldPosition(V3()); if (mouseHand.hasPrev) mouseHand.vel.lerp(tmpD.copy(p).sub(mouseHand.prev).divideScalar(Math.max(dt, 1e-3)), 0.5); mouseHand.prev.copy(p); mouseHand.hasPrev = true;
    // hiire suund liigutab hoidjat
    raycaster.setFromCamera(mouseNDC, camera); camera.getWorldQuaternion(tmpQ).invert(); tmpD.copy(raycaster.ray.direction).applyQuaternion(tmpQ); mouseHolder.position.copy(tmpD.multiplyScalar(0.45));
  }
  const L = hands.find(h => h.connected && h.isHand && h.side === 'left');
  wristG.visible = !!L && renderer.xr.isPresenting && (mode === 'play' || mode === 'pause');
  if (wristG.visible) { const w = L.hand.joints && L.hand.joints['wrist']; if (w) { w.getWorldPosition(tmpV); w.getWorldQuaternion(tmpQ); wristG.position.copy(tmpV).add(tmpD.set(0, 0.035, 0.05).applyQuaternion(tmpQ)); wristG.quaternion.copy(tmpQ); wristG.rotateX(-Math.PI / 2); } }
}
const btnPrev = {};
function readButtons(){ const s = renderer.xr.getSession(); if (!s) return; for (const src of s.inputSources) { const gp = src.gamepad; if (!gp) continue; const b = gp.buttons[5] && gp.buttons[5].pressed; if (b && !btnPrev[src.handedness]) togglePause(); btnPrev[src.handedness] = b; } }

/* =========================================================
   VR SESSIOON JA PÕHITSÜKKEL
   ========================================================= */
const info = document.getElementById('info');
renderer.xr.addEventListener('sessionstart', () => { info.style.display = 'none'; audioInit(); setTimeout(() => { if (mode === 'menu') placeInFront(menuG, 1.1, -0.15); else if (mode === 'end') placeInFront(endG, 1.15, -0.1); else if (mode === 'pause') placeInFront(pauseG, 0.8, -0.15); }, 400); const s = renderer.xr.getSession(); s.addEventListener('visibilitychange', () => { if (s.visibilityState !== 'visible' && mode === 'play') togglePause(); }); });
renderer.xr.addEventListener('sessionend', () => { info.style.display = ''; camera.position.set(0, 1.6, 0.2); rig.position.set(0, 0, 0); if (mode === 'play') togglePause(); });
setupKitchen(1.6); refreshMenu();
camera.rotation.set(deskPitch, 0, 0, 'YXZ'); placeInFront(menuG, 1.1, -0.15);
const clock = new THREE.Clock();
renderer.setAnimationLoop(() => {
  const dt = Math.min(clock.getDelta(), 0.05), T = clock.elapsedTime;
  readButtons();
  if (!renderer.xr.isPresenting) {
    if (keys.KeyA || keys.ArrowLeft) deskYaw += dt * 1.6; if (keys.KeyD || keys.ArrowRight) deskYaw -= dt * 1.6;
    if (keys.KeyW || keys.ArrowUp) deskPitch += dt * 1.2; if (keys.KeyS || keys.ArrowDown) deskPitch -= dt * 1.2;
     deskPitch = clamp(deskPitch, -1.1, 0.4); camera.rotation.set(deskPitch, deskYaw, 0, 'YXZ');
  }
  if (mode === 'play') {
    day.t += dt; music(); updateCooking(dt); updateCustomers(dt); updatePhysics(dt); drawHud();
    if (day.t >= diff().dayLen && !custs.some(c => c.state === 'wait')) endDay();
    else if (day.t >= diff().dayLen + 25) endDay();
  } else if (mode === 'end' || mode === 'menu') { updateCustomers(dt); updatePhysics(dt); }
  updateCat(dt);
  if (up.disco) { disco.rotation.y += dt * 0.8; discoLights.forEach((l, i) => { const a = T * 1.2 + i * 2.1; l.position.set(Math.cos(a) * 1.5, 2.3, -1.5 + Math.sin(a) * 1.2); l.intensity = mode === 'play' ? 3 + Math.sin(T * 6 + i) * 1.5 : 1.5; }); }
  updateHands(dt);
  for (const p of pops) if (p.p.mesh.visible) { p.t -= dt; p.p.mesh.position.y += dt * 0.3; p.p.mesh.material.opacity = clamp(p.t / 0.5, 0, 1); if (p.t <= 0) p.p.mesh.visible = false; }
  if (toast.mesh.visible) { toastT -= dt; toast.mesh.material.opacity = clamp(toastT / 0.4, 0, 1); if (toastT <= 0) toast.mesh.visible = false; }
  updateParticles(dt);
  pMat.uniforms.uScale.value = renderer.xr.isPresenting ? 900 : renderer.domElement.height / 2 * camera.projectionMatrix.elements[5];
  renderer.render(scene, camera);
});
window.__kk = { iceTaps, genOrder, startDay, endDay, goMenu, items, custs, mats, SLOTS, DISP, newItem, addToPlate, serve, trySnap, get day(){ return day; }, save, diff, spawnCustomer, K, putInSlot, pressTap, taps, up, applyUpgrades, mouseHand, grab, release };
</script>
</body>
</html>
