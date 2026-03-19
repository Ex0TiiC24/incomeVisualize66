<script>
	import { onMount } from 'svelte';
	import { extent, geoMercator, geoPath, interpolateYlOrRd, scaleSequential } from 'd3';

	export let data = [];

	const GEOJSON_URL =
		'https://raw.githubusercontent.com/chingchai/OpenGISData-Thailand/5b332fc913afc609c49c25eff5f446983323163b/provinces.geojson';

	const width = 800;
	const height = 920;

	let geoData = null;
	let loading = true;
	let error = null;
	let projection = geoMercator();
	let path = geoPath(projection);

	$: provinceByName = new Map(data.map((item) => [item.province, item]));
	$: values = data.map((item) => item.cv).filter((value) => Number.isFinite(value));
	$: domain = values.length > 0 ? extent(values) : [0, 1];
	$: colorScale = scaleSequential(interpolateYlOrRd).domain(domain);
	$: if (geoData) {
		projection = geoMercator().fitSize([width, height], geoData);
		path = geoPath(projection);
	}
	$: highestProvince = data[0] ?? null;
	$: lowestProvince = data.length > 0 ? data[data.length - 1] : null;

	onMount(async () => {
		try {
			const response = await fetch(GEOJSON_URL);
			if (!response.ok) throw new Error('Failed to load province boundaries');
			geoData = await response.json();
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	});

	function getProvinceRow(feature) {
		return provinceByName.get(feature.properties.pro_th);
	}

	function getFillColor(feature) {
		const row = getProvinceRow(feature);
		return row ? colorScale(row.cv) : '#e5e7eb';
	}

	function getTooltip(feature) {
		const row = getProvinceRow(feature);
		const provinceName = feature.properties.pro_th;

		if (!row) {
			return `${provinceName}: ไม่มีข้อมูล`;
		}

		return `${provinceName}: CV ${row.cv.toFixed(2)}, ช่องว่าง P90-P10 ${Math.round(
			row.p90_p10_gap
		).toLocaleString('th-TH')} บาท`;
	}
</script>

<section class="map-card">
	<div class="map-header">
		<div>
			<h3>แผนที่ความเหลื่อมล้ำรายจังหวัด</h3>
			<p>สีเข้มหมายถึงค่าสัมประสิทธิ์ความแปรปรวน (CV) สูงกว่า</p>
		</div>
		<div class="map-note">
			<span>ใช้ค่า CV หลังตัด outlier แบบ IQR</span>
		</div>
	</div>

	{#if loading}
		<div class="map-state">กำลังโหลดข้อมูลแผนที่...</div>
	{:else if error}
		<div class="map-state error">
			ไม่สามารถโหลดแผนที่จังหวัดได้: {error}
		</div>
	{:else if geoData}
		<div class="map-summary">
			<div class="summary-pill">
				<span>จังหวัด CV สูงสุด</span>
				<strong>{highestProvince?.province}</strong>
			</div>
			<div class="summary-pill">
				<span>จังหวัด CV ต่ำสุด</span>
				<strong>{lowestProvince?.province}</strong>
			</div>
			<div class="summary-pill">
				<span>จำนวนจังหวัดในแผนที่</span>
				<strong>{data.length.toLocaleString('th-TH')}</strong>
			</div>
		</div>

		<div class="map-frame">
			<svg
				viewBox={`0 0 ${width} ${height}`}
				preserveAspectRatio="xMidYMid meet"
				role="img"
				aria-label="แผนที่ความเหลื่อมล้ำรายจังหวัดของประเทศไทย"
			>
				{#each geoData.features as feature}
					<path
						d={path(feature)}
						fill={getFillColor(feature)}
						stroke="#ffffff"
						stroke-width="0.8"
						opacity={getProvinceRow(feature) ? 1 : 0.25}
					>
						<title>{getTooltip(feature)}</title>
					</path>
				{/each}
			</svg>
		</div>

		<div class="legend">
			<span>ต่ำ</span>
			<div class="gradient"></div>
			<span>สูง</span>
		</div>
	{/if}

	<div class="map-footnote">
		<p>
			แผนที่นี้แสดงจังหวัดตามชื่อภาษาไทยในชุดข้อมูล และใช้ค่าสัมประสิทธิ์ CV เพื่อเน้นพื้นที่ที่มี
			ความเหลื่อมล้ำสูง
		</p>
	</div>
</section>

<style>
	.map-card {
		background: var(--color-white);
		border: 1px solid var(--color-border);
		border-radius: 0.5rem;
		padding: var(--spacing-2xl);
		margin-bottom: var(--spacing-2xl);
	}

	.map-header {
		display: flex;
		justify-content: space-between;
		gap: var(--spacing-lg);
		align-items: flex-start;
		margin-bottom: var(--spacing-lg);
	}

	.map-header h3 {
		margin-bottom: var(--spacing-xs);
		font-size: 1.15rem;
	}

	.map-header p {
		margin-bottom: 0;
		color: var(--color-text-light);
	}

	.map-note {
		padding: var(--spacing-sm) var(--spacing-md);
		border-radius: 999px;
		background: rgba(52, 152, 219, 0.08);
		color: var(--color-primary);
		font-size: 0.85rem;
		font-weight: 600;
		white-space: nowrap;
	}

	.map-state {
		padding: var(--spacing-xl);
		text-align: center;
		color: var(--color-text-light);
		background: var(--color-bg);
		border-radius: 0.5rem;
		border: 1px dashed var(--color-border);
	}

	.map-state.error {
		color: var(--color-accent);
	}

	.map-summary {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: var(--spacing-md);
		margin-bottom: var(--spacing-lg);
	}

	.summary-pill {
		padding: var(--spacing-md);
		border: 1px solid var(--color-border);
		border-radius: 0.5rem;
		background: rgba(52, 152, 219, 0.03);
	}

	.summary-pill span {
		display: block;
		margin-bottom: var(--spacing-xs);
		font-size: 0.85rem;
		color: var(--color-text-light);
	}

	.summary-pill strong {
		font-size: 1rem;
		color: var(--color-text);
	}

	.map-frame {
		border: 1px solid var(--color-border);
		border-radius: 0.5rem;
		overflow: hidden;
		background: linear-gradient(180deg, #ffffff 0%, #f8fbff 100%);
	}

	svg {
		display: block;
		width: 100%;
		height: auto;
	}

	.legend {
		display: flex;
		align-items: center;
		gap: var(--spacing-md);
		margin-top: var(--spacing-md);
		color: var(--color-text-light);
		font-size: 0.85rem;
	}

	.gradient {
		flex: 1;
		height: 12px;
		border-radius: 999px;
		background: linear-gradient(90deg, #fff7bc 0%, #fec44f 45%, #f03b20 100%);
		border: 1px solid rgba(0, 0, 0, 0.05);
	}

	.map-footnote {
		margin-top: var(--spacing-md);
	}

	.map-footnote p {
		margin-bottom: 0;
		color: var(--color-text-light);
		font-size: 0.85rem;
	}

	@media (max-width: 768px) {
		.map-card {
			padding: var(--spacing-lg);
		}

		.map-header {
			flex-direction: column;
		}

		.map-summary {
			grid-template-columns: 1fr;
		}
	}
</style>
