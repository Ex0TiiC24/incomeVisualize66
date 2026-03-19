<script>
	import { onMount } from 'svelte';

	export let data = [];

	let sortBy = 'cv';
	let filteredData = [];

	const sortOptions = [
		{ value: 'cv', label: 'ความเหลื่อมล้ำ (CV)' },
		{ value: 'p90_p10_gap', label: 'ช่องว่าง P90-P10' },
		{ value: 'gap', label: 'ช่องว่าง Max-Min' },
		{ value: 'avg_income', label: 'รายได้เฉลี่ย' }
	];

	onMount(() => {
		updateFiltered();
	});

	$: sortBy, updateFiltered();

	function updateFiltered() {
		if (!data) return;
		const sorted = [...data].sort((a, b) => {
			return b[sortBy] - a[sortBy];
		});
		filteredData = sorted;
	}

	function formatNumber(n) {
		return n.toLocaleString('th-TH');
	}

	function getCVColor(cv) {
		if (cv > 0.4) return '#e74c3c';
		if (cv > 0.35) return '#f39c12';
		return '#2ecc71';
	}

	function getGapColor(val, max) {
		const ratio = val / max;
		if (ratio > 0.8) return '#e74c3c';
		if (ratio > 0.5) return '#f39c12';
		return '#2ecc71';
	}
</script>

