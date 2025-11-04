# Generador-de-n-meros-
Genera 3 números aleatorios 
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Generador de 3 números (0100-9999)</title>
  <style>
    body { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; max-width:900px; margin:28px auto; padding:0 18px; color:#111 }
    h1 { font-size:1.4rem; margin-bottom:6px }
    .card { border:1px solid #ddd; padding:14px; border-radius:8px; box-shadow: 0 2px 6px rgba(0,0,0,0.03); margin-bottom:18px }
    label { display:block; margin-top:8px; font-weight:600; font-size:0.9rem }
    input[type="text"], input[type="tel"] { width:100%; padding:10px; margin-top:6px; border-radius:6px; border:1px solid #ccc; box-sizing:border-box }
    button { padding:10px 14px; border-radius:8px; border:none; background:#0b74de; color:white; cursor:pointer; margin-right:8px }
    button.secondary { background:#666 }
    .numbers { display:flex; gap:10px; margin-top:10px; flex-wrap:wrap }
    .num { background:#f3f6fb; padding:12px 16px; border-radius:8px; font-weight:700; font-size:1.1rem; min-width:86px; text-align:center }
    table { width:100%; border-collapse:collapse; margin-top:12px; font-size:0.95rem }
    th, td { padding:8px 10px; border-bottom:1px solid #eee; text-align:left }
    th { background:#fafafa; font-weight:700 }
    .small { font-size:0.85rem; color:#555 }
    .controls { margin-top:10px }
    .actions { display:flex; gap:8px; flex-wrap:wrap; margin-top:10px }
    .danger { background:#c33 }
    .muted { color:#666; font-size:0.9rem; margin-top:6px }
    .inline { display:inline-block; vertical-align:middle }
    .btn-link { background:transparent; color:#0b74de; padding:6px; border: none; cursor:pointer; text-decoration:underline }
    input[type="file"] { display:none }
    footer { margin-top:18px; font-size:0.85rem; color:#444 }
    @media (max-width:520px){ .numbers { flex-direction:column } .num { min-width:unset } }
  </style>
</head>
<body>
  <h1>Generador de 3 números (0100–9999)</h1>

  <div class="card">
    <form id="form">
      <label>Nombre
        <input id="name" type="text" placeholder="Tu nombre" required>
      </label>
      <label>Teléfono
        <input id="phone" type="tel" placeholder="Tu teléfono" required>
      </label>

      <div class="controls">
        <button id="generate" type="submit">Generar 3 números</button>
        <button id="generate-and-save" type="button" class="secondary">Generar y guardar registro</button>
      </div>
      <p class="muted">Los números generados estarán entre <strong>0100</strong> y <strong>9999</strong>. Debes ingresar nombre y teléfono antes de generar.</p>
    </form>

    <div id="last" style="margin-top:12px; display:none">
      <div class="numbers" id="numbersOutput"></div>
      <div class="muted small" id="lastMeta"></div>
    </div>
  </div>

  <div class="card">
    <h2 style="margin:0 0 8px 0">Registro (guardado en tu navegador)</h2>
    <div class="actions">
      <button id="exportCsv" type="button">Exportar CSV</button>
      <button id="exportJson" type="button">Exportar JSON</button>
      <label class="btn inline">
        <button id="importBtn" type="button" class="secondary">Importar JSON</button>
        <input id="importFile" type="file" accept="application/json">
      </label>
      <button id="clearAll" type="button" class="danger">Borrar todo el registro</button>
    </div>
    <p class="muted">El registro se guarda en <strong>localStorage</strong> y permanecerá mientras no lo borres desde aquí o limpies los datos del navegador.</p>

    <table id="logTable" aria-live="polite">
      <thead>
        <tr><th>Fecha / Hora</th><th>Nombre</th><th>Teléfono</th><th>Números</th><th>Acciones</th></tr>
      </thead>
      <tbody id="logBody"></tbody>
    </table>
  </div>

  <footer>
    <strong>Nota sobre privacidad y URL pública:</strong> Esta versión guarda los datos en tu navegador (no se envía a servidores). Si quieres una URL pública y permanente, sigue las instrucciones al final (GitHub Pages u otro hospedaje). 
  </footer>

  <script>
    // --- Configuración ---
    const STORAGE_KEY = 'generador_3_numeros_registro_v1';

    // --- Utilidades ---
    function pad4(n){ return String(n).padStart(4,'0'); }
    function randNumBetween(min, max){
      // inclusive
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }
    function generate3(){
      // Genera 3 números entre 0100 (100) y 9999, con ceros a la izquierda; pueden repetirse
      const min = 100, max = 9999;
      const arr = [];
      for(let i=0;i<3;i++){
        arr.push(pad4(randNumBetween(min,max)));
      }
      return arr;
    }

    // --- Storage ---
    function readLog(){
      try {
        const raw = localStorage.getItem(STORAGE_KEY);
        return raw ? JSON.parse(raw) : [];
      } catch(e){
        console.error('Error leyendo registro:', e);
        return [];
      }
    }
    function writeLog(log){
      localStorage.setItem(STORAGE_KEY, JSON.stringify(log));
      renderLog();
    }

    // --- DOM ---
    const form = document.getElementById('form');
    const nameInput = document.getElementById('name');
    const phoneInput = document.getElementById('phone');
    const numbersOutput = document.getElementById('numbersOutput');
    const last = document.getElementById('last');
    const lastMeta = document.getElementById('lastMeta');
    const generateAndSaveBtn = document.getElementById('generate-and-save');
    const logBody = document.getElementById('logBody');
    const exportCsvBtn = document.getElementById('exportCsv');
    const exportJsonBtn = document.getElementById('exportJson');
    const importFile = document.getElementById('importFile');
    const importBtn = document.getElementById('importBtn');
    const clearAllBtn = document.getElementById('clearAll');

    function showLast(numbers, meta){
      numbersOutput.innerHTML = '';
      numbers.forEach(n=>{
        const div = document.createElement('div');
        div.className = 'num';
        div.textContent = n;
        numbersOutput.appendChild(div);
      });
      lastMeta.textContent = meta;
      last.style.display = 'block';
    }

    form.addEventListener('submit', (e)=>{
      e.preventDefault();
      const name = nameInput.value.trim();
      const phone = phoneInput.value.trim();
      if(!name || !phone){ alert('Ingresa nombre y teléfono.'); return; }
      const numbers = generate3();
      const now = new Date();
      showLast(numbers, `Generado: ${now.toLocaleString()}`);
      // No guarda automáticamente a menos que pulses "Generar y guardar registro"
    });

    generateAndSaveBtn.addEventListener('click', ()=>{
      const name = nameInput.value.trim();
      const phone = phoneInput.value.trim();
      if(!name || !phone){ alert('Ingresa nombre y teléfono.'); return; }
      const numbers = generate3();
      const now = new Date();
      showLast(numbers, `Generado y guardado: ${now.toLocaleString()}`);
      const log = readLog();
      const entry = {
        id: Date.now() + '_' + Math.random().toString(36).slice(2,8),
        timestamp: now.toISOString(),
        name,
        phone,
        numbers
      };
      log.unshift(entry); // newest arriba
      writeLog(log);
    });

    function renderLog(){
      const log = readLog();
      logBody.innerHTML = '';
      if(log.length === 0){
        const tr = document.createElement('tr');
        const td = document.createElement('td');
        td.colSpan = 5;
        td.className = 'small';
        td.textContent = 'Registro vacío';
        tr.appendChild(td);
        logBody.appendChild(tr);
        return;
      }
      log.forEach(entry=>{
        const tr = document.createElement('tr');
        const tdDate = document.createElement('td');
        tdDate.textContent = new Date(entry.timestamp).toLocaleString();
        const tdName = document.createElement('td');
        tdName.textContent = entry.name;
        const tdPhone = document.createElement('td');
        tdPhone.textContent = entry.phone;
        const tdNums = document.createElement('td');
        tdNums.textContent = entry.numbers.join(' — ');
        const tdAct = document.createElement('td');

        const delBtn = document.createElement('button');
        delBtn.textContent = 'Eliminar';
        delBtn.className = 'secondary';
        delBtn.addEventListener('click', ()=>{
          if(!confirm('Eliminar esta entrada?')) return;
          const filtered = readLog().filter(e => e.id !== entry.id);
          writeLog(filtered);
        });

        const copyBtn = document.createElement('button');
        copyBtn.textContent = 'Copiar';
        copyBtn.addEventListener('click', ()=>{
          navigator.clipboard?.writeText(entry.numbers.join(', ')).then(()=> {
            alert('Números copiados al portapapeles.');
          }).catch(()=> {
            alert('No se pudo copiar (tu navegador bloqueó el portapapeles).');
          });
        });

        tdAct.appendChild(copyBtn);
        tdAct.appendChild(delBtn);

        tr.appendChild(tdDate);
        tr.appendChild(tdName);
        tr.appendChild(tdPhone);
        tr.appendChild(tdNums);
        tr.appendChild(tdAct);
        logBody.appendChild(tr);
      });
    }

    // Export CSV
    exportCsvBtn.addEventListener('click', ()=>{
      const log = readLog();
      if(log.length === 0){ alert('Registro vacío'); return; }
      const rows = [['timestamp','name','phone','n1','n2','n3']];
      log.forEach(e=>{
        rows.push([e.timestamp, e.name, e.phone, e.numbers[0], e.numbers[1], e.numbers[2]]);
      });
      const csv = rows.map(r => r.map(field => {
        if(field === null || field === undefined) return '';
        const s = String(field).replace(/"/g,'""');
        return `"${s}"`;
      }).join(',')).join('\\n');
      const blob = new Blob([csv], {type:'text/csv;charset=utf-8;'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'registro_numeros.csv';
      document.body.appendChild(a);
      a.click();
      a.remove();
      URL.revokeObjectURL(url);
    });

    // Export JSON
    exportJsonBtn.addEventListener('click', ()=>{
      const log = readLog();
      if(log.length === 0){ alert('Registro vacío'); return; }
      const blob = new Blob([JSON.stringify(log, null, 2)], {type:'application/json'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'registro_numeros.json';
      document.body.appendChild(a);
      a.click();
      a.remove();
      URL.revokeObjectURL(url);
    });

    // Import JSON
    importFile.addEventListener('change', (ev)=>{
      const f = ev.target.files[0];
      if(!f) return;
      const reader = new FileReader();
      reader.onload = function(e){
        try {
          const parsed = JSON.parse(e.target.result);
          if(!Array.isArray(parsed)){ alert('JSON inválido: se esperaba un arreglo de entradas.'); return; }
          // opcional: concatenar al registro actual
          const current = readLog();
          const merged = parsed.concat(current);
          writeLog(merged);
          alert('Importado con éxito. Las nuevas entradas se añadieron al inicio del registro.');
        } catch(err){
          alert('Error al leer JSON: ' + err.message);
        }
      };
      reader.readAsText(f);
      // reset
      importFile.value = '';
    });

    importBtn.addEventListener('click', ()=> importFile.click());

    clearAllBtn.addEventListener('click', ()=>{
      if(!confirm('¿Borrar todo el registro? Esta acción no se puede deshacer.')) return;
      localStorage.removeItem(STORAGE_KEY);
      renderLog();
    });

    // init
    renderLog();

    // Optional: keep name/phone in session for convenience (not required)
    const NAME_KEY = 'generador_last_name_v1';
    const PHONE_KEY = 'generador_last_phone_v1';
    // restore
    try {
      const lastName = localStorage.getItem(NAME_KEY);
      const lastPhone = localStorage.getItem(PHONE_KEY);
      if(lastName) nameInput.value = lastName;
      if(lastPhone) phoneInput.value = lastPhone;
    } catch(e){}
    // save on change
    nameInput.addEventListener('input', ()=> localStorage.setItem(NAME_KEY, nameInput.value));
    phoneInput.addEventListener('input', ()=> localStorage.setItem(PHONE_KEY, phoneInput.value));
  </script>
</body>
</html>
