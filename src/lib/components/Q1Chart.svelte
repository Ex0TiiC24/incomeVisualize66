<script>
	export let data = [];

	let maxIncome = 0;

	$: maxIncome = data && data.length > 0 ? Math.max(...data.map((d) => d.avg_income)) : 0;
</script>

<div class="q1-container">
	<div class="chart-wrapper">
		<div class="bars-container">
			{#each data as item, idx (idx)}
				<div class="bar-item">
					<div class="bar-label">
						<span class="occupation-name">{item.soc_eco_class2}</span>
						<span
							class="occupation-type"
							class:employer={item.soc_eco_class1 === 'ลูกจ้าง'}
							class:business={item.soc_eco_class1 === 'ผู้ประกอบธุรกิจของตนเองที่ไม่ใช่การเกษตร'}
							class:farmer={item.soc_eco_class1 === 'ผู้ถือครองทำการเกษตร/เพาะเลี้ยง'}
						>
							{item.soc_eco_class1}
						</span>
					</div>
					<div class="bar-wrapper">
						<div 
							class="bar" 
							style="width: {maxIncome > 0 ? (item.avg_income / maxIncome) * 100 : 0}%"
						>
							<span class="bar-value">
								{item.avg_income.toLocaleString('th-TH')}
							</span>
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>

	<div class="table-wrapper">
		<table>
			<thead>
				<tr>
					<th>อันดับ</th>
					<th>กลุ่มอาชีพ</th>
					<th>ประเภท</th>
					<th class="text-right">รายได้เฉลี่ย (บาท/เดือน)</th>
					<th class="text-right">ช่องว่างจากสูงสุด</th>
				</tr>
			</thead>
			<tbody>
				{#each data as item, idx (idx)}
					<tr class:top-row={idx === 0} class:bottom-row={idx === data.length - 1}>
						<td class="text-center">{item.rank}</td>
						<td>{item.soc_eco_class2}</td>
						<td>
							<span class="type-badge" class:employer={item.soc_eco_class1 === 'ลูกจ้าง'} class:business={item.soc_eco_class1 === 'เจ้าของกิจการ'} class:farmer={item.soc_eco_class1 === 'เกษตรกร'}>
								{item.soc_eco_class1}
							</span>
						</td>
						<td class="text-right income-value">
							{item.avg_income.toLocaleString('th-TH')}
						</td>
						<td class="text-right gap-value">
							{item.gap_from_max.toLocaleString('th-TH')}
						</td>
					</tr>
				{/each}
			</tbody>
		</table>
	</div>
</div>

<style>
	.q1-container {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-xl);
	}

	.chart-wrapper {
		width: 100%;
		background: var(--color-white);
		border-radius: 0.375rem;
		overflow-x: auto;
	}

	.bars-container {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-md);
		padding: var(--spacing-lg);
		min-width: 800px;
	}

	.bar-item {
		display: flex;
		flex-direction: column;
		gap: var(--spacing-sm);
	}

	.bar-label {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: var(--spacing-xs);
	}

	.occupation-name {
		font-size: 0.95rem;
		font-weight: 500;
		color: var(--color-text);
		flex: 1;
	}

	.occupation-type {
		font-size: 0.75rem;
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 9999px;
		margin-left: var(--spacing-sm);
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

	.bar-wrapper {
		width: 100%;
		height: 32px;
		background-color: var(--color-bg);
		border-radius: 0.25rem;
		overflow: hidden;
		position: relative;
	}

	.bar {
		height: 100%;
		background: linear-gradient(to right, var(--color-primary), #2980b9);
		display: flex;
		align-items: center;
		padding: 0 var(--spacing-md);
		transition: all 0.3s ease;
		min-width: 50px;
	}

	.bar:hover {
		box-shadow: inset 0 0 0 2px rgba(255, 255, 255, 0.3);
	}

	.bar-value {
		color: var(--color-white);
		font-weight: 600;
		font-size: 0.875rem;
		white-space: nowrap;
	}

	.table-wrapper {
		overflow-x: auto;
		border-radius: 0.375rem;
		border: 1px solid var(--color-border);
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

	tr:hover {
		background-color: rgba(52, 152, 219, 0.02);
	}

	tr.top-row {
		background-color: rgba(46, 204, 113, 0.05);
	}

	tr.bottom-row {
		background-color: rgba(231, 76, 60, 0.05);
	}

	.text-center {
		text-align: center;
	}

	.text-right {
		text-align: right;
	}

	.type-badge {
		display: inline-block;
		padding: var(--spacing-xs) var(--spacing-sm);
		border-radius: 0.25rem;
		font-size: 0.75rem;
		font-weight: 600;
	}

	.type-badge.employer {
		background-color: rgba(52, 152, 219, 0.1);
		color: var(--color-primary);
	}

	.type-badge.business {
		background-color: rgba(46, 204, 113, 0.1);
		color: var(--color-secondary);
	}

	.type-badge.farmer {
		background-color: rgba(243, 156, 18, 0.1);
		color: var(--color-warning);
	}

	.income-value {
		font-weight: 600;
		color: var(--color-primary);
	}

	.gap-value {
		color: var(--color-text-light);
		font-size: 0.85rem;
	}

	@media (max-width: 768px) {
		.bars-container {
			min-width: 100%;
		}

		table {
			font-size: 0.8rem;
		}

		th,
		td {
			padding: var(--spacing-sm);
		}
	}
</style>
