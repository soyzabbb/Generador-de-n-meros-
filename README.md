

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
