<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Rotina da Galera — Produtividade Semanal</title>

<!-- Tailwind CSS via CDN -->
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          bg: "#0B0D12",
          surface: "#14161C",
          surfaceHover: "#1B1E26",
          border: "#262A35",
          textPrimary: "#E8E9ED",
          textMuted: "#8B8FA3",
          accent: "#6E8CFF",
          prod: "#3DD68C",
          streak: "#F5A65B",
        },
        fontFamily: {
          display: ["Sora", "sans-serif"],
          body: ["Inter", "sans-serif"],
        },
      },
    },
  };
</script>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Sora:wght@400;600;700;800&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">

<!-- Supabase JS client -->
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js"></script>

<style>
  html, body { background-color: #0B0D12; }
  body { font-family: 'Inter', sans-serif; }
  h1, h2, h3, .font-display { font-family: 'Sora', sans-serif; }

  ::-webkit-scrollbar { height: 8px; width: 8px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: #262A35; border-radius: 8px; }
  ::-webkit-scrollbar-thumb:hover { background: #333847; }

  .tab-btn { position: relative; transition: color .2s ease; }
  .tab-btn::after {
    content: "";
    position: absolute;
    left: 0; right: 0; bottom: -1px;
    height: 2px;
    background: #6E8CFF;
    transform: scaleX(0);
    transform-origin: center;
    transition: transform .25s ease;
    border-radius: 2px;
  }
  .tab-btn.active::after { transform: scaleX(1); }
  .tab-btn.active { color: #E8E9ED; }

  .profile-btn { transition: all .18s ease; }
  .profile-btn.active {
    background: #6E8CFF22;
    border-color: #6E8CFF;
    color: #E8E9ED;
  }

  .hour-cell {
    transition: transform .12s ease, box-shadow .12s ease, background-color .15s ease;
    cursor: pointer;
  }
  .hour-cell:hover {
    transform: scale(1.15);
    box-shadow: 0 0 0 2px rgba(110,140,255,0.5);
    z-index: 10;
  }

  .card-enter { animation: cardIn .35s ease both; }
  @keyframes cardIn {
    from { opacity: 0; transform: translateY(6px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .pulse-dot { animation: pulseDot 1.6s ease-in-out infinite; }
  @keyframes pulseDot { 0%, 100% { opacity: 1; } 50% { opacity: .35; } }

  .fade-in { animation: fadeIn .3s ease both; }
  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

  .mural-post { animation: postIn .3s ease both; }
  @keyframes postIn {
    from { opacity: 0; transform: translateX(-6px); }
    to { opacity: 1; transform: translateX(0); }
  }

  .toast { animation: toastIn .25s ease both, toastOut .25s ease 2.5s forwards; }
  @keyframes toastIn { from { opacity: 0; transform: translateY(10px);} to {opacity:1; transform: translateY(0);} }
  @keyframes toastOut { to { opacity: 0; transform: translateY(10px); } }

  select, input, textarea { color-scheme: dark; }

  .glow-border {
    box-shadow: 0 0 0 1px #262A35, 0 8px 24px -12px rgba(0,0,0,0.6);
  }
</style>
</head>

<body class="bg-bg text-textPrimary font-body min-h-screen pb-24">

  <!-- ============ HEADER ============ -->
  <header class="border-b border-border sticky top-0 z-30 bg-bg/90 backdrop-blur">
    <div class="max-w-6xl mx-auto px-4 sm:px-6 py-4 flex items-center justify-between">
      <div class="flex items-center gap-3">
        <div class="w-9 h-9 rounded-lg bg-accent/15 flex items-center justify-center">
          <span class="text-accent font-display font-bold text-lg">R</span>
        </div>
        <div>
          <h1 class="font-display font-bold text-lg leading-tight">Rotina da Galera</h1>
          <p class="text-xs text-textMuted -mt-0.5">produtividade em tempo real, 06h–00h</p>
        </div>
      </div>
      <span id="conn-indicator" class="flex items-center gap-1.5 text-xs text-textMuted">
        <span id="conn-dot" class="w-2 h-2 rounded-full bg-textMuted"></span>
        <span id="conn-text">conectando…</span>
      </span>
    </div>
  </header>

  <main class="max-w-6xl mx-auto px-4 sm:px-6 pt-6">

    <!-- ============ ALTERNÂNCIA DE PERFIL ============ -->
    <section class="mb-6">
      <div class="flex items-center justify-between mb-2">
        <span class="text-xs text-textMuted">Marcando como:</span>
        <button id="add-friend-btn"
          class="text-xs px-3 py-1.5 rounded-full border border-dashed border-border text-textMuted hover:text-accent hover:border-accent transition-colors">
          + adicionar amigo
        </button>
      </div>
      <div id="profile-switcher" class="flex flex-wrap gap-2"></div>
    </section>

    <!-- ============ ABAS DE DIAS ============ -->
    <nav class="flex gap-1 overflow-x-auto border-b border-border mb-6" id="day-tabs"></nav>

    <!-- ============ GRADE DE HORAS POR AMIGO ============ -->
    <section class="mb-10">
      <div class="flex items-center justify-between mb-3">
        <h2 class="font-display font-semibold text-base">Grade do dia</h2>
        <div class="flex items-center gap-3 text-[11px] text-textMuted">
          <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-sm bg-border inline-block"></span> livre</span>
          <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-sm bg-streak inline-block"></span> distraído</span>
          <span class="flex items-center gap-1"><span class="w-2.5 h-2.5 rounded-sm bg-prod inline-block"></span> produtivo</span>
        </div>
      </div>
      <p class="text-xs text-textMuted mb-3">Clique numa célula para alternar entre <b class="text-textPrimary">livre → distraído → produtivo</b>. Você só pode editar seu próprio perfil selecionado acima.</p>
      <div id="grid-container" class="space-y-3"></div>
    </section>

    <!-- ============ PLACAR DE PRODUTIVIDADE ============ -->
    <section class="mb-10">
      <h2 class="font-display font-semibold text-base mb-3">Placar da semana</h2>
      <div id="scoreboard" class="grid gap-2 sm:grid-cols-2 lg:grid-cols-3"></div>
    </section>

    <!-- ============ MURAL ============ -->
    <section class="mb-10">
      <h2 class="font-display font-semibold text-base mb-3">Mural</h2>
      <div class="bg-surface border border-border rounded-xl p-3 mb-4 glow-border">
        <div class="flex gap-2">
          <input id="mural-input" type="text" placeholder="Escreva um recado pro grupo…"
            class="flex-1 bg-surfaceHover border border-border rounded-lg px-3 py-2 text-sm placeholder:text-textMuted focus:outline-none focus:ring-2 focus:ring-accent/40" />
          <button id="mural-send"
            class="px-4 py-2 rounded-lg bg-accent text-white text-sm font-medium hover:bg-accent/90 active:scale-95 transition">
            Enviar
          </button>
        </div>
      </div>
      <div id="mural-list" class="space-y-2 max-h-96 overflow-y-auto pr-1"></div>
    </section>

  </main>

  <div id="toast-container" class="fixed bottom-4 right-4 z-50 flex flex-col gap-2"></div>

  <!-- ============ MODAL: adicionar amigo ============ -->
  <div id="modal-backdrop" class="fixed inset-0 bg-black/60 hidden items-center justify-center z-40 p-4">
    <div class="bg-surface border border-border rounded-xl p-5 w-full max-w-sm fade-in">
      <h3 class="font-display font-semibold text-base mb-3">Novo perfil</h3>
      <input id="new-friend-name" type="text" placeholder="Nome do amigo"
        class="w-full bg-surfaceHover border border-border rounded-lg px-3 py-2 text-sm mb-3 focus:outline-none focus:ring-2 focus:ring-accent/40" />
      <div class="flex gap-2 mb-4" id="color-picker"></div>
      <div class="flex justify-end gap-2">
        <button id="cancel-friend-btn" class="px-3 py-2 text-sm text-textMuted hover:text-textPrimary transition">Cancelar</button>
        <button id="confirm-friend-btn" class="px-4 py-2 text-sm bg-accent rounded-lg text-white font-medium hover:bg-accent/90 transition">Adicionar</button>
      </div>
    </div>
  </div>

<script>
/* =========================================================================
   ROTINA DA GALERA — lógica completa + Supabase (tempo real)
   ========================================================================= */

/* 1) CONFIGURAÇÃO DO SUPABASE (credenciais do usuário já embutidas) */
const SUPABASE_CONFIG = {
  SUPABASE_URL: "https://dxkuzccriemkyfxfojku.supabase.co",
  SUPABASE_ANON_KEY: "sb_publishable_tzlR36HMhYvCUalD1P5mhg_QOJNk-cM",
};

const supabaseClient = window.supabase.createClient(
  SUPABASE_CONFIG.SUPABASE_URL,
  SUPABASE_CONFIG.SUPABASE_ANON_KEY
);

/* 2) CONSTANTES DE DOMÍNIO */
const DAYS = [
  { idx: 0, label: "Dom", full: "Domingo" },
  { idx: 1, label: "Seg", full: "Segunda" },
  { idx: 2, label: "Ter", full: "Terça" },
  { idx: 3, label: "Qua", full: "Quarta" },
  { idx: 4, label: "Qui", full: "Quinta" },
  { idx: 5, label: "Sex", full: "Sexta" },
  { idx: 6, label: "Sáb", full: "Sábado" },
];

const HOURS = [6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0];

const STATUS_CYCLE = ["livre", "distraido", "produtivo"];
const STATUS_POINTS = { livre: 0, distraido: -1, produtivo: 2 };
const STATUS_COLOR = {
  livre: "#262A35",
  distraido: "#F5A65B",
  produtivo: "#3DD68C",
};

const FRIEND_COLORS = [
  "#6E8CFF", "#3DD68C", "#F5A65B", "#F06A8C",
  "#B98CFF", "#3DBEE0", "#E0C23D", "#E05C5C",
];

const DEFAULT_FRIENDS = ["Você", "Lucas", "Mariana"];

/* 3) ESTADO LOCAL */
const state = {
  friends: [],
  activities: new Map(),
  muralPosts: [],
  currentDay: new Date().getDay(),
  activeFriendId: null,   // perfil selecionado para marcar a rotina
  chosenColor: FRIEND_COLORS[0],
};

/* 4) HELPERS DE UI */
function $(sel) { return document.querySelector(sel); }
function $all(sel) { return Array.from(document.querySelectorAll(sel)); }

function showToast(message, type = "info") {
  const container = $("#toast-container");
  const colors = {
    info: "border-accent/40 text-textPrimary",
    success: "border-prod/50 text-textPrimary",
    error: "border-red-500/50 text-textPrimary",
  };
  const el = document.createElement("div");
  el.className = `toast bg-surface border ${colors[type] || colors.info} rounded-lg px-4 py-2.5 text-sm shadow-lg`;
  el.textContent = message;
  container.appendChild(el);
  setTimeout(() => el.remove(), 3000);
}

function setConnectionStatus(connected) {
  const dot = $("#conn-dot");
  const text = $("#conn-text");
  if (connected) {
    dot.className = "w-2 h-2 rounded-full bg-prod pulse-dot";
    text.textContent = "sincronizado";
    text.className = "text-prod";
  } else {
    dot.className = "w-2 h-2 rounded-full bg-streak pulse-dot";
    text.textContent = "conectando…";
    text.className = "text-textMuted";
  }
}

function friendById(id) { return state.friends.find((f) => f.id === id); }
function activityKey(friendId, day, hour) { return `${friendId}_${day}_${hour}`; }

function escapeHtml(str) {
  const div = document.createElement("div");
  div.textContent = str;
  return div.innerHTML;
}
function formatHour(h) { return `${String(h).padStart(2, "0")}:00`; }
function statusLabel(s) { return { livre: "livre", distraido: "distraído", produtivo: "produtivo" }[s] || s; }

/* 5) RENDER: SELETOR DE PERFIL ATIVO */
function renderProfileSwitcher() {
  const wrap = $("#profile-switcher");
  wrap.innerHTML = "";

  if (state.friends.length === 0) {
    wrap.innerHTML = `<span class="text-sm text-textMuted">Nenhum perfil ainda. Adicione um amigo para começar.</span>`;
    return;
  }

  if (!state.activeFriendId || !friendById(state.activeFriendId)) {
    state.activeFriendId = state.friends[0].id;
  }

  state.friends.forEach((f) => {
    const btn = document.createElement("button");
    const isActive = f.id === state.activeFriendId;
    btn.className =
      "profile-btn flex items-center gap-1.5 pl-1 pr-3 py-1.5 rounded-full border border-border bg-surface text-sm" +
      (isActive ? " active" : " text-textMuted hover:text-textPrimary");
    btn.innerHTML = `
      <span class="w-6 h-6 rounded-full flex items-center justify-center text-[11px] font-semibold text-black/80"
            style="background:${f.color}">${f.name.charAt(0).toUpperCase()}</span>
      <span>${escapeHtml(f.name)}</span>
    `;
    btn.addEventListener("click", () => {
      state.activeFriendId = f.id;
      renderProfileSwitcher();
      renderGrid();
    });

    const del = document.createElement("button");
    del.className = "ml-1 text-textMuted hover:text-red-400 transition text-xs";
    del.textContent = "✕";
    del.title = `Remover ${f.name}`;
    del.addEventListener("click", async (e) => {
      e.stopPropagation();
      if (!confirm(`Remover ${f.name} do grupo? Isso apaga o histórico dele.`)) return;
      const { error } = await supabaseClient.from("friends").delete().eq("id", f.id);
      if (error) showToast("Erro ao remover: " + error.message, "error");
    });
    btn.appendChild(del);

    wrap.appendChild(btn);
  });
}

/* 6) RENDER: ABAS DE DIA */
function renderDayTabs() {
  const nav = $("#day-tabs");
  nav.innerHTML = "";
  DAYS.forEach((d) => {
    const btn = document.createElement("button");
    btn.className =
      "tab-btn px-4 py-2.5 text-sm font-medium text-textMuted hover:text-textPrimary whitespace-nowrap" +
      (d.idx === state.currentDay ? " active" : "");
    btn.textContent = d.full;
    btn.dataset.day = d.idx;
    btn.addEventListener("click", () => {
      state.currentDay = d.idx;
      renderDayTabs();
      renderGrid();
    });
    nav.appendChild(btn);
  });
}

/* 7) RENDER: GRADE DE HORAS (heatmap por amigo) */
function renderGrid() {
  const container = $("#grid-container");
  container.innerHTML = "";

  if (state.friends.length === 0) {
    container.innerHTML = `
      <div class="border border-dashed border-border rounded-xl py-10 text-center text-textMuted text-sm">
        Nenhum perfil ainda. Clique em "+ adicionar amigo" para começar.
      </div>`;
    return;
  }

  let firstRowDrawn = false;

  state.friends.forEach((friend) => {
    const row = document.createElement("div");
    const isEditable = friend.id === state.activeFriendId;
    row.className =
      "bg-surface border rounded-xl p-3 card-enter glow-border " +
      (isEditable ? "border-accent/50" : "border-border");

    const header = document.createElement("div");
    header.className = "flex items-center justify-between mb-2.5";
    header.innerHTML = `
      <div class="flex items-center gap-2">
        <span class="w-6 h-6 rounded-full flex items-center justify-center text-[11px] font-semibold text-black/80"
              style="background:${friend.color}">${friend.name.charAt(0).toUpperCase()}</span>
        <span class="text-sm font-medium">${escapeHtml(friend.name)}</span>
        ${isEditable ? '<span class="text-[10px] px-1.5 py-0.5 rounded bg-accent/20 text-accent">editando</span>' : ""}
      </div>
      <span class="text-xs text-textMuted" id="day-score-${friend.id}"></span>
    `;
    row.appendChild(header);

    const gridWrap = document.createElement("div");
    gridWrap.className = "grid gap-1";
    gridWrap.style.gridTemplateColumns = `repeat(${HOURS.length}, minmax(0, 1fr))`;

    let dayScore = 0;

    HOURS.forEach((hour) => {
      const key = activityKey(friend.id, state.currentDay, hour);
      const activity = state.activities.get(key);
      const status = activity ? activity.status : "livre";
      dayScore += STATUS_POINTS[status];

      const cell = document.createElement("div");
      cell.className = "hour-cell aspect-square rounded-md flex items-center justify-center" +
        (isEditable ? "" : " cursor-not-allowed opacity-80");
      cell.style.backgroundColor = STATUS_COLOR[status];
      cell.title = `${friend.name} · ${formatHour(hour)} · ${statusLabel(status)}` +
        (isEditable ? "" : " (selecione este perfil para editar)");

      if (isEditable) {
        cell.addEventListener("click", () => cycleStatus(friend.id, state.currentDay, hour));
      }
      gridWrap.appendChild(cell);
    });

    row.appendChild(gridWrap);

    if (!firstRowDrawn) {
      const labelsRow = document.createElement("div");
      labelsRow.className = "grid gap-1 mt-1";
      labelsRow.style.gridTemplateColumns = `repeat(${HOURS.length}, minmax(0, 1fr))`;
      HOURS.forEach((h) => {
        const l = document.createElement("span");
        l.className = "text-[9px] text-textMuted text-center";
        l.textContent = h;
        labelsRow.appendChild(l);
      });
      container.appendChild(labelsRow);
      firstRowDrawn = true;
    }

    container.appendChild(row);

    const scoreEl = header.querySelector(`#day-score-${friend.id}`);
    scoreEl.textContent = `${dayScore >= 0 ? "+" : ""}${dayScore} pts hoje`;
    scoreEl.className = dayScore > 0 ? "text-xs text-prod" : dayScore < 0 ? "text-xs text-streak" : "text-xs text-textMuted";
  });
}

/* 8) AÇÃO: ciclar status de uma célula */
async function cycleStatus(friendId, day, hour) {
  const key = activityKey(friendId, day, hour);
  const current = state.activities.get(key);
  const currentStatus = current ? current.status : "livre";
  const nextStatus = STATUS_CYCLE[(STATUS_CYCLE.indexOf(currentStatus) + 1) % STATUS_CYCLE.length];

  state.activities.set(key, { id: current ? current.id : null, status: nextStatus });
  renderGrid();
  renderScoreboard();

  const { data, error } = await supabaseClient
    .from("activities")
    .upsert(
      {
        id: current ? current.id : undefined,
        friend_id: friendId,
        day_of_week: day,
        hour: hour,
        status: nextStatus,
      },
      { onConflict: "friend_id,day_of_week,hour" }
    )
    .select()
    .single();

  if (error) {
    showToast("Erro ao salvar: " + error.message, "error");
    return;
  }
  state.activities.set(key, { id: data.id, status: data.status });
}

/* 9) RENDER: PLACAR DE PRODUTIVIDADE (semana inteira) */
function renderScoreboard() {
  const board = $("#scoreboard");
  board.innerHTML = "";

  if (state.friends.length === 0) return;

  const scores = state.friends.map((f) => {
    let total = 0;
    for (const [key, activity] of state.activities.entries()) {
      if (key.startsWith(`${f.id}_`)) total += STATUS_POINTS[activity.status];
    }
    return { friend: f, total };
  });

  scores.sort((a, b) => b.total - a.total);

  scores.forEach((s, i) => {
    const card = document.createElement("div");
    card.className = "bg-surface border border-border rounded-xl p-3 flex items-center gap-3 card-enter";
    const medal = i === 0 ? "🥇" : i === 1 ? "🥈" : i === 2 ? "🥉" : `#${i + 1}`;
    card.innerHTML = `
      <span class="w-8 text-center font-display font-semibold text-textMuted">${medal}</span>
      <span class="w-7 h-7 rounded-full flex items-center justify-center text-[11px] font-semibold text-black/80"
            style="background:${s.friend.color}">${s.friend.name.charAt(0).toUpperCase()}</span>
      <span class="flex-1 text-sm font-medium">${escapeHtml(s.friend.name)}</span>
      <span class="font-display font-bold text-sm ${s.total > 0 ? "text-prod" : s.total < 0 ? "text-streak" : "text-textMuted"}">
        ${s.total >= 0 ? "+" : ""}${s.total} pts
      </span>
    `;
    board.appendChild(card);
  });
}

/* 10) MURAL */
function renderMural() {
  const list = $("#mural-list");
  list.innerHTML = "";

  if (state.muralPosts.length === 0) {
    list.innerHTML = `<p class="text-sm text-textMuted text-center py-6">Ainda não tem nenhum recado. Seja o primeiro!</p>`;
    return;
  }

  const sorted = [...state.muralPosts].sort((a, b) => new Date(b.created_at) - new Date(a.created_at));

  sorted.forEach((post) => {
    const item = document.createElement("div");
    item.className = "mural-post bg-surface border border-border rounded-lg px-3 py-2.5 flex items-start gap-2.5";
    const time = new Date(post.created_at).toLocaleString("pt-BR", {
      day: "2-digit", month: "2-digit", hour: "2-digit", minute: "2-digit",
    });
    item.innerHTML = `
      <div class="flex-1">
        <div class="flex items-center gap-2 mb-0.5">
          <span class="text-sm font-medium">${escapeHtml(post.author)}</span>
          <span class="text-[11px] text-textMuted">${time}</span>
        </div>
        <p class="text-sm text-textPrimary/90">${escapeHtml(post.content)}</p>
      </div>
    `;
    list.appendChild(item);
  });
}

async function sendMuralPost() {
  const friend = friendById(state.activeFriendId);
  const input = $("#mural-input");
  const content = input.value.trim();
  if (!friend) {
    showToast("Selecione um perfil antes de enviar.", "error");
    return;
  }
  if (!content) return;

  const { error } = await supabaseClient.from("mural_posts").insert({
    author: friend.name,
    content,
  });

  if (error) {
    showToast("Erro ao enviar: " + error.message, "error");
    return;
  }
  input.value = "";
}

/* 11) MODAL: ADICIONAR AMIGO */
function renderColorPicker() {
  const wrap = $("#color-picker");
  wrap.innerHTML = "";
  FRIEND_COLORS.forEach((color) => {
    const dot = document.createElement("button");
    dot.className = "w-7 h-7 rounded-full border-2 transition-transform hover:scale-110";
    dot.style.backgroundColor = color;
    dot.style.borderColor = color === state.chosenColor ? "#E8E9ED" : "transparent";
    dot.addEventListener("click", () => {
      state.chosenColor = color;
      renderColorPicker();
    });
    wrap.appendChild(dot);
  });
}

function openModal() {
  $("#modal-backdrop").classList.remove("hidden");
  $("#modal-backdrop").classList.add("flex");
  renderColorPicker();
  $("#new-friend-name").value = "";
  $("#new-friend-name").focus();
}
function closeModal() {
  $("#modal-backdrop").classList.add("hidden");
  $("#modal-backdrop").classList.remove("flex");
}

async function confirmAddFriend() {
  const name = $("#new-friend-name").value.trim();
  if (!name) {
    showToast("Digite um nome antes de adicionar.", "error");
    return;
  }
  const { error } = await supabaseClient.from("friends").insert({ name, color: state.chosenColor });

  if (error) {
    showToast("Erro ao adicionar: " + error.message, "error");
    return;
  }
  showToast(`${name} entrou no grupo! 🎉`, "success");
  closeModal();
}

/* 12) SEED: cria os 3 amigos padrão na primeira execução, se a tabela estiver vazia */
async function seedDefaultFriendsIfEmpty() {
  if (state.friends.length > 0) return;
  const rows = DEFAULT_FRIENDS.map((name, i) => ({
    name,
    color: FRIEND_COLORS[i % FRIEND_COLORS.length],
  }));
  const { error } = await supabaseClient.from("friends").insert(rows);
  if (error) {
    console.error("Falha ao criar perfis padrão:", error);
  }
}

/* 13) CARREGAMENTO INICIAL DOS DADOS */
async function loadInitialData() {
  const [{ data: friends, error: friendsErr }, { data: activities, error: actErr }, { data: mural, error: muralErr }] =
    await Promise.all([
      supabaseClient.from("friends").select("*").order("created_at", { ascending: true }),
      supabaseClient.from("activities").select("*"),
      supabaseClient.from("mural_posts").select("*").order("created_at", { ascending: false }).limit(100),
    ]);

  if (friendsErr || actErr || muralErr) {
    showToast("Falha ao carregar dados. Confira se rodou o SQL no Supabase.", "error");
    console.error(friendsErr, actErr, muralErr);
    return;
  }

  state.friends = friends || [];
  state.muralPosts = mural || [];
  state.activities = new Map();
  (activities || []).forEach((a) => {
    state.activities.set(activityKey(a.friend_id, a.day_of_week, a.hour), { id: a.id, status: a.status });
  });

  if (state.friends.length === 0) {
    await seedDefaultFriendsIfEmpty();
    const { data: seeded } = await supabaseClient.from("friends").select("*").order("created_at", { ascending: true });
    state.friends = seeded || [];
  }

  renderAll();
}

function renderAll() {
  renderProfileSwitcher();
  renderDayTabs();
  renderGrid();
  renderScoreboard();
  renderMural();
}

/* 14) REALTIME */
function setupRealtime() {
  supabaseClient
    .channel("rotina-da-galera-realtime")
    .on("postgres_changes", { event: "*", schema: "public", table: "friends" }, handleFriendsChange)
    .on("postgres_changes", { event: "*", schema: "public", table: "activities" }, handleActivitiesChange)
    .on("postgres_changes", { event: "*", schema: "public", table: "mural_posts" }, handleMuralChange)
    .subscribe((status) => {
      if (status === "SUBSCRIBED") setConnectionStatus(true);
      if (status === "CHANNEL_ERROR" || status === "TIMED_OUT" || status === "CLOSED") setConnectionStatus(false);
    });
}

function handleFriendsChange(payload) {
  if (payload.eventType === "INSERT") {
    if (!state.friends.some((f) => f.id === payload.new.id)) state.friends.push(payload.new);
  } else if (payload.eventType === "UPDATE") {
    const idx = state.friends.findIndex((f) => f.id === payload.new.id);
    if (idx !== -1) state.friends[idx] = payload.new;
  } else if (payload.eventType === "DELETE") {
    state.friends = state.friends.filter((f) => f.id !== payload.old.id);
    for (const key of state.activities.keys()) {
      if (key.startsWith(`${payload.old.id}_`)) state.activities.delete(key);
    }
    if (state.activeFriendId === payload.old.id) state.activeFriendId = null;
  }
  renderProfileSwitcher();
  renderGrid();
  renderScoreboard();
}

function handleActivitiesChange(payload) {
  const row = payload.new || payload.old;
  const key = activityKey(row.friend_id, row.day_of_week, row.hour);
  if (payload.eventType === "DELETE") {
    state.activities.delete(key);
  } else {
    state.activities.set(key, { id: row.id, status: row.status });
  }
  renderGrid();
  renderScoreboard();
}

function handleMuralChange(payload) {
  if (payload.eventType === "INSERT") {
    if (!state.muralPosts.some((p) => p.id === payload.new.id)) state.muralPosts.unshift(payload.new);
  } else if (payload.eventType === "DELETE") {
    state.muralPosts = state.muralPosts.filter((p) => p.id !== payload.old.id);
  }
  renderMural();
}

/* 15) EVENTOS ESTÁTICOS */
function bindStaticEvents() {
  $("#add-friend-btn").addEventListener("click", openModal);
  $("#cancel-friend-btn").addEventListener("click", closeModal);
  $("#confirm-friend-btn").addEventListener("click", confirmAddFriend);
  $("#modal-backdrop").addEventListener("click", (e) => {
    if (e.target.id === "modal-backdrop") closeModal();
  });
  $("#new-friend-name").addEventListener("keydown", (e) => {
    if (e.key === "Enter") confirmAddFriend();
  });

  $("#mural-send").addEventListener("click", sendMuralPost);
  $("#mural-input").addEventListener("keydown", (e) => {
    if (e.key === "Enter") sendMuralPost();
  });
}

/* 16) BOOTSTRAP */
(async function init() {
  bindStaticEvents();
  setConnectionStatus(false);
  await loadInitialData();
  setupRealtime();
})();
</script>
</body>
</html>