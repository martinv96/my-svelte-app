<!-- ===== SCRIPT ===== -->
<script lang="ts">
  import { onMount } from "svelte";

  interface Task {
    id: number;
    text: string;
    completed: boolean;
    priority: "haute" | "normale" | "basse";
  }

  let tasks: Task[] = [];
  let darkMode = false;
  let draggedTaskId: number | null = null;

  // Calculs réactifs pour les statistiques
  $: totalTasks = tasks.length;
  $: completedTasks = tasks.filter((t) => t.completed).length;
  $: activeTasks = tasks.filter((t) => !t.completed).length;
  $: completionRate =
    totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0;

  // Données pour les graphiques, calculées automatiquement
  $: statusData = [
    { label: "Complétées", value: completedTasks, color: "#27ae60" },
    { label: "Actives", value: activeTasks, color: "#3498db" },
  ];

  $: priorityData = {
    haute: tasks.filter((t) => t.priority === "haute").length,
    normale: tasks.filter((t) => t.priority === "normale").length,
    basse: tasks.filter((t) => t.priority === "basse").length,
  };

  onMount(() => {
    // Charger les tâches depuis le localStorage (partagé avec TodoList)
    const saved = localStorage.getItem("svelte-todos");
    if (saved) {
      tasks = JSON.parse(saved);
    }

    // Charger le thème sauvegardé
    const savedTheme = localStorage.getItem("dashboardTheme");
    darkMode = savedTheme === "dark";
    if (darkMode) {
      document.body.classList.add("dark");
    }

    // Vérifier les changements du localStorage toutes les 500ms
    // pour synchroniser avec les modifications de TodoList
    const interval = setInterval(() => {
      const updated = localStorage.getItem("svelte-todos");
      if (updated) {
        const newTasks = JSON.parse(updated);
        if (JSON.stringify(newTasks) !== JSON.stringify(tasks)) {
          tasks = newTasks;
        }
      }
    }, 500);

    return () => clearInterval(interval);
  });

  // Basculer entre mode clair et mode sombre
  function toggleTheme() {
    darkMode = !darkMode;
    localStorage.setItem("dashboardTheme", darkMode ? "dark" : "light");
    document.body.classList.toggle("dark");
  }

  // Initialiser le drag
  function handleDragStart(e: DragEvent, id: number) {
    draggedTaskId = id;
    if (e.dataTransfer) {
      e.dataTransfer.effectAllowed = "move";
    }
  }

  // Permettre le drop à cette position
  function handleDragOver(e: DragEvent) {
    e.preventDefault();
    if (e.dataTransfer) {
      e.dataTransfer.dropEffect = "move";
    }
  }

  // Effectuer le drop et réorganiser les tâches
  function handleDrop(e: DragEvent, targetId: number) {
    e.preventDefault();
    if (draggedTaskId === null || draggedTaskId === targetId) return;

    const draggedIndex = tasks.findIndex((t) => t.id === draggedTaskId);
    const targetIndex = tasks.findIndex((t) => t.id === targetId);

    if (draggedIndex !== -1 && targetIndex !== -1) {
      const newTasks = [...tasks];
      [newTasks[draggedIndex], newTasks[targetIndex]] = [
        newTasks[targetIndex],
        newTasks[draggedIndex],
      ];
      tasks = newTasks;
      saveTasks();
    }

    draggedTaskId = null;
  }

  // Terminer le drag
  function handleDragEnd() {
    draggedTaskId = null;
  }

  // Sauvegarder dans le localStorage
  function saveTasks() {
    localStorage.setItem("svelte-todos", JSON.stringify(tasks));
  }

  // Modifier la priorité d'une tâche
  function changePriority(id: number, priority: "haute" | "normale" | "basse") {
    tasks = tasks.map((t) => (t.id === id ? { ...t, priority } : t));
    saveTasks();
  }

  // Marquer une tâche comme complétée/active
  function toggleTask(id: number) {
    tasks = tasks.map((t) =>
      t.id === id ? { ...t, completed: !t.completed } : t
    );
    saveTasks();
  }

  // Supprimer une tâche
  function deleteTask(id: number) {
    tasks = tasks.filter((t) => t.id !== id);
    saveTasks();
  }
</script>

