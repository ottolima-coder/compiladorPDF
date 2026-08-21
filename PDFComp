<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Juntar PDFs - Time MM Shippify</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf-lib/1.17.1/pdf-lib.min.js"></script>
<style>
  body{
    font-family: Arial, Helvetica, sans-serif;
    max-width: 600px;
    margin: 40px auto;
    padding: 0 16px;
    background: #121212;
    color: #e6e6e6;
  }
  header{
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 4px;
  }
  .badge{
    background: #ff6a00;
    color: #121212;
    font-size: 11px;
    font-weight: bold;
    padding: 3px 8px;
    border-radius: 3px;
    letter-spacing: 0.03em;
  }
  h1{
    font-size: 22px;
    margin: 0;
    color: #fff;
  }
  p.desc{
    color: #aaa;
    font-size: 14px;
    margin-top: 4px;
  }
  #dropzone{
    border: 2px dashed #444;
    padding: 30px;
    text-align: center;
    cursor: pointer;
    margin-top: 20px;
    background: #1a1a1a;
    color: #ccc;
  }
  #dropzone.drag{
    border-color: #ff6a00;
    background: #1f1f1f;
  }
  input[type=file]{ display:none; }
  ul#list{
    list-style: none;
    padding: 0;
    margin-top: 20px;
  }
  ul#list li{
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 4px;
    border-bottom: 1px solid #2a2a2a;
  }
  ul#list li.dragging{ opacity: 0.4; }
  .handle{ cursor: grab; color: #666; }
  .fname{
    flex: 1;
    font-size: 14px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: #ddd;
  }
  .fpages{
    font-size: 12px;
    color: #888;
  }
  .rm{
    border: none;
    background: none;
    color: #888;
    cursor: pointer;
    font-size: 16px;
  }
  .rm:hover{ color: #ff6a00; }
  .totals{
    font-size: 13px;
    color: #999;
    margin-top: 10px;
  }
  .actions{
    margin-top: 20px;
  }
  button.primary{
    background: #ff6a00;
    color: #121212;
    border: none;
    padding: 10px 18px;
    font-size: 14px;
    font-weight: bold;
    cursor: pointer;
    border-radius: 4px;
  }
  button.primary:disabled{
    background: #444;
    color: #888;
    cursor: not-allowed;
  }
  button.secondary{
    background: none;
    border: none;
    color: #aaa;
    text-decoration: underline;
    cursor: pointer;
    font-size: 13px;
    margin-left: 12px;
  }
  #status{
    margin-top: 14px;
    font-size: 13px;
    color: #999;
  }
  #status.ok{ color: #4ade80; }
  #status.err{ color: #ff6a6a; }
  footer{
    margin-top: 40px;
    font-size: 11px;
    color: #555;
    text-align: center;
  }
</style>
</head>
<body>

<header>
  <h1>Juntar PDFs</h1>
  <span class="badge">Time MM · Shippify</span>
</header>
<p class="desc">Escolha vários arquivos PDF, reordene se precisar, e baixe tudo junto em um único arquivo.</p>

<div id="dropzone">
  Arraste os PDFs aqui ou clique para escolher
</div>
<input type="file" id="fileInput" accept="application/pdf" multiple>

<ul id="list"></ul>

<div class="totals" id="totals" style="display:none;"></div>

<div class="actions">
  <button class="primary" id="mergeBtn" disabled>Unir e baixar PDF</button>
  <button class="secondary" id="clearBtn" style="display:none;">Limpar tudo</button>
</div>

<div id="status"></div>

<footer>Time MM · Shippify</footer>

<script>
const { PDFDocument } = PDFLib;

const dropzone = document.getElementById('dropzone');
const fileInput = document.getElementById('fileInput');
const listEl = document.getElementById('list');
const mergeBtn = document.getElementById('mergeBtn');
const clearBtn = document.getElementById('clearBtn');
const statusEl = document.getElementById('status');
const totalsEl = document.getElementById('totals');

let items = [];
let dragSrcId = null;

dropzone.addEventListener('click', () => fileInput.click());
dropzone.addEventListener('dragover', e => { e.preventDefault(); dropzone.classList.add('drag'); });
dropzone.addEventListener('dragleave', () => dropzone.classList.remove('drag'));
dropzone.addEventListener('drop', e => {
  e.preventDefault();
  dropzone.classList.remove('drag');
  handleFiles(e.dataTransfer.files);
});
fileInput.addEventListener('change', e => handleFiles(e.target.files));

async function handleFiles(fileList){
  const files = Array.from(fileList).filter(f => f.type === 'application/pdf' || f.name.toLowerCase().endsWith('.pdf'));
  if(!files.length) return;
  setStatus('Lendo arquivos...', '');
  for(const file of files){
    try{
      const bytes = await file.arrayBuffer();
      const doc = await PDFDocument.load(bytes, { ignoreEncryption: true });
      items.push({
        id: crypto.randomUUID(),
        name: file.name,
        pages: doc.getPageCount(),
        bytes
      });
    }catch(err){
      setStatus(`Não foi possível ler "${file.name}".`, 'err');
    }
  }
  fileInput.value = '';
  render();
  setStatus('', '');
}

function render(){
  listEl.innerHTML = '';
  items.forEach((it, i) => {
    const li = document.createElement('li');
    li.draggable = true;
    li.dataset.id = it.id;

    li.addEventListener('dragstart', () => { dragSrcId = it.id; li.classList.add('dragging'); });
    li.addEventListener('dragend', () => li.classList.remove('dragging'));
    li.addEventListener('dragover', e => e.preventDefault());
    li.addEventListener('drop', e => {
      e.preventDefault();
      if(dragSrcId === it.id) return;
      const from = items.findIndex(x => x.id === dragSrcId);
      const to = items.findIndex(x => x.id === it.id);
      const [moved] = items.splice(from, 1);
      items.splice(to, 0, moved);
      render();
    });

    li.innerHTML = `
      <span class="handle">↕</span>
      <span class="fname" title="${it.name}">${i+1}. ${it.name}</span>
      <span class="fpages">${it.pages}p</span>
      <button class="rm" data-id="${it.id}" title="Remover">×</button>
    `;
    listEl.appendChild(li);
  });

  listEl.querySelectorAll('.rm').forEach(btn => {
    btn.addEventListener('click', (e) => {
      const id = e.currentTarget.dataset.id;
      items = items.filter(x => x.id !== id);
      render();
    });
  });

  const totalPages = items.reduce((s, it) => s + it.pages, 0);
  totalsEl.style.display = items.length ? 'block' : 'none';
  totalsEl.textContent = `${items.length} arquivo(s), ${totalPages} página(s) no total`;
  mergeBtn.disabled = items.length < 2;
  clearBtn.style.display = items.length ? 'inline' : 'none';
}

clearBtn.addEventListener('click', () => {
  items = [];
  render();
  setStatus('', '');
});

mergeBtn.addEventListener('click', async () => {
  if(items.length < 2) return;
  mergeBtn.disabled = true;
  setStatus('Unindo arquivos...', '');
  try{
    const merged = await PDFDocument.create();
    for(const it of items){
      const src = await PDFDocument.load(it.bytes, { ignoreEncryption: true });
      const pages = await merged.copyPages(src, src.getPageIndices());
      pages.forEach(p => merged.addPage(p));
    }
    const outBytes = await merged.save();
    const blob = new Blob([outBytes], { type: 'application/pdf' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'pdfs-unidos.pdf';
    document.body.appendChild(a);
    a.click();
    a.remove();
    URL.revokeObjectURL(url);
    setStatus(`Pronto! ${items.length} arquivos unidos em "pdfs-unidos.pdf".`, 'ok');
  }catch(err){
    console.error(err);
    setStatus('Erro ao unir os PDFs. Tente novamente.', 'err');
  }finally{
    mergeBtn.disabled = items.length < 2;
  }
});

function setStatus(msg, cls){
  statusEl.textContent = msg;
  statusEl.className = cls || '';
}
</script>
</body>
</html>
