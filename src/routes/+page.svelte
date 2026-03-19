<script>
	import { onMount } from 'svelte';
	import Q1Chart from '$lib/components/Q1Chart.svelte';
	import Q2Map from '$lib/components/Q2Map.svelte';
	import Q2Chart from '$lib/components/Q2Chart.svelte';
	import Q3Chart from '$lib/components/Q3Chart.svelte';
	import StatCard from '$lib/components/StatCard.svelte';

	let data = null;
	let loading = true;
	let error = null;

	onMount(async () => {
		try {
			const response = await fetch('/data.json');
			if (!response.ok) throw new Error('Failed to load data');
			data = await response.json();
			loading = false;
		} catch (err) {
			error = err.message;
			loading = false;
		}
	});
</script>

<div class="page-wrapper">
	<header class="header">
		<div class="container">
			<div class="header-content">
				<div>
					<h1>รายได้เฉลี่ยต่อเดือนของครัวเรือนไทย</h1>
					<p class="subtitle">สำนักงานสถิติแห่งชาติ (NSO) | ปีงบประมาณ 2566</p>
					<p class="subtitle">600345 Taiyo</p>
				</div>
				<div class="header-meta">
					<a href="https://data.go.th/dataset/os_08_00007" target="_blank" rel="noopener">
						ที่มาข้อมูล
					</a>
				</div>
			</div>
		</div>
	</header>

	<main class="container">
		{#if loading}
			<div class="loading">
				<p>กำลังโหลดข้อมูล...</p>
			</div>
		{:else if error}
			<div class="error">
				<p>เกิดข้อผิดพลาด: {error}</p>
			</div>
		{:else if data}
			<!-- Summary Stats -->
			<section class="summary-section">
				<h2>สรุปข้อมูลทั่วไป</h2>
				<div class="grid grid-cols-4">
					<StatCard
						label="จำนวนรายการข้อมูล"
						value={data.summary.total_records}
						unit="รายการ"
					/>
					<StatCard
						label="จำนวนจังหวัด"
						value={data.summary.total_provinces}
						unit="จังหวัด"
					/>
					<StatCard
						label="รายได้เฉลี่ยของข้อมูลทั้งหมด"
						value={Math.round(data.summary.avg_income_all).toLocaleString('th-TH')}
						unit="บาท/เดือน"
						highlight={true}
					/>
					<StatCard
						label="รายได้มัธยฐานของข้อมูลทั้งหมด"
						value={Math.round(data.summary.median_income_all).toLocaleString('th-TH')}
						unit="บาท/เดือน"
					/>
				</div>
				<p class="summary-footnote">
					ตัวเลขสรุปนี้อ้างอิงจากข้อมูลที่ทำความสะอาดแล้วทั้งหมด
				</p>
			</section>

			<!-- Data Cleaning Summary -->
			<section class="cleaning-section">
				<div class="section-header">
					<h2>Data cleaning</h2>
					<p class="section-subtitle">การทำความสะอาดข้อมูลก่อนนำมาวิเคราะห์</p>
				</div>

				<div class="grid grid-cols-4">
					<StatCard label="ข้อมูลดิบ" value={data.cleaning_summary.metrics.raw_records} unit="รายการ" />
					<StatCard
						label="ข้อมูลหลังทำความสะอาด"
						value={data.cleaning_summary.metrics.clean_records}
						unit="รายการ"
						highlight={true}
					/>
					<StatCard
						label="ข้อมูลซ้ำที่ลบออก"
						value={data.cleaning_summary.metrics.duplicate_rows_removed}
						unit="รายการ"
					/>
					<StatCard
						label="แถวสำคัญที่ขาดหาย"
						value={data.cleaning_summary.metrics.missing_required_rows_removed}
						unit="รายการ"
					/>
				</div>

				<div class="cleaning-highlights">
					<div class="cleaning-highlight">
						<span class="cleaning-label">ข้อมูลรายได้ติดลบ</span>
						<strong>{data.cleaning_summary.metrics.negative_income_records.toLocaleString('th-TH')}</strong>
						<span class="cleaning-unit">รายการ</span>
					</div>
					<div class="cleaning-highlight">
						<span class="cleaning-label">ข้อมูลที่มีค่าเป็นศูนย์</span>
						<strong>{data.cleaning_summary.metrics.zero_income_records.toLocaleString('th-TH')}</strong>
						<span class="cleaning-unit">รายการ</span>
					</div>
				</div>

				<p class="cleaning-takeaway">{data.cleaning_summary.takeaway}</p>

				<div class="cleaning-steps">
					<h3>ขั้นตอนหลักที่ใช้</h3>
					<ul>
						{#each data.cleaning_summary.steps as step}
							<li>{step}</li>
						{/each}
					</ul>
				</div>
			</section>

			<!-- Question 1: Who earns what -->
			<section class="question-section">
				<div class="section-header">
					<h2>{data.q1_occupational_income.title}</h2>
					<p class="section-subtitle">{data.q1_occupational_income.description}</p>
				</div>
				<Q1Chart data={data.q1_occupational_income.data} />
				<div class="section-insight">
					<h3>สรุปประเด็นสำคัญ</h3>
					<ul>
						<li>
							<strong>รายได้สูงสุด:</strong>
							{data.q1_occupational_income.data[0].soc_eco_class2}
							({data.q1_occupational_income.data[0].soc_eco_class1})
							รับ
							<strong>{data.q1_occupational_income.data[0].avg_income.toLocaleString('th-TH')} บาท/เดือน</strong>
						</li>
						<li>
							<strong>รายได้ต่ำสุด:</strong>
							{data.q1_occupational_income.data[data.q1_occupational_income.data.length - 1].soc_eco_class2}
							({data.q1_occupational_income.data[data.q1_occupational_income.data.length - 1].soc_eco_class1})
							รับ
							<strong>
								{data.q1_occupational_income.data[data.q1_occupational_income.data.length - 1].avg_income.toLocaleString('th-TH')} บาท/เดือน
							</strong>
						</li>
						<li>
							<strong>ช่องว่างรายได้:</strong>
							ความแตกต่างระหว่างกลุ่มอาชีพสูงสุดและต่ำสุด คือ
							<strong>
								{(data.q1_occupational_income.data[0].avg_income - data.q1_occupational_income.data[data.q1_occupational_income.data.length - 1].avg_income).toLocaleString('th-TH')} บาท/เดือน
							</strong>
						</li>
						<li>
							ลูกจ้างมีความหลากหลายในระดับรายได้มากที่สุด ตั้งแต่ผู้บริหารระดับสูงไปจนถึงคนงานเกษตร
						</li>
					</ul>
				</div>
			</section>

			<!-- Question 2: Where is inequality -->
			<section class="question-section">
				<div class="section-header">
					<h2>{data.q2_geographic_inequality.title}</h2>
					<p class="section-subtitle">
						{data.q2_geographic_inequality.description} พร้อมมุมมองแผนที่เพื่อเห็นรูปแบบเชิงพื้นที่
					</p>
				</div>
				<div class="q2-explainer">
					<strong>ทำไมใช้ CV:</strong>
					<span>
						CV = ส่วนเบี่ยงเบนมาตรฐาน ÷ ค่าเฉลี่ย จึงเหมาะสำหรับเปรียบเทียบจังหวัดที่มีระดับรายได้
						ต่างกัน เพราะวัดความกระจายตัวในรูปสัดส่วน ทำให้เห็นความเหลื่อมล้ำภายในจังหวัดได้อย่าง
						ยุติธรรมกว่าเมื่อเทียบกับการดูตัวเลขดิบเพียงอย่างเดียว
					</span>
				</div>
				<Q2Map data={data.q2_geographic_inequality.data} />
				<Q2Chart data={data.q2_geographic_inequality.data} />
				<div class="section-insight">
					<h3>สรุปประเด็นสำคัญ</h3>
					<ul>
						<li>
							<strong>จังหวัดที่มีความเหลื่อมล้ำมากที่สุด:</strong>
							{data.q2_geographic_inequality.data[0].province}
							(ความแตกต่าง p90-p10:
							<strong>{data.q2_geographic_inequality.data[0].p90_p10_gap.toLocaleString('th-TH')} บาท</strong>)
						</li>
						<li>
							ความเหลื่อมล้ำภูมิศาสตร์นั้นมีผลต่อโอกาสในการหารายได้อย่างมาก
						</li>
						<li>
							จังหวัดที่มีรายได้เฉลี่ยสูง เช่น กรุงเทพ อาจยังคงมีช่องว่างรายได้ภายในจังหวัดอยู่
						</li>
						<li>
							นโยบายต้องพิจารณาทั้ง "การเพิ่มรายได้ทั้งหมด" และ "การลดความเหลื่อมล้ำ" พร้อมกัน
						</li>
					</ul>
				</div>
			</section>

			<!-- Question 3: Income Structure -->
			<section class="question-section">
				<div class="section-header">
					<h2>{data.q3_income_structure.title}</h2>
					<p class="section-subtitle">{data.q3_income_structure.description}</p>
				</div>
				<Q3Chart data={data.q3_income_structure.data} />
				<div class="section-insight">
					<h3>สรุปประเด็นสำคัญ</h3>
					<ul>
						<li>
							<strong>ลูกจ้าง:</strong> พึ่งพา "ค่าจ้างและเงินเดือน" เป็นหลัก (85-92%)
							- โครงสร้างรายได้เสถียร แต่มีความเสี่ยงจากการว่างงาน
						</li>
						<li>
							<strong>เจ้าของกิจการ:</strong> พึ่งพา "กำไรสุทธิจากธุรกิจ" เป็นหลัก (88-90%)
							- รายได้ขึ้นอยู่กับประสิทธิภาพของธุรกิจ
						</li>
						<li>
							<strong>เกษตรกร:</strong> พึ่งพา "กำไรจากการเกษตร" เป็นหลัก (74%)
							- มีหลายแหล่งรายได้เสริม แต่ยังคงเสี่ยงต่อภัยธรรมชาติและราคาสินค้า
						</li>
						<li>
							ไม่มีกลุ่มใดพึ่งพา "รายได้จากทรัพย์สิน" เป็นหลัก - แสดงถึงการกระจุกตัวของทรัพย์สินในประเทศ
						</li>
					</ul>
				</div>
			</section>

			<!-- Policy Recommendations -->
			<section class="recommendations-section">
				<h2>ข้อเสนอแนะการกำหนดนโยบาย</h2>
				<div class="grid grid-cols-3">
					<div class="card">
						<h3>เพิ่มทักษะแรงงาน</h3>
						<p>
							โปรแกรมฝึกอบรมสำหรับกลุ่มลูกจ้างระดับต่ำ เพื่อยกระดับให้เป็นผู้จัดการและนักวิชาการ
						</p>
					</div>
					<div class="card">
						<h3>สนับสนุนเกษตรกร</h3>
						<p>
							นโยบายเกษตรยั่งยืนและการประกันราคา เพื่อลดความเสี่ยงและเพิ่มรายได้อย่างต่อเนื่อง
						</p>
					</div>
					<div class="card">
						<h3>พัฒนาเศรษฐกิจท้องถิ่น</h3>
						<p>
							สนับสนุนเอสเอ็มอีและผู้ประกอบการขนาดเล็ก เพื่อลดความเหลื่อมล้ำทางภูมิศาสตร์
						</p>
					</div>
				</div>
			</section>
		{/if}
	</main>

	<footer class="footer">
		<div class="container">
			<p>
				ข้อมูลจาก
				<a href="https://data.go.th/dataset/os_08_00007" target="_blank" rel="noopener">
					Open Government Data of Thailand
				</a>
				| ปีงบประมาณ 2566
			</p>
			<p>
			    Visualize Mini Hackathon Data 600345
			</p>
		</div>
	</footer>
</div>

<style>
	:global(body) {
		background-color: var(--color-bg);
	}

	.page-wrapper {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
	}

	.header {
		background: var(--color-white);
		border-bottom: 1px solid var(--color-border);
		padding: var(--spacing-2xl) 0;
		margin-bottom: var(--spacing-2xl);
	}

	.header-content {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
	}

	.header h1 {
		margin-bottom: var(--spacing-sm);
	}

	.subtitle {
		color: var(--color-text-light);
		font-size: 0.95rem;
	}

	.header-meta a {
		padding: var(--spacing-sm) var(--spacing-md);
		background-color: var(--color-primary);
		color: var(--color-white);
		border-radius: 0.375rem;
		font-size: 0.875rem;
		transition: var(--transition);
	}

	.header-meta a:hover {
		background-color: #2980b9;
		text-decoration: none;
	}

	main {
		flex: 1;
		padding-bottom: var(--spacing-2xl);
	}

	.summary-section {
		margin-bottom: var(--spacing-2xl);
	}

	.summary-section h2 {
		margin-bottom: var(--spacing-lg);
	}

	.summary-footnote {
		margin-top: var(--spacing-md);
		color: var(--color-text-light);
		font-size: 0.875rem;
	}

	.cleaning-section {
		margin-bottom: var(--spacing-2xl);
		background: var(--color-white);
		padding: var(--spacing-2xl);
		border-radius: 0.5rem;
		border: 1px solid var(--color-border);
	}

	.cleaning-highlights {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: var(--spacing-md);
		margin-top: var(--spacing-lg);
	}

	.cleaning-highlight {
		display: flex;
		align-items: baseline;
		gap: var(--spacing-sm);
		padding: var(--spacing-md);
		border-radius: 0.375rem;
		background-color: rgba(52, 152, 219, 0.04);
		border: 1px solid var(--color-border);
	}

	.cleaning-label {
		font-size: 0.9rem;
		color: var(--color-text-light);
	}

	.cleaning-unit {
		font-size: 0.85rem;
		color: var(--color-text-light);
	}

	.cleaning-takeaway {
		margin-top: var(--spacing-lg);
		margin-bottom: 0;
		color: var(--color-text);
	}

	.cleaning-steps {
		padding: var(--spacing-md);
		margin-top: var(--spacing-lg);
	}

	.cleaning-steps h3 {
		margin-bottom: var(--spacing-sm);
		font-size: 1rem;
	}

	.cleaning-steps ul {
		margin: 0;
		padding-left: 1.25rem;
		color: var(--color-text);
	}

	.cleaning-steps li {
		margin-bottom: var(--spacing-sm);
		line-height: 1.7;
	}

	.question-section {
		margin-bottom: var(--spacing-2xl);
		background: var(--color-white);
		padding: var(--spacing-2xl);
		border-radius: 0.5rem;
		border: 1px solid var(--color-border);
	}

	.section-header {
		margin-bottom: var(--spacing-xl);
	}

	.section-header h2 {
		margin-bottom: var(--spacing-sm);
	}

	.section-subtitle {
		color: var(--color-text-light);
		font-size: 1rem;
	}

	.q2-explainer {
		margin-bottom: var(--spacing-lg);
		padding: var(--spacing-md) var(--spacing-lg);
		background: rgba(52, 152, 219, 0.06);
		border-left: 4px solid var(--color-primary);
		border-radius: 0.375rem;
		color: var(--color-text);
		line-height: 1.7;
	}

	.q2-explainer strong {
		display: inline-block;
		margin-right: var(--spacing-xs);
	}

	.section-insight {
		margin-top: var(--spacing-xl);
		padding-top: var(--spacing-xl);
		border-top: 1px solid var(--color-border);
	}

	.section-insight h3 {
		margin-bottom: var(--spacing-md);
		font-size: 1.1rem;
	}

	.section-insight ul {
		list-style: none;
		padding: 0;
	}

	.section-insight li {
		margin-bottom: var(--spacing-md);
		color: var(--color-text);
		line-height: 1.8;
	}

	.section-insight li strong {
		color: var(--color-primary);
		font-weight: 600;
	}

	.recommendations-section {
		background: var(--color-white);
		padding: var(--spacing-2xl);
		border-radius: 0.5rem;
		border: 1px solid var(--color-border);
		margin-bottom: var(--spacing-2xl);
	}

	.recommendations-section h2 {
		margin-bottom: var(--spacing-xl);
	}

	.recommendations-section .card {
		padding: var(--spacing-lg);
	}

	.recommendations-section .card h3 {
		color: var(--color-primary);
		margin-bottom: var(--spacing-md);
	}

	.recommendations-section .card p {
		margin-bottom: 0;
		font-size: 0.95rem;
	}

	.footer {
		background: var(--color-white);
		border-top: 1px solid var(--color-border);
		padding: var(--spacing-xl) 0;
		margin-top: auto;
	}

	.footer p {
		margin-bottom: 0;
		color: var(--color-text-light);
		font-size: 0.875rem;
	}

	.footer a {
		color: var(--color-primary);
	}

	.loading,
	.error {
		text-align: center;
		padding: var(--spacing-2xl);
		background: var(--color-white);
		border-radius: 0.5rem;
		border: 1px solid var(--color-border);
	}

	.error {
		background-color: rgba(231, 76, 60, 0.05);
		border-color: rgba(231, 76, 60, 0.2);
	}

	.error p {
		color: var(--color-accent);
	}

	@media (max-width: 768px) {
		.header-content {
			flex-direction: column;
			gap: var(--spacing-lg);
		}

		.header {
			padding: var(--spacing-lg) 0;
		}

		.question-section {
			padding: var(--spacing-lg);
		}

		.recommendations-section {
			padding: var(--spacing-lg);
		}
	}
</style>
