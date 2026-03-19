<script>
	export let label = '';
	export let value = '0';
	export let unit = '';
	export let highlight = false;
</script>

<div class="stat-card" class:highlight>
	<div class="stat-label">{label}</div>
	<div class="stat-value">{value}</div>
	<div class="stat-unit">{unit}</div>
</div>

<style>
	.stat-card {
		background: var(--color-white);
		border: 1px solid var(--color-border);
		border-radius: 0.5rem;
		padding: var(--spacing-lg);
		text-align: center;
		transition: var(--transition);
	}

	.stat-card:hover {
		box-shadow: var(--shadow-md);
		border-color: var(--color-primary);
	}

	.stat-card.highlight {
		border-color: var(--color-primary);
		background: rgba(52, 152, 219, 0.02);
	}

	.stat-label {
		font-size: 0.875rem;
		color: var(--color-text-light);
		margin-bottom: var(--spacing-sm);
		font-weight: 500;
	}

	.stat-value {
		font-size: 1.75rem;
		font-weight: 700;
		color: var(--color-text);
		margin-bottom: var(--spacing-xs);
	}

	.stat-card.highlight .stat-value {
		color: var(--color-primary);
	}

	.stat-unit {
		font-size: 0.8rem;
		color: var(--color-text-light);
	}
</style>