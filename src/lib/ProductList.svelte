<!-- ===== SCRIPT ===== -->
<script lang="ts">
	import { onMount } from 'svelte';

	interface Product {
		id: number;
		name: string;
		price: number;
		rating: number;
		category: string;
		description: string;
		inStock: boolean;
		image: string;
	}

	let products: Product[] = [];
	let searchQuery = '';
	let sortBy: 'name' | 'price' | 'rating' = 'name';
	let sortOrder: 'asc' | 'desc' = 'asc';
	let selectedCategory = 'all';
	let priceRange: [number, number] = [0, 1000];
	let minPrice = 0;
	let maxPrice = 1000;
	let searchTimeout: ReturnType<typeof setTimeout>;

	// Extraire les catégories uniques (calculé une seule fois)
	$: categories = ['all', ...Array.from(new Set(products.map(p => p.category)))];

	// Filtrer les produits par recherche, catégorie et prix
	$: filteredProducts = products
		.filter(p => {
			const matchesSearch = p.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
				p.description.toLowerCase().includes(searchQuery.toLowerCase());
			const matchesCategory = selectedCategory === 'all' || p.category === selectedCategory;
			const matchesPrice = p.price >= priceRange[0] && p.price <= priceRange[1];
			return matchesSearch && matchesCategory && matchesPrice;
		});

	// Trier les produits filtrés (séparé du filtrage pour plus d'efficacité)
	$: sortedProducts = [...filteredProducts].sort((a, b) => {
		let comparison = 0;
		switch(sortBy) {
			case 'name':
				comparison = a.name.localeCompare(b.name);
				break;
			case 'price':
				comparison = a.price - b.price;
				break;
			case 'rating':
				comparison = b.rating - a.rating;
				break;
		}
		return sortOrder === 'asc' ? comparison : -comparison;
	});

	// Statistiques réactives
	$: totalProducts = products.length;
	$: displayedProducts = sortedProducts.length;
	$: averagePrice = filteredProducts.length > 0
		? (filteredProducts.reduce((sum, p) => sum + p.price, 0) / filteredProducts.length).toFixed(2)
		: 0;

	onMount(async () => {
		// Générer les produits une seule fois
		products = generateMockProducts(10);
		maxPrice = Math.max(...products.map(p => p.price));
		priceRange = [0, maxPrice];
	});

	// Générer des produits avec noms cohérents avec leur catégorie
	function generateMockProducts(count: number): Product[] {
		const productsByCategory: Record<string, string[]> = {
			'Électronique': ['Téléphone', 'Ordinateur', 'Montre', 'Tablette'],
			'Vêtements': ['Chemise', 'Pantalon', 'Veste', 'Chaussures'],
			'Alimentation': ['Pizza', 'Burger', 'Café', 'Gâteau'],
			'Livres': ['Roman', 'Manuel', 'Bande dessinée', 'Journal'],
			'Maison': ['Chaise', 'Table', 'Lampe', 'Tapis']
		};
		const adjectives = ['Premium', 'Eco', 'Smart', 'Ultra', 'Pro'];
		const categories = Object.keys(productsByCategory);

		return Array.from({ length: count }, (_, i) => {
			const category = categories[Math.floor(Math.random() * categories.length)];
			const noun = productsByCategory[category][Math.floor(Math.random() * productsByCategory[category].length)];
			const adjective = adjectives[Math.floor(Math.random() * adjectives.length)];
			
			return {
				id: i + 1,
				name: `${adjective} ${noun}`,
				category: category,
				price: Math.floor(Math.random() * 1000) + 10,
				rating: Number((Math.random() * 2 + 3).toFixed(1)),
				description: `Description du produit #${i + 1}`,
				inStock: Math.random() > 0.2,
				image: ''
			};
		});
	}

	// Réinitialiser tous les filtres
	function resetFilters() {
		searchQuery = '';
		selectedCategory = 'all';
		priceRange = [0, maxPrice];
		sortBy = 'name';
		sortOrder = 'asc';
	}

	// Débounce pour la recherche (évite de refiltrer à chaque lettre tapée)
	function handleSearch(event: Event) {
		clearTimeout(searchTimeout);
		const value = (event.target as HTMLInputElement).value;
		searchTimeout = setTimeout(() => {
			searchQuery = value;
		}, 300);
	}

	// Mapper les catégories à des emojis (fait pour eviter d'avoir des images vides)
	function getCategoryIcon(category: string): string {
		const icons: Record<string, string> = {
			'Électronique': '📱',
			'Vêtements': '👕',
			'Alimentation': '🍔',
			'Livres': '📚',
			'Maison': '🏠'
		};
		return icons[category] || '📦';
	}
