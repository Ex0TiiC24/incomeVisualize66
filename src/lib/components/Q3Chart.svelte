<script>
	import { onMount } from 'svelte';

	export let data = [];

	let occupations = [];
	let selectedOccupation = null;
	let filteredData = [];

	onMount(() => {
		if (data && data.length > 0) {
			// Get unique occupations
			const uniqueOccupations = [...new Set(data.map(d => d.soc_eco_class2))];
			occupations = uniqueOccupations;
			selectedOccupation = uniqueOccupations[0];
			updateFiltered();
		}
	});

	$: selectedOccupation, updateFiltered();

	function updateFiltered() {
		if (!selectedOccupation) return;
		filteredData = data.filter(d => d.soc_eco_class2 === selectedOccupation);
	}

	$: mainSource = filteredData.length > 0 ? [...filteredData].sort((a, b) => b.percentage - a.percentage)[0] : null;
	$: supportingSources = mainSource ? filteredData.filter(d => d.percentage > 5 && d.percentage < mainSource.percentage) : [];

	function formatNumber(n) {
		return n.toLocaleString('th-TH');
	}

	function getColorBySource(source) {
		const colors = {
			'ค่าจ้างและเงินเดือน': '#3498db',
			'กำไรสุทธิจากการทำธุรกิจ': '#2ecc71',
			'กำไรสุทธิจากการทำการเกษตร': '#f39c12',
			'เงินที่ได้รับเป็นการช่วยเหลือ': '#e74c3c',
			'รายได้จากทรัพย์สิน': '#9b59b6',
			'รายได้ไม่ประจำ (ที่เป็นตัวเงิน)': '#1abc9c',
			'รายได้ที่ไม่เป็นตัวเงิน': '#95a5a6'
		};
		return colors[source] || '#34495e';
	}

	function getOccupationType(occ) {
		const first = data.find(d => d.soc_eco_class2 === occ);
		return first?.soc_eco_class1 || '';
	}

	function getTotalIncome(occ) {
		const filtered = data.filter(d => d.soc_eco_class2 === occ);
		if (filtered.length === 0) return 0;
		return filtered[0].total_income;
	}
</script>

