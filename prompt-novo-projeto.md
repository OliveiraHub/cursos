# Prompt — Sistema de Guias de Estudo em HTML

Use este prompt ao iniciar um novo projeto de guias de estudo com a mesma metodologia e convenções do projeto Guidewire.

---

## Contexto do projeto

Vou criar um sistema de guias de estudo em HTML a partir de capturas de tela de e-learning e materiais de curso. O sistema é composto por:

1. **Uma página Hub** (`[Tema]_Home.html`) — lista todos os cursos disponíveis, funciona como índice central, e contém um glossário consolidado de todos os módulos.
2. **Páginas de guia por curso** (`[Curso]_Overview.html`) — cada curso é um único arquivo HTML com navegação por páginas internas (sidebar + paginação).
3. **CSS compartilhado** (`[tema].css`) — todo estilo comum extraído em arquivo separado; cada HTML tem apenas CSS específico no `<style>`.
4. **Pasta `images/`** — todas as imagens em kebab-case (sem espaços, sem `+`, sem parênteses), prontas para GitHub Pages.
5. **`robots.txt`** — `Disallow: /` para não indexar pelo Google.

---

## Regras absolutas de conteúdo

- **Quando eu enviar o conteúdo real de uma página, você remove 100% do texto placeholder e substitui pelo meu conteúdo.** Só a estrutura HTML/CSS permanece.
- Todo o conteúdo é escrito em **PT-BR**. Não mencione isso — é assumido.
- Onde o original usar "P&C" ou "Property & Casualty", traduzir para **"seguro de danos"**.
- Headings H1 e H2 em inglês devem ser **traduzidos para PT-BR**.
- Todo H2 recebe um atributo `id` em kebab-case (ex: `id="arquitetura-em-camadas"`).

---

## Arquitetura técnica

### CSS compartilhado (`[tema].css`)

Contém obrigatoriamente:
- Variáveis CSS no `:root`
- Reset (`box-sizing`, `margin`, `padding`)
- `body` com fonte Barlow (Google Fonts) e fallbacks
- Header (sticky, logo, botão ← Home, breadcrumb)
- Layout grid (sidebar 260px + `1fr` main)
- Sidebar: `.nav-btn`, `.nav-group-header`, dots (`.dot.done`, `.dot.current`, `.dot.pending`)
- `.page-section { display:none }` / `.page-section.active { display:block }`
- Componentes: `.callout`, `.goal-list`, `.accordion`, `.note`, `.info-box`, `.term-table`, `.tooltip-wrap/.tooltip-box`
- Paginação (`.pagination`, `.btn`)
- Menu gaveta mobile (`.menu-toggle`, `.sidebar-overlay`)
- Glossário (`.glossary-item`)
- `@media (max-width: 768px)` completo

Cada HTML tem apenas CSS **específico daquele arquivo** no `<style>`.

### Fonte
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Barlow:wght@400;500;600;700&display=swap" rel="stylesheet">
```
```css
body { font-family: 'Barlow', 'Segoe UI', system-ui, sans-serif; }
```

### `<head>` obrigatório (todos os HTMLs)
```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="robots" content="noindex, nofollow">
<meta name="description" content="[descrição da página]">
<link rel="icon" type="image/png" href="images/logo.png">
<title>[Título da Página]</title>
<!-- Barlow (Google Fonts) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Barlow:wght@400;500;600;700&display=swap" rel="stylesheet">
<!-- CSS compartilhado -->
<link rel="stylesheet" href="[tema].css">
```

---

## Variáveis CSS (adaptar cores ao tema)

```css
:root {
  --dark:    #1a2e44;   /* cor principal escura (header, sidebar) */
  --blue:    #0077b6;   /* cor de ação/link */
  --teal:    #00b4d8;   /* cor de destaque/acento */
  --light:   #f0f6fb;   /* fundo suave */
  --text:    #1e293b;
  --muted:   #64748b;
  --border:  #cbd5e1;
  --white:   #ffffff;
  --success: #22c55e;
  --warning: #f59e0b;
  --header-h: 60px;
}
```

---

## JavaScript — padrões obrigatórios

### Estrutura do bloco `<script>` (guia de curso)

```js
// SAFETY NET — garante init mesmo se o arquivo for truncado pelo formatter
const TOTAL_PAGES = N;
const _saved = parseInt(localStorage.getItem('[chave]') || '1') || 1;
let currentPage = Math.min(Math.max(_saved, 1), TOTAL_PAGES);

