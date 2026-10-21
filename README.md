<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SARS-CoV-2 — интерактивный портал</title>
<style>
  :root {
    --primary: #0b3d91; --primary-light: #e3eaf6; --bg: #f4f7fb; --card: #fff;
    --text: #1a1a1a; --muted: #5a6a7a; --border: #d8e0ea;
    --danger: #c0392b; --success: #1e8449;
  }
  * { box-sizing: border-box; }
  body { margin: 0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    background: var(--bg); color: var(--text); line-height: 1.6; }
  header { background: var(--primary); color: #fff; padding: 1rem 2rem;
    display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: .5rem; }
  header h1 { margin: 0; font-size: 1.2rem; }
  header h1 a { color: #fff; text-decoration: none; }
  nav { display: flex; gap: .3rem; flex-wrap: wrap; }
  nav button { background: transparent; border: 1px solid rgba(255,255,255,.35); color: #fff;
    padding: .4rem .9rem; border-radius: 6px; cursor: pointer; font-size: .85rem; }
  nav button:hover, nav button.active { background: rgba(255,255,255,.2); }
  main { max-width: 1200px; margin: 0 auto; padding: 1.2rem; }
  section.page { display: none; } section.page.active { display: block; }
  h2 { color: var(--primary); margin-top: 0; } h3 { color: var(--primary); }
  .card { background: var(--card); border-radius: 12px; padding: 1.2rem 1.4rem;
    box-shadow: 0 2px 10px rgba(0,0,0,.06); margin-bottom: 1rem; }
  .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  @media (max-width: 800px) { .grid2 { grid-template-columns: 1fr; } }
  .badge { display: inline-block; background: var(--primary-light); color: var(--primary);
    padding: .15rem .6rem; border-radius: 20px; font-size: .78rem; margin: .1rem .2rem .1rem 0; }
  table { width: 100%; border-collapse: collapse; background: #fff; margin: .8rem 0; }
  th, td { border: 1px solid var(--border); padding: .45rem .6rem; text-align: left; font-size: .88rem; }
  th { background: var(--primary-light); color: var(--primary); }
  .btn { display: inline-block; padding: .5rem 1.1rem; background: var(--primary); color: #fff;
    border: none; border-radius: 8px; cursor: pointer; font-size: .92rem; margin: .2rem .3rem .2rem 0; }
  .btn:hover { background: #09306f; }
  .btn.secondary { background: var(--primary-light); color: var(--primary); }
  .btn.secondary:hover { background: #d0dcf0; }
  .hidden { display: none !important; }
  .muted { color: var(--muted); font-size: .88rem; }

  /* ===== CSS 3D вирион ===== */
  .scene3d { perspective: 1200px; width: 100%; height: 460px;
    background: radial-gradient(circle at 50% 45%, #1a1a2e 0%, #08080f 100%);
    border-radius: 16px; overflow: hidden; position: relative;
    display: flex; align-items: center; justify-content: center; cursor: grab; }
  .scene3d:active { cursor: grabbing; }
  .world { width: 0; height: 0; transform-style: preserve-3d; position: relative;
    transition: transform .05s linear; }
  .particle { position: absolute; transform-style: preserve-3d; }

  /* Оболочка: полый шар из колец */
  .shell-ring { position: absolute; width: 280px; height: 280px; margin: -140px 0 0 -140px;
    border-radius: 50%; border: 2px solid rgba(155,89,182,.5);
    transform-style: preserve-3d; }
  .shell-fill { position: absolute; width: 280px; height: 280px; margin: -140px 0 0 -140px;
    border-radius: 50%; background: radial-gradient(circle at 40% 35%, rgba(74,44,106,.7), rgba(26,15,45,.85));
    box-shadow: inset -20px -20px 60px rgba(0,0,0,.6), inset 20px 20px 40px rgba(155,89,182,.2); }

  /* Шип S — стержень + головка, направлен наружу */
  .spike { position: absolute; transform-style: preserve-3d; }
  .spike-stalk { position: absolute; width: 8px; height: 50px; left: -4px; top: 0;
    background: linear-gradient(180deg, #ff6b6b, #c0392b); border-radius: 4px; }
  .spike-head { position: absolute; width: 22px; height: 22px; left: -11px; top: -14px;
    border-radius: 50%; background: radial-gradient(circle at 35% 30%, #ff9b9b, #e74c3c);
    box-shadow: 0 0 12px rgba(231,76,60,.7); }

  /* M-белок */
  .m-prot { position: absolute; width: 14px; height: 14px; left: -7px; top: -7px;
    border-radius: 50%; background: radial-gradient(circle at 35% 30%, #f5b041, #d68910);
    box-shadow: 0 0 8px rgba(243,156,18,.6); }
  /* E-белок */
  .e-prot { position: absolute; width: 9px; height: 9px; left: -4.5px; top: -4.5px;
    border-radius: 50%; background: #b07cc6; box-shadow: 0 0 6px rgba(176,124,198,.7); }
  /* РНК */
  .rna-seg { position: absolute; width: 10px; height: 10px; left: -5px; top: -5px;
    border-radius: 50%; background: radial-gradient(circle, #2ecc71, #1e8449);
    box-shadow: 0 0 8px rgba(46,204,113,.8); }
  /* N-белок */
  .n-prot { position: absolute; width: 7px; height: 7px; left: -3.5px; top: -3.5px;
    border-radius: 50%; background: #58d68d; }

  /* Подписи-кнопки слоёв */
  .layer-toggles { display: flex; flex-wrap: wrap; gap: .5rem; margin-top: 1rem; }
  .layer-toggles label { display: flex; align-items: center; gap: .4rem; background: #fff;
    padding: .45rem .9rem; border-radius: 8px; border: 1px solid var(--border);
    cursor: pointer; font-size: .88rem; user-select: none; }
  .layer-toggles input { accent-color: var(--primary); }

  /* ===== Заражение ===== */
  .infection-scene { position: relative; height: 420px; border-radius: 16px; overflow: hidden;
    background: radial-gradient(circle at 70% 50%, #0f2238 0%, #060b14 100%); }
  /* Клетка */
  .cell { position: absolute; right: -80px; top: 50%; transform: translateY(-50%);
    width: 420px; height: 420px; border-radius: 50%;
    background: radial-gradient(circle at 40% 40%, rgba(46,134,193,.35), rgba(15,34,56,.9));
    border: 3px solid #2e86c1; box-shadow: 0 0 60px rgba(46,134,193,.3) inset; }
  /* Рецепторы ACE2 */
  .ace2 { position: absolute; width: 10px; height: 26px; background: #f1c40f;
    border-radius: 3px; box-shadow: 0 0 8px rgba(241,196,15,.7); }
  /* Вирус на сцене заражения */
  .inf-virus { position: absolute; width: 90px; height: 90px; border-radius: 50%;
    background: radial-gradient(circle at 35% 30%, #3d566e, #1a252f);
    box-shadow: 0 0 20px rgba(231,76,60,.4); transition: all .8s cubic-bezier(.4,0,.2,1); z-index: 5; }
  .inf-virus .inf-spike { position: absolute; width: 4px; height: 22px; background: #e74c3c;
    left: 50%; top: 50%; transform-origin: 2px 0; border-radius: 2px; }
  .inf-virus .inf-head { position: absolute; width: 10px; height: 10px; border-radius: 50%;
    background: #ff6b6b; left: 50%; top: 50%; transform-origin: 5px 0; }
  /* РНК и новые вирионы */
  .inf-rna { position: absolute; width: 12px; height: 12px; border-radius: 50%;
    background: #27ae60; box-shadow: 0 0 10px #2ecc71; opacity: 0; transition: opacity .6s; }
  .inf-new { position: absolute; width: 40px; height: 40px; border-radius: 50%;
    background: radial-gradient(circle at 35% 30%, #3d566e, #1a252f);
    border: 2px solid #e74c3c; opacity: 0; transition: all .6s; }
  /* Этапные подсказки */
  .stage-flash { position: absolute; padding: .5rem .9rem; background: rgba(241,196,15,.15);
    border: 1px solid #f1c40f; color: #f1c40f; border-radius: 8px; font-size: .82rem;
    font-weight: 600; opacity: 0; transition: opacity .4s; pointer-events: none; }
  .stage-flash.show { opacity: 1; }

  .progress-dots { display: flex; gap: .35rem; margin: .8rem 0; flex-wrap: wrap; }
  .progress-dots span { width: 30px; height: 30px; border-radius: 50%; background: #d8e0ea;
    display: flex; align-items: center; justify-content: center; font-size: .75rem;
    cursor: pointer; transition: all .2s; }
  .progress-dots span.active { background: var(--primary); color: #fff; }
  .progress-dots span.done { background: var(--success); color: #fff; }
  .step-info { background: var(--primary-light); border-left: 4px solid var(--primary);
    padding: .8rem 1rem; border-radius: 0 8px 8px 0; margin-top: .5rem; }
  .step-info h4 { margin: 0 0 .3rem; color: var(--primary); }

  /* ===== Карта ===== */
  .map-wrap { position: relative; background: #0b1a2e; border-radius: 16px; overflow: hidden; }
  .map-svg { width: 100%; height: auto; display: block; }
  .map-tooltip { position: absolute; background: #fff; border-radius: 8px; padding: .5rem .8rem;
    box-shadow: 0 4px 16px rgba(0,0,0,.3); font-size: .82rem; pointer-events: none;
    max-width: 230px; z-index: 10; display: none; line-height: 1.4; }
  .map-legend { display: flex; gap: 1rem; flex-wrap: wrap; margin-top: .8rem; font-size: .82rem; }
  .map-legend span { display: flex; align-items: center; gap: .3rem; }
  .map-legend i { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }

  /* ===== Тест ===== */
  .level-tabs { display: flex; gap: .5rem; margin-bottom: 1rem; flex-wrap: wrap; }
  .level-tabs button { padding: .5rem 1.2rem; border-radius: 8px; border: 1px solid var(--border);
    background: #fff; cursor: pointer; font-size: .92rem; }
  .level-tabs button.active { background: var(--primary); color: #fff; border-color: var(--primary); }
  .quiz-q { margin-bottom: 1rem; } .quiz-q p { margin: 0 0 .4rem; font-weight: 600; }
  .quiz-q label { display: block; padding: .3rem .5rem; border-radius: 6px; cursor: pointer; }
  .quiz-q label:hover { background: var(--primary-light); }
  .quiz-q input { margin-right: .5rem; }
  .quiz-result { margin-top: 1rem; padding: .8rem; border-radius: 8px; font-weight: 600; }
  .quiz-result.good { background: #d4edda; color: var(--success); }
  .quiz-result.bad { background: #f8d7da; color: var(--danger); }
  footer { text-align: center; padding: 1.2rem; color: var(--muted); font-size: .82rem; }
</style>
</head>
<body>

<header>
  <h1><a href="#" onclick="showPage('home');return false;">🧬 SARS-CoV-2 — портал</a></h1>
  <nav>
    <button data-page="home" class="active" onclick="showPage('home')">Главная</button>
    <button data-page="structure" onclick="showPage('structure')">Строение</button>
    <button data-page="infection" onclick="showPage('infection')">Заражение</button>
    <button data-page="map" onclick="showPage('map')">Карта</button>
    <button data-page="quiz" onclick="showPage('quiz')">Тест</button>
  </nav>
</header>

<main>

<!-- ГЛАВНАЯ -->
<section id="page-home" class="page active">
  <div class="card">
    <h2>SARS-CoV-2 — возбудитель COVID-19</h2>
    <p>
      <b>Семейство:</b> Coronaviridae • <b>Род:</b> Betacoronavirus •
      <b>Геном:</b> (+)РНК ~29.9 т.п.н. • <b>Диаметр:</b> 80–200 нм
    </p>
    <p>
      Оболочечный вирус с шиповидными белками S, формирующими «корону».
      S-белок связывается с <b>ACE2</b>, затем протеаза <b>TMPRSS2</b>
      разрезает его и запускает слияние мембран.
    </p>
    <p>
      <span class="badge">Betacoronavirus</span> <span class="badge">ssRNA(+)</span>
      <span class="badge">ACE2</span> <span class="badge">TMPRSS2</span>
      <span class="badge">80–200 нм</span> <span class="badge">R0 2.0–3.8</span>
    </p>
  </div>

  <div class="grid2">
    <div class="card">
      <h3>Структурные белки</h3>
      <table>
        <tr><th>Белок</th><th>Функция</th></tr>
        <tr><td><b>S</b></td><td>Связывание ACE2, вход</td></tr>
        <tr><td><b>N</b></td><td>Упаковка РНК</td></tr>
        <tr><td><b>M</b></td><td>Оболочка, сборка</td></tr>
        <tr><td><b>E</b></td><td>Ионный канал</td></tr>
      </table>
    </div>
    <div class="card">
      <h3>Ключевые факты</h3>
      <ul style="margin:.3rem 0; padding-left:1.2rem;">
        <li>16 неструктурных белков (nsp1–16), включая RdRp и 3CLpro</li>
        <li>Инкубация 2–14 дней, в среднем 5–7</li>
        <li>Передача: капельная, аэрозольная, контактная</li>
        <li>Мутации: D614G, N501Y, E484K</li>
        <li>Поражает лёгкие, кишечник, почки, сердце</li>
      </ul>
    </div>
  </div>
</section>

<!-- СТРОЕНИЕ (CSS 3D) -->
<section id="page-structure" class="page">
  <h2>🔬 Строение вириона (3D)</h2>
  <p class="muted">Потяните мышью — вращение. Отключайте слои чекбоксами.</p>

  <div class="scene3d" id="scene3d">
    <div class="world" id="world"></div>
  </div>

  <div class="layer-toggles" id="layer-toggles">
    <label><input type="checkbox" checked data-layer="spike"> 🔴 S-белки</label>
    <label><input type="checkbox" checked data-layer="envelope"> 🟣 Оболочка</label>
    <label><input type="checkbox" checked data-layer="membrane"> 🟠 M-белки</label>
    <label><input type="checkbox" checked data-layer="e"> 🟪 E-белки</label>
    <label><input type="checkbox" checked data-layer="rna"> 🟢 РНК + N</label>
  </div>
</section>

<!-- ЗАРАЖЕНИЕ -->
<section id="page-infection" class="page">
  <h2>🦠 Заражение клетки</h2>
  <p class="muted">Нажимайте «Далее» — вирус будет двигаться по этапам.</p>

  <div class="infection-scene" id="infection-scene">
    <div class="cell" id="cell"></div>
    <div class="stage-flash" id="stage-flash"></div>
    <!-- вирус создаётся скриптом -->
  </div>

  <div class="progress-dots" id="infection-dots"></div>
  <div class="step-info" id="infection-info"></div>

  <div style="margin-top:.6rem;">
    <button class="btn secondary" onclick="prevStep()">← Назад</button>
    <button class="btn" onclick="nextStep()">Далее →</button>
    <button class="btn secondary" onclick="resetSteps()">Сброс</button>
  </div>
</section>

<!-- КАРТА -->
<section id="page-map" class="page">
  <h2>🌍 Карта вспышек (2020)</h2>
  <p class="muted">Наведите на точку.</p>

  <div class="map-wrap" id="map-wrap">
    <svg class="map-svg" viewBox="0 0 900 480" xmlns="http://www.w3.org/2000/svg">
      <rect width="900" height="480" fill="#0b1a2e"/>
      <g fill="#1c3a5c" opacity="0.8">
        <path d="M110 130 Q170 80 250 95 Q300 115 280 170 Q250 220 200 235 Q140 220 110 180 Z"/>
        <path d="M195 260 Q240 245 265 290 Q275 345 240 395 Q205 415 185 380 Q165 330 175 290 Z"/>
        <path d="M395 85 Q440 65 485 78 Q510 100 490 135 Q455 155 420 140 Q390 115 395 85 Z"/>
        <path d="M400 165 Q455 145 505 165 Q530 220 505 295 Q470 345 425 330 Q385 295 375 235 Q375 195 400 165 Z"/>
        <path d="M505 65 Q610 40 730 75 Q810 105 825 170 Q800 225 735 240 Q675 250 615 215 Q555 195 515 145 Q485 105 505 65 Z"/>
        <path d="M700 330 Q755 305 805 328 Q825 360 790 395 Q745 405 705 380 Q685 355 700 330 Z"/>
      </g>
      <g id="map-points"></g>
    </svg>
    <div class="map-tooltip" id="map-tooltip"></div>
  </div>

  <div class="map-legend">
    <span><i style="background:#e74c3c"></i> Дек 2019 — Янв 2020</span>
    <span><i style="background:#e67e22"></i> Фев 2020</span>
    <span><i style="background:#f1c40f"></i> Мар 2020</span>
    <span><i style="background:#2ecc71"></i> Апр–Май 2020</span>
  </div>

  <div class="card" style="margin-top:1rem;">
    <h3>Хронология</h3>
    <ul id="event-list" style="padding-left:1.2rem;"></ul>
  </div>
</section>

<!-- ТЕСТ -->
<section id="page-quiz" class="page">
  <h2>📝 Тест</h2>
  <div class="level-tabs" id="level-tabs">
    <button data-level="basic" class="active" onclick="setLevel('basic')">🟢 Базовый</button>
    <button data-level="medium" onclick="setLevel('medium')">🟡 Средний</button>
    <button data-level="hard" onclick="setLevel('hard')">🔴 Продвинутый</button>
  </div>
  <div class="card">
    <div id="quiz-container"></div>
    <button class="btn" onclick="checkQuiz()">Проверить</button>
    <div id="quiz-result" class="quiz-result hidden"></div>
  </div>
</section>

</main>
<footer>Учебно-научный прототип • без внешних зависимостей</footer>

<script>
/* ============================================================
   НАВИГАЦИЯ
   ============================================================ */
const PAGES = ["home","structure","infection","map","quiz"];
let structInit = false, infInit = false, mapInit = false;

function showPage(name) {
  PAGES.forEach(p => document.getElementById("page-" + p).classList.toggle("active", p === name));
  document.querySelectorAll("nav button").forEach(b => b.classList.toggle("active", b.dataset.page === name));
  window.scrollTo({ top: 0, behavior: "smooth" });
  if (name === "structure" && !structInit) { buildVirion(); structInit = true; }
  if (name === "infection" && !infInit) { buildInfection(); infInit = true; }
  if (name === "map" && !mapInit) { buildMap(); mapInit = true; }
  if (name === "quiz") renderQuiz();
}

/* ============================================================
   CSS 3D ВИРИОН
   ============================================================ */
const world = document.getElementById("world");
let rotX = -15, rotY = 0, dragging = false, lastX = 0, lastY = 0;
const LAYERS = { spike: [], envelope: [], membrane: [], e: [], rna: [] };

function buildVirion() {
  world.innerHTML = "";

  // --- Оболочка: сфера из колец ---
  for (let i = 0; i < 8; i++) {
    const ring = document.createElement("div");
    ring.className = "shell-ring";
    ring.style.transform = `rotateY(${i * 22.5}deg) rotateX(90deg)`;
    world.appendChild(ring);
    LAYERS.envelope.push(ring);
  }
  // Заливка оболочки
  const fill = document.createElement("div");
  fill.className = "shell-fill";
  world.appendChild(fill);
  LAYERS.envelope.push(fill);

  // --- S-белки (шипы) по сфере ---
  const spikeDirs = [];
  const nSpikes = 26;
  for (let i = 0; i < nSpikes; i++) {
    const phi = Math.acos(1 - 2 * (i + 0.5) / nSpikes); // равномерно по сфере
    const theta = Math.PI * (1 + Math.sqrt(5)) * i;
    const x = Math.sin(phi) * Math.cos(theta);
    const y = Math.cos(phi);
    const z = Math.sin(phi) * Math.sin(theta);
    spikeDirs.push({ x, y, z });
  }
  spikeDirs.forEach(d => {
    const spike = document.createElement("div");
    spike.className = "spike";
    // Позиция на поверхности радиуса 140
    spike.style.transform = `translate3d(${d.x*140}px, ${d.y*140}px, ${d.z*140}px) rotateY(${Math.atan2(d.x, d.z)*180/Math.PI}deg) rotateX(${-Math.asin(d.y)*180/Math.PI}deg)`;
    const stalk = document.createElement("div");
    stalk.className = "spike-stalk";
    const head = document.createElement("div");
    head.className = "spike-head";
    spike.appendChild(stalk);
    spike.appendChild(head);
    world.appendChild(spike);
    LAYERS.spike.push(spike);
  });

  // --- M-белки ---
  for (let i = 0; i < 30; i++) {
    const phi = Math.acos(1 - 2 * Math.random());
    const theta = Math.random() * Math.PI * 2;
    const r = 138;
    const x = r * Math.sin(phi) * Math.cos(theta);
    const y = r * Math.cos(phi);
    const z = r * Math.sin(phi) * Math.sin(theta);
    const m = document.createElement("div");
    m.className = "m-prot";
    m.style.transform = `translate3d(${x}px, ${y}px, ${z}px)`;
    world.appendChild(m);
    LAYERS.membrane.push(m);
  }

  // --- E-белки ---
  for (let i = 0; i < 18; i++) {
    const phi = Math.acos(1 - 2 * Math.random());
    const theta = Math.random() * Math.PI * 2;
    const r = 136;
    const x = r * Math.sin(phi) * Math.cos(theta);
    const y = r * Math.cos(phi);
    const z = r * Math.sin(phi) * Math.sin(theta);
    const e = document.createElement("div");
    e.className = "e-prot";
    e.style.transform = `translate3d(${x}px, ${y}px, ${z}px)`;
    world.appendChild(e);
    LAYERS.e.push(e);
  }

  // --- РНК: спираль внутри ---
  for (let t = 0; t < 40; t++) {
    const angle = t * 0.6;
    const radius = 40 + Math.sin(t * 0.3) * 20;
    const x = Math.cos(angle) * radius;
    const y = Math.sin(t * 0.4) * 40;
    const z = Math.sin(angle) * radius;
    const seg = document.createElement("div");
    seg.className = "rna-seg";
    seg.style.transform = `translate3d(${x}px, ${y}px, ${z}px)`;
    world.appendChild(seg);
    LAYERS.rna.push(seg);
  }
  // N-белки на РНК
  for (let t = 0; t < 20; t++) {
    const angle = t * 0.8;
    const radius = 40 + Math.sin(t * 0.4) * 15;
    const x = Math.cos(angle) * radius;
    const y = Math.sin(t * 0.5) * 35;
    const z = Math.sin(angle) * radius;
    const n = document.createElement("div");
    n.className = "n-prot";
    n.style.transform = `translate3d(${x}px, ${y}px, ${z}px)`;
    world.appendChild(n);
    LAYERS.rna.push(n);
  }

  updateWorldTransform();
  animateRotation();

  // Управление мышью
  const scene = document.getElementById("scene3d");
  scene.addEventListener("mousedown", e => { dragging = true; lastX = e.clientX; lastY = e.clientY; });
  window.addEventListener("mouseup", () => dragging = false);
  window.addEventListener("mousemove", e => {
    if (!dragging) return;
    rotY += (e.clientX - lastX) * 0.5;
    rotX -= (e.clientY - lastY) * 0.5;
    rotX = Math.max(-90, Math.min(90, rotX));
    lastX = e.clientX; lastY = e.clientY;
    updateWorldTransform();
  });
  // Тач
  scene.addEventListener("touchstart", e => { const t = e.touches[0]; dragging = true; lastX = t.clientX; lastY = t.clientY; });
  scene.addEventListener("touchend", () => dragging = false);
  scene.addEventListener("touchmove", e => {
    if (!dragging) return;
    const t = e.touches[0];
    rotY += (t.clientX - lastX) * 0.5;
    rotX -= (t.clientY - lastY) * 0.5;
    lastX = t.clientX; lastY = t.clientY;
    updateWorldTransform();
    e.preventDefault();
  }, { passive: false });

  // Чекбоксы
  document.querySelectorAll("#layer-toggles input").forEach(cb => {
    cb.addEventListener("change", () => {
      LAYERS[cb.dataset.layer].forEach(el => el.style.display = cb.checked ? "" : "none");
    });
  });
}

function updateWorldTransform() {
  world.style.transform = `rotateX(${rotX}deg) rotateY(${rotY}deg)`;
}

let autoRotate = true;
function animateRotation() {
  if (autoRotate && !dragging) {
    rotY += 0.25;
    updateWorldTransform();
  }
  requestAnimationFrame(animateRotation);
}

/* ============================================================
   ЗАРАЖЕНИЕ
   ============================================================ */
const STEPS = [
  { t: "Шаг 1. Приближение", d: "Вирус приближается к клетке эпителия дыхательных путей." },
  { t: "Шаг 2. Связывание с ACE2", d: "S-белок связывается с рецептором ACE2." },
  { t: "Шаг 3. TMPRSS2", d: "Протеаза хозяина разрезает S-белок, активируя слияние." },
  { t: "Шаг 4. Слияние мембран", d: "Оболочка вируса сливается с мембраной клетки." },
  { t: "Шаг 5. РНК внутри", d: "(+)РНК высвобождается в цитоплазму." },
  { t: "Шаг 6. Трансляция", d: "Синтезируются неструктурные белки (RdRp, протеазы)." },
  { t: "Шаг 7. Репликация", d: "RdRp создаёт новые копии РНК и субгеномные мРНК." },
  { t: "Шаг 8. Выход", d: "Собираются новые вирионы и выходят из клетки." }
];
let currentStep = 0;
let infVirus, infRna = [], infNew = [], infFlash;

function buildInfection() {
  const scene = document.getElementById("infection-scene");

  // Вирус
  infVirus = document.createElement("div");
  infVirus.className = "inf-virus";
  infVirus.style.left = "20px";
  infVirus.style.top = "165px";
  // шипы
  for (let i = 0; i < 10; i++) {
    const a = (i / 10) * 360;
    const spike = document.createElement("div");
    spike.className = "inf-spike";
    spike.style.transform = `rotate(${a}deg) translateY(-45px)`;
    infVirus.appendChild(spike);
  }
  scene.appendChild(infVirus);

  // РНК-точки (появятся позже)
  for (let i = 0; i < 6; i++) {
    const r = document.createElement("div");
    r.className = "inf-rna";
    r.style.left = (500 + Math.random() * 200) + "px";
    r.style.top = (140 + Math.random() * 140) + "px";
    scene.appendChild(r);
    infRna.push(r);
  }

  // Новые вирионы
  for (let i = 0; i < 4; i++) {
    const v = document.createElement("div");
    v.className = "inf-new";
    v.style.left = (700 + i * 40) + "px";
    v.style.top = (100 + Math.random() * 200) + "px";
    scene.appendChild(v);
    infNew.push(v);
  }

  infFlash = document.getElementById("stage-flash");

  // Точки прогресса
  const dots = document.getElementById("infection-dots");
  STEPS.forEach((s, i) => {
    const span = document.createElement("span");
    span.textContent = i + 1;
    span.onclick = () => { currentStep = i; updateInfection(); };
    dots.appendChild(span);
  });

  updateInfection();
}

function updateInfection() {
  // Позиции вируса (left, top) по шагам
  const pos = [
    { l: 20, t: 165, s: 1 },
    { l: 180, t: 140, s: 1 },
    { l: 320, t: 130, s: 1 },
    { l: 430, t: 140, s: 0.9 },
    { l: 520, t: 160, s: 0.8 },
    { l: 560, t: 180, s: 0.7 },
    { l: 580, t: 190, s: 0.6 },
    { l: 650, t: 200, s: 0.5 }
  ];
  const p = pos[currentStep];
  infVirus.style.left = p.l + "px";
  infVirus.style.top = p.t + "px";
  infVirus.style.transform = `scale(${p.s})`;
  infVirus.style.opacity = currentStep >= 5 ? 0.3 : 1;

  // РНК видна с шага 5
  infRna.forEach(r => r.style.opacity = currentStep >= 4 ? 1 : 0);
  // Новые вирионы — с шага 8
  infNew.forEach(v => v.style.opacity = currentStep >= 7 ? 1 : 0);

  // Точки
  document.querySelectorAll("#infection-dots span").forEach((s, i) => {
    s.classList.toggle("active", i === currentStep);
    s.classList.toggle("done", i < currentStep);
  });

  // Текст
  document.getElementById("infection-info").innerHTML =
    `<h4>${STEPS[currentStep].t}</h4><p>${STEPS[currentStep].d}</p>`;

  // Вспышка на сцене
  infFlash.textContent = STEPS[currentStep].t;
  infFlash.style.left = "20px";
  infFlash.style.top = "20px";
  infFlash.classList.add("show");
  clearTimeout(infFlash._timeout);
  infFlash._timeout = setTimeout(() => infFlash.classList.remove("show"), 1500);
}

function nextStep() { if (currentStep < STEPS.length - 1) { currentStep++; updateInfection(); } }
function prevStep() { if (currentStep > 0) { currentStep--; updateInfection(); } }
function resetSteps() { currentStep = 0; updateInfection(); }

/* ============================================================
   КАРТА
   ============================================================ */
const OUTBREAKS = [
  { x: 620, y: 155, city: "Ухань, Китай", date: "31 дек 2019", desc: "Первый кластер пневмонии", color: "#e74c3c" },
  { x: 645, y: 205, city: "Таиланд", date: "13 янв 2020", desc: "Первый случай вне Китая", color: "#e74c3c" },
  { x: 565, y: 105, city: "Япония", date: "16 янв 2020", desc: "Diamond Princess", color: "#e74c3c" },
  { x: 460, y: 115, city: "Италия", date: "31 янв 2020", desc: "Эпицентр в Европе", color: "#e67e22" },
  { x: 440, y: 98, city: "Франция", date: "24 янв 2020", desc: "Первые случаи", color: "#e67e22" },
  { x: 235, y: 155, city: "США", date: "21 янв 2020", desc: "Более 1 млн к апрелю", color: "#f1c40f" },
  { x: 485, y: 205, city: "Иран", date: "19 фев 2020", desc: "Эпицентр на Ближнем Востоке", color: "#e67e22" },
  { x: 525, y: 235, city: "Индия", date: "30 янв 2020", desc: "Локдаун в марте", color: "#f1c40f" },
  { x: 225, y: 325, city: "Бразилия", date: "26 фев 2020", desc: "Первый случай в Латинской Америке", color: "#2ecc71" },
  { x: 745, y: 365, city: "Австралия", date: "25 янв 2020", desc: "Локальные вспышки", color: "#2ecc71" }
];

const EVENTS = [
  "31 дек 2019 — ВОЗ уведомлена о пневмонии в Ухане",
  "30 янв 2020 — ЧС в области общественного здравоохранения",
  "11 фев 2020 — болезнь названа COVID-19",
  "11 мар 2020 — объявлена пандемия",
  "Апр 2020 — более 1 млн случаев в мире",
  "Май 2020 — Южная Америка становится эпицентром"
];

function buildMap() {
  const g = document.getElementById("map-points");
  const tooltip = document.getElementById("map-tooltip");
  const wrap = document.getElementById("map-wrap");

  OUTBREAKS.forEach(o => {
    const c = document.createElementNS("http://www.w3.org/2000/svg", "circle");
    c.setAttribute("cx", o.x); c.setAttribute("cy", o.y); c.setAttribute("r", 9);
    c.setAttribute("fill", o.color); c.setAttribute("stroke", "#fff"); c.setAttribute("stroke-width", 2);
    c.setAttribute("cursor", "pointer");
    c.onmouseenter = () => {
      c.setAttribute("r", 14);
      tooltip.style.display = "block";
      tooltip.innerHTML = `<b>${o.city}</b><br>${o.date}<br><span style="color:#555">${o.desc}</span>`;
    };
    c.onmousemove = e => {
      const rect = wrap.getBoundingClientRect();
      tooltip.style.left = Math.min(rect.width - 240, e.clientX - rect.left + 15) + "px";
      tooltip.style.top = Math.max(0, e.clientY - rect.top - 20) + "px";
    };
    c.onmouseleave = () => { c.setAttribute("r", 9); tooltip.style.display = "none"; };
    g.appendChild(c);
  });

  const list = document.getElementById("event-list");
  EVENTS.forEach(ev => { const li = document.createElement("li"); li.textContent = ev; list.appendChild(li); });
}

/* ============================================================
   ТЕСТ
   ============================================================ */
const QUIZ = {
  basic: [
    { q: "Какой белок связывается с ACE2?", opts: ["S", "N", "M"], a: 0 },
    { q: "Семейство SARS-CoV-2?", opts: ["Orthomyxoviridae", "Coronaviridae", "Filoviridae"], a: 1 },
    { q: "Тип генома?", opts: ["ДНК", "(−)РНК", "(+)РНК"], a: 2 },
    { q: "D614G — это…", opts: ["Белок", "Мутация в Spike", "Рецептор"], a: 1 }
  ],
  medium: [
    { q: "Протеаза, активирующая S?", opts: ["TMPRSS2", "ACE2", "Furin"], a: 0 },
    { q: "ORF1a/1b кодируют…", opts: ["Структурные белки", "pp1ab → 16 nsp", "Только Spike"], a: 1 },
    { q: "Сборка вирионов происходит в…", opts: ["Ядре", "ERGIC", "Митохондриях"], a: 1 },
    { q: "Мутация, повышающая аффинность к ACE2?", opts: ["D614G", "N501Y", "P681H"], a: 1 }
  ],
  hard: [
    { q: "Сколько событий расщепления S нужно для входа?", opts: ["1", "2 (S1/S2 и S2')", "0"], a: 1 },
    { q: "Ионный канал, участвующий в высвобождении?", opts: ["M", "E", "N"], a: 1 },
    { q: "Что НЕ является ко-рецептором?", opts: ["NRP1", "CD147", "Гемоглобин"], a: 2 },
    { q: "Отличие S от SARS-CoV-1?", opts: ["Нет RBD", "Фурин-подобный сайт S1/S2", "Нет S2"], a: 1 },
    { q: "Репликация РНК происходит в…", opts: ["Ядре", "Репликационных органеллах ER", "Митохондриях"], a: 1 }
  ]
};

let currentLevel = "basic";

function setLevel(level) {
  currentLevel = level;
  document.querySelectorAll("#level-tabs button").forEach(b => b.classList.toggle("active", b.dataset.level === level));
  renderQuiz();
}

function renderQuiz() {
  const c = document.getElementById("quiz-container");
  if (!c) return;
  c.innerHTML = "";
  document.getElementById("quiz-result").classList.add("hidden");
  QUIZ[currentLevel].forEach((item, i) => {
    const div = document.createElement("div");
    div.className = "quiz-q";
    div.innerHTML = `<p>${i+1}. ${item.q}</p>` +
      item.opts.map((o, j) => `<label><input type="radio" name="q${i}" value="${j}"> ${o}</label>`).join("");
    c.appendChild(div);
  });
}

function checkQuiz() {
  const items = QUIZ[currentLevel];
  let score = 0;
  items.forEach((item, i) => {
    const sel = document.querySelector(`input[name="q${i}"]:checked`);
    if (sel && parseInt(sel.value) === item.a) score++;
  });
  const r = document.getElementById("quiz-result");
  r.classList.remove("hidden", "good", "bad");
  r.classList.add(score >= items.length * 0.7 ? "good" : "bad");
  r.textContent = `Результат: ${score} из ${items.length} (${Math.round(score/items.length*100)}%)`;
  items.forEach((item, i) => {
    document.querySelectorAll(`input[name="q${i}"]`).forEach((inp, j) => {
      if (j === item.a) inp.parentElement.style.background = "#d4edda";
      else if (inp.checked && j !== item.a) inp.parentElement.style.background = "#f8d7da";
    });
  });
}

/* ============================================================
   СТАРТ
   ============================================================ */
document.addEventListener("DOMContentLoaded", () => {
  showPage("home");
  renderQuiz();
});
</script>
</body>
</html>
