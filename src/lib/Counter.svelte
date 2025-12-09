<!-- ===== SCRIPT ===== -->
<script> 
  let count = 0;
  function increment() {
  count += 1; }
  function decrement() {
  count -= 1; }
  function reset() {
    count = 0;
  }
</script>

<!-- ===== STYLES ===== -->
<style>
  .counter-container {
    display: flex;
    flex-direction:
    column;
    align-items: center;
    padding: 2rem;
    background:
    linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  }

  h1 {
    color: white;
    font-size: 2.5rem;
    margin-bottom: 1rem;
  }

  .count-display {
    font-size:
    4rem;
    color: white;
    font-weight: bold;
    margin: 1rem 0;
  }

  .button-group {
  display: flex;
  gap: 1rem; }
  button {
    padding:
    0.8rem 1.5rem;
    font-size: 1.1rem;
    border: none;
    border-radius: 8px;
    background: white;
    color: #667eea;
    cursor: pointer;
    transition:
    transform 0.2s;
  }

  button:hover {
    transform: scale(1.05);
  }

  button:active {
    transform: scale(0.95);
  }
</style>

<!-- ===== TEMPLATE ===== -->
<div class="counter-container">
  <h1>Mon Compteur Svelte</h1>
  <div class="count-display">{count}</div>
  <div class="button-group">
    <button on:click={decrement}>-</button>
    <button on:click={reset}>Reset</button>
    <button on:click={increment}>+</button>
  </div>
</div>