</script>

<!-- ===== STYLES ===== -->
<style>
	.container {
		max-width: 1200px;
		margin: 0 auto;
		padding: 2rem;
	}

	.header {
		text-align: center;
		margin-bottom: 2rem;
	}

	h1 {
		color: #2c3e50;
		font-size: 2.5rem;
	}

	.stats {
		display: flex;
		justify-content: center;
		gap: 2rem;
		margin: 1rem 0;
		padding: 1rem;
		background: #ecf0f1;
		border-radius: 10px;
	}

	.stat {
		text-align: center;
	}

	.stat-label {
		font-size: 0.875rem;
		color: #7f8c8d;
	}

	.stat-value {
		font-size: 1.5rem;
		font-weight: bold;
		color: #2c3e50;
	}

	.controls {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
		gap: 1rem;
		margin-bottom: 2rem;
		padding: 1.5rem;
		background: white;
		border-radius: 10px;
		box-shadow: 0 2px 10px rgba(0,0,0,0.1);
	}

	.control-group {
		display: flex;
		flex-direction: column;
	}

	label {
		font-size: 0.875rem;
		color: #7f8c8d;
		margin-bottom: 0.5rem;
	}

	input, select {
		padding: 0.5rem;
		border: 1px solid #ddd;
		border-radius: 5px;
		font-size: 1rem;
	}

	input[type="range"] {
		width: 100%;
	}

	.price-display {
		display: flex;
		justify-content: space-between;
		font-size: 0.875rem;
		color: #2c3e50;
		margin-top: 0.25rem;
	}

	.reset-btn {
		grid-column: span 2;
		padding: 0.75rem;
		background: #e74c3c;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
		font-size: 1rem;
		transition: background 0.3s;
	}

	.reset-btn:hover {
		background: #c0392b;
	}

	.products-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
		gap: 1.5rem;
		margin-bottom: 2rem;
	}

	.product-card {
		background: white;
		border-radius: 10px;
		overflow: hidden;
		box-shadow: 0 2px 10px rgba(0,0,0,0.1);
		transition: transform 0.3s, box-shadow 0.3s;
	}

	.product-card:hover {
		transform: translateY(-5px);
		box-shadow: 0 5px 20px rgba(0,0,0,0.15);
	}

	.product-image-container {
		width: 100%;
		height: 200px;
		background: linear-gradient(135deg, #f5f5f5 0%, #eeeeee 100%);
		display: flex;
		align-items: center;
		justify-content: center;
		border-bottom: 1px solid #e0e0e0;
	}

	.product-image-svg {
		width: 100%;
		height: 100%;
	}

	.product-info {
		padding: 1rem;
	}

	.product-name {
		font-size: 1.1rem;
		font-weight: bold;
		margin-bottom: 0.5rem;
		color: #2c3e50;
	}

	.product-category {
		display: inline-block;
		padding: 0.25rem 0.5rem;
		background: #3498db;
		color: white;
		border-radius: 3px;
		font-size: 0.75rem;
		margin-bottom: 0.5rem;
	}

	.product-price {
		font-size: 1.5rem;
		color: #27ae60;
		font-weight: bold;
	}

	.product-rating {
		display: flex;
		align-items: center;
		gap: 0.25rem;
		margin-top: 0.5rem;
	}

	.star {
		color: #f39c12;
	}

	.stock-status {
		margin-top: 0.5rem;
		font-size: 0.875rem;
	}

	.in-stock {
		color: #27ae60;
	}

	.out-of-stock {
		color: #e74c3c;
	}

	.empty-state {
		text-align: center;
		padding: 3rem;
		color: #7f8c8d;
	}

	.loading {
		text-align: center;
		padding: 2rem;
		font-size: 1.2rem;
		color: #3498db;
	}

	/* DARK MODE */
	:global(body.dark) .container,
	:global(body.dark) .stats,
	:global(body.dark) .controls,
	:global(body.dark) .product-card {
		background: #2a2a2a;
		color: white;
	}

	:global(body.dark) h1 {
		color: white;
	}

	:global(body.dark) .stat-label {
		color: #aaa;
	}

	:global(body.dark) .control-group label {
		color: #aaa;
	}

	:global(body.dark) input,
	:global(body.dark) select {
		background: #333;
		color: white;
		border-color: #444 !important;
	}

	:global(body.dark) .reset-btn {
		background: #e74c3c;
	}

	:global(body.dark) .reset-btn:hover {
		background: #c0392b;
	}

	:global(body.dark) .product-card {
		border: 1px solid #444;
	}

	:global(body.dark) .product-image-container {
		background: linear-gradient(135deg, #333 0%, #2a2a2a 100%);
		border-color: #444 !important;
	}

	:global(body.dark) .product-name {
		color: white;
	}

	:global(body.dark) .product-category {
		background: #667eea;
		color: white;
	}

	:global(body.dark) .product-price {
		color: #27ae60;
	}

	:global(body.dark) .in-stock {
		color: #27ae60;
	}

	:global(body.dark) .out-of-stock {
		color: #e74c3c;
	}

	:global(body.dark) .empty-state,
	:global(body.dark) .price-display {
		color: #aaa;
	}

	:global(body.dark) .loading {
		color: #667eea;
	}
</style>

<!-- ===== TEMPLATE ===== -->
<div class="container">
	<div class="header">
		<h1>🛍 Boutique Svelte</h1>
		<div class="stats">
			<div class="stat">
				<div class="stat-label">Total produits</div>
				<div class="stat-value">{totalProducts}</div>
			</div>
			<div class="stat">
				<div class="stat-label">Affichés</div>
				<div class="stat-value">{displayedProducts}</div>
			</div>
			<div class="stat">
				<div class="stat-label">Prix moyen</div>
				<div class="stat-value">{averagePrice}€</div>
			</div>
		</div>
	</div>

	<div class="controls">
		<div class="control-group">
			<label for="search">Rechercher</label>
			<input
				type="text"
				id="search"
				on:input={handleSearch}
				placeholder="Nom ou description..."
			/>
		</div>

		<div class="control-group">
			<label for="category">Catégorie</label>
			<select id="category" bind:value={selectedCategory}>
				{#each categories as category}
					<option value={category}>
						{category === 'all' ? 'Toutes' : category}
					</option>
				{/each}
			</select>
		</div>

		<div class="control-group">
			<label for="price-range">Prix: {priceRange[0]}€ - {priceRange[1]}€</label>
			<input
				type="range"
				id="price-range"
				min={minPrice}
				max={maxPrice}
				bind:value={priceRange[1]}
			/>
			<div class="price-display">
				<span>{minPrice}€</span>
				<span>{maxPrice}€</span>
			</div>
		</div>

		<div class="control-group">
			<label for="sort">Trier par</label>
			<select id="sort" bind:value={sortBy}>
				<option value="name">Nom</option>
				<option value="price">Prix</option>
				<option value="rating">Note</option>
			</select>
		</div>

		<div class="control-group">
			<label for="order">Ordre</label>
			<select id="order" bind:value={sortOrder}>
				<option value="asc">Croissant</option>
				<option value="desc">Décroissant</option>
			</select>
		</div>

		<button class="reset-btn" on:click={resetFilters}>
			Réinitialiser les filtres
		</button>
	</div>

	{#if products.length === 0}
		<div class="loading">Chargement des produits...</div>
	{:else if sortedProducts.length === 0}
		<div class="empty-state">
			<h2>Aucun produit trouvé</h2>
			<p>Essayez de modifier vos critères de recherche</p>
		</div>
	{:else}
		<div class="products-grid">
			{#each sortedProducts as product (product.id)}
				<div class="product-card">
					<div class="product-image-container">
						<svg width="100%" height="100%" viewBox="0 0 200 200" class="product-image-svg">
							<rect fill="#f5f5f5" width="200" height="200" />
							<text x="100" y="110" text-anchor="middle" font-size="80" font-family="Arial, sans-serif">
								{getCategoryIcon(product.category)}
							</text>
						</svg>
					</div>
					<div class="product-info">
						<div class="product-name">{product.name}</div>
						<span class="product-category">{product.category}</span>
						<div class="product-price">{product.price}€</div>
						<div class="product-rating">
							{#each Array(5) as _, i}
								<span class="star">
									{i < Math.floor(product.rating) ? '★' : '☆'}
								</span>
							{/each}
							<span>{product.rating}</span>
						</div>
						<div class="stock-status">
							{#if product.inStock}
								<span class="in-stock">✓ En stock</span>
							{:else}
								<span class="out-of-stock">✗ Rupture</span>
							{/if}
						</div>
					</div>
				</div>
			{/each}
		</div>
	{/if}
</div>