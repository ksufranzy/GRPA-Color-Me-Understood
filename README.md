# GRPA-Color-Me-Understood
GRPA Online Temperament Assessment
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Color Me Understood – Temperament Assessment</title>
<style>
  :root { --text:#1f2937; --muted:#6b7280; --card:#f9fafb; }
  * { box-sizing: border-box; }
  body { margin:0; font-family: system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial; color: var(--text); }
  .wrap { max-width: 760px; margin: 0 auto; padding: 16px; }
  .card { background: var(--card); border-radius: 12px; padding: 14px; margin: 12px 0; box-shadow: 0 1px 2px rgba(0,0,0,.05); }
  .q { font-weight: 600; margin-bottom: 6px; }
  .scale { margin-top: 8px; }
  .scale label { margin-right: 10px; }
  button { appearance:none; border:none; background:#111; color:#fff; border-radius:10px; padding:12px 14px; font-weight:700; cursor:pointer; }
  .btns { display:flex; gap:10px; flex-wrap:wrap; margin-top:12px; }
  .btn-secondary { background:#e5e7eb; color:#111; }
  .results { display:none; margin-top:10px; }
  .muted { color: var(--muted); font-size:.95rem; }
</style>
</head>
<body>
<div class="wrap">
  <h1 style="text-align:center; margin:8px 0 4px;">Color Me Understood</h1>
  <p class="muted" style="text-align:center;">2–3 Minute Temperament Assessment (questions randomize on load)</p>

  <div class="card">
    <p><strong>How to take it:</strong> For each statement, pick what’s true of you <em>most of the time</em>. Scoring happens automatically — you’ll see your <strong>dominant</strong> and <strong>secondary</strong> results at the end.</p>
    <p class="muted">Scale: 0 = Not me · 1 = Sometimes · 2 = Often · 3 = Very true</p>
  </div>

  <form id="form" class="card">
    <div id="questions"></div>
    <div class="btns">
      <button type="button" id="scoreBtn">See My Results</button>
      <button type="reset" class="btn-secondary">Reset</button>
    </div>
  </form>

  <div id="results" class="card results">
    <h2 style="margin:6px 0 10px; font-size:1.15rem;">Your Results</h2>
    <p><strong>Yellow (Sanguine):</strong> <span id="scoreY">0</span></p>
    <p><strong>Red (Choleric):</strong> <span id="scoreR">0</span></p>
    <p><strong>Blue (Melancholic):</strong> <span id="scoreB">0</span></p>
    <p><strong>Green (Phlegmatic):</strong> <span id="scoreG">0</span></p>
    <p id="summary" style="margin-top:12px; font-weight:700;"></p>
    <p class="muted" style="font-size:.85rem;">This page scores locally in your browser. No data is collected.</p>
  </div>
</div>

<script>
(function(){
  // Question bank (no visible color labels; colors used only for scoring)
  const qbank = [
    {id:'q1',color:'Y',text:'I bring energy and get people talking.'},
    {id:'q2',color:'Y',text:'I think out loud and brainstorm easily.'},
    {id:'q3',color:'Y',text:'I’m spontaneous and comfortable meeting new people.'},
    {id:'q4',color:'Y',text:'I persuade and inspire more than I analyze details.'},
    {id:'q5',color:'R',text:'I take charge quickly and decide with confidence.'},
    {id:'q6',color:'R',text:'I’m comfortable with conflict if it helps us move forward.'},
    {id:'q7',color:'R',text:'I focus on results and efficiency over process.'},
    {id:'q8',color:'R',text:'I’d rather make a quick call than wait for consensus.'},
    {id:'q9',color:'B',text:'I value accuracy, quality, and well-thought-out plans.'},
    {id:'q10',color:'B',text:'I prefer clear expectations and time to prepare.'},
    {id:'q11',color:'B',text:'I notice risks and ask careful questions before acting.'},
    {id:'q12',color:'B',text:'I’m thorough and detail-oriented, sometimes to a fault.'},
    {id:'q13',color:'G',text:'I stay calm and steady when others are stressed.'},
    {id:'q14',color:'G',text:'I’m patient, loyal, and people know they can count on me.'},
    {id:'q15',color:'G',text:'I avoid unnecessary conflict and prefer harmony.'},
    {id:'q16',color:'G',text:'I’m consistent and supportive rather than attention-seeking.'}
  ];

  function shuffle(arr){
    for(let i=arr.length-1;i>0;i--){
      const j = Math.floor(Math.random()*(i+1));
      [arr[i],arr[j]] = [arr[j],arr[i]];
    }
    return arr;
  }

  function render(){
    const container = document.getElementById('questions');
    container.innerHTML = '';
    shuffle(qbank.slice()).forEach((q, idx)=>{
      const d = document.createElement('div');
      d.style.marginBottom = '10px';
      d.innerHTML = `
        <div class="q">${idx+1}. ${q.text}</div>
        <div class="scale">
          <label><input type="radio" name="${q.id}" value="0">0</label>
          <label><input type="radio" name="${q.id}" value="1">1</label>
          <label><input type="radio" name="${q.id}" value="2">2</label>
          <label><input type="radio" name="${q.id}" value="3">3</label>
        </div>`;
      container.appendChild(d);
    });
  }

  function val(name){
    const el = document.querySelector('input[name="'+name+'"]:checked');
    return el ? parseInt(el.value, 10) : 0;
  }

  function score(){
    const totals = {Y:0,R:0,B:0,G:0};
    qbank.forEach(q => { totals[q.color] += val(q.id); });

    document.getElementById('scoreY').textContent = totals.Y;
    document.getElementById('scoreR').textContent = totals.R;
    document.getElementById('scoreB').textContent = totals.B;
    document.getElementById('scoreG').textContent = totals.G;

    const entries = Object.entries(totals).sort((a,b)=>b[1]-a[1]);
    const labels = {Y:'Yellow (Sanguine)', R:'Red (Choleric)', B:'Blue (Melancholic)', G:'Green (Phlegmatic)'};

    let summary = '';
    if(entries[0][1] === entries[1][1]){
      summary = 'Tie for dominant: ' + labels[entries[0][0]] + ' & ' + labels[entries[1][0]];
    } else {
      summary = 'Dominant: ' + labels[entries[0][0]] + ' • Secondary: ' + labels[entries[1][0]];
    }

    document.getElementById('summary').textContent = summary;
    document.getElementById('results').style.display = 'block';
    window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' });
  }

  document.getElementById('scoreBtn').addEventListener('click', score);
  document.getElementById('form').addEventListener('reset', () => {
    setTimeout(() => { render(); document.getElementById('results').style.display='none'; }, 0);
  });

  render();
})();
</script>
</body>
</html>