document.addEventListener('DOMContentLoaded', function() { goToPage(currentPage); });

function pageExists(n) { return !!document.getElementById('page-' + n); }

function goToPage(n) {
  if (n < 1 || n > TOTAL_PAGES || !pageExists(n)) return;
  document.querySelectorAll('.page-section').forEach(s => s.classList.remove('active'));
  document.getElementById('page-' + n).classList.add('active');
  currentPage = n;
  try { localStorage.setItem('[chave]', n); } catch(e) {}
  window.scrollTo({ top: 0, behavior: 'smooth' });
  updateUI();
}

function updateUI() {
  // atualiza breadcrumb com H1 da página atual
  const h1 = document.getElementById('page-' + currentPage)?.querySelector('h1');
  if (h1) { const el = document.getElementById('bc-page'); if (el) el.textContent = h1.textContent; }
  // botões prev/next
  document.getElementById('btn-prev').disabled = currentPage === 1;
  document.getElementById('btn-next').disabled = currentPage === TOTAL_PAGES;
  // dots
  document.querySelectorAll('.dot[data-page]').forEach(d => {
    const p = Number(d.dataset.page);
    d.className = 'dot ' + (p < currentPage ? 'done' : p === currentPage ? 'current' : 'pending');
  });
  // barra de progresso
  const pct = Math.round(((currentPage - 1) / (TOTAL_PAGES - 1)) * 100) || 0;
  document.getElementById('progress-fill').style.width = pct + '%';
}

// Sidebar / menu gaveta
function openSidebar()  { document.getElementById('sidebar').classList.add('open'); document.getElementById('sidebar-overlay').classList.add('active'); }
function closeSidebar() { document.getElementById('sidebar').classList.remove('open'); document.getElementById('sidebar-overlay').classList.remove('active'); }
document.getElementById('menu-toggle').addEventListener('click', openSidebar);
document.getElementById('sidebar-overlay').addEventListener('click', closeSidebar);

// Accordions
function toggleAcc(btn) {
  const body = btn.nextElementSibling;
  const open = btn.getAttribute('aria-expanded') === 'true';
  btn.setAttribute('aria-expanded', !open);
  body.style.display = open ? 'none' : 'block';
  btn.querySelector('.acc-arrow').textContent = open ? '▾' : '▴';
}

// Nav buttons
document.querySelectorAll('.nav-btn[data-page]').forEach(btn => {
  btn.addEventListener('click', () => {
    goToPage(Number(btn.dataset.page));
    if (window.innerWidth <= 768) closeSidebar();
  });
});

goToPage(currentPage);
```

**Regra crítica:** o `document.addEventListener('DOMContentLoaded', ...)` deve vir logo após `let currentPage = ...`, antes de qualquer função. Isso protege contra truncamento de arquivo pelo formatter.

### localStorage
- Cada guia usa uma chave única: ex `'gwcp_page'`, `'pc_page'`, `'[sigla]_page'`
- A barra de progresso **nunca** tem `style="width: X%"` hardcoded — é sempre calculada pelo JS

---

## Estrutura HTML da página de guia

```html
<header>
  <img src="images/logo.png" class="header-logo" alt="Logo">
  <div>
    <h1>[Nome do Curso]</h1>
  </div>
  <a href="[Tema]_Home.html" class="hub-link">← Home</a>
</header>

<div class="breadcrumb">
  <a href="[Tema]_Home.html">Home</a>
  <span class="bc-sep">›</span>
  <span>[Nome do Curso]</span>
  <span class="bc-sep">›</span>
  <span id="bc-page">Bem-vindo</span>
</div>