<div class="q2-container">
	<div class="controls">
		<label for="sort-select">เรียงลำดับตาม:</label>
		<select id="sort-select" bind:value={sortBy}>
			{#each sortOptions as option}
				<option value={option.value}>{option.label}</option>
			{/each}
		</select>
	</div>

	<div class="charts-grid">
		<!-- Scatter Plot: Avg Income vs Inequality -->
		<div class="chart-card">
			<h3>รายได้เฉลี่ย vs ความเหลื่อมล้ำ</h3>
			<div class="scatter-plot">
				{#each filteredData as item}
					<div 
						class="scatter-point" 
						title="{item.province}: รายได้ {formatNumber(item.avg_income)} บาท, CV {item.cv.toFixed(2)}"
						style="left: {(item.avg_income / Math.max(...filteredData.map(d => d.avg_income))) * 90 + 5}%; bottom: {(item.cv / Math.max(...filteredData.map(d => d.cv))) * 90 + 5}%"
					>
						<div class="tooltip">{item.province}</div>
					</div>
				{/each}
			</div>
			<div class="axis-labels">
				<span class="x-label">รายได้เฉลี่ย →</span>
				<span class="y-label">ความเหลื่อมล้ำ (CV) →</span>
			</div>
		</div>

		<!-- Bar Chart: Top 15 Inequality -->
		<div class="chart-card">
			<h3>15 จังหวัดที่มีความเหลื่อมล้ำมากที่สุด</h3>
			<div class="bars-container">
				{#each filteredData.slice(0, 15) as item, idx}
					<div class="bar-row">
						<div class="bar-label">
							<span class="rank">{idx + 1}</span>
							<span class="province-name">{item.province}</span>
						</div>
						<div class="bar-wrapper">
							<div 
								class="bar" 
								style="width: {(item.cv / Math.max(...filteredData.map(d => d.cv))) * 100}%; background-color: {getCVColor(item.cv)}"
							>
								<span class="bar-value">{item.cv.toFixed(2)}</span>
							</div>
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>

	<!-- Detailed Table -->
	<div class="table-wrapper">
		<table>
			<thead>
				<tr>
					<th>จังหวัด</th>
					<th class="text-right">รายได้เฉลี่ย</th>
					<th class="text-right">ค่ามัธยฐาน</th>
					<th class="text-right">P10</th>
					<th class="text-right">P90</th>
					<th class="text-right">ช่องว่าง P90-P10</th>
					<th class="text-right">ค่าสัมประสิทธิ์ CV</th>
					<th class="text-center">ระดับความเหลื่อมล้ำ</th>
				</tr>
			</thead>
			<tbody>
				{#each filteredData as item, idx}
					<tr class:alert={item.cv > 0.4}>
						<td><strong>{item.province}</strong></td>
						<td class="text-right">{formatNumber(Math.round(item.avg_income))}</td>
						<td class="text-right">{formatNumber(Math.round(item.median_income))}</td>
						<td class="text-right">{formatNumber(Math.round(item.p10))}</td>
						<td class="text-right">{formatNumber(Math.round(item.p90))}</td>
						<td class="text-right"><strong>{formatNumber(Math.round(item.p90_p10_gap))}</strong></td>
						<td class="text-right">
							<span class="cv-badge" style="background-color: {getCVColor(item.cv)}">
								{item.cv.toFixed(2)}
							</span>
						</td>
						<td class="text-center">
							{#if item.cv > 0.4}
								<span class="level-badge high">สูง</span>
							{:else if item.cv > 0.35}
								<span class="level-badge medium">ปานกลาง</span>
							{:else}
								<span class="level-badge low">ต่ำ</span>
							{/if}
						</td>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>
</div>

<style>
	.q2-container {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-lg);
	}

	.controls {
		display: flex;
		align-items: center;
		gap: var(--spacing-md);
	}

	.controls label {
		font-weight: 500;
		color: var(--color-text);
	}

	.controls select {
		padding: var(--spacing-sm) var(--spacing-md);
		border: 1px solid var(--color-border);
		border-radius: 0.375rem;
		font-size: 0.9rem;
		background: var(--color-white);
		cursor: pointer;
	}

	.charts-grid {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: var(--spacing-lg);
	}

	.chart-card {
		background: var(--color-white);
		border: 1px solid var(--color-border);
		border-radius: 0.375rem;
		padding: var(--spacing-lg);
	}

	.chart-card h3 {
		margin-bottom: var(--spacing-lg);
		font-size: 1rem;
	}

	.scatter-plot {
		position: relative;
		width: 100%;
		aspect-ratio: 1;
		background: linear-gradient(to top, var(--color-bg), var(--color-white));
		border: 1px solid var(--color-border);
		border-radius: 0.25rem;
		margin-bottom: var(--spacing-md);
	}

	.scatter-point {
		position: absolute;
		width: 12px;
		height: 12px;
		background: var(--color-primary);
		border-radius: 50%;
		cursor: pointer;
		transform: translate(-50%, -50%);
		transition: all 0.2s ease;
		border: 2px solid var(--color-white);
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
	}

	.scatter-point:hover {
		width: 16px;
		height: 16px;
		z-index: 10;
	}

	.scatter-point .tooltip {
		display: none;
		position: absolute;
		bottom: 20px;
		left: -50px;
		width: 100px;
		background: var(--color-text);
		color: var(--color-white);
		padding: var(--spacing-sm);
		border-radius: 0.25rem;
		font-size: 0.75rem;
		white-space: nowrap;
		text-overflow: ellipsis;
		overflow: hidden;
		z-index: 20;
	}

	.scatter-point:hover .tooltip {
		display: block;
	}

	.axis-labels {
		display: flex;
		justify-content: space-between;
		font-size: 0.8rem;
		color: var(--color-text-light);
	}

	.bars-container {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-sm);
	}

	.bar-row {
		display: flex;
		align-items: center;
		gap: var(--spacing-md);
	}

	.bar-label {
		display: flex;
		align-items: center;
		gap: var(--spacing-sm);
		width: 200px;
		min-width: 200px;
	}

	.rank {
		font-weight: 600;
		color: var(--color-primary);
		font-size: 0.9rem;
		width: 30px;
		text-align: right;
	}

	.province-name {
		font-size: 0.9rem;
		color: var(--color-text);
	}

	.bar-wrapper {
		flex: 1;
		height: 24px;
		background-color: var(--color-bg);
		border-radius: 0.25rem;
		overflow: hidden;
		position: relative;
	}

	.bar {
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: flex-end;
		padding-right: var(--spacing-sm);
		transition: all 0.2s ease;
	}

	.bar-value {
		color: var(--color-white);
		font-weight: 600;
		font-size: 0.75rem;
	}

	.table-wrapper {
		border-radius: 0.375rem;
		border: 1px solid var(--color-border);
		overflow-x: auto;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		background: var(--color-white);
		font-size: 0.85rem;
	}

	thead {
		background-color: var(--color-bg);
		border-bottom: 2px solid var(--color-border);
		position: sticky;
		top: 0;
	}

	th {
		padding: var(--spacing-md);
		text-align: left;
		font-weight: 600;
		color: var(--color-text);
		border-right: 1px solid var(--color-border);
	}

	th:last-child {
		border-right: none;
	}

	td {
		padding: var(--spacing-md);
		border-right: 1px solid var(--color-border);
		border-bottom: 1px solid var(--color-border);
	}

	td:last-child {
		border-right: none;
	}

	tbody tr:hover {
		background-color: rgba(52, 152, 219, 0.02);
	}

	tbody tr.alert {
		background-color: rgba(231, 76, 60, 0.05);
	}

	.text-right {
		text-align: right;
	}

	.text-center {
		text-align: center;
	}

	.cv-badge {
		display: inline-block;
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 0.25rem;
		color: var(--color-white);
		font-weight: 600;
		font-size: 0.8rem;
	}

	.level-badge {
		display: inline-block;
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 0.25rem;
		font-weight: 600;
		font-size: 0.75rem;
	}

	.level-badge.high {
		background-color: rgba(231, 76, 60, 0.1);
		color: #c0392b;
	}

	.level-badge.medium {
		background-color: rgba(243, 156, 18, 0.1);
		color: #d68910;
	}

	.level-badge.low {
		background-color: rgba(46, 204, 113, 0.1);
		color: #27ae60;
	}

	@media (max-width: 768px) {
		.charts-grid {
			grid-template-columns: 1fr;
		}

		.bar-label {
			width: 150px;
			min-width: 150px;
		}

		table {
			font-size: 0.75rem;
		}

		th,
		td {
			padding: var(--spacing-sm);
		}
	}
</style>