<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FLUX — Gestor de Proyectos</title>
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@300;400;500&display=swap" rel="stylesheet">
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0a0a0f;
    --surface: #12121a;
    --surface2: #1c1c28;
    --border: #2a2a3d;
    --accent: #7c6fff;
    --accent2: #ff6f91;
    --accent3: #6fffd4;
    --text: #e8e8f0;
    --muted: #6b6b8a;
    --done: #3d3d55;
    --radius: 12px;
    --shadow: 0 8px 32px rgba(0,0,0,0.4);
  }

  body {
    font-family: 'DM Mono', monospace;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Background mesh */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background:
      radial-gradient(ellipse 60% 50% at 80% 10%, rgba(124,111,255,0.08) 0%, transparent 60%),
      radial-gradient(ellipse 40% 40% at 10% 90%, rgba(111,255,212,0.05) 0%, transparent 60%);
    pointer-events: none;
    z-index: 0;
  }

  /* Header */
  header {
    position: sticky;
    top: 0;
    z-index: 100;
    background: rgba(10,10,15,0.85);
    backdrop-filter: blur(20px);
    border-bottom: 1px solid var(--border);
    padding: 0 32px;
    height: 64px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .logo {
    font-family: 'Syne', sans-serif;
    font-weight: 800;
    font-size: 22px;
    letter-spacing: -0.5px;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .header-stats {
    display: flex;
    gap: 24px;
    font-size: 11px;
    color: var(--muted);
  }

  .header-stats span strong {
    color: var(--text);
    font-weight: 500;
  }

  /* Layout */
  .app {
    position: relative;
    z-index: 1;
    display: grid;
    grid-template-columns: 280px 1fr;
    min-height: calc(100vh - 64px);
  }

  /* Sidebar */
  .sidebar {
    border-right: 1px solid var(--border);
    padding: 24px 16px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    background: rgba(18,18,26,0.5);
    overflow-y: auto;
    max-height: calc(100vh - 64px);
    position: sticky;
    top: 64px;
  }

  .sidebar-title {
    font-family: 'Syne', sans-serif;
    font-size: 10px;
    letter-spacing: 2px;
    text-transform: uppercase;
    color: var(--muted);
    padding: 8px 12px 16px;
  }

  .project-item {
    border-radius: 10px;
    padding: 12px 14px;
    cursor: pointer;
    transition: all 0.2s;
    border: 1px solid transparent;
    position: relative;
    overflow: hidden;
  }

  .project-item::before {
    content: '';
    position: absolute;
    left: 0; top: 0; bottom: 0;
    width: 3px;
    border-radius: 0 2px 2px 0;
    background: var(--proj-color, var(--accent));
    transform: scaleY(0);
    transition: transform 0.2s;
  }

  .project-item:hover::before,
  .project-item.active::before { transform: scaleY(1); }

  .project-item:hover, .project-item.active {
    background: var(--surface2);
    border-color: var(--border);
  }

  .project-item.active { background: var(--surface2); }

  .proj-name {
    font-family: 'Syne', sans-serif;
    font-weight: 600;
    font-size: 13px;
    margin-bottom: 4px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .proj-meta {
    font-size: 10px;
    color: var(--muted);
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .proj-progress-bar {
    height: 2px;
    background: var(--border);
    border-radius: 2px;
    margin-top: 8px;
    overflow: hidden;
  }

  .proj-progress-fill {
    height: 100%;
    border-radius: 2px;
    background: var(--proj-color, var(--accent));
    transition: width 0.5s cubic-bezier(0.34,1.56,0.64,1);
  }

  .btn-add-project {
    margin-top: 8px;
    padding: 10px 14px;
    border-radius: 10px;
    border: 1px dashed var(--border);
    background: transparent;
    color: var(--muted);
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    cursor: pointer;
    transition: all 0.2s;
    width: 100%;
    text-align: left;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .btn-add-project:hover {
    border-color: var(--accent);
    color: var(--accent);
    background: rgba(124,111,255,0.05);
  }

  /* Main area */
  .main {
    padding: 32px;
    overflow-y: auto;
    max-height: calc(100vh - 64px);
  }

  .project-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    margin-bottom: 32px;
    gap: 16px;
  }

  .project-title-wrap h1 {
    font-family: 'Syne', sans-serif;
    font-size: 32px;
    font-weight: 800;
    letter-spacing: -1px;
    line-height: 1;
    margin-bottom: 8px;
  }

  .project-title-wrap p {
    font-size: 12px;
    color: var(--muted);
  }

  .view-toggle {
    display: flex;
    gap: 4px;
    background: var(--surface);
    padding: 4px;
    border-radius: 10px;
    border: 1px solid var(--border);
  }

  .view-btn {
    padding: 6px 14px;
    border-radius: 7px;
    border: none;
    background: transparent;
    color: var(--muted);
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .view-btn.active {
    background: var(--accent);
    color: white;
  }

  /* Task columns */
  .columns {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 16px;
    margin-bottom: 24px;
  }

  .columns.compact { grid-template-columns: repeat(4, 1fr); gap: 12px; }

  .column {
    background: var(--surface);
    border-radius: var(--radius);
    border: 1px solid var(--border);
    overflow: hidden;
  }

  .column-header {
    padding: 14px 16px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid var(--border);
  }

  .column-label {
    font-family: 'Syne', sans-serif;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .col-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
  }

  .column-count {
    font-size: 10px;
    color: var(--muted);
    background: var(--surface2);
    padding: 2px 8px;
    border-radius: 20px;
  }

  .column-tasks {
    padding: 12px;
    display: flex;
    flex-direction: column;
    gap: 8px;
    min-height: 60px;
  }

  /* Task card */
  .task-card {
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 12px 14px;
    cursor: pointer;
    transition: all 0.25s;
    position: relative;
    animation: slideIn 0.3s cubic-bezier(0.34,1.56,0.64,1);
  }

  @keyframes slideIn {
    from { opacity: 0; transform: translateY(-8px) scale(0.97); }
    to { opacity: 1; transform: translateY(0) scale(1); }
  }

  .task-card:hover {
    border-color: var(--accent);
    transform: translateY(-2px);
    box-shadow: 0 8px 24px rgba(124,111,255,0.15);
  }

  .task-card.done {
    opacity: 0.5;
  }

  .task-card.done .task-title {
    text-decoration: line-through;
    color: var(--muted);
  }

  .task-top {
    display: flex;
    align-items: flex-start;
    gap: 8px;
    margin-bottom: 6px;
  }

  .task-check {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    border: 1.5px solid var(--border);
    flex-shrink: 0;
    margin-top: 2px;
    cursor: pointer;
    transition: all 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 9px;
  }

  .task-check.checked {
    background: var(--accent3);
    border-color: var(--accent3);
    color: #0a0a0f;
  }

  .task-title {
    font-size: 13px;
    font-weight: 500;
    line-height: 1.4;
    flex: 1;
  }

  .task-meta {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
  }

  .task-tag {
    font-size: 9px;
    padding: 2px 8px;
    border-radius: 20px;
    letter-spacing: 0.5px;
    text-transform: uppercase;
    font-weight: 500;
  }

  .tag-high { background: rgba(255,111,145,0.15); color: var(--accent2); }
  .tag-med { background: rgba(255,200,80,0.12); color: #ffcc50; }
  .tag-low { background: rgba(111,255,212,0.1); color: var(--accent3); }

  .subtask-preview {
    margin-top: 8px;
    padding-top: 8px;
    border-top: 1px solid var(--border);
  }

  .subtask-row {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 3px 0;
    font-size: 11px;
    color: var(--muted);
  }

  .subtask-row.done-sub {
    text-decoration: line-through;
    opacity: 0.5;
  }

  .sub-check {
    width: 10px;
    height: 10px;
    border-radius: 2px;
    border: 1px solid var(--border);
    cursor: pointer;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 7px;
    transition: all 0.15s;
  }

  .sub-check.checked {
    background: var(--accent3);
    border-color: var(--accent3);
    color: #0a0a0f;
  }

  .subtask-progress {
    margin-top: 6px;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .sub-bar {
    flex: 1;
    height: 3px;
    background: var(--border);
    border-radius: 3px;
    overflow: hidden;
  }

  .sub-bar-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--accent), var(--accent3));
    border-radius: 3px;
    transition: width 0.4s cubic-bezier(0.34,1.56,0.64,1);
  }

  .sub-count { font-size: 9px; color: var(--muted); }

  /* Add task */
  .add-task-btn {
    width: 100%;
    padding: 8px;
    background: transparent;
    border: 1px dashed transparent;
    border-radius: 8px;
    color: var(--muted);
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    cursor: pointer;
    transition: all 0.2s;
    text-align: left;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .add-task-btn:hover {
    border-color: var(--border);
    color: var(--text);
    background: rgba(255,255,255,0.03);
  }

  /* Modal */
  .modal-overlay {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.7);
    z-index: 1000;
    backdrop-filter: blur(4px);
    align-items: center;
    justify-content: center;
  }

  .modal-overlay.open { display: flex; animation: fadeIn 0.2s; }

  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

  .modal {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 16px;
    padding: 28px;
    width: 520px;
    max-width: 95vw;
    max-height: 85vh;
    overflow-y: auto;
    box-shadow: var(--shadow), 0 0 0 1px rgba(124,111,255,0.1);
    animation: modalIn 0.3s cubic-bezier(0.34,1.56,0.64,1);
  }

  @keyframes modalIn {
    from { opacity: 0; transform: scale(0.9) translateY(20px); }
    to { opacity: 1; transform: scale(1) translateY(0); }
  }

  .modal h2 {
    font-family: 'Syne', sans-serif;
    font-size: 20px;
    font-weight: 700;
    margin-bottom: 24px;
  }

  .form-group {
    margin-bottom: 16px;
  }

  .form-label {
    display: block;
    font-size: 10px;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: 8px;
  }

  .form-input, .form-select, .form-textarea {
    width: 100%;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 10px 14px;
    color: var(--text);
    font-family: 'DM Mono', monospace;
    font-size: 13px;
    transition: border-color 0.2s;
    outline: none;
  }

  .form-input:focus, .form-select:focus, .form-textarea:focus {
    border-color: var(--accent);
    box-shadow: 0 0 0 3px rgba(124,111,255,0.1);
  }

  .form-select { cursor: pointer; appearance: none; }

  .form-textarea { resize: vertical; min-height: 80px; }

  .subtasks-section {
    margin-bottom: 16px;
  }

  .subtask-input-row {
    display: flex;
    gap: 8px;
    margin-top: 8px;
  }

  .subtask-input-row input {
    flex: 1;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 8px 12px;
    color: var(--text);
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    outline: none;
  }

  .subtask-input-row input:focus { border-color: var(--accent); }

  .btn-icon {
    width: 34px;
    height: 34px;
    border-radius: 8px;
    border: none;
    background: var(--accent);
    color: white;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 18px;
    line-height: 1;
    transition: all 0.15s;
    flex-shrink: 0;
  }

  .btn-icon:hover { background: #6a5fff; transform: scale(1.05); }

  .subtask-list {
    display: flex;
    flex-direction: column;
    gap: 6px;
    margin-top: 8px;
  }

  .subtask-edit-item {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 10px;
    background: var(--surface2);
    border-radius: 6px;
    font-size: 12px;
    animation: slideIn 0.2s;
  }

  .subtask-edit-item .del-sub {
    margin-left: auto;
    background: none;
    border: none;
    color: var(--muted);
    cursor: pointer;
    font-size: 14px;
    line-height: 1;
    padding: 0 4px;
    transition: color 0.15s;
  }

  .subtask-edit-item .del-sub:hover { color: var(--accent2); }

  .modal-actions {
    display: flex;
    gap: 10px;
    justify-content: flex-end;
    margin-top: 24px;
  }

  .btn {
    padding: 10px 20px;
    border-radius: 8px;
    border: none;
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    cursor: pointer;
    transition: all 0.2s;
  }

  .btn-primary {
    background: var(--accent);
    color: white;
  }

  .btn-primary:hover { background: #6a5fff; transform: translateY(-1px); }

  .btn-ghost {
    background: transparent;
    color: var(--muted);
    border: 1px solid var(--border);
  }

  .btn-ghost:hover { color: var(--text); border-color: var(--text); }

  /* Color picker */
  .color-picker {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
  }

  .color-dot {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    cursor: pointer;
    border: 2px solid transparent;
    transition: all 0.15s;
  }

  .color-dot.selected, .color-dot:hover {
    border-color: white;
    transform: scale(1.15);
  }

  /* Empty state */
  .empty-state {
    text-align: center;
    padding: 60px 24px;
    color: var(--muted);
  }

  .empty-state .big-icon {
    font-size: 48px;
    margin-bottom: 16px;
    opacity: 0.5;
  }

  .empty-state p { font-size: 13px; margin-bottom: 20px; }

  /* Scrollbar */
  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 4px; }

  /* List view */
  .list-view { display: flex; flex-direction: column; gap: 8px; }

  .list-task {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 14px 16px;
    display: flex;
    align-items: center;
    gap: 12px;
    cursor: pointer;
    transition: all 0.2s;
    animation: slideIn 0.3s cubic-bezier(0.34,1.56,0.64,1);
  }

  .list-task:hover {
    border-color: var(--accent);
    transform: translateX(4px);
  }

  .list-task.done { opacity: 0.5; }
  .list-task.done .list-title { text-decoration: line-through; color: var(--muted); }

  .list-title { flex: 1; font-size: 13px; }

  .list-status {
    font-size: 9px;
    padding: 3px 10px;
    border-radius: 20px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .status-todo { background: var(--surface2); color: var(--muted); }
  .status-doing { background: rgba(124,111,255,0.15); color: var(--accent); }
  .status-done-s { background: rgba(111,255,212,0.1); color: var(--accent3); }

  /* Responsive */
  @media (max-width: 768px) {
    .app { grid-template-columns: 1fr; }
    .sidebar { display: none; }
    .columns { grid-template-columns: 1fr; }
  }
</style>
</head>
<body>

<header>
  <div class="logo">FLUX</div>
  <div class="header-stats">
    <span id="stat-projects">Proyectos: <strong>0</strong></span>
    <span id="stat-tasks">Tareas: <strong>0</strong></span>
    <span id="stat-done">Completadas: <strong>0%</strong></span>
  </div>
</header>

<div class="app">
  <aside class="sidebar">
    <div class="sidebar-title">Proyectos</div>
    <div id="project-list"></div>
    <button class="btn-add-project" onclick="openProjectModal()">
      <span>＋</span> Nuevo proyecto
    </button>
  </aside>

  <main class="main">
    <div id="main-content">
      <div class="empty-state">
        <div class="big-icon">⚡</div>
        <p>Crea tu primer proyecto para empezar</p>
        <button class="btn btn-primary" onclick="openProjectModal()">Crear proyecto</button>
      </div>
    </div>
  </main>
</div>

<!-- Project Modal -->
<div class="modal-overlay" id="project-modal">
  <div class="modal">
    <h2 id="proj-modal-title">Nuevo Proyecto</h2>
    <div class="form-group">
      <label class="form-label">Nombre del proyecto</label>
      <input class="form-input" id="proj-name-input" placeholder="Ej: Rediseño web…" />
    </div>
    <div class="form-group">
      <label class="form-label">Descripción (opcional)</label>
      <textarea class="form-textarea" id="proj-desc-input" placeholder="¿De qué trata este proyecto?"></textarea>
    </div>
    <div class="form-group">
      <label class="form-label">Color</label>
      <div class="color-picker" id="color-picker">
        <div class="color-dot selected" style="background:#7c6fff" data-color="#7c6fff"></div>
        <div class="color-dot" style="background:#ff6f91" data-color="#ff6f91"></div>
        <div class="color-dot" style="background:#6fffd4" data-color="#6fffd4"></div>
        <div class="color-dot" style="background:#ffcc50" data-color="#ffcc50"></div>
        <div class="color-dot" style="background:#ff8c5a" data-color="#ff8c5a"></div>
        <div class="color-dot" style="background:#5ad4ff" data-color="#5ad4ff"></div>
        <div class="color-dot" style="background:#c77dff" data-color="#c77dff"></div>
      </div>
    </div>
    <div class="modal-actions">
      <button class="btn btn-ghost" onclick="closeModal('project-modal')">Cancelar</button>
      <button class="btn btn-primary" onclick="saveProject()">Crear proyecto</button>
    </div>
  </div>
</div>

<!-- Task Modal -->
<div class="modal-overlay" id="task-modal">
  <div class="modal">
    <h2 id="task-modal-title">Nueva tarea</h2>
    <div class="form-group">
      <label class="form-label">Título</label>
      <input class="form-input" id="task-title-input" placeholder="¿Qué hay que hacer?" />
    </div>
    <div class="form-group">
      <label class="form-label">Estado</label>
      <select class="form-select" id="task-status-input">
        <option value="todo">Por hacer</option>
        <option value="doing">En progreso</option>
        <option value="done">Completado</option>
      </select>
    </div>
    <div class="form-group">
      <label class="form-label">Prioridad</label>
      <select class="form-select" id="task-priority-input">
        <option value="low">Baja</option>
        <option value="med">Media</option>
        <option value="high">Alta</option>
      </select>
    </div>
    <div class="subtasks-section">
      <label class="form-label">Subtareas</label>
      <div id="subtask-list-edit" class="subtask-list"></div>
      <div class="subtask-input-row">
        <input type="text" id="subtask-input" placeholder="Añadir subtarea…" onkeydown="if(event.key==='Enter') addSubtaskToForm()" />
        <button class="btn-icon" onclick="addSubtaskToForm()">＋</button>
      </div>
    </div>
    <div class="modal-actions">
      <button class="btn btn-ghost" onclick="closeModal('task-modal')">Cancelar</button>
      <button class="btn btn-primary" onclick="saveTask()">Guardar tarea</button>
    </div>
  </div>
</div>

<script>
// ─── State ───────────────────────────────────────────────────────────────────
let state = {
  projects: [],
  activeProject: null,
  view: 'board' // board | list
};

let editingTask = null;
let editingTaskColumn = null;
let formSubtasks = [];
let selectedColor = '#7c6fff';

const COLORS = {
  todo: '#6b6b8a',
  doing: '#7c6fff',
  done: '#6fffd4'
};

const COL_LABELS = {
  todo: 'Por hacer',
  doing: 'En progreso',
  done: 'Completado'
};

// ─── Helpers ─────────────────────────────────────────────────────────────────
function uid() {
  return Math.random().toString(36).slice(2, 9);
}

function getProject(id) {
  return state.projects.find(p => p.id === id);
}

function getActiveProject() {
  return state.projects.find(p => p.id === state.activeProject);
}

// ─── Render ──────────────────────────────────────────────────────────────────
function render() {
  renderSidebar();
  renderMain();
  updateStats();
}

function renderSidebar() {
  const list = document.getElementById('project-list');
  list.innerHTML = '';

  state.projects.forEach(proj => {
    const allTasks = [...proj.tasks.todo, ...proj.tasks.doing, ...proj.tasks.done];
    const total = allTasks.length;
    const done = proj.tasks.done.length;
    const pct = total ? Math.round((done / total) * 100) : 0;

    const div = document.createElement('div');
    div.className = 'project-item' + (proj.id === state.activeProject ? ' active' : '');
    div.style.setProperty('--proj-color', proj.color);
    div.innerHTML = `
      <div class="proj-name">${proj.name}</div>
      <div class="proj-meta">
        <span>${total} tarea${total !== 1 ? 's' : ''}</span>
        <span>${pct}%</span>
      </div>
      <div class="proj-progress-bar">
        <div class="proj-progress-fill" style="width:${pct}%"></div>
      </div>
    `;
    div.onclick = () => setActiveProject(proj.id);
    list.appendChild(div);
  });
}

function renderMain() {
  const proj = getActiveProject();
  const container = document.getElementById('main-content');

  if (!proj) {
    container.innerHTML = `
      <div class="empty-state">
        <div class="big-icon">⚡</div>
        <p>Crea tu primer proyecto para empezar</p>
        <button class="btn btn-primary" onclick="openProjectModal()">Crear proyecto</button>
      </div>`;
    return;
  }

  const allTasks = [...proj.tasks.todo, ...proj.tasks.doing, ...proj.tasks.done];
  const total = allTasks.length;
  const donePct = total ? Math.round((proj.tasks.done.length / total) * 100) : 0;

  container.innerHTML = `
    <div class="project-header">
      <div class="project-title-wrap">
        <h1 style="color:${proj.color}">${proj.name}</h1>
        <p>${proj.description || 'Sin descripción'} · ${total} tareas · ${donePct}% completado</p>
      </div>
      <div class="view-toggle">
        <button class="view-btn ${state.view==='board'?'active':''}" onclick="setView('board')">Tablero</button>
        <button class="view-btn ${state.view==='list'?'active':''}" onclick="setView('list')">Lista</button>
      </div>
    </div>
    <div id="task-area"></div>
  `;

  if (state.view === 'board') renderBoard(proj);
  else renderList(proj);
}

function renderBoard(proj) {
  const area = document.getElementById('task-area');
  const cols = ['todo', 'doing', 'done'];

  // Adapt grid based on task count
  const totalTasks = cols.reduce((s, c) => s + proj.tasks[c].length, 0);
  const colClass = totalTasks > 12 ? 'columns compact' : 'columns';

  area.innerHTML = `<div class="${colClass}" id="board-cols"></div>`;
  const board = document.getElementById('board-cols');

  cols.forEach(col => {
    const tasks = proj.tasks[col];
    const colDiv = document.createElement('div');
    colDiv.className = 'column';
    colDiv.innerHTML = `
      <div class="column-header">
        <div class="column-label">
          <span class="col-dot" style="background:${COLORS[col]}"></span>
          ${COL_LABELS[col]}
        </div>
        <span class="column-count">${tasks.length}</span>
      </div>
      <div class="column-tasks" id="col-${col}"></div>
      <div style="padding:8px 12px 12px">
        <button class="add-task-btn" onclick="openTaskModal('${col}')">＋ Añadir tarea</button>
      </div>
    `;
    board.appendChild(colDiv);

    const colTasksEl = document.getElementById(`col-${col}`);
    tasks.forEach(task => {
      colTasksEl.appendChild(buildTaskCard(task, col, proj));
    });
  });
}

function buildTaskCard(task, col, proj) {
  const card = document.createElement('div');
  card.className = 'task-card' + (task.done ? ' done' : '');

  const subs = task.subtasks || [];
  const subsDone = subs.filter(s => s.done).length;
  const subPct = subs.length ? Math.round((subsDone / subs.length) * 100) : 0;

  const tagClass = { high: 'tag-high', med: 'tag-med', low: 'tag-low' }[task.priority] || 'tag-low';
  const tagLabel = { high: 'Alta', med: 'Media', low: 'Baja' }[task.priority] || 'Baja';

  let subsHtml = '';
  if (subs.length > 0) {
    const shown = subs.slice(0, 3);
    subsHtml = `
      <div class="subtask-preview">
        ${shown.map(s => `
          <div class="subtask-row ${s.done ? 'done-sub' : ''}">
            <div class="sub-check ${s.done ? 'checked' : ''}" onclick="toggleSubtask(event,'${proj.id}','${task.id}','${s.id}','${col}')">
              ${s.done ? '✓' : ''}
            </div>
            <span>${s.text}</span>
          </div>
        `).join('')}
        ${subs.length > 3 ? `<div class="subtask-row" style="padding-left:16px">+${subs.length - 3} más…</div>` : ''}
        <div class="subtask-progress">
          <div class="sub-bar"><div class="sub-bar-fill" style="width:${subPct}%"></div></div>
          <span class="sub-count">${subsDone}/${subs.length}</span>
        </div>
      </div>`;
  }

  card.innerHTML = `
    <div class="task-top">
      <div class="task-check ${task.done ? 'checked' : ''}" onclick="toggleTask(event,'${proj.id}','${task.id}','${col}')">
        ${task.done ? '✓' : ''}
      </div>
      <div class="task-title">${task.title}</div>
    </div>
    <div class="task-meta">
      <span class="task-tag ${tagClass}">${tagLabel}</span>
      ${subs.length > 0 ? `<span class="sub-count">${subs.length} subtarea${subs.length > 1 ? 's' : ''}</span>` : ''}
    </div>
    ${subsHtml}
  `;

  card.onclick = (e) => {
    if (e.target.closest('.task-check') || e.target.closest('.sub-check')) return;
    openTaskModal(col, task, proj.id);
  };

  return card;
}

function renderList(proj) {
  const area = document.getElementById('task-area');
  const cols = ['todo', 'doing', 'done'];
  const statusClass = { todo: 'status-todo', doing: 'status-doing', done: 'status-done-s' };
  const statusLabel = { todo: 'Por hacer', doing: 'En progreso', done: 'Completado' };

  let html = '<div class="list-view">';
  cols.forEach(col => {
    proj.tasks[col].forEach(task => {
      const subs = task.subtasks || [];
      const subsDone = subs.filter(s => s.done).length;
      html += `
        <div class="list-task ${task.done ? 'done' : ''}" onclick="openTaskModal('${col}',null,'${proj.id}')">
          <div class="task-check ${task.done ? 'checked' : ''}" onclick="event.stopPropagation();toggleTask(event,'${proj.id}','${task.id}','${col}')">
            ${task.done ? '✓' : ''}
          </div>
          <div class="list-title">${task.title}</div>
          ${subs.length ? `<span class="sub-count">${subsDone}/${subs.length} subtareas</span>` : ''}
          <span class="list-status ${statusClass[col]}">${statusLabel[col]}</span>
        </div>`;
    });
  });

  html += `<div style="margin-top:16px;display:flex;gap:8px">
    <button class="btn btn-primary" onclick="openTaskModal('todo')">＋ Nueva tarea</button>
  </div></div>`;

  area.innerHTML = html;
}

// ─── Stats ────────────────────────────────────────────────────────────────────
function updateStats() {
  let totalTasks = 0, doneTasks = 0;
  state.projects.forEach(p => {
    totalTasks += p.tasks.todo.length + p.tasks.doing.length + p.tasks.done.length;
    doneTasks += p.tasks.done.length;
  });
  const pct = totalTasks ? Math.round((doneTasks / totalTasks) * 100) : 0;
  document.getElementById('stat-projects').innerHTML = `Proyectos: <strong>${state.projects.length}</strong>`;
  document.getElementById('stat-tasks').innerHTML = `Tareas: <strong>${totalTasks}</strong>`;
  document.getElementById('stat-done').innerHTML = `Completadas: <strong>${pct}%</strong>`;
}

// ─── Actions ──────────────────────────────────────────────────────────────────
function setActiveProject(id) {
  state.activeProject = id;
  render();
}

function setView(v) {
  state.view = v;
  renderMain();
}

function toggleTask(e, projId, taskId, col) {
  e.stopPropagation();
  const proj = getProject(projId);
  const task = proj.tasks[col].find(t => t.id === taskId);
  if (!task) return;

  task.done = !task.done;
  if (task.done && col !== 'done') {
    proj.tasks[col] = proj.tasks[col].filter(t => t.id !== taskId);
    proj.tasks.done.push({ ...task, done: true });
  } else if (!task.done && col === 'done') {
    proj.tasks.done = proj.tasks.done.filter(t => t.id !== taskId);
    proj.tasks.todo.push({ ...task, done: false });
  }
  render();
}

function toggleSubtask(e, projId, taskId, subId, col) {
  e.stopPropagation();
  const proj = getProject(projId);
  const task = proj.tasks[col].find(t => t.id === taskId);
  if (!task) return;
  const sub = task.subtasks.find(s => s.id === subId);
  if (sub) sub.done = !sub.done;
  render();
}

// ─── Modals ───────────────────────────────────────────────────────────────────
function openModal(id) {
  document.getElementById(id).classList.add('open');
}

function closeModal(id) {
  document.getElementById(id).classList.remove('open');
}

// Project modal
function openProjectModal() {
  document.getElementById('proj-name-input').value = '';
  document.getElementById('proj-desc-input').value = '';
  selectedColor = '#7c6fff';
  document.querySelectorAll('.color-dot').forEach(d => {
    d.classList.toggle('selected', d.dataset.color === selectedColor);
  });
  openModal('project-modal');
  setTimeout(() => document.getElementById('proj-name-input').focus(), 100);
}

document.getElementById('color-picker').addEventListener('click', e => {
  const dot = e.target.closest('.color-dot');
  if (!dot) return;
  selectedColor = dot.dataset.color;
  document.querySelectorAll('.color-dot').forEach(d => d.classList.remove('selected'));
  dot.classList.add('selected');
});

function saveProject() {
  const name = document.getElementById('proj-name-input').value.trim();
  if (!name) { document.getElementById('proj-name-input').focus(); return; }

  const proj = {
    id: uid(),
    name,
    description: document.getElementById('proj-desc-input').value.trim(),
    color: selectedColor,
    tasks: { todo: [], doing: [], done: [] }
  };

  state.projects.push(proj);
  state.activeProject = proj.id;
  closeModal('project-modal');
  render();
}

// Task modal
function openTaskModal(defaultCol = 'todo', existingTask = null, projId = null) {
  editingTask = existingTask;
  editingTaskColumn = defaultCol;

  formSubtasks = existingTask ? [...(existingTask.subtasks || [])] : [];

  document.getElementById('task-modal-title').textContent = existingTask ? 'Editar tarea' : 'Nueva tarea';
  document.getElementById('task-title-input').value = existingTask ? existingTask.title : '';
  document.getElementById('task-status-input').value = defaultCol;
  document.getElementById('task-priority-input').value = existingTask ? (existingTask.priority || 'low') : 'low';
  document.getElementById('subtask-input').value = '';

  renderFormSubtasks();
  openModal('task-modal');
  setTimeout(() => document.getElementById('task-title-input').focus(), 100);
}

function renderFormSubtasks() {
  const list = document.getElementById('subtask-list-edit');
  list.innerHTML = '';
  formSubtasks.forEach((s, i) => {
    const div = document.createElement('div');
    div.className = 'subtask-edit-item';
    div.innerHTML = `
      <div class="sub-check ${s.done ? 'checked' : ''}" onclick="formSubtasks[${i}].done=!formSubtasks[${i}].done;renderFormSubtasks()">
        ${s.done ? '✓' : ''}
      </div>
      <span style="flex:1;font-size:12px">${s.text}</span>
      <button class="del-sub" onclick="formSubtasks.splice(${i},1);renderFormSubtasks()">✕</button>
    `;
    list.appendChild(div);
  });
}

function addSubtaskToForm() {
  const inp = document.getElementById('subtask-input');
  const text = inp.value.trim();
  if (!text) return;
  formSubtasks.push({ id: uid(), text, done: false });
  inp.value = '';
  renderFormSubtasks();
  inp.focus();
}

function saveTask() {
  const title = document.getElementById('task-title-input').value.trim();
  if (!title) { document.getElementById('task-title-input').focus(); return; }

  const newCol = document.getElementById('task-status-input').value;
  const priority = document.getElementById('task-priority-input').value;
  const proj = getActiveProject();

  if (editingTask) {
    // Remove from old column
    const oldTasks = proj.tasks[editingTaskColumn];
    const idx = oldTasks.findIndex(t => t.id === editingTask.id);
    if (idx > -1) oldTasks.splice(idx, 1);

    const updated = {
      ...editingTask,
      title,
      priority,
      subtasks: formSubtasks,
      done: newCol === 'done'
    };
    proj.tasks[newCol].push(updated);
  } else {
    proj.tasks[newCol].push({
      id: uid(),
      title,
      priority,
      subtasks: formSubtasks,
      done: newCol === 'done'
    });
  }

  closeModal('task-modal');
  render();
}

// Close modals on overlay click
document.querySelectorAll('.modal-overlay').forEach(overlay => {
  overlay.addEventListener('click', e => {
    if (e.target === overlay) overlay.classList.remove('open');
  });
});

// ─── Demo data ────────────────────────────────────────────────────────────────
(function seed() {
  const p1 = {
    id: uid(), name: 'Rediseño Web', description: 'Nuevo branding y UX',
    color: '#7c6fff',
    tasks: {
      todo: [
        { id: uid(), title: 'Auditoría de contenido', priority: 'med', done: false,
          subtasks: [
            { id: uid(), text: 'Revisar páginas principales', done: true },
            { id: uid(), text: 'Identificar contenido obsoleto', done: false },
            { id: uid(), text: 'Proponer nueva estructura', done: false }
          ]
        },
        { id: uid(), title: 'Briefing con el cliente', priority: 'high', done: false, subtasks: [] }
      ],
      doing: [
        { id: uid(), title: 'Wireframes home y about', priority: 'high', done: false,
          subtasks: [
            { id: uid(), text: 'Boceto en papel', done: true },
            { id: uid(), text: 'Versión digital Figma', done: true },
            { id: uid(), text: 'Revisión interna', done: false }
          ]
        }
      ],
      done: [
        { id: uid(), title: 'Investigación de competencia', priority: 'low', done: true, subtasks: [] }
      ]
    }
  };

  const p2 = {
    id: uid(), name: 'App Móvil', description: 'MVP lanzamiento Q2',
    color: '#ff6f91',
    tasks: {
      todo: [
        { id: uid(), title: 'Definir flujo de usuario', priority: 'high', done: false, subtasks: [] },
        { id: uid(), title: 'Setup repositorio GitHub', priority: 'med', done: false, subtasks: [
          { id: uid(), text: 'Crear repo privado', done: false },
          { id: uid(), text: 'Configurar CI/CD', done: false }
        ]}
      ],
      doing: [
        { id: uid(), title: 'Diseño sistema de auth', priority: 'high', done: false, subtasks: [
          { id: uid(), text: 'Login con email', done: true },
          { id: uid(), text: 'OAuth Google', done: false },
          { id: uid(), text: 'Recuperar contraseña', done: false }
        ]}
      ],
      done: [
        { id: uid(), title: 'Planning inicial', priority: 'low', done: true, subtasks: [] },
        { id: uid(), title: 'Elección de tecnologías', priority: 'med', done: true, subtasks: [] }
      ]
    }
  };

  state.projects = [p1, p2];
  state.activeProject = p1.id;
  render();
})();
</script>
</body>
</html>
