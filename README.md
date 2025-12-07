<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<title>AE86 Cup Test — Initial D G‑Meter</title>
<style>
  :root{
    --bg:#0b0d10; --fg:#e6fff4; --accent:#00ffa6; --warn:#ff3b30; --muted:#7aa390;
  }
  *{box-sizing:border-box}
  body{
    margin:0; font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, "Helvetica Neue", Arial, "Noto Sans", sans-serif;
    background: radial-gradient(1200px 1200px at 50% -20%, #0f1317, var(--bg));
    color:var(--fg);
  }
  header{
    text-align:center; padding:14px 12px 6px; letter-spacing:.5px; user-select:none;
  }
  header h1{
    margin:.1rem 0 0; font-weight:800; font-size:1.05rem; color:var(--accent);
    text-transform:uppercase;
  }
  header small{
    color:var(--muted); font-family:monospace; opacity:.9;
  }

  .wrap{max-width:760px; margin:0 auto; padding:8px 14px 24px;}

  /* Cup UI */
  .cup{
    position:relative; margin:16px auto; width:min(74vw,420px); aspect-ratio:1/1; border-radius:50%;
    border:2px solid color-mix(in oklab, var(--accent) 70%, #fff 0%);
    background: radial-gradient(closest-side, #0f1619, #0b0d10 60%);
    box-shadow: 0 0 0 1px rgba(0,255,166,.35) inset, 0 6px 30px rgba(0,0,0,.45);
    overflow:hidden;
  }
  .rim{position:absolute; inset:10px; border-radius:50%; border:1px dashed rgba(0,255,166,.25)}
  .drop{
    --r:16px;
    position:absolute; width:var(--r); height:var(--r);
    border-radius:50%;
    background: radial-gradient(circle at 35% 35%, #baffea, #4ff6c1 45%, #00b87e);
    left:50%; top:50%; transform:translate(-50%,-50%);
    box-shadow: 0 0 12px rgba(0,255,166,.45);
    will-change: transform;
  }
  .aura{
    position:absolute; inset:-6px; border-radius:50%;
    background: conic-gradient(from 0deg, transparent 0 45deg, rgba(0,255,166,.06) 45deg 135deg, transparent 135deg 225deg, rgba(0,255,166,.06) 225deg 315deg, transparent 315deg 360deg);
    filter:blur(8px); opacity:.4; pointer-events:none;
  }

  /* Readouts & controls */
  .grid{display:grid; grid-template-columns:1fr 1fr; gap:10px; align-items:end}
  .panel{
    padding:10px 12px; border:1px solid rgba(0,255,166,.18); border-radius:10px; background: #0c1115cc;
    backdrop-filter: blur(6px);
  }
  .row{display:flex; align-items:center; gap:10px; margin:8px 0;}
  label{font-size:.95rem}
  input[type="checkbox"]{transform:scale(1.1)}
  input[type="range"]{width:100%}
  .val{font-family:monospace; color:var(--accent)}
  .muted{color:var(--muted)}
  .btn{
    appearance:none; border:1px solid var(--accent); background:transparent; color:var(--accent);
    padding:10px 14px; border-radius:999px; font-weight:700; letter-spacing:.4px; cursor:pointer;
    transition:.15s transform, .15s opacity, .2s background;
  }
  .btn[disabled]{opacity:.4; cursor:not-allowed}
  .btn.primary{background:color-mix(in oklab, var(--accent) 15%, #000 85%);}
  .btn.warn{border-color:var(--warn); color:#ffdada}
  .stack{display:flex; gap:8px; flex-wrap:wrap}
  .kpi{
    display:grid; grid-template-columns:auto 1fr auto; gap:6px 10px; align-items:center;
    font-family:monospace; line-height:1.35;
  }
  .kpi div:nth-child(odd){color:var(--muted)}
  .badge{
    display:inline-flex; align-items:center; gap:6px; padding:6px 10px; border-radius:999px;
    border:1px solid rgba(255,255,255,.15); background:#0f151a80; font-family:monospace; font-size:.9rem;
  }

  /* Visual flash */
  .flash::before{
    content:"⚠️  Spill Alert — ease up, Takumi!"; position:fixed; inset:0; display:grid; place-items:center;
    background:rgba(255,59,48,.15); color:#ffdcdc; font-weight:900; font-size:clamp(18px, 5vw, 36px);
    text-shadow:0 0 10px #ff6b6b; z-index:10; animation:strobe .4s ease-in-out 1;
    pointer-events:none;
  }
  @keyframes strobe{0%,100%{background:rgba(255,59,48,.15)} 50%{background:rgba(255,59,48,.35)}}

  @media (max-width:720px){
    .grid{grid-template-columns:1fr}
  }

  /* Reduce motion preference */
  @media (prefers-reduced-motion: reduce){
    .flash::before{animation:none}
    .drop{transition:none}
  }
</style>
</head>
<body>
  <header>
    <small>Fujiwara Tofu Shop (Private Use Only)</small>
    <h1>AE86 Cup Test</h1>
  </header>

  <div class="wrap">
    <div class="cup">
      <div class="aura"></div>
      <div class="rim"></div>
      <div id="drop" class="drop" aria-hidden="true"></div>
    </div>

    <div class="grid">
      <section class="panel">
        <div class="kpi" style="margin-bottom:6px">
          <div>Current G</div><div class="muted">lateral/longitudinal (plane)</div><div class="val" id="gCur">0.00 g</div>
          <div>Peak G</div><div class="muted">since start</div><div class="val" id="gPeak">0.00 g</div>
          <div>Jerk</div><div class="muted">rate of change</div><div class="val" id="jerk">0.00 m/s³</div>
        </div>
        <div class="row">
          <label for="threshold">Threshold: <span class="val" id="thVal">0.20 g</span></label>
        </div>
        <input id="threshold" type="range" min="0.05" max="0.80" step="0.01" value="0.20" aria-label="G threshold"/>
        <div class="row stack">
          <label class="badge"><input id="audioOn" type="checkbox" checked /> Audio</label>
          <label class="badge"><input id="visualOn" type="checkbox" checked /> Visual</label>
          <label class="badge"><input id="vibrateOn" type="checkbox" /> Vibrate</label>
          <label class="badge"><input id="hudOn" type="checkbox" /> HUD (mirror)</label>
        </div>
      </section>

      <section class="panel">
        <div class="stack">
          <button id="startBtn" class="btn primary">▶ Start</button>
          <button id="stopBtn" class="btn" disabled>⏸ Stop</button>
          <button id="testBeep" class="btn">🔊 Test</button>
          <button id="resetPeak" class="btn">↺ Reset Peak</button>
          <button id="aboutBtn" class="btn">ℹ About</button>
        </div>
        <p class="muted" style="margin-top:10px">
          Mount phone securely. Configure while parked. Don’t stare at the screen—drive smooth like <strong>Takumi</strong>.
        </p>
      </section>
    </div>
  </div>

<script>
(() => {
  const G = 9.80665;
  const el = {
    gCur: document.getElementById('gCur'),
    gPeak: document.getElementById('gPeak'),
    jerk: document.getElementById('jerk'),
    th: document.getElementById('threshold'),
    thVal: document.getElementById('thVal'),
    audioOn: document.getElementById('audioOn'),
    visualOn: document.getElementById('visualOn'),
    vibrateOn: document.getElementById('vibrateOn'),
    hudOn: document.getElementById('hudOn'),
    start: document.getElementById('startBtn'),
    stop: document.getElementById('stopBtn'),
    test: document.getElementById('testBeep'),
    resetPeak: document.getElementById('resetPeak'),
    about: document.getElementById('aboutBtn'),
    drop: document.getElementById('drop')
  };

  // Persist settings
  const store = {
    get k(){ try{return JSON.parse(localStorage.ae86||'{}')}catch{ return {} } },
    set k(v){ localStorage.ae86 = JSON.stringify(v) }
  };
  const cfg = Object.assign({th:0.20, audio:true, visual:true, vibrate:false, hud:false}, store.k);
  el.th.value = cfg.th;
  el.thVal.textContent = (+cfg.th).toFixed(2) + ' g';
  el.audioOn.checked = cfg.audio; el.visualOn.checked = cfg.visual; el.vibrateOn.checked = cfg.vibrate; el.hudOn.checked = cfg.hud;
  if (cfg.hud) document.body.style.transform = 'scaleX(-1)';

  // Audio
  let audioCtx = null;
  function ensureAudio(){
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
    return audioCtx;
  }
  function beep({dur=120, freq=880, vol=0.15}={}){
    if (!el.audioOn.checked) return;
    try{
      const ac = ensureAudio();
      const o = ac.createOscillator();
      const g = ac.createGain();
      o.type='square'; o.frequency.value=freq;
      o.connect(g); g.connect(ac.destination);
      const t = ac.currentTime;
      g.gain.setValueAtTime(0, t);
      g.gain.linearRampToValueAtTime(vol, t+0.01);
      g.gain.exponentialRampToValueAtTime(0.0001, t + dur/1000);
      o.start(t); o.stop(t + dur/1000 + 0.02);
    }catch(e){}
  }

  // WakeLock (best-effort)
  let wakeLock = null;
  async function lockScreen(){
    try{ wakeLock = await navigator.wakeLock?.request('screen');
      wakeLock?.addEventListener('release',()=>{});
    }catch(e){}
  }
  async function releaseScreen(){ try{ await wakeLock?.release(); wakeLock=null }catch(e){} }

  // Sensor processing
  let running=false, lastT=null, gPeak=0;
  let grav=[0,0,0]; // low-pass gravity estimate
  let lin=[0,0,0];  // linear acceleration (m/s^2)
  let ema=[0,0,0];  // smoothed linear accel for UI
  const TAU_GRAV = 0.5; // seconds; larger -> slower gravity tracking
  const TAU_SMOOTH = 0.08; // seconds; UI smoothing
  let lastMag = 0, lastAlert = 0;
  const ALERT_COOLDOWN = 800; // ms
  const rimPx = () => document.querySelector('.cup').clientWidth * 0.5 - 20; // approx usable radius for drop

  function onMotion(ev){
    const now = ev.timeStamp ? ev.timeStamp/1000 : performance.now()/1000;
    const dt = Math.min(0.1, Math.max(0.008, lastT ? (now - lastT) : 1/60));
    lastT = now;

    // Prefer acceleration (already gravity-compensated). Fallback: estimate via LPF.
    const ai = ev.accelerationIncludingGravity || ev.acceleration || {x:0,y:0,z:0};
    const a = ev.acceleration || {x: null, y: null, z: null};

    if (a.x !== null){ lin[0]=a.x; lin[1]=a.y; lin[2]=a.z; }
    else {
      const alpha = TAU_GRAV/(TAU_GRAV+dt); // LPF for gravity
      grav[0] = alpha*grav[0] + (1-alpha)*(ai.x||0);
      grav[1] = alpha*grav[1] + (1-alpha)*(ai.y||0);
      grav[2] = alpha*grav[2] + (1-alpha)*(ai.z||0);
      lin[0] = (ai.x||0) - grav[0];
      lin[1] = (ai.y||0) - grav[1];
      lin[2] = (ai.z||0) - grav[2];
    }

    // Smooth for UI display (EMA)
    const beta = TAU_SMOOTH/(TAU_SMOOTH+dt);
    ema[0] = beta*ema[0] + (1-beta)*lin[0];
    ema[1] = beta*ema[1] + (1-beta)*lin[1];
    ema[2] = beta*ema[2] + (1-beta)*lin[2];

    // Use phone plane magnitude (x/y) — orientation‑agnostic for lateral/longitudinal
    const mag = Math.hypot(ema[0], ema[1]); // m/s^2
    const gPlane = mag / G;
    if (gPlane > gPeak){ gPeak = gPlane; el.gPeak.textContent = gPeak.toFixed(2) + ' g'; }

    // Jerk (rate of change)
    const jerk = (mag - lastMag) / dt; lastMag = mag;

    // UI: move the "water drop" opposite accel vector
    const R = rimPx();
    const k = 0.9; // sensitivity for visual displacement
    const dx = -k * ema[0] / G * R;
    const dy = -k * ema[1] / G * R;
    el.drop.style.transform = `translate(calc(-50% + ${dx}px), calc(-50% + ${dy}px))`;

    // Readouts
    el.gCur.textContent = gPlane.toFixed(2) + ' g';
    el.jerk.textContent = jerk.toFixed(2) + ' m/s³';

    // Threshold + alert with cooldown + slight hysteresis
    const th = +el.th.value;
    const over = gPlane >= th;
    const timeOK = (performance.now() - lastAlert) > ALERT_COOLDOWN;
    if (over && timeOK){
      lastAlert = performance.now();
      if (el.visualOn.checked){ document.body.classList.add('flash'); setTimeout(()=>document.body.classList.remove('flash'), 350); }
      if (el.vibrateOn.checked && navigator.vibrate){ navigator.vibrate(100); }
      beep({dur:120, freq:880});
    }
  }

  // Generic Sensor API fallback (if available)
  let linearSensor = null;
  function startGenericSensor(){
    try{
      if ('LinearAccelerationSensor' in window){
        linearSensor = new LinearAccelerationSensor({frequency:60});
        linearSensor.addEventListener('reading', () => {
          onMotion({
            acceleration: {x:linearSensor.x, y:linearSensor.y, z:linearSensor.z},
            accelerationIncludingGravity: null,
            timeStamp: performance.now()
          });
        });
        linearSensor.start();
        return true;
      }
    }catch(e){}
    return false;
  }

  // Start/Stop
  function start(){
    if (running) return;
    running = true; gPeak = 0; lastT=null; lastMag=0;
    el.gPeak.textContent = '0.00 g';
    lastAlert = 0;
    // Permissions (mainly affects iOS, but harmless elsewhere)
    if (typeof DeviceMotionEvent !== 'undefined' && typeof DeviceMotionEvent.requestPermission === 'function'){
      DeviceMotionEvent.requestPermission().catch(()=>{}).finally(bindMotion);
    } else { bindMotion(); }
    lockScreen();
    el.start.disabled = true; el.stop.disabled = false; el.start.classList.remove('primary');
  }
  function bindMotion(){
    window.addEventListener('devicemotion', onMotion, {passive:true});
    // If no events arrive, try Generic Sensor after 1s
    let got = false;
    const tmp = (e)=>{ got=true; window.removeEventListener('devicemotion', tmp, {passive:true}); };
    window.addEventListener('devicemotion', tmp, {passive:true});
    setTimeout(()=>{ if(!got) startGenericSensor(); }, 1000);
  }
  function stop(){
    if (!running) return;
    running = false;
    window.removeEventListener('devicemotion', onMotion, {passive:true});
    try{ linearSensor?.stop(); }catch(e){}
    linearSensor = null;
    releaseScreen();
    el.start.disabled = false; el.stop.disabled = true; el.start.classList.add('primary');
  }

  // UI bindings
  el.th.addEventListener('input', ()=>{ el.thVal.textContent = (+el.th.value).toFixed(2) + ' g'; saveCfg(); });
  el.audioOn.addEventListener('change', saveCfg);
  el.visualOn.addEventListener('change', saveCfg);
  el.vibrateOn.addEventListener('change', saveCfg);
  el.hudOn.addEventListener('change', ()=>{ document.body.style.transform = el.hudOn.checked ? 'scaleX(-1)' : ''; saveCfg(); });

  function saveCfg(){
    store.k = {
      th:+el.th.value, audio:el.audioOn.checked, visual:el.visualOn.checked,
      vibrate:el.vibrateOn.checked, hud:el.hudOn.checked
    };
  }

  el.start.addEventListener('click', () => { ensureAudio(); start(); });
  el.stop.addEventListener('click', stop);
  el.test.addEventListener('click', ()=>{ ensureAudio(); beep({dur:140, freq:920}); });
  el.resetPeak.addEventListener('click', ()=>{ gPeak=0; el.gPeak.textContent='0.00 g'; });
  el.about.addEventListener('click', ()=>{
    alert(
`AE86 Cup Test — Initial D inspired "no‑spill" trainer.

• Flash/beep when plane‑G exceeds your threshold.
• The dot moves opposite acceleration (water inertia).
• Tip: start around 0.20 g, then tighten.

Safety:
• Set it up while parked. Mount your phone.
• Don’t look at the screen while driving.
• Use at your own risk. Drive smooth, not fast.`
    );
  });

  // Visibility cleanup
  document.addEventListener('visibilitychange', ()=>{ if (document.hidden) releaseScreen(); else if (running) lockScreen(); });

})();
</script>
</body>
</html>