<div class="container">
  <aside id="sidebar">
    <!-- grupos de navegação + dots -->
  </aside>
  <div id="sidebar-overlay" class="sidebar-overlay"></div>

  <main>
    <button id="menu-toggle" class="menu-toggle" aria-label="Abrir menu">☰</button>
    <div class="progress-bar-wrap">
      <div class="progress-bar-fill" id="progress-fill"></div>
    </div>

    <!-- Páginas -->
    <section class="page-section" id="page-1"> ... </section>
    <section class="page-section" id="page-2"> ... </section>
    <!-- ... -->

    <!-- Paginação -->
    <div class="pagination">
      <button class="btn btn-secondary" id="btn-prev" onclick="goToPage(currentPage-1)">← Anterior</button>
      <span id="page-indicator">Página 1 de N</span>
      <button class="btn btn-primary"   id="btn-next" onclick="goToPage(currentPage+1)">Próximo →</button>
    </div>

    <a href="[Tema]_Home.html" class="home-footer-link">← Voltar ao Home</a>
  </main>
</div>
```

---

## Dots na sidebar

```html
<div class="nav-group">
  <div class="nav-group-header" onclick="toggleGroup(this)">Módulo 1 <span>▾</span></div>
  <div class="nav-group-body">
    <button class="nav-btn" data-page="1">
      <span class="dot pending" data-page="1"></span> Nome da página
    </button>
  </div>
</div>
```

Estados: `dot done` (visitado), `dot current` (atual), `dot pending` (ainda não visto). Nunca usar `dot locked`.

---

## Lightbox (quando houver imagens clicáveis)

```html
<div class="lightbox" id="lightbox" onclick="closeLightbox()">
  <button class="lightbox-close" onclick="closeLightbox()">✕</button>
  <button class="lightbox-prev" id="lb-prev" onclick="event.stopPropagation();lbNav(-1)">‹</button>
  <img id="lightbox-img" src="" alt="" onclick="event.stopPropagation()">
  <button class="lightbox-next" id="lb-next" onclick="event.stopPropagation();lbNav(1)">›</button>
</div>
```

JS do lightbox inclui navegação por teclado (ArrowLeft, ArrowRight, Escape).

---

## Página Hub ([Tema]_Home.html)

- Header com logo + título + subtítulo
- Cards de cursos: ativos mostram número de páginas (ex: "17 páginas"), **sem** badge de idioma
- Cards "Em breve": mostram "Em desenvolvimento" no subtítulo
- Seção de glossário com `id="glossario"` para anchor links funcionarem: `<p id="glossario" class="section-title">Glossário Consolidado</p>`
- Campo de busca do glossário com `aria-label="Buscar no glossário"`
- O Hub **não usa** `guidewire.css` — tem CSS próprio embutido no `<style>`, mas usa a mesma fonte Barlow e as mesmas variáveis CSS

---

## Imagens

- Todas em `images/` com nomes **kebab-case**: `nome-do-arquivo.png`
- Sem espaços, `+`, `(`, `)` ou maiúsculas no nome do arquivo
- Obrigatório para compatibilidade com GitHub Pages (Linux, case-sensitive)

---

## Deployment (GitHub Pages)

- `robots.txt` na raiz:
  ```
  User-agent: *
  Disallow: /
  ```
- Todos os HTMLs com `<meta name="robots" content="noindex, nofollow">`
- Abrir arquivos localmente via `file://` funciona normalmente antes do deploy

---

## Componentes de conteúdo disponíveis

| Componente | Uso |
|---|---|
| `.callout` | Destaque colorido com ícone |
| `.goal-list` | Lista de objetivos de aprendizado |
| `.accordion` | Seção expansível (toggleAcc) |
| `.note` | Nota informativa com borda lateral |
| `.info-box` | Caixa de informação com fundo colorido |
| `.term-table` | Tabela de terminologia/glossário |
| `.capability-grid` | Grid de capacidades/features |
| `.tooltip-wrap > .tooltip-term + .tooltip-box` | Tooltip ao hover |

---

## Fluxo de trabalho

1. Criar o CSS compartilhado (`[tema].css`) com todas as variáveis e componentes base
2. Criar o Hub (`[Tema]_Home.html`) com cards dos cursos e glossário vazio
3. Para cada curso: criar o `[Curso]_Overview.html` com estrutura completa e conteúdo placeholder
4. Ao receber o conteúdo real de cada página: substituir 100% do placeholder pelo conteúdo enviado
5. Ao receber novas entradas de glossário: adicionar no Hub e, se o módulo tiver glossário próprio, remover dele (glossário centralizado no Hub)
6. Testar JS com `node --check` antes de confirmar cada arquivo

---

*Gerado a partir do projeto Guidewire — metodologia de guias de estudo HTML.*
