<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <title>Вирусологический портал</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
      margin: 0; background: #f5f7fa; color: #222; line-height: 1.5;
    }
    header {
      background: #0b3d91; color: #fff; padding: 1rem 2rem;
      display: flex; justify-content: space-between; align-items: center;
      flex-wrap: wrap;
    }
    header h1 { margin: 0; font-size: 1.3rem; }
    header a { color: #fff; text-decoration: none; }
    nav a {
      margin-left: 1rem; padding: .3rem .6rem;
      border-radius: 6px; transition: background .2s;
    }
    nav a:hover { background: rgba(255,255,255,.15); }
    main { padding: 2rem; max-width: 1100px; margin: 0 auto; }
    h2 { color: #0b3d91; }
    .cards {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
      gap: 1rem;
    }
    .card {
      background: #fff; border-radius: 10px; padding: 1rem 1.2rem;
      box-shadow: 0 2px 8px rgba(0,0,0,.08);
    }
    .btn {
      display: inline-block; padding: .5rem 1rem;
      background: #0b3d91; color: #fff; border-radius: 6px;
      text-decoration: none; border: none; cursor: pointer;
      margin: .2rem .2rem .2rem 0; font-size: .95rem;
    }
    .btn:hover { background: #09306f; }
    .btn.secondary { background: #e3eaf6; color: #0b3d91; }
    .btn.secondary:hover { background: #d0dcf0; }
    table {
      width: 100%; border-collapse: collapse; background: #fff;
      margin: 1rem 0; border-radius: 8px; overflow: hidden;
    }
    th, td { border: 1px solid #ddd; padding: .5rem .7rem; text-align: left; }
    th { background: #e3eaf6; color: #0b3d91; }
    textarea {
      width: 100%; font-family: "Courier New", monospace;
      padding: .6rem; border: 1px solid #ccc; border-radius: 6px;
      resize: vertical;
    }
    pre {
      background: #111; color: #0f0; padding: 1rem;
      border-radius: 6px; overflow: auto;
      font-family: "Courier New", monospace; min-height: 2rem;
    }
    footer { text-align: center; padding: 1rem; color: #666; font-size: .9rem; }
    .hidden { display: none; }
    #quiz p { margin-bottom: .3rem; }
    #quiz label { display: inline-block; margin-right: 1rem; }
  </style>
</head>
<body>
<header>
  <h1><a href="#" onclick="showPage('home'); return false;">🧬 Вирусологический портал</a></h1>
  <nav>
    <a href="#" onclick="showPage('home'); return false;">Главная</a>
    <a href="#" onclick="showPage('learn'); return false;">Учебный модуль</a>
    <a href="#" onclick="showPage('research'); return false;">Исследование</a>
  </nav>
</header>

<main>
  <!-- ГЛАВНАЯ -->
  <section id="page-home">
    <h2>Выберите вирус</h2>
    <div class="cards" id="virus-cards"></div>
  </section>

  <!-- КАРТОЧКА ВИРУСА -->
  <section id="page-virus" class="hidden">
    <h2 id="virus-title"></h2>
    <p id="virus-meta"></p>
    <p id="virus-desc"></p>
    <h3>Белки</h3>
    <table>
      <thead>
        <tr><th>ID</th><th>Название</th><th>PDB</th><th>Функция</th></tr>
      </thead>
      <tbody id="virus-proteins"></tbody>
    </table>
  </section>

  <!-- УЧЕБНЫЙ МОДУЛЬ -->
  <section id="page-learn" class="hidden">
    <h2>📚 Учебный модуль: SARS-CoV-2</h2>
    <h3>Шаг 1. Что такое коронавирус?</h3>
    <p>
      Коронавирусы — семейство <i>Coronaviridae</i> с одноцепочечной РНК.
      Название связано с шиповидными белками на поверхности, напоминающими корону.
    </p>
    <h3>Шаг 2. Ключевой белок S</h3>
    <p>
      Шиповидный белок (Spike, PDB: 6VXX) связывается с рецептором ACE2.
      Именно он — основная мишень антител.
    </p>
    <iframe src="https://www.rcsb.org/3d-view/6VXX" width="100%" height="400"
            style="border:1px solid #ccc"></iframe>
    <h3>Шаг 3. Проверь себя</h3>
    <div id="quiz"></div>
  </section>

  <!-- ИССЛЕДОВАТЕЛЬСКИЙ МОДУЛЬ -->
  <section id="page-research" class="hidden">
    <h2>🔬 Исследовательский модуль: SARS-CoV-2</h2>
    <p>
      Вставьте нуклеотидную или аминокислотную последовательность (демо).
      Система найдёт совпадения с известными мутациями.
    </p>
    <form id="analyze-form">
      <textarea name="sequence" rows="8" placeholder="Вставьте последовательность..."></textarea>
      <button class="btn" type="submit">Анализировать</button>
    </form>
    <pre id="result">Результат появится здесь...</pre>
    <h3>Справочник мутаций</h3>
    <table>
      <thead>
        <tr><th>Позиция</th><th>Мутация</th><th>Вариант</th><th>Эффект</th></tr>
      </thead>
      <tbody id="mut-table"></tbody>
    </table>
  </section>
</main>

<footer>Учебно-научный прототип • данные: PDB, Nextstrain, GISAID (демо)</footer>

<script>
/* ============================================================
   ДАННЫЕ (встроены — сервер не нужен)
   ============================================================ */
const VIRUSES = {
  "sars-cov-2": {
    name: "SARS-CoV-2",
    family: "Coronaviridae",
    genome: "ssRNA(+) ~29.9 kb",
    description: "Возбудитель COVID-19. Ключевой белок — шиповидный (S), отвечает за связывание с рецептором ACE2.",
    proteins: [
      { id: "S", name: "Spike", pdb: "6VXX", function: "Связывание с рецептором ACE2, вход в клетку" },
      { id: "N", name: "Nucleocapsid", pdb: "6M3M", function: "Упаковка РНК" },
      { id: "M", name: "Membrane", pdb: "8R7D", function: "Формирование оболочки" },
      { id: "E", name: "Envelope", pdb: "5X29", function: "Ионный канал, сборка вириона" }
    ]
  }
};

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

/* ============================================================
   НАВИГАЦИЯ МЕЖДУ СТРАНИЦАМИ (SPA-стиль)
   ============================================================ */
const PAGES = ["home", "virus", "learn", "research"];

function showPage(name) {
  PAGES.forEach(p => {
    const el = document.getElementById("page-" + p);
    if (el) el.classList.toggle("hidden", p !== name);
  });
  window.scrollTo({ top: 0, behavior: "smooth" });
}

/* ============================================================
   ГЛАВНАЯ: карточки вирусов
   ============================================================ */
function renderHome() {
  const container = document.getElementById("virus-cards");
  container.innerHTML = "";
  Object.entries(VIRUSES).forEach(([id, v]) => {
    const div = document.createElement("div");
    div.className = "card";
    div.innerHTML = `
      <h3>${v.name}</h3>
      <p><b>Семейство:</b> ${v.family}</p>
      <p><b>Геном:</b> ${v.genome}</p>
      <p>${v.description}</p>
      <button class="btn" onclick="openVirus('${id}')">Открыть карточку</button>
      <button class="btn secondary" onclick="showPage('learn')">Учиться</button>
      <button class="btn secondary" onclick="showPage('research')">Исследовать</button>
    `;
    container.appendChild(div);
  });
}

/* ============================================================
   КАРТОЧКА ВИРУСА
   ============================================================ */
function openVirus(id) {
  const v = VIRUSES[id];
  if (!v) return;

  document.getElementById("virus-title").textContent = v.name;
  document.getElementById("virus-meta").innerHTML =
    `<b>Семейство:</b> ${v.family} • <b>Геном:</b> ${v.genome}`;
  document.getElementById("virus-desc").textContent = v.description;

  const tbody = document.getElementById("virus-proteins");
  tbody.innerHTML = "";
  v.proteins.forEach(p => {
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${p.id}</td>
      <td>${p.name}</td>
      <td><a href="https://www.rcsb.org/structure/${p.pdb}" target="_blank">${p.pdb}</a></td>
      <td>${p.function}</td>
    `;
    tbody.appendChild(tr);
  });

  showPage("virus");
}

/* ============================================================
   ТЕСТ В УЧЕБНОМ МОДУЛЕ
   ============================================================ */
function renderQuiz() {
  const el = document.getElementById("quiz");
  if (!el || el.dataset.done) return;
  el.dataset.done = "1";

  const questions = [
    { q: "Какой белок SARS-CoV-2 связывается с ACE2?", a: "S", options: ["S", "N", "M"] },
    { q: "Что такое D614G?", a: "Мутация в Spike", options: ["Белок", "Мутация в Spike", "Рецептор"] },
    { q: "К какому семейству относится SARS-CoV-2?", a: "Coronaviridae", options: ["Orthomyxoviridae", "Coronaviridae", "Filoviridae"] }
  ];

  questions.forEach((item, i) => {
    const div = document.createElement("div");
    div.innerHTML =
      `<p><b>${i + 1}. ${item.q}</b></p>` +
      item.options.map(o =>
        `<label><input type="radio" name="q${i}" value="${o}"> ${o}</label>`
      ).join("");
    el.appendChild(div);
  });

  const btn = document.createElement("button");
  btn.className = "btn";
  btn.textContent = "Проверить";
  btn.onclick = () => {
    let score = 0;
    questions.forEach((item, i) => {
      const sel = document.querySelector(`input[name="q${i}"]:checked`);
      if (sel && sel.value === item.a) score++;
    });
    alert(`Правильно: ${score} из ${questions.length}`);
  };
  el.appendChild(btn);
}

/* ============================================================
   ИССЛЕДОВАТЕЛЬСКИЙ МОДУЛЬ: справочник + анализ
   ============================================================ */
function renderMutations() {
  const tbody = document.getElementById("mut-table");
  if (!tbody || tbody.dataset.done) return;
  tbody.dataset.done = "1";

  MUTATIONS.spike_mutations.forEach(m => {
    const tr = document.createElement("tr");
    tr.innerHTML = `
      <td>${m.pos}</td>
      <td>${m.ref}${m.pos}${m.alt}</td>
      <td>${m.variant}</td>
      <td>${m.effect}</td>
    `;
    tbody.appendChild(tr);
  });
}

function analyzeSequence(seq) {
  seq = (seq || "").trim().toUpperCase().replace(/[\n\s]/g, "");
  if (!seq) {
    return { error: "Последовательность не загружена" };
  }
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

const analyzeForm = document.getElementById("analyze-form");
if (analyzeForm) {
  analyzeForm.addEventListener("submit", (e) => {
    e.preventDefault();
    const seq = analyzeForm.elements["sequence"].value;
    const resultEl = document.getElementById("result");
    resultEl.textContent = "Анализ...";
    setTimeout(() => {
      const data = analyzeSequence(seq);
      resultEl.textContent = JSON.stringify(data, null, 2);
    }, 200);
  });
}

/* ============================================================
   ИНИЦИАЛИЗАЦИЯ
   ============================================================ */
document.addEventListener("DOMContentLoaded", () => {
  renderHome();
  renderQuiz();
  renderMutations();
});
</script>
</body>
</html>