<!-- ===== STYLES ===== -->
<style>
  .dashboard {
    padding: 2rem;
    max-width: 1000px;
    border-radius: 8px;
    margin: 0 auto;
    color: #000;
    background: white;
    transition: all 0.3s ease;
  }

  .dashboard.dark {
    background: #1a1a1a;
    color: white;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
  }
  .theme-btn {
    background: none;
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    padding: 0.5rem;
  }

  h1,
  h2 {
    margin-bottom: 1rem;
    color: #000;
  }
  .dark h1,
  .dark h2 {
    color: white;
  }
  h1 {
    font-size: 2rem;
    margin-bottom: 2rem;
  }
  h2 {
    font-size: 1.3rem;
    margin-top: 2rem;
  }

  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(-20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /* Statistiques */
  .stats-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
  }
  .stat-widget {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    padding: 1.5rem;
    border-radius: 10px;
    text-align: center;
    animation: slideIn 0.5s ease-out;
  }
  .stat-number {
    font-size: 2.5rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
  }
  .stat-label {
    font-size: 0.9rem;
    opacity: 0.9;
  }

  /* Graphiques */
  .charts-section {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
  }
  .chart-container {
    background: #f8f9fa;
    padding: 1.5rem;
    border-radius: 10px;
  }
  .dark .chart-container {
    background: #2a2a2a;
  }
  .chart {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  .chart.horizontal {
    gap: 1.5rem;
  }
  .chart-item {
    display: flex;
    align-items: center;
    gap: 1rem;
  }
  .chart-label {
    min-width: 100px;
    font-weight: bold;
    color: #2c3e50;
  }
  .dark .chart-label {
    color: white;
  }
  .chart-bar-container {
    flex: 1;
    background: #e0e0e0;
    border-radius: 5px;
    overflow: hidden;
    height: 30px;
    position: relative;
  }
  .dark .chart-bar-container {
    background: #333;
  }
  .chart-bar {
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: width 0.3s ease;
    min-width: 0;
  }
  .chart-value {
    color: white;
    font-weight: bold;
    font-size: 0.9rem;
    padding: 0 0.5rem;
  }

  /* Priorité */
  .priority-chart {
    flex-direction: row;
    align-items: flex-end;
    gap: 2rem;
    justify-content: space-around;
  }
  .priority-bar {
    display: flex;
    gap: 2rem;
    width: 100%;
    justify-content: space-around;
    align-items: flex-end;
    height: 250px;
  }
  .priority-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
    flex: 1;
    max-width: 100px;
  }
  .priority-label {
    font-weight: bold;
    font-size: 0.9rem;
    color: #2c3e50;
  }
  .dark .priority-label {
    color: white;
  }
  .priority-bar-inner {
    width: 60px;
    height: 200px;
    border-radius: 5px;
    overflow: hidden;
    display: flex;
    flex-direction: column-reverse;
    background: #e0e0e0;
  }
  .dark .priority-bar-inner {
    background: #333;
  }
  .priority-fill {
    width: 100%;
    transition: height 0.3s ease;
  }
  .priority-count {
    font-weight: bold;
    font-size: 1.2rem;
    color: #2c3e50;
  }
  .dark .priority-count {
    color: white;
  }

  /* Tâches */
  .tasks-section {
    background: #f8f9fa;
    padding: 1.5rem;
    border-radius: 10px;
    margin-top: 2rem;
  }
  .dark .tasks-section {
    background: #2a2a2a;
  }
  .tasks-list {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }
  .task-item {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    background: white;
    border: 2px solid #ddd;
    border-radius: 5px;
    cursor: move;
    transition: all 0.2s ease;
  }
  .dark .task-item {
    background: #333;
    border-color: #444;
    color: white;
  }
  .task-item:hover {
    background: #f0f0f0;
    border-color: #667eea;
  }
  .task-item.completed {
    opacity: 0.6;
  }
  .task-item input[type="checkbox"] {
    width: 20px;
    height: 20px;
    cursor: pointer;
  }
  .task-content {
    flex: 1;
    display: flex;
    align-items: center;
    gap: 1rem;
  }
  .task-text {
    flex: 1;
    color: #000;
  }
  .dark .task-text {
    color: white;
  }
  .task-item.completed .task-text {
    text-decoration: line-through;
    color: #999;
  }
  .priority-select {
    padding: 0.5rem;
    border: 1px solid #ddd;
    border-radius: 3px;
    cursor: pointer;
    font-size: 0.85rem;
  }
  .dark .priority-select {
    background: #444;
    border-color: #555;
    color: white;
  }
  .delete-btn {
    width: 30px;
    height: 30px;
    background: #ff6b6b;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1.2rem;
    font-weight: bold;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .delete-btn:hover {
    background: #ff5252;
  }

  @media (max-width: 768px) {
    .task-item {
      flex-direction: column;
      align-items: flex-start;
    }
    .task-content {
      flex-direction: column;
      width: 100%;
    }
  }
</style>

<!-- ===== TEMPLATE ===== -->
<div class="dashboard {darkMode ? 'dark' : ''}">
  <div class="header">
    <h1>📊 Dashboard</h1>
    <button class="theme-btn" on:click={toggleTheme}>
      {darkMode ? "☀️" : "🌙"}
    </button>
  </div>

  <!-- WIDGETS STATISTIQUES -->
  <div class="stats-container">
    <div class="stat-widget">
      <div class="stat-number">{totalTasks}</div>
      <div class="stat-label">Total</div>
    </div>
    <div class="stat-widget">
      <div class="stat-number">{completedTasks}</div>
      <div class="stat-label">Complétées</div>
    </div>
    <div class="stat-widget">
      <div class="stat-number">{activeTasks}</div>
      <div class="stat-label">Actives</div>
    </div>
    <div class="stat-widget">
      <div class="stat-number">{completionRate}%</div>
      <div class="stat-label">Taux</div>
    </div>
  </div>

  <!-- GRAPHIQUES -->
  <div class="charts-section">
    <!-- Graphique 1: Complétées vs Actives -->
    <div class="chart-container">
      <h2>📊 Statut des Tâches</h2>
      <div class="chart horizontal">
        {#each statusData as data (data.label)}
          <div class="chart-item">
            <div class="chart-label">{data.label}</div>
            <div class="chart-bar-container">
              <div
                class="chart-bar"
                style="width: {totalTasks > 0
                  ? (data.value / totalTasks) * 100
                  : 0}%; background: {data.color};"
              >
                <span class="chart-value">{data.value}</span>
              </div>
            </div>
          </div>
        {/each}
      </div>
    </div>

    <!-- Graphique 2: Distribution par Priorité -->
    <div class="chart-container">
      <h2>⚡ Distribution par Priorité</h2>
      <div class="chart priority-chart">
        <div class="priority-bar">
          <div class="priority-item">
            <span class="priority-label">Haute</span>
            <div class="priority-bar-inner">
              <div
                class="priority-fill"
                style="height: {totalTasks > 0
                  ? (priorityData.haute / totalTasks) * 100
                  : 0}%; background: #e74c3c;"
              ></div>
            </div>
            <span class="priority-count">{priorityData.haute}</span>
          </div>
          <div class="priority-item">
            <span class="priority-label">Normale</span>
            <div class="priority-bar-inner">
              <div
                class="priority-fill"
                style="height: {totalTasks > 0
                  ? (priorityData.normale / totalTasks) * 100
                  : 0}%; background: #f39c12;"
              ></div>
            </div>
            <span class="priority-count">{priorityData.normale}</span>
          </div>
          <div class="priority-item">
            <span class="priority-label">Basse</span>
            <div class="priority-bar-inner">
              <div
                class="priority-fill"
                style="height: {totalTasks > 0
                  ? (priorityData.basse / totalTasks) * 100
                  : 0}%; background: #27ae60;"
              ></div>
            </div>
            <span class="priority-count">{priorityData.basse}</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- LISTE DE TÂCHES AVEC DRAG & DROP -->
  <div class="tasks-section">
    <h2>📋 Tâches</h2>
    <div class="tasks-list">
      {#each tasks as task (task.id)}
        <div
          class="task-item {task.completed ? 'completed' : ''}"
          draggable="true"
          role="button"
          tabindex="0"
          on:dragstart={(e) => handleDragStart(e, task.id)}
          on:dragover={handleDragOver}
          on:drop={(e) => handleDrop(e, task.id)}
          on:dragend={handleDragEnd}
        >
          <input
            type="checkbox"
            checked={task.completed}
            on:change={() => toggleTask(task.id)}
          />
          <div class="task-content">
            <span class="task-text">{task.text}</span>
            <select
              value={task.priority}
              on:change={(e) =>
                changePriority(task.id, e.currentTarget.value as any)}
              class="priority-select"
            >
              <option value="basse">Basse</option>
              <option value="normale">Normale</option>
              <option value="haute">Haute</option>
            </select>
          </div>
          <button class="delete-btn" on:click={() => deleteTask(task.id)}
            >×</button
          >
        </div>
      {/each}
    </div>
  </div>
</div>


