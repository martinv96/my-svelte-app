<!-- ===== SCRIPT ===== -->
<script lang="ts">
	import { onMount } from 'svelte';

	interface Todo {
		id: number;
		text: string;
		completed: boolean;
		priority: 'haute' | 'normale' | 'basse';
		createdAt: string;
	}

	let todos: Todo[] = [];
	let newTodo = '';
	let newPriority: 'haute' | 'normale' | 'basse' = 'normale';
	let filter: 'all' | 'active' | 'completed' = 'all';
	let searchTerm = '';
	let editingId: number | null = null;
	let editingText = '';
	let nextId = 1;

	// Statistiques réactives
	$: totalTodos = todos.length;
	$: completedTodos = todos.filter(t => t.completed).length;
	$: activeTodos = totalTodos - completedTodos;

	// Filtrage par statut
	$: filteredTodos = filter === 'all'
		? todos
		: filter === 'active'
			? todos.filter(t => !t.completed)
			: todos.filter(t => t.completed);

	// Recherche sur les tâches filtrées
	$: searchedTodos = filteredTodos.filter(t =>
		t.text.toLowerCase().includes(searchTerm.toLowerCase())
	);

	// Sauvegarder automatiquement dans le localStorage
	$: if (todos.length > 0) {
		localStorage.setItem('svelte-todos', JSON.stringify(todos));
	}

	onMount(() => {
		// Charger les tâches depuis le localStorage
		const stored = localStorage.getItem('svelte-todos');
		if (stored) {
			const loaded = JSON.parse(stored);
			todos = loaded.map((t: any) => ({ ...t, priority: t.priority || 'normale' }));
			nextId = Math.max(...todos.map(t => t.id)) + 1;
		}

		// Vérifier les changements du localStorage toutes les 500ms
		// pour synchroniser avec les modifications du Dashboard
		const interval = setInterval(() => {
			const updated = localStorage.getItem('svelte-todos');
			if (updated) {
				const newTodos = JSON.parse(updated);
				if (JSON.stringify(newTodos) !== JSON.stringify(todos)) {
					todos = newTodos;
					nextId = Math.max(...todos.map(t => t.id), 0) + 1;
				}
			}
		}, 500);

		return () => clearInterval(interval);
	});

	// Ajouter une nouvelle tâche
	function addTodo() {
		if (newTodo.trim()) {
			todos = [...todos, { id: nextId++, text: newTodo, completed: false, priority: newPriority, createdAt: new Date().toISOString() }];
			newTodo = '';
		}
	}

	// Marquer une tâche comme complétée/active
	function toggleTodo(id: number) {
		todos = todos.map(t => t.id === id ? { ...t, completed: !t.completed } : t);
	}

	// Supprimer une tâche
	function removeTodo(id: number) {
		todos = todos.filter(t => t.id !== id);
	}

	// Supprimer toutes les tâches complétées
	function clearCompleted() {
		todos = todos.filter(t => !t.completed);
	}

	// Commencer à éditer une tâche (double-clic)
	function startEditing(id: number, text: string) {
		editingId = id;
		editingText = text;
	}

	// Sauvegarder les modifications
	function saveEdit(id: number) {
		if (editingText.trim()) {
			todos = todos.map(t => t.id === id ? { ...t, text: editingText } : t);
		}
		editingId = null;
		editingText = '';
	}

	// Annuler l'édition
	function cancelEdit() {
		editingId = null;
		editingText = '';
	}

	// Modifier la priorité d'une tâche
	function changePriority(id: number, priority: 'haute' | 'normale' | 'basse') {
		todos = todos.map(t => t.id === id ? { ...t, priority } : t);
	}

	// Retourner la couleur en fonction de la priorité
	function getPriorityColor(priority: string): string {
		const colors = { 'haute': '#ff6b6b', 'normale': '#ffc93c', 'basse': '#4ecdc4' };
		return colors[priority as keyof typeof colors] || '#95a5a6';
	}
</script>

