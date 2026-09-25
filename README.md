<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Калькулятор калорий и расписание дня</title>
<style>
  :root{
    --bg:#0f1420; --card:#171d2c; --card2:#1e2637; --line:#2a3346;
    --text:#eef1f8; --muted:#9aa4bd; --accent:#5ee1b2; --accent2:#7aa2ff;
    --warn:#ffcf6b; --danger:#ff7a7a; --radius:18px;
  }
  *{box-sizing:border-box}
  body{
    margin:0; font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
    background:radial-gradient(1200px 600px at 10% -10%, #1c2740 0%, var(--bg) 55%),var(--bg);
    color:var(--text); min-height:100vh; padding:24px 14px 60px;
  }
  .wrap{max-width:780px; margin:0 auto}
  header{text-align:center; margin-bottom:22px}
  header h1{font-size:1.5rem; margin:0 0 6px}
  header p{color:var(--muted); margin:0; font-size:.92rem}
  .card{
    background:linear-gradient(180deg,var(--card),var(--card2));
    border:1px solid var(--line); border-radius:var(--radius);
    padding:20px; margin-bottom:18px;
  }
  .card h2{font-size:1.05rem; margin:0 0 14px; display:flex; align-items:center; gap:8px}
  .grid{display:grid; grid-template-columns:1fr 1fr; gap:12px}
  @media (max-width:520px){ .grid{grid-template-columns:1fr} }
  label{display:block; font-size:.82rem; color:var(--muted); margin-bottom:5px}
  input, select{
    width:100%; padding:11px 12px; border-radius:12px; border:1px solid var(--line);
    background:#0f1522; color:var(--text); font-size:.95rem; outline:none;
  }
  input:focus, select:focus{border-color:var(--accent2)}
  .field{margin-bottom:12px}
  .row3{display:grid; grid-template-columns:1fr 1fr 1fr; gap:12px}
  @media (max-width:520px){ .row3{grid-template-columns:1fr} }
  .toggle-row{display:flex; align-items:center; gap:10px; margin-bottom:12px}
  .toggle-row input[type=checkbox]{width:auto}
  button{
    width:100%; padding:14px; border:none; border-radius:14px; font-size:1rem; font-weight:600;
    background:linear-gradient(90deg,var(--accent),var(--accent2)); color:#0b0f18; cursor:pointer;
    margin-top:6px;
  }
  button:disabled{opacity:.6; cursor:default}
  #error{
    display:none; background:#341318; border:1px solid #5a2530; color:var(--danger);
    padding:12px 14px; border-radius:12px; margin-bottom:16px; font-size:.9rem;
  }
  #results{display:none}
  .stats{display:grid; grid-template-columns:repeat(3,1fr); gap:10px; margin-bottom:6px}
  @media (max-width:520px){ .stats{grid-template-columns:repeat(2,1fr)} }
  .stat{background:#0f1522; border:1px solid var(--line); border-radius:14px; padding:14px; text-align:center}
  .stat .num{font-size:1.35rem; font-weight:700; color:var(--accent)}
  .stat .lbl{font-size:.75rem; color:var(--muted); margin-top:2px}
  .notes{margin-top:12px}
  .note{
    background:#2a2312; border:1px solid #4a3c15; color:var(--warn);
    padding:10px 12px; border-radius:12px; font-size:.85rem; margin-top:8px;
  }
  .timeline{margin-top:8px}
  .item{
    display:flex; gap:12px; padding:12px 0; border-bottom:1px solid var(--line);
  }
  .item:last-child{border-bottom:none}
  .item .time{
    min-width:96px; font-variant-numeric:tabular-nums; color:var(--accent2);
    font-size:.85rem; font-weight:600; padding-top:2px;
  }
  .item .icon{font-size:1.2rem; width:26px; text-align:center}
  .item .body h4{margin:0 0 2px; font-size:.95rem}
  .item .body p{margin:0; color:var(--muted); font-size:.85rem; line-height:1.35}
  .item.meal .time{color:var(--accent)}
  footer{text-align:center; color:var(--muted); font-size:.78rem; margin-top:26px}
  .spinner{display:inline-block; width:16px; height:16px; border:2px solid #0b0f18; border-top-color:transparent; border-radius:50%; animation:spin .7s linear infinite; vertical-align:-3px; margin-right:6px}
  @keyframes spin{to{transform:rotate(360deg)}}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1 id="app-title">🍎 Калькулятор калорий и расписание дня</h1>
    <p>Введи данные — получишь норму калорий, БЖУ и готовый план на день</p>
  </header>

  <form class="card" id="form">
    <h2>👤 О тебе</h2>
    <div class="row3">
      <div class="field">
        <label>Пол</label>
        <select id="sex">
          <option value="m">Мужской</option>
          <option value="f">Женский</option>
        </select>
      </div>
      <div class="field">
        <label>Возраст</label>
        <input type="number" id="age" min="10" max="100" value="15" required>
      </div>
      <div class="field">
        <label>Вес, кг</label>
        <input type="number" id="weight" min="30" max="250" value="67" required>
      </div>
    </div>
    <div class="grid">
      <div class="field">
        <label>Рост, см</label>
        <input type="number" id="height" min="120" max="230" value="181" required>
      </div>
      <div class="field">
        <label>Уровень активности</label>
        <select id="activity">
          <option value="low">Минимальная (сидячий образ жизни)</option>
          <option value="light">Лёгкая (1–3 тренировки/нед)</option>
          <option value="mid" selected>Средняя (3–5 тренировок/нед)</option>
          <option value="high">Высокая (6–7 тренировок/нед)</option>
          <option value="extreme">Очень высокая (спорт + физ. работа)</option>
        </select>
      </div>
    </div>
    <div class="field">
      <label>Цель</label>
      <select id="goal">
        <option value="maintain" selected>Поддержание формы</option>
        <option value="gain">Набор массы</option>
        <option value="loss">Снижение веса</option>
      </select>
    </div>

    <h2 style="margin-top:18px">⏰ Режим дня</h2>
    <div class="grid">
      <div class="field">
        <label>Подъём</label>
        <input type="time" id="wake" value="07:00" required>
      </div>
      <div class="field">
        <label>Отбой</label>
        <input type="time" id="sleep" value="22:30" required>
      </div>
    </div>
    <div class="grid">
      <div class="field">
        <label>Начало учёбы / работы</label>
        <input type="time" id="schoolStart" value="08:30">
      </div>
      <div class="field">
        <label>Конец учёбы / работы</label>
        <input type="time" id="schoolEnd" value="14:30">
      </div>
    </div>
    <div class="field">
      <label>Приёмов пищи в день</label>
      <select id="meals">
        <option value="3">3</option>
        <option value="4" selected>4</option>
        <option value="5">5</option>
      </select>
    </div>

    <h2 style="margin-top:18px">🏋️ Тренировка</h2>
    <div class="toggle-row">
      <input type="checkbox" id="hasTraining" checked>
      <label style="margin:0" for="hasTraining">Сегодня есть тренировка</label>
    </div>
    <div class="grid" id="trainingFields">
      <div class="field">
        <label>Время начала</label>
        <input type="time" id="trainingTime" value="17:00">
      </div>
      <div class="field">
        <label>Длительность, мин</label>
        <input type="number" id="trainingDuration" min="15" max="240" value="60">
      </div>
    </div>

    <button type="submit" id="submitBtn">Составить план</button>
  </form>

  <div id="error"></div>

  <div id="results">
    <div class="card">
      <h2>📊 Твоя норма</h2>
      <div class="stats">
        <div class="stat"><div class="num" id="s-cal">–</div><div class="lbl">ккал/день</div></div>
        <div class="stat"><div class="num" id="s-protein">–</div><div class="lbl">белки, г</div></div>
        <div class="stat"><div class="num" id="s-fat">–</div><div class="lbl">жиры, г</div></div>
        <div class="stat"><div class="num" id="s-carbs">–</div><div class="lbl">углеводы, г</div></div>
        <div class="stat"><div class="num" id="s-water">–</div><div class="lbl">литров воды</div></div>
        <div class="stat"><div class="num" id="s-sleep">–</div><div class="lbl">часов сна</div></div>
      </div>
      <div class="notes" id="notes"></div>
    </div>

    <div class="card">
      <h2>🗓️ Расписание на день</h2>
      <div class="timeline" id="timeline"></div>
    </div>
  </div>

  <footer id="footer">Расчёт по формуле Миффлина – Сан Жеора · не заменяет консультацию врача или тренера</footer>
</div>

<script>
const form = document.getElementById('form');
const errorBox = document.getElementById('error');
const results = document.getElementById('results');
const submitBtn = document.getElementById('submitBtn');
const hasTraining = document.getElementById('hasTraining');
const trainingFields = document.getElementById('trainingFields');

hasTraining.addEventListener('change', () => {
  trainingFields.style.opacity = hasTraining.checked ? '1' : '.35';
  trainingFields.style.pointerEvents = hasTraining.checked ? 'auto' : 'none';
});

fetch('/api/plan').then(r => r.json()).then(d => {
  if (d.appName) document.getElementById('app-title').textContent = '🍎 ' + d.appName;
}).catch(() => {});

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  errorBox.style.display = 'none';
  results.style.display = 'none';
  submitBtn.disabled = true;
  submitBtn.innerHTML = '<span class="spinner"></span>Считаю…';

  const payload = {
    sex: document.getElementById('sex').value,
    age: document.getElementById('age').value,
    weight: document.getElementById('weight').value,
    height: document.getElementById('height').value,
    activity: document.getElementById('activity').value,
    goal: document.getElementById('goal').value,
    wake: document.getElementById('wake').value,
    sleep: document.getElementById('sleep').value,
    schoolStart: document.getElementById('schoolStart').value,
    schoolEnd: document.getElementById('schoolEnd').value,
    meals: document.getElementById('meals').value,
    trainingTime: hasTraining.checked ? document.getElementById('trainingTime').value : '',
    trainingDuration: document.getElementById('trainingDuration').value,
  };

  try {
    const res = await fetch('/api/plan', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    const data = await res.json();
    if (!res.ok) throw new Error(data.error || 'Ошибка сервера');
    render(data);
  } catch (err) {
    errorBox.textContent = '⚠️ ' + err.message;
    errorBox.style.display = 'block';
  } finally {
    submitBtn.disabled = false;
    submitBtn.textContent = 'Составить план';
  }
});

function render(data) {
  const s = data.summary;
  document.getElementById('s-cal').textContent = s.calories;
  document.getElementById('s-protein').textContent = s.protein;
  document.getElementById('s-fat').textContent = s.fat;
  document.getElementById('s-carbs').textContent = s.carbs;
  document.getElementById('s-water').textContent = s.water;
  document.getElementById('s-sleep').textContent = s.sleepHours;

  const notesBox = document.getElementById('notes');
  notesBox.innerHTML = '';
  (data.notes || []).forEach(n => {
    const d = document.createElement('div');
    d.className = 'note';
    d.textContent = '💡 ' + n;
    notesBox.appendChild(d);
  });

  const tl = document.getElementById('timeline');
  tl.innerHTML = '';
  data.schedule.forEach(item => {
    const row = document.createElement('div');
    row.className = 'item' + (item.type === 'meal' ? ' meal' : '');
    row.innerHTML = `
      <div class="time">${item.time}</div>
      <div class="icon">${item.icon}</div>
      <div class="body"><h4>${item.title}</h4><p>${item.text}</p></div>
    `;
    tl.appendChild(row);
  });

  results.style.display = 'block';
  results.scrollIntoView({ behavior: 'smooth', block: 'start' });
}
</script>
</body>
</html>