<div class="q3-container">
	<div class="occupation-selector">
		<label for="occupation-select">เลือกกลุ่มอาชีพ:</label>
		<select id="occupation-select" bind:value={selectedOccupation}>
			{#each occupations as occ}
				<option value={occ}>{occ}</option>
			{/each}
		</select>
		<span class="occupation-type" class:employer={getOccupationType(selectedOccupation) === 'ลูกจ้าง'} class:business={getOccupationType(selectedOccupation) === 'เจ้าของกิจการ'} class:farmer={getOccupationType(selectedOccupation) === 'เกษตรกร'}>
			{getOccupationType(selectedOccupation)}
		</span>
	</div>

	{#if filteredData.length > 0}
		<div class="charts-grid">
			<!-- Pie Chart / Donut -->
			<div class="chart-card">
				<h3>โครงสร้างรายได้</h3>
				<div class="pie-container">
					<div class="pie-chart">
						{#each filteredData as item, idx}
							{@const startAngle = filteredData.slice(0, idx).reduce((sum, d) => sum + d.percentage, 0)}
							{@const angle = item.percentage}
							<div 
								class="pie-segment" 
								style="
									--start-angle: {startAngle}deg;
									--angle: {angle}deg;
									--color: {getColorBySource(item.source_income3)};
									background: conic-gradient(
										from {startAngle}deg,
										{getColorBySource(item.source_income3)} {angle}%,
										transparent {angle}%
									);
								"
								title="{item.source_income3}: {item.percentage}%"
							></div>
						{/each}
						<div class="pie-center">
							<div class="total-income">
								{formatNumber(Math.round(getTotalIncome(selectedOccupation)))}
								<span>บาท</span>
							</div>
						</div>
					</div>
				</div>
				<div class="legend">
					{#each filteredData as item}
						<div class="legend-item">
							<span class="legend-color" style="background-color: {getColorBySource(item.source_income3)}"></span>
							<span class="legend-label">{item.source_income3}</span>
							<span class="legend-value">{item.percentage}%</span>
						</div>
					{/each}
				</div>
			</div>

			<!-- Horizontal Stacked Bar -->
			<div class="chart-card">
				<h3>สัดส่วนรายได้แต่ละแหล่ง</h3>
				<div class="stacked-bar-wrapper">
					<div class="stacked-bar">
						{#each filteredData as item}
							<div 
								class="segment" 
								style="
									width: {item.percentage}%;
									background-color: {getColorBySource(item.source_income3)};
								"
								title="{item.source_income3}: {item.percentage}%"
							>
								{#if item.percentage > 8}
									<span class="segment-label">{item.percentage}%</span>
								{/if}
							</div>
						{/each}
					</div>
				</div>
				<div class="bar-details">
					{#each filteredData as item}
						<div class="detail-row">
							<div class="detail-source">
								<span class="dot" style="background-color: {getColorBySource(item.source_income3)}"></span>
								{item.source_income3}
							</div>
							<div class="detail-amount">
								{formatNumber(Math.round(item.income))} บาท
							</div>
							<div class="detail-percent">
								{item.percentage}%
							</div>
						</div>
					{/each}
				</div>
			</div>
		</div>

		<!-- Income Composition Table -->
		<div class="table-wrapper">
			<h3>รายละเอียดโครงสร้างรายได้</h3>
			<table>
				<thead>
					<tr>
						<th>แหล่งที่มาของรายได้</th>
						<th class="text-right">รายได้ (บาท/เดือน)</th>
						<th class="text-right">สัดส่วน (%)</th>
						<th class="text-center">ความสำคัญ</th>
					</tr>
				</thead>
				<tbody>
					{#each filteredData.sort((a, b) => b.percentage - a.percentage) as item}
						<tr class:primary={item.percentage > 50} class:secondary={item.percentage > 20} class:tertiary={item.percentage <= 20}>
							<td>
								<div class="source-label">
									<span class="source-dot" style="background-color: {getColorBySource(item.source_income3)}"></span>
									{item.source_income3}
								</div>
							</td>
							<td class="text-right">
								<strong>{formatNumber(Math.round(item.income))}</strong>
							</td>
							<td class="text-right">
								<span class="percentage-badge" style="background-color: {getColorBySource(item.source_income3)}">
									{item.percentage}%
								</span>
							</td>
							<td class="text-center">
								{#if item.percentage > 50}
									<span class="importance primary">หลัก</span>
								{:else if item.percentage > 20}
									<span class="importance secondary">ทุติยภูมิ</span>
								{:else}
									<span class="importance tertiary">เพิ่มเติม</span>
								{/if}
							</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>

		<!-- Insights -->
		<div class="insights-card">
			<h3>💡 วิเคราะห์โครงสร้างรายได้</h3>
			<div class="insights-list">
				{#if mainSource}
				<div class="insight-item">
					<strong>แหล่งรายได้หลัก:</strong>
					<p>
						{mainSource.source_income3} 
						<span class="highlight">{mainSource.percentage}%</span>
						ของรายได้ทั้งหมด
					</p>
				</div>
				{#if supportingSources.length > 0}
					<div class="insight-item">
						<strong>แหล่งรายได้เสริม:</strong>
						<p>
							{supportingSources.map(s => `${s.source_income3} (${s.percentage}%)`).join(', ')}
						</p>
					</div>
				{/if}
				<div class="insight-item">
					<strong>ความเสี่ยง:</strong>
					<p>
						{#if mainSource.percentage > 80}
							⚠️ ความเสี่ยงสูง: พึ่งพาแหล่งรายได้เพียงแหล่งเดียวมากเกินไป
						{:else if mainSource.percentage > 60}
							⚠️ ความเสี่ยงปานกลาง: หากแหล่งรายได้หลักขาด อาจสัมผัสปัญหาทางเศรษฐกิจ
						{:else}
							✅ ความเสี่ยงต่ำ: มีความหลากหลายในแหล่งรายได้
						{/if}
					</p>
				</div>
			{/if}
			</div>
		</div>
	{/if}
</div>

<style>
	.q3-container {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-lg);
	}

	.occupation-selector {
		display: flex;
		align-items: center;
		gap: var(--spacing-md);
		padding: var(--spacing-lg);
		background: var(--color-white);
		border: 1px solid var(--color-border);
		border-radius: 0.375rem;
	}

	.occupation-selector label {
		font-weight: 500;
		color: var(--color-text);
		white-space: nowrap;
	}

	.occupation-selector select {
		padding: var(--spacing-sm) var(--spacing-md);
		border: 1px solid var(--color-border);
		border-radius: 0.375rem;
		font-size: 0.9rem;
		background: var(--color-white);
		cursor: pointer;
		flex: 1;
	}

	.occupation-type {
		padding: var(--spacing-xs) var(--spacing-md);
		border-radius: 9999px;
		font-size: 0.8rem;
		font-weight: 600;
		white-space: nowrap;
	}

	.occupation-type.employer {
		background-color: rgba(52, 152, 219, 0.1);
		color: var(--color-primary);
	}

	.occupation-type.business {
		background-color: rgba(46, 204, 113, 0.1);
		color: var(--color-secondary);
	}

	.occupation-type.farmer {
		background-color: rgba(243, 156, 18, 0.1);
		color: var(--color-warning);
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

	.pie-container {
		display: flex;
		justify-content: center;
		margin-bottom: var(--spacing-lg);
	}

	.pie-chart {
		position: relative;
		width: 200px;
		height: 200px;
		border-radius: 50%;
		background: conic-gradient(
			#3498db 0deg 90deg,
			#2ecc71 90deg 180deg,
			#f39c12 180deg 270deg,
			#e74c3c 270deg 360deg
		);
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	}

	.pie-segment {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		border-radius: 50%;
	}

	.pie-center {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		width: 140px;
		height: 140px;
		background: var(--color-white);
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: inset 0 0 0 2px var(--color-border);
	}

	.total-income {
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: var(--spacing-xs);
	}

	.total-income {
		font-weight: 700;
		font-size: 1.1rem;
		color: var(--color-primary);
	}

	.total-income span {
		font-size: 0.75rem;
		color: var(--color-text-light);
		font-weight: 400;
	}

	.legend {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-sm);
		margin-top: var(--spacing-lg);
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: var(--spacing-sm);
		font-size: 0.85rem;
	}

	.legend-color {
		width: 16px;
		height: 16px;
		border-radius: 0.25rem;
		flex-shrink: 0;
	}

	.legend-label {
		flex: 1;
		color: var(--color-text);
	}

	.legend-value {
		font-weight: 600;
		color: var(--color-text);
		min-width: 40px;
		text-align: right;
	}

	.stacked-bar-wrapper {
		margin-bottom: var(--spacing-lg);
	}

	.stacked-bar {
		display: flex;
		height: 40px;
		border-radius: 0.25rem;
		overflow: hidden;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
	}

	.segment {
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		transition: all 0.2s ease;
	}

	.segment:hover {
		opacity: 0.8;
		z-index: 10;
	}

	.segment-label {
		color: var(--color-white);
		font-weight: 600;
		font-size: 0.75rem;
		text-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
	}

	.bar-details {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-sm);
	}

	.detail-row {
		display: flex;
		align-items: center;
		gap: var(--spacing-md);
		padding: var(--spacing-sm) 0;
		border-bottom: 1px solid var(--color-border);
		font-size: 0.9rem;
	}

	.detail-row:last-child {
		border-bottom: none;
	}

	.detail-source {
		flex: 1;
		display: flex;
		align-items: center;
		gap: var(--spacing-sm);
		color: var(--color-text);
	}

	.dot {
		width: 12px;
		height: 12px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	.detail-amount {
		font-weight: 600;
		color: var(--color-primary);
		min-width: 120px;
		text-align: right;
	}

	.detail-percent {
		min-width: 50px;
		text-align: right;
		color: var(--color-text-light);
	}

	.table-wrapper {
		background: var(--color-white);
		border: 1px solid var(--color-border);
		border-radius: 0.375rem;
		padding: var(--spacing-lg);
	}

	.table-wrapper h3 {
		margin-bottom: var(--spacing-lg);
		font-size: 1rem;
	}

	.table-wrapper {
		overflow-x: auto;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		background: var(--color-white);
		font-size: 0.9rem;
	}

	thead {
		background-color: var(--color-bg);
		border-bottom: 2px solid var(--color-border);
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

	tbody tr.primary {
		background-color: rgba(52, 152, 219, 0.05);
	}

	.source-label {
		display: flex;
		align-items: center;
		gap: var(--spacing-sm);
		color: var(--color-text);
	}

	.source-dot {
		width: 12px;
		height: 12px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	.percentage-badge {
		display: inline-block;
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 0.25rem;
		color: var(--color-white);
		font-weight: 600;
		font-size: 0.8rem;
	}

	.importance {
		display: inline-block;
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 0.25rem;
		font-weight: 600;
		font-size: 0.75rem;
	}

	.importance.primary {
		background-color: rgba(52, 152, 219, 0.1);
		color: var(--color-primary);
	}

	.importance.secondary {
		background-color: rgba(243, 156, 18, 0.1);
		color: var(--color-warning);
	}

	.importance.tertiary {
		background-color: rgba(159, 168, 218, 0.1);
		color: #5d7e9f;
	}

	.text-right {
		text-align: right;
	}

	.text-center {
		text-align: center;
	}

	.insights-card {
		background: rgba(52, 152, 219, 0.05);
		border: 1px solid rgba(52, 152, 219, 0.2);
		border-radius: 0.375rem;
		padding: var(--spacing-lg);
	}

	.insights-card h3 {
		margin-bottom: var(--spacing-lg);
		color: var(--color-primary);
	}

	.insights-list {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-md);
	}

	.insight-item {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-xs);
	}

	.insight-item strong {
		color: var(--color-text);
		font-weight: 600;
	}

	.insight-item p {
		margin-bottom: 0;
		color: var(--color-text);
		line-height: 1.6;
	}

	.highlight {
		color: var(--color-primary);
		font-weight: 700;
		background-color: rgba(52, 152, 219, 0.1);
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 0.25rem;
	}

	@media (max-width: 768px) {
		.charts-grid {
			grid-template-columns: 1fr;
		}

		.occupation-selector {
			flex-direction: column;
			align-items: flex-start;
		}

		.occupation-selector select {
			width: 100%;
		}

		table {
			font-size: 0.8rem;
		}

		th,
		td {
			padding: var(--spacing-sm);
		}

		.detail-row {
			flex-wrap: wrap;
		}
	}
</style>