<!-- ===== STYLES ===== -->
<style>
	.todo-app {
		max-width: 600px;
		margin: 0 auto;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
	}

	.header {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		color: white;
		padding: 2rem;
		border-radius: 15px 15px 0 0;
		text-align: center;
	}

	.stats {
		display: flex;
		justify-content: space-around;
		margin-top: 1rem;
		font-size: 0.9rem;
	}

	.stat {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.stat-value {
		font-size: 1.5rem;
		font-weight: bold;
	}

	.input-group {
		display: flex;
		padding: 1rem;
		background: #f7f7f7;
		gap: 0.5rem;
	}

	input[type="text"] {
		flex: 1;
		padding: 0.8rem;
		border: 2px solid #e0e0e0;
		border-radius: 8px;
		font-size: 1rem;
	}

	input[type="text"]:focus {
		outline: none;
		border-color: #667eea;
	}

	select {
		padding: 0.6rem;
		border: 1px solid #ddd;
		border-radius: 6px;
		cursor: pointer;
	}

	.btn-add {
		padding: 0.8rem 1.5rem;
		background: #667eea;
		color: white;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		font-size: 1rem;
	}

	.btn-add:hover {
		background: #5a67d8;
	}

	.search-box {
		padding: 1rem;
		background: #f0f0f0;
	}

	.search-box input {
		width: 100%;
		padding: 0.8rem;
		border: 2px solid #e0e0e0;
		border-radius: 8px;
		box-sizing: border-box;
	}

	.search-box input:focus {
		outline: none;
		border-color: #667eea;
	}

	.filters {
		display: flex;
		justify-content: center;
		gap: 1rem;
		padding: 1rem;
		background: #f0f0f0;
	}

	.filter-btn {
		padding: 0.5rem 1rem;
		background: white;
		border: 2px solid #e0e0e0;
		border-radius: 6px;
		cursor: pointer;
	}

	.filter-btn.active {
		background: #667eea;
		color: white;
		border-color: #667eea;
	}

	.todo-list {
		list-style: none;
		padding: 0;
		margin: 0;
		background: white;
	}

	.todo-item {
		display: flex;
		align-items: center;
		padding: 1rem;
		border-bottom: 1px solid #f0f0f0;
		gap: 0.8rem;
	}

	.todo-item:hover {
		background: #fafafa;
	}

	.todo-checkbox {
		width: 20px;
		height: 20px;
		cursor: pointer;
	}

	.priority-badge {
		padding: 0.25rem 0.6rem;
		border-radius: 4px;
		color: white;
		font-size: 0.75rem;
		font-weight: bold;
		min-width: 25px;
		text-align: center;
	}

	.todo-text {
		flex: 1;
		font-size: 1rem;
		cursor: pointer;
		color: #000;
	}

	.todo-text.completed {
		text-decoration: line-through;
		color: #999;
	}

	.edit-input {
		flex: 1;
		padding: 0.5rem;
		border: 2px solid #667eea;
		border-radius: 4px;
		font-size: 1rem;
	}

	.btn-save, .btn-cancel, .btn-remove {
		padding: 0.3rem 0.8rem;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-size: 0.875rem;
	}

	.btn-save {
		background: #27ae60;
		color: white;
	}

	.btn-cancel {
		background: #95a5a6;
		color: white;
	}

	.btn-remove {
		background: #e74c3c;
		color: white;
	}

	.todo-date {
		font-size: 0.75rem;
		color: #999;
	}

	.footer {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 1rem;
		background: #f7f7f7;
		border-radius: 0 0 15px 15px;
	}

	.btn-clear {
		padding: 0.5rem 1rem;
		background: #95a5a6;
		color: white;
		border: none;
		border-radius: 6px;
		cursor: pointer;
	}

	.btn-clear:hover {
		background: #7f8c8d;
	}

	.empty-state {
		padding: 3rem;
		text-align: center;
		color: #999;
	}

	/* DARK MODE */
	:global(body.dark) .todo-app,
	:global(body.dark) .todo-list {
		background: #2a2a2a;
	}

	:global(body.dark) .input-group,
	:global(body.dark) .search-box,
	:global(body.dark) .filters,
	:global(body.dark) .footer {
		background: #2a2a2a;
	}

	:global(body.dark) input[type="text"],
	:global(body.dark) select,
	:global(body.dark) .search-box input,
	:global(body.dark) .edit-input {
		background: #333;
		color: white;
		border-color: #444 !important;
	}

	:global(body.dark) .filter-btn {
		background: #333;
		color: white;
		border-color: #444 !important;
	}

	:global(body.dark) .filter-btn.active {
		background: #667eea;
		border-color: #667eea !important;
	}

	:global(body.dark) .todo-item {
		background: #2a2a2a;
		border-color: #444 !important;
		color: white;
	}

	:global(body.dark) .todo-item:hover {
		background: #333;
	}

	:global(body.dark) .todo-text {
		color: white;
	}

	:global(body.dark) .todo-text.completed,
	:global(body.dark) .todo-date {
		color: #999;
	}

	:global(body.dark) .empty-state {
		color: #aaa;
	}
</style>

<!-- ===== TEMPLATE ===== -->
<div class="todo-app">
	<div class="header">
		<h1>📝 Ma Todo List Svelte</h1>
		<div class="stats">
			<div class="stat">
				<span class="stat-value">{totalTodos}</span>
				<span>Total</span>
			</div>
			<div class="stat">
				<span class="stat-value">{activeTodos}</span>
				<span>Actives</span>
			</div>
			<div class="stat">
				<span class="stat-value">{completedTodos}</span>
				<span>Complétées</span>
			</div>
		</div>
	</div>

	<div class="input-group">
		<input type="text" bind:value={newTodo} on:keydown={e => e.key === 'Enter' && addTodo()} placeholder="Ajouter une tâche..." />
		<select bind:value={newPriority}>
			<option value="basse">Basse</option>
			<option value="normale">Normale</option>
			<option value="haute">Haute</option>
		</select>
		<button class="btn-add" on:click={addTodo}>Ajouter</button>
	</div>

	<div class="search-box">
		<input type="text" bind:value={searchTerm} placeholder="Rechercher..." />
	</div>

	<div class="filters">
		<button class="filter-btn" class:active={filter === 'all'} on:click={() => filter = 'all'}>Toutes</button>
		<button class="filter-btn" class:active={filter === 'active'} on:click={() => filter = 'active'}>Actives</button>
		<button class="filter-btn" class:active={filter === 'completed'} on:click={() => filter = 'completed'}>Complétées</button>
	</div>

	{#if searchedTodos.length > 0}
		<ul class="todo-list">
			{#each searchedTodos as todo (todo.id)}
				<li class="todo-item">
					<input type="checkbox" class="todo-checkbox" checked={todo.completed} on:change={() => toggleTodo(todo.id)} />
					<div class="priority-badge" style="background-color: {getPriorityColor(todo.priority)}">{todo.priority.charAt(0).toUpperCase()}</div>

					{#if editingId === todo.id}
						<input type="text" class="edit-input" bind:value={editingText} on:keydown={(e: KeyboardEvent) => { if (e.key === 'Enter') saveEdit(todo.id); if (e.key === 'Escape') cancelEdit(); }} />
						<button class="btn-save" on:click={() => saveEdit(todo.id)}>✓</button>
						<button class="btn-cancel" on:click={cancelEdit}>✕</button>
					{:else}
						<span class="todo-text" class:completed={todo.completed} on:dblclick={() => startEditing(todo.id, todo.text)} role="button" tabindex="0" on:keydown={(e: KeyboardEvent) => { if (e.key === 'Enter') startEditing(todo.id, todo.text); }}>{todo.text}</span>
						<select bind:value={todo.priority} on:change={(e: Event) => changePriority(todo.id, (e.target as HTMLSelectElement).value as 'haute' | 'normale' | 'basse')}>
							<option value="basse">Basse</option>
							<option value="normale">Normale</option>
							<option value="haute">Haute</option>
						</select>
					{/if}

					<span class="todo-date">{new Date(todo.createdAt).toLocaleDateString()}</span>
					<button class="btn-remove" on:click={() => removeTodo(todo.id)}>✕</button>
				</li>
			{/each}
		</ul>
	{:else}
		<div class="empty-state">
			<p>Aucune tâche</p>
		</div>
	{/if}

	{#if completedTodos > 0}
		<div class="footer">
			<span>{completedTodos} tâche{completedTodos > 1 ? 's' : ''} complétée{completedTodos > 1 ? 's' : ''}</span>
			<button class="btn-clear" on:click={clearCompleted}>Effacer</button>
		</div>
	{/if}
</div>