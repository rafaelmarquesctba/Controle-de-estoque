<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EstoqueApp</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
<style>
  /* ── Reset & Base ── */
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --verde:       #2d6a4f;
    --verde-mid:   #40916c;
    --verde-light: #d8f3dc;
    --verde-pale:  #f0faf3;
    --amber:       #e9a825;
    --red:         #c0392b;
    --red-light:   #fdecea;
    --blue:        #1a6fa0;
    --blue-light:  #e8f4fb;
    --text:        #1a1a2e;
    --text-muted:  #6b7280;
    --border:      #e5e7eb;
    --bg:          #f8fafb;
    --surface:     #ffffff;
    --radius:      10px;
    --shadow:      0 1px 3px rgba(0,0,0,.08), 0 1px 2px rgba(0,0,0,.04);
    --shadow-md:   0 4px 12px rgba(0,0,0,.10);
  }
  body { font-family: 'Segoe UI', system-ui, sans-serif; background: var(--bg); color: var(--text); min-height: 100vh; }

  /* ── Layout ── */
  .app { display: flex; min-height: 100vh; }
  .sidebar {
    width: 220px; min-height: 100vh; background: var(--verde);
    display: flex; flex-direction: column; padding: 0; flex-shrink: 0;
    position: fixed; top: 0; left: 0; z-index: 100;
  }
  .sidebar-brand {
    padding: 20px 20px 16px; border-bottom: 1px solid rgba(255,255,255,.15);
  }
  .sidebar-brand h1 { font-size: 18px; font-weight: 700; color: #fff; letter-spacing: -.3px; }
  .sidebar-brand p  { font-size: 11px; color: rgba(255,255,255,.6); margin-top: 2px; }
  .sidebar-nav { padding: 12px 0; flex: 1; }
  .nav-item {
    display: flex; align-items: center; gap: 10px;
    padding: 10px 20px; color: rgba(255,255,255,.75); cursor: pointer;
    font-size: 14px; transition: all .15s; border-left: 3px solid transparent;
    user-select: none;
  }
  .nav-item:hover { background: rgba(255,255,255,.1); color: #fff; }
  .nav-item.active { background: rgba(255,255,255,.15); color: #fff; border-left-color: #b7e4c7; font-weight: 600; }
  .nav-icon { font-size: 18px; width: 20px; text-align: center; }
  .main { margin-left: 220px; flex: 1; padding: 28px 32px; max-width: 1200px; }

  /* ── Setup banner ── */
  #setup-banner {
    background: #fffbeb; border: 1px solid #fbbf24; border-radius: var(--radius);
    padding: 16px 20px; margin-bottom: 24px; display: none;
    align-items: center; gap: 16px; flex-wrap: wrap;
  }
  #setup-banner.show { display: flex; }
  #setup-banner p { font-size: 14px; color: #92400e; flex: 1; }
  #setup-banner input {
    flex: 1; min-width: 240px; padding: 8px 12px; border: 1px solid #fbbf24;
    border-radius: 6px; font-size: 14px; background: #fff;
  }
  #setup-banner button { padding: 8px 18px; background: var(--amber); border: none; border-radius: 6px; font-weight: 600; color: #fff; cursor: pointer; white-space: nowrap; }

  /* ── Page sections ── */
  .page { display: none; }
  .page.active { display: block; }
  .page-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; flex-wrap: wrap; gap: 12px; }
  .page-header h2 { font-size: 22px; font-weight: 700; color: var(--text); }

  /* ── Cards métricas ── */
  .metrics { display: grid; grid-template-columns: repeat(auto-fit, minmax(170px, 1fr)); gap: 16px; margin-bottom: 28px; }
  .metric-card {
    background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius);
    padding: 18px 20px; box-shadow: var(--shadow);
  }
  .metric-label { font-size: 12px; color: var(--text-muted); text-transform: uppercase; letter-spacing: .5px; margin-bottom: 6px; }
  .metric-value { font-size: 26px; font-weight: 700; color: var(--text); }
  .metric-value.green  { color: var(--verde); }
  .metric-value.red    { color: var(--red); }
  .metric-value.amber  { color: var(--amber); }

  /* ── Tabelas ── */
  .table-wrap { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); overflow: hidden; box-shadow: var(--shadow); }
  table { width: 100%; border-collapse: collapse; font-size: 14px; }
  thead th { background: var(--verde-pale); padding: 12px 16px; text-align: left; font-weight: 600; font-size: 12px; color: var(--verde); text-transform: uppercase; letter-spacing: .5px; border-bottom: 1px solid var(--border); }
  tbody tr { border-bottom: 1px solid var(--border); transition: background .1s; }
  tbody tr:last-child { border-bottom: none; }
  tbody tr:hover { background: var(--verde-pale); }
  tbody td { padding: 12px 16px; color: var(--text); vertical-align: middle; }
  .badge { display: inline-flex; align-items: center; padding: 3px 10px; border-radius: 20px; font-size: 12px; font-weight: 600; }
  .badge-compra { background: var(--blue-light); color: var(--blue); }
  .badge-venda  { background: var(--verde-light); color: var(--verde); }
  .badge-baixo  { background: var(--red-light); color: var(--red); }
  .badge-ok     { background: var(--verde-light); color: var(--verde); }

  /* ── Botões ── */
  .btn {
    display: inline-flex; align-items: center; gap: 6px;
    padding: 9px 18px; border-radius: 8px; font-size: 14px; font-weight: 600;
    cursor: pointer; border: none; transition: all .15s; white-space: nowrap;
  }
  .btn-primary { background: var(--verde); color: #fff; }
  .btn-primary:hover { background: var(--verde-mid); }
  .btn-secondary { background: #fff; color: var(--text); border: 1px solid var(--border); }
  .btn-secondary:hover { background: var(--bg); }
  .btn-danger { background: var(--red-light); color: var(--red); border: 1px solid #fca5a5; }
  .btn-danger:hover { background: #fca5a5; }
  .btn-sm { padding: 5px 12px; font-size: 13px; }

  /* ── Modal ── */
  .modal-overlay {
    position: fixed; inset: 0; background: rgba(0,0,0,.45); z-index: 200;
    display: none; align-items: center; justify-content: center; padding: 20px;
  }
  .modal-overlay.open { display: flex; }
  .modal {
    background: var(--surface); border-radius: 14px; padding: 28px; width: 100%; max-width: 500px;
    box-shadow: var(--shadow-md); max-height: 90vh; overflow-y: auto;
  }
  .modal h3 { font-size: 18px; font-weight: 700; margin-bottom: 20px; color: var(--text); }
  .form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }
  .form-grid .full { grid-column: 1 / -1; }
  .form-group { display: flex; flex-direction: column; gap: 6px; }
  .form-group label { font-size: 13px; font-weight: 600; color: var(--text-muted); }
  .form-group input, .form-group select, .form-group textarea {
    padding: 9px 12px; border: 1px solid var(--border); border-radius: 7px;
    font-size: 14px; color: var(--text); background: #fff;
    transition: border-color .15s;
  }
  .form-group input:focus, .form-group select:focus { outline: none; border-color: var(--verde-mid); }
  .modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 22px; }

  /* ── Filtros ── */
  .filters { display: flex; gap: 12px; margin-bottom: 20px; flex-wrap: wrap; }
  .filters input, .filters select {
    padding: 8px 12px; border: 1px solid var(--border); border-radius: 7px;
    font-size: 14px; background: var(--surface);
  }
  .filters input { flex: 1; min-width: 180px; }

  /* ── Gráfico ── */
  .chart-row { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 28px; }
  .chart-card { background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); padding: 20px; box-shadow: var(--shadow); }
  .chart-card h3 { font-size: 14px; font-weight: 600; color: var(--text-muted); margin-bottom: 16px; }

  /* ── Alerta estoque baixo ── */
  .alerta-card { background: var(--red-light); border: 1px solid #fca5a5; border-radius: var(--radius); padding: 16px 20px; margin-bottom: 24px; }
  .alerta-card h3 { font-size: 14px; font-weight: 700; color: var(--red); margin-bottom: 10px; }
  .alerta-list { list-style: none; display: flex; flex-wrap: wrap; gap: 8px; }
  .alerta-list li { background: #fff; border: 1px solid #fca5a5; border-radius: 20px; padding: 4px 12px; font-size: 13px; color: var(--red); }

  /* ── Loading ── */
  .loading { text-align: center; padding: 48px; color: var(--text-muted); font-size: 15px; }
  .spinner { display: inline-block; width: 32px; height: 32px; border: 3px solid var(--verde-light); border-top-color: var(--verde); border-radius: 50%; animation: spin .7s linear infinite; margin-bottom: 12px; }
  @keyframes spin { to { transform: rotate(360deg); } }

  /* ── Toast ── */
  #toast {
    position: fixed; bottom: 28px; right: 28px; background: var(--text); color: #fff;
    padding: 12px 20px; border-radius: 10px; font-size: 14px; z-index: 999;
    opacity: 0; transform: translateY(10px); transition: all .25s; pointer-events: none;
  }
  #toast.show { opacity: 1; transform: translateY(0); }
  #toast.error { background: var(--red); }
  #toast.success { background: var(--verde); }

  /* ── Empty state ── */
  .empty { text-align: center; padding: 48px 20px; color: var(--text-muted); }
  .empty p { font-size: 15px; margin-top: 8px; }

  /* ── Responsive ── */
  @media (max-width: 768px) {
    .sidebar { width: 100%; min-height: auto; position: relative; flex-direction: row; flex-wrap: wrap; }
    .main { margin-left: 0; padding: 16px; }
    .sidebar-nav { display: flex; flex-wrap: wrap; padding: 8px; }
    .nav-item { border-left: none; border-bottom: 2px solid transparent; padding: 8px 12px; font-size: 13px; }
    .nav-item.active { border-left-color: transparent; border-bottom-color: #b7e4c7; }
    .chart-row { grid-template-columns: 1fr; }
    .form-grid { grid-template-columns: 1fr; }
    .app { flex-direction: column; }
  }
</style>
</head>
<body>

<div class="app">
  <!-- Sidebar -->
  <aside class="sidebar">
    <div class="sidebar-brand">
      <h1>📦 EstoqueApp</h1>
      <p>Controle de estoque</p>
    </div>
    <nav class="sidebar-nav">
      <div class="nav-item active" data-page="dashboard">
        <span class="nav-icon">📊</span> Dashboard
      </div>
      <div class="nav-item" data-page="produtos">
        <span class="nav-icon">🗂️</span> Produtos
      </div>
      <div class="nav-item" data-page="compras">
        <span class="nav-icon">🛒</span> Compras
      </div>
      <div class="nav-item" data-page="vendas">
        <span class="nav-icon">💰</span> Vendas
      </div>
      <div class="nav-item" data-page="relatorios">
        <span class="nav-icon">📈</span> Relatórios
      </div>
    </nav>
  </aside>

  <!-- Main -->
  <main class="main">
    <!-- Setup Banner -->
    <div id="setup-banner" class="show">
      <p>⚙️ <strong>Configuração inicial:</strong> cole a URL do seu Google Apps Script para conectar à planilha.</p>
      <input type="text" id="api-url-input" placeholder="https://script.google.com/macros/s/SEU_ID/exec">
      <button onclick="salvarConfig()">Conectar</button>
    </div>

    <!-- ── DASHBOARD ── -->
    <div id="page-dashboard" class="page active">
      <div class="page-header">
        <h2>Dashboard</h2>
        <button class="btn btn-secondary btn-sm" onclick="carregarDashboard()">🔄 Atualizar</button>
      </div>
      <div id="dash-content"><div class="loading"><div class="spinner"></div><br>Carregando...</div></div>
    </div>

    <!-- ── PRODUTOS ── -->
    <div id="page-produtos" class="page">
      <div class="page-header">
        <h2>Produtos</h2>
        <button class="btn btn-primary" onclick="abrirModalProduto()">+ Novo produto</button>
      </div>
      <div class="filters">
        <input type="text" id="filtro-produto" placeholder="Buscar produto..." oninput="renderProdutos()">
        <select id="filtro-categoria" onchange="renderProdutos()">
          <option value="">Todas categorias</option>
        </select>
      </div>
      <div class="table-wrap">
        <table>
          <thead>
            <tr>
              <th>Produto</th><th>Categoria</th><th>Estoque</th><th>Mín.</th><th>Custo</th><th>Venda</th><th>Status</th><th></th>
            </tr>
          </thead>
          <tbody id="tbody-produtos">
            <tr><td colspan="8"><div class="loading"><div class="spinner"></div></div></td></tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ── COMPRAS ── -->
    <div id="page-compras" class="page">
      <div class="page-header">
        <h2>Compras (Entrada)</h2>
        <button class="btn btn-primary" onclick="abrirModalMovimento('compra')">+ Registrar compra</button>
      </div>
      <div class="table-wrap">
        <table>
          <thead>
            <tr><th>Data</th><th>Produto</th><th>Qtd</th><th>Preço unit.</th><th>Total</th><th>Observação</th></tr>
          </thead>
          <tbody id="tbody-compras">
            <tr><td colspan="6"><div class="loading"><div class="spinner"></div></div></td></tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ── VENDAS ── -->
    <div id="page-vendas" class="page">
      <div class="page-header">
        <h2>Vendas (Saída)</h2>
        <button class="btn btn-primary" onclick="abrirModalMovimento('venda')">+ Registrar venda</button>
      </div>
      <div class="table-wrap">
        <table>
          <thead>
            <tr><th>Data</th><th>Produto</th><th>Qtd</th><th>Preço unit.</th><th>Total</th><th>Observação</th></tr>
          </thead>
          <tbody id="tbody-vendas">
            <tr><td colspan="6"><div class="loading"><div class="spinner"></div></div></td></tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ── RELATÓRIOS ── -->
    <div id="page-relatorios" class="page">
      <div class="page-header">
        <h2>Relatórios</h2>
        <button class="btn btn-secondary btn-sm" onclick="exportarCSV()">⬇️ Exportar CSV</button>
      </div>
      <div id="rel-content"><div class="loading"><div class="spinner"></div></div></div>
    </div>
  </main>
</div>

<!-- ── Modal Produto ── -->
<div class="modal-overlay" id="modal-produto">
  <div class="modal">
    <h3 id="modal-produto-titulo">Novo produto</h3>
    <div class="form-grid">
      <div class="form-group full">
        <label>Nome do produto *</label>
        <input type="text" id="p-nome" placeholder="Ex: Caderno universitário">
      </div>
      <div class="form-group">
        <label>Categoria</label>
        <input type="text" id="p-categoria" placeholder="Ex: Papelaria">
      </div>
      <div class="form-group">
        <label>Unidade</label>
        <select id="p-unidade">
          <option value="un">un (unidade)</option>
          <option value="kg">kg</option>
          <option value="g">g (grama)</option>
          <option value="l">l (litro)</option>
          <option value="m">m (metro)</option>
          <option value="cx">cx (caixa)</option>
          <option value="pct">pct (pacote)</option>
        </select>
      </div>
      <div class="form-group">
        <label>Preço de custo (R$)</label>
        <input type="number" id="p-custo" step="0.01" placeholder="0,00">
      </div>
      <div class="form-group">
        <label>Preço de venda (R$)</label>
        <input type="number" id="p-venda" step="0.01" placeholder="0,00">
      </div>
      <div class="form-group">
        <label>Estoque inicial</label>
        <input type="number" id="p-estoque" placeholder="0">
      </div>
      <div class="form-group">
        <label>Estoque mínimo</label>
        <input type="number" id="p-minimo" placeholder="5">
      </div>
    </div>
    <div class="modal-actions">
      <button class="btn btn-secondary" onclick="fecharModais()">Cancelar</button>
      <button class="btn btn-primary" onclick="salvarProduto()">Salvar produto</button>
    </div>
  </div>
</div>

<!-- ── Modal Movimento ── -->
<div class="modal-overlay" id="modal-movimento">
  <div class="modal">
    <h3 id="modal-mov-titulo">Registrar movimento</h3>
    <div class="form-grid">
      <div class="form-group full">
        <label>Produto *</label>
        <select id="m-produto">
          <option value="">Selecione um produto...</option>
        </select>
      </div>
      <div class="form-group">
        <label>Quantidade *</label>
        <input type="number" id="m-quantidade" min="0.01" step="0.01" placeholder="1">
      </div>
      <div class="form-group">
        <label>Preço unitário (R$) *</label>
        <input type="number" id="m-preco" step="0.01" placeholder="0,00">
      </div>
      <div class="form-group full">
        <label>Observação</label>
        <textarea id="m-obs" rows="2" placeholder="Opcional..." style="resize:vertical"></textarea>
      </div>
    </div>
    <div id="m-total-preview" style="font-size:14px;color:var(--verde);font-weight:600;margin-top:4px;"></div>
    <div class="modal-actions">
      <button class="btn btn-secondary" onclick="fecharModais()">Cancelar</button>
      <button class="btn btn-primary" id="btn-salvar-mov" onclick="salvarMovimento()">Salvar</button>
    </div>
  </div>
</div>

<!-- Toast -->
<div id="toast"></div>

<script>
// ── Estado global ──
let API_URL = localStorage.getItem('estoque_api_url') || '';
let produtos = [];
let movimentos = [];
let tipoMovimento = 'compra';
let produtoEditando = null;
let chartComprasVendas = null;
let chartEstoque = null;

// ── Init ──
window.addEventListener('DOMContentLoaded', () => {
  if (API_URL) {
    document.getElementById('setup-banner').classList.remove('show');
    document.getElementById('api-url-input').value = API_URL;
    inicializar();
  }
  document.querySelectorAll('.nav-item').forEach(item => {
    item.addEventListener('click', () => navegarPara(item.dataset.page));
  });
  // Preview total no modal de movimento
  ['m-quantidade','m-preco'].forEach(id => {
    document.getElementById(id).addEventListener('input', atualizarPreviewTotal);
  });
});

function inicializar() {
  carregarDashboard();
  carregarProdutos();
  carregarMovimentos();
}

// ── Navegação ──
function navegarPara(page) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  document.getElementById('page-' + page).classList.add('active');
  document.querySelector(`[data-page="${page}"]`).classList.add('active');
  if (page === 'relatorios') renderRelatorios();
}

// ── Config ──
function salvarConfig() {
  const url = document.getElementById('api-url-input').value.trim();
  if (!url || !url.startsWith('https://')) { toast('URL inválida.', 'error'); return; }
  API_URL = url;
  localStorage.setItem('estoque_api_url', url);
  document.getElementById('setup-banner').classList.remove('show');
  toast('Conectado! Configurando planilha...', 'success');
  api({ action: 'setup' }).then(() => { toast('Planilha configurada!', 'success'); inicializar(); });
}

// ── API ──
async function api(payload) {
  if (!API_URL) { toast('Configure a URL da API primeiro.', 'error'); return null; }
  try {
    const res = await fetch(API_URL, {
      method: 'POST',
      body: JSON.stringify(payload),
      headers: { 'Content-Type': 'application/json' }
    });
    const data = await res.json();
    if (data.erro) { toast('Erro: ' + data.erro, 'error'); return null; }
    return data;
  } catch (e) {
    toast('Falha na comunicação com a API.', 'error');
    return null;
  }
}

// ── Dashboard ──
async function carregarDashboard() {
  const data = await api({ action: 'getDashboard' });
  if (!data) return;
  const lucroClass = data.lucroBrutoMes >= 0 ? 'green' : 'red';
  const lucroBruto = formatR(data.lucroBrutoMes);

  let alertaHTML = '';
  if (data.estoqueBaixo && data.estoqueBaixo.length > 0) {
    const items = data.estoqueBaixo.map(p =>
      `<li>⚠️ ${p.nome} — ${p.estoque_atual} ${p.unidade || 'un'}</li>`
    ).join('');
    alertaHTML = `<div class="alerta-card"><h3>🚨 Estoque abaixo do mínimo</h3><ul class="alerta-list">${items}</ul></div>`;
  }

  let ultimosHTML = '';
  if (data.ultimosMovimentos && data.ultimosMovimentos.length > 0) {
    const rows = data.ultimosMovimentos.map(m => `
      <tr>
        <td>${formatData(m.data)}</td>
        <td>${m.produto_nome}</td>
        <td><span class="badge badge-${m.tipo}">${m.tipo === 'compra' ? '🛒 Compra' : '💰 Venda'}</span></td>
        <td>${m.quantidade}</td>
        <td>${formatR(m.total)}</td>
      </tr>`).join('');
    ultimosHTML = `
      <h3 style="font-size:15px;font-weight:700;margin:28px 0 14px">Últimas movimentações</h3>
      <div class="table-wrap"><table>
        <thead><tr><th>Data</th><th>Produto</th><th>Tipo</th><th>Qtd</th><th>Total</th></tr></thead>
        <tbody>${rows}</tbody>
      </table></div>`;
  }

  document.getElementById('dash-content').innerHTML = `
    ${alertaHTML}
    <div class="metrics">
      <div class="metric-card">
        <div class="metric-label">Total de produtos</div>
        <div class="metric-value">${data.totalProdutos}</div>
      </div>
      <div class="metric-card">
        <div class="metric-label">Compras no mês</div>
        <div class="metric-value red">${formatR(data.totalComprasMes)}</div>
      </div>
      <div class="metric-card">
        <div class="metric-label">Vendas no mês</div>
        <div class="metric-value green">${formatR(data.totalVendasMes)}</div>
      </div>
      <div class="metric-card">
        <div class="metric-label">Lucro bruto (mês)</div>
        <div class="metric-value ${lucroClass}">${lucroBruto}</div>
      </div>
    </div>
    <div class="chart-row">
      <div class="chart-card"><h3>Compras x Vendas — últimos 6 meses</h3><canvas id="chart-cv" height="200"></canvas></div>
      <div class="chart-card"><h3>Itens com estoque baixo</h3><canvas id="chart-estoque" height="200"></canvas></div>
    </div>
    ${ultimosHTML}
  `;

  // Gráfico Compras x Vendas
  const ctxCV = document.getElementById('chart-cv').getContext('2d');
  if (chartComprasVendas) chartComprasVendas.destroy();
  chartComprasVendas = new Chart(ctxCV, {
    type: 'bar',
    data: {
      labels: data.grafico.map(g => g.mes),
      datasets: [
        { label: 'Compras', data: data.grafico.map(g => g.compras), backgroundColor: '#c0392b33', borderColor: '#c0392b', borderWidth: 2, borderRadius: 6 },
        { label: 'Vendas',  data: data.grafico.map(g => g.vendas),  backgroundColor: '#2d6a4f33', borderColor: '#2d6a4f', borderWidth: 2, borderRadius: 6 }
      ]
    },
    options: { responsive: true, plugins: { legend: { position: 'bottom' } }, scales: { y: { beginAtZero: true, ticks: { callback: v => 'R$' + v.toLocaleString('pt-BR') } } } }
  });

  // Gráfico estoque baixo
  const ctxE = document.getElementById('chart-estoque').getContext('2d');
  if (chartEstoque) chartEstoque.destroy();
  const baixo = (data.estoqueBaixo || []).slice(0, 8);
  chartEstoque = new Chart(ctxE, {
    type: 'bar',
    data: {
      labels: baixo.map(p => p.nome.substring(0, 16)),
      datasets: [{
        label: 'Estoque atual', data: baixo.map(p => p.estoque_atual),
        backgroundColor: '#e9a82566', borderColor: '#e9a825', borderWidth: 2, borderRadius: 6
      }]
    },
    options: {
      indexAxis: 'y', responsive: true,
      plugins: { legend: { display: false } },
      scales: { x: { beginAtZero: true } }
    }
  });
}

// ── Produtos ──
async function carregarProdutos() {
  const data = await api({ action: 'getProdutos' });
  if (!data) return;
  produtos = data.filter(p => p.ativo === true || p.ativo === 'TRUE' || p.ativo === 'true');
  renderProdutos();
  atualizarSelectProdutos();
  atualizarFiltrosCategoria();
}

function renderProdutos() {
  const busca = (document.getElementById('filtro-produto').value || '').toLowerCase();
  const cat   = document.getElementById('filtro-categoria').value;
  const filtrados = produtos.filter(p => {
    const matchBusca = !busca || p.nome.toLowerCase().includes(busca);
    const matchCat   = !cat || p.categoria === cat;
    return matchBusca && matchCat;
  });

  const tbody = document.getElementById('tbody-produtos');
  if (!filtrados.length) {
    tbody.innerHTML = `<tr><td colspan="8"><div class="empty"><p>Nenhum produto encontrado.</p></div></td></tr>`;
    return;
  }
  tbody.innerHTML = filtrados.map(p => {
    const atual = Number(p.estoque_atual);
    const min   = Number(p.estoque_minimo);
    const status = atual <= min
      ? '<span class="badge badge-baixo">⚠️ Baixo</span>'
      : '<span class="badge badge-ok">✅ OK</span>';
    const margem = p.preco_venda > 0 && p.preco_custo > 0
      ? ((p.preco_venda - p.preco_custo) / p.preco_custo * 100).toFixed(1) + '%'
      : '—';
    return `<tr>
      <td><strong>${p.nome}</strong></td>
      <td>${p.categoria || '—'}</td>
      <td><strong>${atual}</strong> ${p.unidade || 'un'}</td>
      <td>${min}</td>
      <td>${formatR(p.preco_custo)}</td>
      <td>${formatR(p.preco_venda)} <small style="color:var(--verde);font-size:11px">(${margem})</small></td>
      <td>${status}</td>
      <td>
        <button class="btn btn-secondary btn-sm" onclick='editarProduto(${JSON.stringify(p)})'>✏️</button>
        <button class="btn btn-danger btn-sm" onclick="excluirProduto('${p.id}')">🗑️</button>
      </td>
    </tr>`;
  }).join('');
}

function atualizarFiltrosCategoria() {
  const cats = [...new Set(produtos.map(p => p.categoria).filter(Boolean))].sort();
  const sel = document.getElementById('filtro-categoria');
  sel.innerHTML = '<option value="">Todas categorias</option>' + cats.map(c => `<option value="${c}">${c}</option>`).join('');
}

function atualizarSelectProdutos() {
  const sel = document.getElementById('m-produto');
  sel.innerHTML = '<option value="">Selecione um produto...</option>' +
    produtos.map(p => `<option value="${p.id}" data-nome="${p.nome}" data-custo="${p.preco_custo}" data-venda="${p.preco_venda}">${p.nome} (estq: ${p.estoque_atual})</option>`).join('');
}

function abrirModalProduto() {
  produtoEditando = null;
  document.getElementById('modal-produto-titulo').textContent = 'Novo produto';
  ['p-nome','p-categoria','p-custo','p-venda','p-estoque','p-minimo'].forEach(id => document.getElementById(id).value = '');
  document.getElementById('p-unidade').value = 'un';
  document.getElementById('modal-produto').classList.add('open');
}

function editarProduto(p) {
  produtoEditando = p;
  document.getElementById('modal-produto-titulo').textContent = 'Editar produto';
  document.getElementById('p-nome').value     = p.nome;
  document.getElementById('p-categoria').value = p.categoria || '';
  document.getElementById('p-unidade').value  = p.unidade || 'un';
  document.getElementById('p-custo').value    = p.preco_custo;
  document.getElementById('p-venda').value    = p.preco_venda;
  document.getElementById('p-estoque').value  = p.estoque_atual;
  document.getElementById('p-minimo').value   = p.estoque_minimo;
  document.getElementById('modal-produto').classList.add('open');
}

async function salvarProduto() {
  const nome = document.getElementById('p-nome').value.trim();
  if (!nome) { toast('Informe o nome do produto.', 'error'); return; }

  const payload = {
    nome,
    categoria:     document.getElementById('p-categoria').value.trim(),
    unidade:       document.getElementById('p-unidade').value,
    preco_custo:   document.getElementById('p-custo').value,
    preco_venda:   document.getElementById('p-venda').value,
    estoque_atual: document.getElementById('p-estoque').value || 0,
    estoque_minimo:document.getElementById('p-minimo').value || 5
  };

  if (produtoEditando) {
    payload.action = 'editProduto';
    payload.id = produtoEditando.id;
  } else {
    payload.action = 'addProduto';
  }

  const res = await api(payload);
  if (res?.ok) {
    fecharModais();
    toast(produtoEditando ? 'Produto atualizado!' : 'Produto criado!', 'success');
    carregarProdutos();
  }
}

async function excluirProduto(id) {
  if (!confirm('Remover este produto?')) return;
  const res = await api({ action: 'deleteProduto', id });
  if (res?.ok) { toast('Produto removido.', 'success'); carregarProdutos(); }
}

// ── Movimentos ──
async function carregarMovimentos() {
  const data = await api({ action: 'getMovimentos' });
  if (!data) return;
  movimentos = data;
  renderMovimentos();
}

function renderMovimentos() {
  ['compras','vendas'].forEach(tipo => {
    const filtrados = movimentos.filter(m => m.tipo === tipo.slice(0,-1));
    const tbody = document.getElementById('tbody-' + tipo);
    if (!filtrados.length) {
      tbody.innerHTML = `<tr><td colspan="6"><div class="empty"><p>Nenhuma ${tipo === 'compras' ? 'compra' : 'venda'} registrada.</p></div></td></tr>`;
      return;
    }
    tbody.innerHTML = filtrados.map(m => `<tr>
      <td>${formatData(m.data)}</td>
      <td>${m.produto_nome}</td>
      <td>${m.quantidade}</td>
      <td>${formatR(m.preco_unitario)}</td>
      <td><strong>${formatR(m.total)}</strong></td>
      <td style="color:var(--text-muted);font-size:13px">${m.observacao || '—'}</td>
    </tr>`).join('');
  });
}

function abrirModalMovimento(tipo) {
  tipoMovimento = tipo;
  document.getElementById('modal-mov-titulo').textContent = tipo === 'compra' ? '🛒 Registrar compra' : '💰 Registrar venda';
  document.getElementById('btn-salvar-mov').textContent = tipo === 'compra' ? 'Salvar compra' : 'Salvar venda';
  ['m-quantidade','m-preco','m-obs'].forEach(id => document.getElementById(id).value = '');
  document.getElementById('m-produto').value = '';
  document.getElementById('m-total-preview').textContent = '';
  // Preenche preço sugerido ao selecionar produto
  document.getElementById('m-produto').onchange = function() {
    const opt = this.options[this.selectedIndex];
    const preco = tipo === 'compra' ? opt.dataset.custo : opt.dataset.venda;
    if (preco) document.getElementById('m-preco').value = preco;
    atualizarPreviewTotal();
  };
  document.getElementById('modal-movimento').classList.add('open');
}

function atualizarPreviewTotal() {
  const q = parseFloat(document.getElementById('m-quantidade').value) || 0;
  const p = parseFloat(document.getElementById('m-preco').value) || 0;
  const total = q * p;
  document.getElementById('m-total-preview').textContent = total > 0 ? `Total: ${formatR(total)}` : '';
}

async function salvarMovimento() {
  const selProd = document.getElementById('m-produto');
  const produtoId = selProd.value;
  const produtoNome = selProd.options[selProd.selectedIndex]?.dataset?.nome || '';
  const quantidade = parseFloat(document.getElementById('m-quantidade').value);
  const preco = parseFloat(document.getElementById('m-preco').value);

  if (!produtoId) { toast('Selecione um produto.', 'error'); return; }
  if (!quantidade || quantidade <= 0) { toast('Quantidade inválida.', 'error'); return; }
  if (!preco || preco <= 0) { toast('Preço inválido.', 'error'); return; }

  const res = await api({
    action: 'addMovimento',
    produto_id: produtoId,
    produto_nome: produtoNome,
    tipo: tipoMovimento,
    quantidade,
    preco_unitario: preco,
    observacao: document.getElementById('m-obs').value.trim()
  });

  if (res?.ok) {
    fecharModais();
    toast(tipoMovimento === 'compra' ? 'Compra registrada!' : 'Venda registrada!', 'success');
    carregarProdutos();
    carregarMovimentos();
    carregarDashboard();
  }
}

// ── Relatórios ──
function renderRelatorios() {
  const totalCompras = movimentos.filter(m => m.tipo === 'compra').reduce((s, m) => s + Number(m.total), 0);
  const totalVendas  = movimentos.filter(m => m.tipo === 'venda').reduce((s, m) => s + Number(m.total), 0);
  const lucro = totalVendas - totalCompras;

  // Top 5 produtos mais vendidos
  const vendas = movimentos.filter(m => m.tipo === 'venda');
  const porProduto = {};
  vendas.forEach(m => {
    porProduto[m.produto_nome] = (porProduto[m.produto_nome] || 0) + Number(m.total);
  });
  const topProdutos = Object.entries(porProduto).sort((a,b) => b[1]-a[1]).slice(0,5);

  const topHTML = topProdutos.length ? topProdutos.map(([nome, total], i) => `
    <tr>
      <td>${i+1}. ${nome}</td>
      <td><strong>${formatR(total)}</strong></td>
    </tr>`).join('') : '<tr><td colspan="2">Sem vendas ainda.</td></tr>';

  document.getElementById('rel-content').innerHTML = `
    <div class="metrics">
      <div class="metric-card"><div class="metric-label">Total de compras (geral)</div><div class="metric-value red">${formatR(totalCompras)}</div></div>
      <div class="metric-card"><div class="metric-label">Total de vendas (geral)</div><div class="metric-value green">${formatR(totalVendas)}</div></div>
      <div class="metric-card"><div class="metric-label">Lucro bruto acumulado</div><div class="metric-value ${lucro >= 0 ? 'green' : 'red'}">${formatR(lucro)}</div></div>
      <div class="metric-card"><div class="metric-label">Total de movimentações</div><div class="metric-value">${movimentos.length}</div></div>
    </div>
    <h3 style="font-size:15px;font-weight:700;margin:24px 0 14px">Top 5 produtos por receita</h3>
    <div class="table-wrap" style="max-width:420px"><table>
      <thead><tr><th>Produto</th><th>Receita total</th></tr></thead>
      <tbody>${topHTML}</tbody>
    </table></div>
  `;
}

// ── Exportar CSV ──
function exportarCSV() {
  const cabecalho = 'Data,Produto,Tipo,Quantidade,Preço unitário,Total,Observação\n';
  const linhas = movimentos.map(m =>
    `"${formatData(m.data)}","${m.produto_nome}","${m.tipo}",${m.quantidade},${m.preco_unitario},${m.total},"${m.observacao || ''}"`
  ).join('\n');
  const blob = new Blob(['\uFEFF' + cabecalho + linhas], { type: 'text/csv;charset=utf-8;' });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement('a');
  a.href = url; a.download = 'movimentos_estoque.csv'; a.click();
  URL.revokeObjectURL(url);
  toast('CSV exportado!', 'success');
}

// ── Helpers ──
function fecharModais() {
  document.querySelectorAll('.modal-overlay').forEach(m => m.classList.remove('open'));
}
document.querySelectorAll('.modal-overlay').forEach(overlay => {
  overlay.addEventListener('click', e => { if (e.target === overlay) fecharModais(); });
});

function formatR(v) {
  return 'R$ ' + (Number(v) || 0).toLocaleString('pt-BR', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}

function formatData(iso) {
  if (!iso) return '—';
  const d = new Date(iso);
  return d.toLocaleDateString('pt-BR') + ' ' + d.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' });
}

let toastTimer;
function toast(msg, type = '') {
  const el = document.getElementById('toast');
  el.textContent = msg;
  el.className = 'show ' + type;
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => { el.className = ''; }, 3200);
}
</script>
</body>
</html>
