<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>SARS-CoV-2 — интерактивный вирусологический портал</title>
<style>
  :root {
    --primary: #0b3d91;
    --primary-light: #e3eaf6;
    --bg: #f4f7fb;
    --card: #ffffff;
    --text: #1a1a1a;
    --muted: #5a6a7a;
    --border: #d8e0ea;
    --danger: #c0392b;
    --success: #1e8449;
    --spike: #e74c3c;
    --membrane: #f39c12;
    --envelope: #9b59b6;
    --rna: #27ae60;
  }
  * { box-sizing: border-box; }
  body {
    margin: 0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    background: var(--bg); color: var(--text); line-height: 1.6;
  }
  header {
    background: var(--primary); color: #fff; padding: 1rem 2rem;
    display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: .5rem;
  }
  header h1 { margin: 0; font-size: 1.25rem; }
  header h1 a { color: #fff; text-decoration: none; }
  nav { display: flex; gap: .3rem; flex-wrap: wrap; }
  nav button {
    background: transparent; border: 1px solid rgba(255,255,255,.35); color: #fff;
    padding: .4rem .9rem; border-radius: 6px; cursor: pointer; font-size: .9rem;
  }
  nav button:hover, nav button.active { background: rgba(255,255,255,.2); }
  main { max-width: 1100px; margin: 0 auto; padding: 1.5rem; }
  section.page { display: none; }
  section.page.active { display: block; }
  h2 { color: var(--primary); margin-top: 0; }
  h3 { color: var(--primary); }
  .card {
    background: var(--card); border-radius: 12px; padding: 1.2rem 1.4rem;
    box-shadow: 0 2px 10px rgba(0,0,0,.06); margin-bottom: 1.2rem;
  }
  .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1.2rem; }
  @media (max-width: 800px) { .grid2 { grid-template-columns: 1fr; } }
  .badge {
    display: inline-block; background: var(--primary-light); color: var(--primary);
    padding: .15rem .6rem; border-radius: 20px; font-size: .8rem; margin: .1rem .2rem .1rem 0;
  }
  table { width: 100%; border-collapse: collapse; background: #fff; margin: .8rem 0; }
  th, td { border: 1px solid var(--border); padding: .45rem .6rem; text-align: left; font-size: .9rem; }
  th { background: var(--primary-light); color: var(--primary); }
  .btn {
    display: inline-block; padding: .5rem 1.1rem; background: var(--primary); color: #fff;
    border: none; border-radius: 8px; cursor: pointer; font-size: .95rem; margin: .2rem .3rem .2rem 0;
  }
  .btn:hover { background: #09306f; }
  .btn.secondary { background: var(--primary-light); color: var(--primary); }
  .btn.secondary:hover { background: #d0dcf0; }
  .btn.small { padding: .3rem .7rem; font-size: .85rem; }
  .hidden { display: none !important; }
  .muted { color: var(--muted); font-size: .9rem; }

  /* ---------- Визуализация вириона ---------- */
  .virion-wrap { display: flex; flex-wrap: wrap; gap: 1.5rem; align-items: flex-start; }
  .virion-stage {
    flex: 1 1 380px; min-height: 380px; position: relative;
    background: radial-gradient(circle at 50% 45%, #1a1a2e 0%, #0f0f1a 100%);
    border-radius: 16px; overflow: hidden; display: flex; align-items: center; justify-content: center;
  }
  .virion-svg { width: 100%; max-width: 420px; height: auto; }
  .layer-label { cursor: pointer; transition: opacity .2s; }
  .layer-label:hover { opacity: .8; }
  .controls { flex: 1 1 260px; }
  .layer-btn {
    display: flex; align-items: center; gap: .5rem; width: 100%;
    padding: .6rem .8rem; margin-bottom: .5rem; border-radius: 8px;
    border: 1px solid var(--border); background: #fff; cursor: pointer;
    font-size: .95rem; text-align: left; transition: all .15s;
  }
  .layer-btn:hover { border-color: var(--primary); }
  .layer-btn .dot { width: 14px; height: 14px; border-radius: 50%; flex-shrink: 0; }
  .layer-btn.off { opacity: .45; text-decoration: line-through; }

  /* ---------- Анимация заражения ---------- */
  .infection-stage {
    background: #0f172a; border-radius: 16px; padding: 1rem; min-height: 300px;
    position: relative; overflow: hidden;
  }
  .infection-svg { width: 100%; height: auto; max-height: 420px; }
  .step-info {
    background: var(--primary-light); border-left: 4px solid var(--primary);
    padding: .8rem 1rem; border-radius: 0 8px 8px 0; margin-top: 1rem;
  }
  .step-info h4 { margin: 0 0 .3rem; color: var(--primary); }
  .progress-dots { display: flex; gap: .4rem; margin: .8rem 0; flex-wrap: wrap; }
  .progress-dots span {
    width: 28px; height: 28px; border-radius: 50%; background: #d8e0ea;
    display: flex; align-items: center; justify-content: center;
    font-size: .75rem; cursor: pointer; transition: all .2s;
  }
  .progress-dots span.active { background: var(--primary); color: #fff; }
  .progress-dots span.done { background: var(--success); color: #fff; }

  /* ---------- Карта ---------- */
  .map-container { position: relative; background: #0b1a2e; border-radius: 16px; overflow: hidden; }
  .map-svg { width: 100%; height: auto; display: block; }
  .map-tooltip {
    position: absolute; background: #fff; border-radius: 8px; padding: .5rem .8rem;
    box-shadow: 0 4px 16px rgba(0,0,0,.25); font-size: .85rem; pointer-events: none;
    max-width: 220px; z-index: 10; display: none;
  }
  .map-legend { display: flex; gap: 1rem; flex-wrap: wrap; margin-top: .8rem; font-size: .85rem; }
  .map-legend span { display: flex; align-items: center; gap: .3rem; }
  .map-legend i { width: 12px; height: 12px; border-radius: 50%; display: inline-block; }

  /* ---------- Тест ---------- */
  .level-tabs { display: flex; gap: .5rem; margin-bottom: 1rem; flex-wrap: wrap; }
  .level-tabs button {
    padding: .5rem 1.2rem; border-radius: 8px; border: 1px solid var(--border);
    background: #fff; cursor: pointer; font-size: .95rem;
  }
  .level-tabs button.active { background: var(--primary); color: #fff; border-color: var(--primary); }
  .quiz-q { margin-bottom: 1rem; }
  .quiz-q p { margin: 0 0 .4rem; font-weight: 600; }
  .quiz-q label { display: block; padding: .3rem .5rem; border-radius: 6px; cursor: pointer; }
  .quiz-q label:hover { background: var(--primary-light); }
  .quiz-q input { margin-right: .5rem; }
  .quiz-result { margin-top: 1rem; padding: .8rem; border-radius: 8px; font-weight: 600; }
  .quiz-result.good { background: #d4edda; color: var(--success); }
  .quiz-result.bad { background: #f8d7da; color: var(--danger); }

  textarea {
    width: 100%; font-family: "Courier New", monospace; padding: .6rem;
    border: 1px solid var(--border); border-radius: 8px; resize: vertical;
  }
  pre {
    background: #111; color: #0f0; padding: 1rem; border-radius: 8px;
    overflow: auto; font-family: "Courier New", monospace; font-size: .85rem;
  }
  footer { text-align: center; padding: 1.5rem; color: var(--muted); font-size: .85rem; }
</style>
</head>
<body>

<header>
  <h1><a href="#" onclick="showPage('home');return false;">🧬 SARS-CoV-2 — интерактивный портал</a></h1>
  <nav>
    <button data-page="home" class="active" onclick="showPage('home')">Главная</button>
    <button data-page="structure" onclick="showPage('structure')">Строение</button>
    <button data-page="infection" onclick="showPage('infection')">Заражение</button>
    <button data-page="map" onclick="showPage('map')">Карта вспышек</button>
    <button data-page="quiz" onclick="showPage('quiz')">Тест</button>
    <button data-page="research" onclick="showPage('research')">Анализ</button>
  </nav>
</header>

<main>

<!-- ============================================================
     ГЛАВНАЯ
     ============================================================ -->
<section id="page-home" class="page active">
  <div class="card">
    <h2>SARS-CoV-2 — возбудитель COVID-19</h2>
    <p>
      <b>Семейство:</b> Coronaviridae • <b>Род:</b> Betacoronavirus •
      <b>Геном:</b> одноцепочечная РНК (+) ~29.9 т.п.н. •
      <b>Диаметр вириона:</b> 70–90 нм [citation:1]
    </p>
    <p>
      SARS-CoV-2 — оболочечный вирус с характерными шиповидными белками на поверхности,
      из-за которых под электронным микроскопом он напоминает корону.
      Шиповидный белок S связывается с рецептором <b>ACE2</b> на поверхности клеток человека,
      после чего вирус проникает внутрь и начинает репликацию [citation:1][citation:7].
    </p>
    <p>
      <span class="badge">Betacoronavirus</span>
      <span class="badge">ssRNA(+)</span>
      <span class="badge">ACE2-рецептор</span>
      <span class="badge">TMPRSS2-прайминг</span>
      <span class="badge">~29.9 kb</span>
    </p>
  </div>

  <div class="grid2">
    <div class="card">
      <h3>Ключевые белки</h3>
      <table>
        <tr><th>Белок</th><th>Функция</th><th>PDB</th></tr>
        <tr><td><b>S</b> (Spike)</td><td>Связывание с ACE2, вход в клетку</td><td><a href="https://www.rcsb.org/structure/6VXX" target="_blank">6VXX</a></td></tr>
        <tr><td><b>N</b> (Nucleocapsid)</td><td>Упаковка РНК</td><td><a href="https://www.rcsb.org/structure/6M3M" target="_blank">6M3M</a></td></tr>
        <tr><td><b>M</b> (Membrane)</td><td>Формирование оболочки</td><td><a href="https://www.rcsb.org/structure/8R7D" target="_blank">8R7D</a></td></tr>
        <tr><td><b>E</b> (Envelope)</td><td>Ионный канал, сборка вириона</td><td><a href="https://www.rcsb.org/structure/5X29" target="_blank">5X29</a></td></tr>
      </table>
    </div>
    <div class="card">
      <h3>Быстрые факты</h3>
      <ul>
        <li>Геном содержит 10 открытых рамок считывания (ORF), две трети занимают ORF1a/ORF1b — они кодируют 16 неструктурных белков [citation:10].</li>
        <li>Помимо ACE2, в качестве ко-рецепторов обсуждаются TMPRSS2 (прайминг S-белка), нейропилин-1 (NRP1) и BASIGIN/CD147 [citation:4].</li>
        <li>Мутация D614G в Spike повысила инфекционность вируса, а варианты Beta/Gamma/Omicron несут N501Y и E484K — ключевые для ухода от антител.</li>
        <li>Первые случаи неизвестной пневмонии в Ухане зафиксированы в декабре 2019; ВОЗ объявила пандемию 11 марта 2020 [citation:5].</li>
      </ul>
    </div>
  </div>
</section>

<!-- ============================================================
     СТРОЕНИЕ — интерактивная разборка вириона
     ============================================================ -->
<section id="page-structure" class="page">
  <h2>🔬 Строение вириона SARS-CoV-2</h2>
  <p class="muted">Нажмите на кнопки слоёв или на подписи вокруг вириона, чтобы убрать или вернуть слой.</p>

  <div class="virion-wrap">
    <div class="virion-stage">
      <svg class="virion-svg" viewBox="0 0 400 400" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <radialGradient id="gradEnv" cx="50%" cy="45%" r="50%">
            <stop offset="0%" stop-color="#2c3e50"/>
            <stop offset="70%" stop-color="#1a252f"/>
            <stop offset="100%" stop-color="#0f1419"/>
          </radialGradient>
        </defs>

        <!-- Оболочка (липидный бислой) -->
        <g id="layer-envelope" class="layer">
          <circle cx="200" cy="200" r="130" fill="url(#gradEnv)" stroke="#9b59b6" stroke-width="6"/>
          <circle cx="200" cy="200" r="124" fill="none" stroke="#b07cc6" stroke-width="2" stroke-dasharray="4 4" opacity="0.5"/>
        </g>

        <!-- M-белки (оранжевые вкрапления) -->
        <g id="layer-membrane" class="layer">
          <circle cx="95" cy="155" r="9" fill="#f39c12"/>
          <circle cx="130" cy="90" r="8" fill="#f39c12"/>
          <circle cx="210" cy="72" r="9" fill="#f39c12"/>
          <circle cx="290" cy="110" r="8" fill="#f39c12"/>
          <circle cx="315" cy="190" r="9" fill="#f39c12"/>
          <circle cx="300" cy="285" r="8" fill="#f39c12"/>
          <circle cx="215" cy="325" r="9" fill="#f39c12"/>
          <circle cx="115" cy="295" r="8" fill="#f39c12"/>
          <circle cx="80" cy="230" r="9" fill="#f39c12"/>
        </g>

        <!-- E-белки (фиолетовые, мелкие) -->
        <g id="layer-e" class="layer">
          <circle cx="160" cy="80" r="5" fill="#b07cc6"/>
          <circle cx="260" cy="88" r="5" fill="#b07cc6"/>
          <circle cx="310" cy="240" r="5" fill="#b07cc6"/>
          <circle cx="250" cy="320" r="5" fill="#b07cc6"/>
          <circle cx="140" cy="318" r="5" fill="#b07cc6"/>
          <circle cx="78" cy="200" r="5" fill="#b07cc6"/>
        </g>

        <!-- РНК + N-белки -->
        <g id="layer-rna" class="layer">
          <path d="M170 170 Q200 140 230 170 Q260 200 230 230 Q200 260 170 230 Q140 200 170 170 Z"
                fill="none" stroke="#27ae60" stroke-width="4" opacity="0.8"/>
          <path d="M185 185 Q200 170 215 185 Q230 200 215 215 Q200 230 185 215 Q170 200 185 185 Z"
                fill="none" stroke="#2ecc71" stroke-width="2" opacity="0.6"/>
          <circle cx="195" cy="190" r="4" fill="#27ae60"/>
          <circle cx="215" cy="200" r="4" fill="#27ae60"/>
          <circle cx="205" cy="215" r="4" fill="#27ae60"/>
          <circle cx="185" cy="205" r="4" fill="#27ae60"/>
        </g>

        <!-- Шиповидные белки (S) — рисуем поверх -->
        <g id="layer-spike" class="layer">
          <!-- верхний -->
          <line x1="200" y1="70" x2="200" y2="30" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="200" cy="26" r="9" fill="#e74c3c"/>
          <!-- верхне-правый -->
          <line x1="255" y1="85" x2="285" y2="55" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="290" cy="50" r="9" fill="#e74c3c"/>
          <!-- правый -->
          <line x1="320" y1="140" x2="360" y2="140" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="366" cy="140" r="9" fill="#e74c3c"/>
          <!-- нижне-правый -->
          <line x1="315" y1="260" x2="350" y2="290" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="356" cy="296" r="9" fill="#e74c3c"/>
          <!-- нижний -->
          <line x1="200" y1="330" x2="200" y2="370" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="200" cy="376" r="9" fill="#e74c3c"/>
          <!-- нижне-левый -->
          <line x1="85" y1="260" x2="50" y2="290" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="44" cy="296" r="9" fill="#e74c3c"/>
          <!-- левый -->
          <line x1="80" y1="140" x2="40" y2="140" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="34" cy="140" r="9" fill="#e74c3c"/>
          <!-- верхне-левый -->
          <line x1="145" y1="85" x2="115" y2="55" stroke="#e74c3c" stroke-width="7" stroke-linecap="round"/>
          <circle cx="110" cy="50" r="9" fill="#e74c3c"/>
        </g>

        <!-- Подписи -->
        <text x="200" y="395" text-anchor="middle" fill="#fff" font-size="12" font-weight="bold" id="label-text">SARS-CoV-2 (70–90 нм)</text>
      </svg>
    </div>

    <div class="controls">
      <h3>Слои</h3>
      <button class="layer-btn" id="btn-spike" onclick="toggleLayer('spike')">
        <span class="dot" style="background:#e74c3c"></span> S-белок (шипы) — связывание с ACE2
      </button>
      <button class="layer-btn" id="btn-envelope" onclick="toggleLayer('envelope')">
        <span class="dot" style="background:#9b59b6"></span> Липидная оболочка
      </button>
      <button class="layer-btn" id="btn-membrane" onclick="toggleLayer('membrane')">
        <span class="dot" style="background:#f39c12"></span> M-белок (мембранный)
      </button>
      <button class="layer-btn" id="btn-e" onclick="toggleLayer('e')">
        <span class="dot" style="background:#b07cc6"></span> E-белок (оболочечный)
      </button>
      <button class="layer-btn" id="btn-rna" onclick="toggleLayer('rna')">
        <span class="dot" style="background:#27ae60"></span> РНК + N-белок (нуклеокапсид)
      </button>

      <p class="muted" style="margin-top:1rem;">
        Совет: отключите все слои, кроме РНК, чтобы увидеть генетический материал внутри.
        Шипы — главная мишень антител и вакцин.
      </p>

      <h3 style="margin-top:1.5rem;">Что означает слой</h3>
      <div id="layer-desc" class="muted">
        Нажмите на слой слева, чтобы увидеть описание.
      </div>
    </div>
  </div>
</section>

<!-- ============================================================
     ЗАРАЖЕНИЕ — пошаговая анимация
     ============================================================ -->
<section id="page-infection" class="page">
  <h2>🦠 Как SARS-CoV-2 заражает клетку</h2>
  <p class="muted">Пошаговая анимация. Нажимайте «Далее» или выбирайте шаг в прогресс-баре.</p>

  <div class="infection-stage">
    <svg class="infection-svg" viewBox="0 0 800 420" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <radialGradient id="cellGrad" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#1a3a5c"/>
          <stop offset="100%" stop-color="#0f2238"/>
        </radialGradient>
        <marker id="arrow" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto">
          <path d="M0,0 L8,4 L0,8 Z" fill="#f1c40f"/>
        </marker>
      </defs>

      <!-- Фон: клетка-хозяин -->
      <ellipse cx="400" cy="240" rx="340" ry="160" fill="url(#cellGrad)" stroke="#2e86c1" stroke-width="3"/>
      <text x="400" y="395" text-anchor="middle" fill="#5dade2" font-size="13">Клетка-хозяин (эпителий дыхательных путей)</text>

      <!-- Рецепторы ACE2 на поверхности -->
      <g id="ace2-receptors">
        <rect x="150" y="195" width="8" height="28" rx="3" fill="#f1c40f"/>
        <rect x="250" y="195" width="8" height="28" rx="3" fill="#f1c40f"/>
        <rect x="350" y="195" width="8" height="28" rx="3" fill="#f1c40f"/>
        <rect x="450" y="195" width="8" height="28" rx="3" fill="#f1c40f"/>
        <rect x="550" y="195" width="8" height="28" rx="3" fill="#f1c40f"/>
        <rect x="620" y="195" width="8" height="28" rx="3" fill="#f1c40f"/>
      </g>

      <!-- Вирус (группа) -->
      <g id="virus-group">
        <circle cx="100" cy="120" r="32" fill="#2c3e50" stroke="#e74c3c" stroke-width="3"/>
        <!-- шипы -->
        <g stroke="#e74c3c" stroke-width="3" stroke-linecap="round">
          <line x1="100" y1="88" x2="100" y2="70"/>
          <line x1="128" y1="100" x2="145" y2="92"/>
          <line x1="128" y1="140" x2="145" y2="148"/>
          <line x1="100" y1="152" x2="100" y2="170"/>
          <line x1="72" y1="140" x2="55" y2="148"/>
          <line x1="72" y1="100" x2="55" y2="92"/>
        </g>
        <text x="100" y="125" text-anchor="middle" fill="#fff" font-size="10">SARS-CoV-2</text>
      </g>

      <!-- Этап 2: связывание -->
      <g id="step-binding" class="hidden">
        <line x1="132" y1="130" x2="350" y2="200" stroke="#f1c40f" stroke-width="2" stroke-dasharray="5 4" marker-end="url(#arrow)"/>
        <text x="240" y="150" fill="#f1c40f" font-size="12">S-белок → ACE2</text>
      </g>

      <!-- Этап 3: TMPRSS2 -->
      <g id="step-tmprss2" class="hidden">
        <rect x="340" y="180" width="18" height="18" rx="4" fill="#e67e22"/>
        <text x="349" y="193" text-anchor="middle" fill="#fff" font-size="9" font-weight="bold">T</text>
        <text x="280" y="175" fill="#e67e22" font-size="11">TMPRSS2 режет S</text>
      </g>

      <!-- Этап 4: слияние и вход -->
      <g id="step-fusion" class="hidden">
        <path d="M132 140 Q250 180 350 210" stroke="#e74c3c" stroke-width="3" fill="none" stroke-dasharray="6 4"/>
        <text x="220" y="225" fill="#e74c3c" font-size="11">слияние мембран</text>
      </g>

      <!-- Этап 5: РНК внутри -->
      <g id="step-rna" class="hidden">
        <path d="M320 250 Q360 230 400 250 Q440 270 480 250" stroke="#27ae60" stroke-width="4" fill="none"/>
        <text x="400" y="290" text-anchor="middle" fill="#2ecc71" font-size="12">РНК вируса в цитоплазме</text>
      </g>

      <!-- Этап 6: трансляция -->
      <g id="step-translation" class="hidden">
        <rect x="420" y="240" width="40" height="20" rx="4" fill="#3498db" opacity="0.8"/>
        <text x="440" y="254" text-anchor="middle" fill="#fff" font-size="9">pp1a/pp1ab</text>
        <text x="440" y="280" text-anchor="middle" fill="#5dade2" font-size="10">трансляция</text>
      </g>

      <!-- Этап 7: репликация -->
      <g id="step-replication" class="hidden">
        <path d="M480 250 Q520 220 560 250 Q600 280 640 250" stroke="#9b59b6" stroke-width="3" fill="none"/>
        <text x="560" y="290" text-anchor="middle" fill="#c39bd3" font-size="11">репликация РНК</text>
      </g>

      <!-- Этап 8: сборка и выход -->
      <g id="step-release" class="hidden">
        <circle cx="600" cy="140" r="20" fill="#2c3e50" stroke="#e74c3c" stroke-width="2"/>
        <circle cx="650" cy="120" r="20" fill="#2c3e50" stroke="#e74c3c" stroke-width="2"/>
        <circle cx="680" cy="160" r="20" fill="#2c3e50" stroke="#e74c3c" stroke-width="2"/>
        <text x="640" y="200" text-anchor="middle" fill="#e74c3c" font-size="11">новые вирионы</text>
        <text x="640" y="215" text-anchor="middle" fill="#e74c3c" font-size="10">→ экзоцитоз</text>
      </g>
    </svg>
  </div>

  <div class="progress-dots" id="progress-dots"></div>

  <div class="step-info" id="step-info">
    <h4>Шаг 1. Приближение</h4>
    <p>Вирус SARS-CoV-2 диаметром 70–90 нм приближается к клетке эпителия дыхательных путей. На его поверхности — шиповидные белки S, которые будут искать рецептор ACE2.</p>
  </div>

  <div style="margin-top:.8rem;">
    <button class="btn secondary" onclick="prevStep()">← Назад</button>
    <button class="btn" onclick="nextStep()">Далее →</button>
    <button class="btn secondary" onclick="resetSteps()">Сброс</button>
  </div>
</section>

<!-- ============================================================
     КАРТА ВСПЫШЕК
     ============================================================ -->
<section id="page-map" class="page">
  <h2>🌍 Хронология и география вспышек</h2>
  <p class="muted">Наведите курсор на точку, чтобы увидеть событие. Цвет — месяц 2020 года.</p>

  <div class="map-container" id="map-container">
    <svg class="map-svg" viewBox="0 0 900 480" xmlns="http://www.w3.org/2000/svg">
      <rect width="900" height="480" fill="#0b1a2e"/>
      <!-- Упрощённые контуры материков -->
      <g fill="#1c3a5c" opacity="0.7">
        <!-- Северная Америка -->
        <path d="M120 120 Q180 80 240 100 Q280 120 260 170 Q240 220 200 240 Q150 230 120 190 Q100 150 120 120Z"/>
        <!-- Южная Америка -->
        <path d="M200 260 Q240 250 260 290 Q270 340 240 390 Q210 410 190 380 Q170 330 180 290Z"/>
        <!-- Европа -->
        <path d="M400 90 Q440 70 480 80 Q500 100 480 130 Q450 150 420 140 Q390 120 400 90Z"/>
        <!-- Африка -->
        <path d="M400 170 Q450 150 500 170 Q520 220 500 290 Q470 340 430 330 Q390 300 380 240 Q380 200 400 170Z"/>
        <!-- Азия -->
        <path d="M500 70 Q600 50 720 80 Q800 110 820 170 Q800 220 740 240 Q680 250 620 220 Q560 200 520 150 Q490 110 500 70Z"/>
        <!-- Австралия -->
        <path d="M700 330 Q750 310 800 330 Q820 360 790 390 Q750 400 710 380 Q690 360 700 330Z"/>
      </g>

      <!-- Точки вспышек -->
      <g id="outbreak-points"></g>
    </svg>
    <div class="map-tooltip" id="map-tooltip"></div>
  </div>

  <div class="map-legend">
    <span><i style="background:#e74c3c"></i> Декабрь 2019 — Январь 2020</span>
    <span><i style="background:#e67e22"></i> Февраль 2020</span>
    <span><i style="background:#f1c40f"></i> Март 2020</span>
    <span><i style="background:#2ecc71"></i> Апрель–Май 2020</span>
  </div>

  <div class="card" style="margin-top:1rem;">
    <h3>Ключевые события</h3>
    <ul id="event-list"></ul>
  </div>
</section>

<!-- ============================================================
     ТЕСТ — 3 уровня
     ============================================================ -->
<section id="page-quiz" class="page">
  <h2>📝 Проверь себя</h2>
  <p class="muted">Выберите уровень сложности. Вопросы различаются глубиной.</p>

  <div class="level-tabs" id="level-tabs">
    <button data-level="basic" class="active" onclick="setLevel('basic')">🟢 Базовый</button>
    <button data-level="medium" onclick="setLevel('medium')">🟡 Средний</button>
    <button data-level="hard" onclick="setLevel('hard')">🔴 Продвинутый</button>
  </div>

  <div class="card">
    <div id="quiz-container"></div>
    <button class="btn" onclick="checkQuiz()">Проверить ответы</button>
    <div id="quiz-result" class="quiz-result hidden"></div>
  </div>
</section>

<!-- ============================================================
     АНАЛИЗ (исследовательский слой)
     ============================================================ -->
<section id="page-research" class="page">
  <h2>🔬 Демо-анализ последовательности</h2>
  <p class="muted">
    Вставьте последовательность (буквы аминокислот или нуклеотидов). Система найдёт совпадения
    с известными мутациями Spike из демо-базы.
  </p>

  <form id="analyze-form">
    <textarea name="sequence" rows="6" placeholder="Например: MSTQYVKGHRS — содержит Y, K, G, H, R"></textarea>
    <button class="btn" type="submit">Анализировать</button>
  </form>

  <pre id="result">Результат появится здесь...</pre>

  <h3>Справочник мутаций Spike</h3>
  <table>
    <thead><tr><th>Позиция</th><th>Мутация</th><th>Вариант</th><th>Эффект</th></tr></thead>
    <tbody id="mut-table"></tbody>
  </table>
</section>

</main>

<footer>Учебно-научный прототип • данные: PDB, Nextstrain, GISAID, литература (демо)</footer>

<script>
/* ============================================================
   ДАННЫЕ
   ============================================================ */
const MUTATIONS = {
  reference: "Wuhan-Hu-1",
  spike_mutations: [
    { pos: 501, ref: "N", alt: "Y", variant: "Beta/Gamma/Omicron", effect: "Повышает аффинность к ACE2" },
    { pos: 484, ref: "E", alt: "K", variant: "Beta/Gamma/Omicron", effect: "Уход от некоторых антител" },
    { pos: 614, ref: "D", alt: "G", variant: "D614G (все варианты)", effect: "Повышает инфекционность" },
    { pos: 681, ref: "P", alt: "H", variant: "Omicron", effect: "Усиливает фуриновый сайт расщепления" },
    { pos: 452, ref: "L", alt: "R", variant: "Omicron", effect: "Уход от нейтрализующих антител" }
  ]
};

const OUTBREAKS = [
  { x: 620, y: 150, city: "Ухань, Китай", date: "31 дек 2019", desc: "ВОЗ получила сообщение о кластере пневмонии неизвестного происхождения", color: "#e74c3c" },
  { x: 640, y: 200, city: "Таиланд", date: "13 янв 2020", desc: "Первый подтверждённый случай за пределами Китая", color: "#e74c3c" },
  { x: 560, y: 100, city: "Япония", date: "16 янв 2020", desc: "Первый случай; позднее — вспышка на круизном лайнере Diamond Princess", color: "#e74c3c" },
  { x: 460, y: 110, city: "Италия", date: "31 янв 2020", desc: "Первые случаи в Европе; к марту — один из эпицентров", color: "#e67e22" },
  { x: 440, y: 95, city: "Франция", date: "24 янв 2020", desc: "Первые случаи; возможная циркуляция уже в конце декабря 2019", color: "#e67e22" },
  { x: 230, y: 150, city: "США", date: "21 янв 2020", desc: "Первый случай в штате Вашингтон; к апрелю — более 1 млн случаев", color: "#f1c40f" },
  { x: 480, y: 200, city: "Иран", date: "19 фев 2020", desc: "Крупная вспышка; новый эпицентр на Ближнем Востоке", color: "#e67e22" },
  { x: 520, y: 230, city: "Индия", date: "30 янв 2020", desc: "Первый случай; в марте — общенациональный локдаун", color: "#f1c40f" },
  { x: 220, y: 320, city: "Бразилия", date: "26 фев 2020", desc: "Первый случай в Латинской Америке", color: "#2ecc71" },
  { x: 740, y: 360, city: "Австралия", date: "25 янв 2020", desc: "Первый случай; локальные вспышки", color: "#2ecc71" }
];

const EVENTS = [
  "31 декабря 2019 — ВОЗ уведомлена о пневмонии неизвестного происхождения в Ухане [citation:5]",
  "30 января 2020 — ВОЗ объявляет чрезвычайную ситуацию в области общественного здравоохранения [citation:5]",
  "11 февраля 2020 — болезнь получает название COVID-19 [citation:5]",
  "11 марта 2020 — ВОЗ объявляет пандемию [citation:5]",
  "Апрель 2020 — число случаев в мире превышает 1 млн [citation:5]",
  "Май 2020 — Южная Америка становится новым эпицентром [citation:5]"
];

/* ============================================================
   НАВИГАЦИЯ
   ============================================================ */
const PAGES = ["home","structure","infection","map","quiz","research"];

function showPage(name) {
  PAGES.forEach(p => {
    document.getElementById("page-" + p).classList.toggle("active", p === name);
  });
  document.querySelectorAll("nav button").forEach(b => {
    b.classList.toggle("active", b.dataset.page === name);
  });
  window.scrollTo({ top: 0, behavior: "smooth" });
  if (name === "infection") initInfection();
  if (name === "map") initMap();
  if (name === "quiz") renderQuiz();
}

/* ============================================================
   СТРОЕНИЕ — переключение слоёв
   ============================================================ */
const LAYER_INFO = {
  spike: "S-белок (Spike). Тример, формирующий «корону». Содержит RBD (receptor-binding domain), который связывается с ACE2. Главная мишень нейтрализующих антител и вакцин.",
  envelope: "Липидная оболочка. Двойной слой липидов, украденный у клетки-хозяина при сборке вириона. Уязвим для спиртов и детергентов.",
  membrane: "M-белок (Membrane). Самый многочисленный структурный белок. Формирует оболочку, участвует в сборке и почковании вириона.",
  e: "E-белок (Envelope). Мелкий ионный канал. Участвует в сборке, почковании и высвобождении вириона; влияет на воспалительный ответ.",
  rna: "Генетический материал: одноцепочечная (+)РНК ~29.9 т.п.н. Упакована N-белком (нуклеокапсид). Содержит 10 ORF; ORF1a/1b кодируют 16 неструктурных белков, включая RdRp и протеазы."
};

const layerState = { spike: true, envelope: true, membrane: true, e: true, rna: true };

function toggleLayer(name) {
  layerState[name] = !layerState[name];
  const g = document.getElementById("layer-" + name);
  const btn = document.getElementById("btn-" + name);
  if (g) g.style.display = layerState[name] ? "" : "none";
  if (btn) btn.classList.toggle("off", !layerState[name]);
  document.getElementById("layer-desc").textContent = LAYER_INFO[name];
}

/* ============================================================
   ЗАРАЖЕНИЕ — шаги
   ============================================================ */
const STEPS = [
  { title: "Шаг 1. Приближение", text: "Вирус SARS-CoV-2 диаметром 70–90 нм приближается к клетке эпителия дыхательных путей.", el: null },
  { title: "Шаг 2. Связывание с ACE2", text: "S-белок связывается с рецептором ACE2 на поверхности клетки. Это первый и самый специфичный шаг [citation:1][citation:10].", el: "step-binding" },
  { title: "Шаг 3. Прайминг TMPRSS2", text: "Хозяинская протеаза TMPRSS2 разрезает S-белок в сайте S2', активируя его для слияния мембран [citation:4][citation:10].", el: "step-tmprss2" },
  { title: "Шаг 4. Слияние мембран", text: "После активации S2-субъединица встраивается в мембрану клетки и запускает слияние. Вирусная оболочка сливается с клеточной [citation:10].", el: "step-fusion" },
  { title: "Шаг 5. РНК в цитоплазме", text: "Генетический материал (+)РНК высвобождается в цитоплазму. Начинается трансляция ORF1a/ORF1b с образованием pp1a/pp1ab [citation:4][citation:10].", el: "step-rna" },
  { title: "Шаг 6. Трансляция и неструктурные белки", text: "Из pp1ab нарезаются 16 неструктурных белков (nsp1–16), включая RdRp и протеазы. Они перестраивают мембраны ER в репликационные органеллы [citation:4].", el: "step-translation" },
  { title: "Шаг 7. Репликация РНК", text: "RdRp синтезирует (−)РНК как матрицу, затем — новые (+)РНК и субгеномные мРНК для структурных белков S, E, M, N [citation:4][citation:10].", el: "step-replication" },
  { title: "Шаг 8. Сборка и выход", text: "Структурные белки и РНК собираются в ERGIC, почкуются в везикулы и выходят через экзоцитоз, заражая соседние клетки [citation:4][citation:10].", el: "step-release" }
];

let currentStep = 0;

function initInfection() {
  if (document.getElementById("progress-dots").children.length) return;
  const dots = document.getElementById("progress-dots");
  STEPS.forEach((s, i) => {
    const span = document.createElement("span");
    span.textContent = i + 1;
    span.onclick = () => { currentStep = i; updateStep(); };
    dots.appendChild(span);
  });
  updateStep();
}

function updateStep() {
  // Скрыть все этапы
  ["step-binding","step-tmprss2","step-fusion","step-rna","step-translation","step-replication","step-release"]
    .forEach(id => {
      const el = document.getElementById(id);
      if (el) el.classList.add("hidden");
    });

  // Показать все этапы до текущего включительно
  for (let i = 0; i <= currentStep; i++) {
    if (STEPS[i].el) document.getElementById(STEPS[i].el).classList.remove("hidden");
  }

  // Двигать вирус
  const virus = document.getElementById("virus-group");
  const positions = [
    { cx: 100, cy: 120 },
    { cx: 150, cy: 150 },
    { cx: 200, cy: 170 },
    { cx: 260, cy: 190 },
    { cx: 320, cy: 220 },
    { cx: 400, cy: 240 },
    { cx: 480, cy: 240 },
    { cx: 560, cy: 200 }
  ];
  const p = positions[currentStep] || positions[0];
  // Плавный переход через transform
  virus.style.transition = "transform 0.6s ease";
  const dx = p.cx - 100;
  const dy = p.cy - 120;
  virus.style.transform = `translate(${dx}px, ${dy}px)`;

  // Обновить прогресс-бар
  document.querySelectorAll("#progress-dots span").forEach((s, i) => {
    s.classList.toggle("active", i === currentStep);
    s.classList.toggle("done", i < currentStep);
  });

  // Обновить текст
  document.getElementById("step-info").innerHTML =
    `<h4>${STEPS[currentStep].title}</h4><p>${STEPS[currentStep].text}</p>`;
}

function nextStep() {
  if (currentStep < STEPS.length - 1) { currentStep++; updateStep(); }
}

function prevStep() {
  if (currentStep > 0) { currentStep--; updateStep(); }
}

function resetSteps() {
  currentStep = 0;
  document.getElementById("virus-group").style.transform = "none";
  updateStep();
}

/* ============================================================
   КАРТА
   ============================================================ */
function initMap() {
  const g = document.getElementById("outbreak-points");
  if (g.children.length) return;

  const tooltip = document.getElementById("map-tooltip");
  const container = document.getElementById("map-container");

  OUTBREAKS.forEach(o => {
    const circle = document.createElementNS("http://www.w3.org/2000/svg", "circle");
    circle.setAttribute("cx", o.x);
    circle.setAttribute("cy", o.y);
    circle.setAttribute("r", 10);
    circle.setAttribute("fill", o.color);
    circle.setAttribute("stroke", "#fff");
    circle.setAttribute("stroke-width", 2);
    circle.setAttribute("cursor", "pointer");
    circle.style.transition = "r .2s";
    circle.onmouseenter = (e) => {
      circle.setAttribute("r", 14);
      tooltip.style.display = "block";
      tooltip.innerHTML = `<b>${o.city}</b><br>${o.date}<br><span style="color:#555">${o.desc}</span>`;
      const rect = container.getBoundingClientRect();
      tooltip.style.left = (e.clientX - rect.left + 15) + "px";
      tooltip.style.top = (e.clientY - rect.top - 10) + "px";
    };
    circle.onmousemove = (e) => {
      const rect = container.getBoundingClientRect();
      tooltip.style.left = (e.clientX - rect.left + 15) + "px";
      tooltip.style.top = (e.clientY - rect.top - 10) + "px";
    };
    circle.onmouseleave = () => {
      circle.setAttribute("r", 10);
      tooltip.style.display = "none";
    };
    g.appendChild(circle);
  });

  // Список событий
  const list = document.getElementById("event-list");
  if (!list.children.length) {
    EVENTS.forEach(e => {
      const li = document.createElement("li");
      li.textContent = e;
      list.appendChild(li);
    });
  }
}

/* ============================================================
   ТЕСТ — 3 уровня
   ============================================================ */
const QUIZ = {
  basic: [
    { q: "Какой белок SARS-CoV-2 связывается с ACE2?", opts: ["S (Spike)", "N (Nucleocapsid)", "M (Membrane)"], a: 0 },
    { q: "К какому семейству относится SARS-CoV-2?", opts: ["Orthomyxoviridae", "Coronaviridae", "Filoviridae"], a: 1 },
    { q: "Что такое D614G?", opts: ["Белок", "Мутация в Spike", "Рецептор"], a: 1 },
    { q: "Какой тип генома у SARS-CoV-2?", opts: ["ДНК", "(−)РНК", "(+)РНК"], a: 2 }
  ],
  medium: [
    { q: "Какая протеаза хозяина активирует S-белок для слияния?", opts: ["TMPRSS2", "ACE2", "Furin (только)"], a: 0 },
    { q: "Что кодируют ORF1a и ORF1b?", opts: ["Структурные белки", "Полипротеин pp1ab → 16 nsp", "Только Spike"], a: 1 },
    { q: "Какая мутация повышает аффинность к ACE2?", opts: ["D614G", "N501Y", "P681H"], a: 1 },
    { q: "Где происходит сборка новых вирионов?", opts: ["В ядре", "В ERGIC (ER–Golgi intermediate compartment)", "В митохондриях"], a: 1 }
  ],
  hard: [
    { q: "Какие два события расщепления S-белка необходимы для входа?", opts: ["Только S1/S2", "S1/S2 и S2'", "Только S2'"], a: 1 },
    { q: "Какой белок образует ионный канал и участвует в высвобождении вириона?", opts: ["M", "E", "N"], a: 1 },
    { q: "Что из перечисленного НЕ является подтверждённым ко-рецептором или фактором входа?", opts: ["Нейропилин-1 (NRP1)", "BASIGIN/CD147", "Гемоглобин"], a: 2 },
    { q: "Какая особенность S-белка SARS-CoV-2 отличает его от SARS-CoV-1?", opts: ["Отсутствие RBD", "Фурин-подобный сайт на границе S1/S2", "Отсутствие S2-субъединицы"], a: 1 },
    { q: "Что происходит на этапе «репликация»?", opts: ["Вирус сразу выходит из клетки", "RdRp синтезирует (−)РНК, затем новые (+)РНК и субгеномные мРНК", "S-белок связывается с ACE2"], a: 1 }
  ]
};

let currentLevel = "basic";

function setLevel(level) {
  currentLevel = level;
  document.querySelectorAll("#level-tabs button").forEach(b => {
    b.classList.toggle("active", b.dataset.level === level);
  });
  renderQuiz();
}

function renderQuiz() {
  const container = document.getElementById("quiz-container");
  if (!container) return;
  container.innerHTML = "";
  document.getElementById("quiz-result").classList.add("hidden");

  QUIZ[currentLevel].forEach((item, i) => {
    const div = document.createElement("div");
    div.className = "quiz-q";
    div.innerHTML = `<p>${i + 1}. ${item.q}</p>` +
      item.opts.map((o, j) =>
        `<label><input type="radio" name="q${i}" value="${j}"> ${o}</label>`
      ).join("");
    container.appendChild(div);
  });
}

function checkQuiz() {
  const items = QUIZ[currentLevel];
  let score = 0;
  items.forEach((item, i) => {
    const sel = document.querySelector(`input[name="q${i}"]:checked`);
    if (sel && parseInt(sel.value) === item.a) score++;
  });

  const result = document.getElementById("quiz-result");
  result.classList.remove("hidden", "good", "bad");
  result.classList.add(score >= items.length * 0.7 ? "good" : "bad");
  result.textContent = `Результат: ${score} из ${items.length} (${Math.round(score / items.length * 100)}%)`;

  // Показать правильные ответы
  items.forEach((item, i) => {
    const labels = document.querySelectorAll(`input[name="q${i}"]`);
    labels.forEach((inp, j) => {
      if (j === item.a) inp.parentElement.style.background = "#d4edda";
      else if (inp.checked && j !== item.a) inp.parentElement.style.background = "#f8d7da";
    });
  });
}

/* ============================================================
   АНАЛИЗ
   ============================================================ */
function analyzeSequence(seq) {
  seq = (seq || "").trim().toUpperCase().replace(/[\n\s]/g, "");
  if (!seq) return { error: "Последовательность не загружена" };
  const findings = MUTATIONS.spike_mutations
    .filter(m => seq.includes(m.alt))
    .map(m => ({
      position: m.pos,
      mutation: `${m.ref}${m.pos}${m.alt}`,
      variant: m.variant,
      effect: m.effect
    }));
  return {
    length: seq.length,
    reference: MUTATIONS.reference,
    found_mutations: findings,
    note: "Демо-анализ: поиск остатков из базы мутаций в загруженной последовательности."
  };
}

document.getElementById("analyze-form").addEventListener("submit", (e) => {
  e.preventDefault();
  const seq = e.target.elements["sequence"].value;
  const resultEl = document.getElementById("result");
  resultEl.textContent = "Анализ...";
  setTimeout(() => {
    resultEl.textContent = JSON.stringify(analyzeSequence(seq), null, 2);
  }, 200);
});

// Заполнить справочник мутаций
(function fillMutTable() {
  const tbody = document.getElementById("mut-table");
  if (!tbody) return;
  MUTATIONS.spike_mutations.forEach(m => {
    const tr = document.createElement("tr");
    tr.innerHTML = `<td>${m.pos}</td><td>${m.ref}${m.pos}${m.alt}</td><td>${m.variant}</td><td>${m.effect}</td>`;
    tbody.appendChild(tr);
  });
})();

// Старт
document.addEventListener("DOMContentLoaded", () => {
  showPage("home");
  renderQuiz();
});
</script>
</body>
</html>
