# Thai Household Income Dashboard - Data Cleaning / Data visualization project
## Overview

This is a complete **SvelteKit-based web visualization dashboard** designed to analyze Thai household income data from the National Statistical Office (NSO). The dashboard answers 3 critical policy questions through interactive data visualizations.

**Data Source:** https://data.go.th/dataset/os_08_00007  
**Year:** 2566 (2023)  
**Framework:** SvelteKit + Vite  
**Design:** Clean white aesthetic with responsive layout

---

## Three Core Questions Answered

### Q1: Who earns what? (ใครได้รับค่าตอบแทนแค่ไหน?)
- **Purpose:** Identify income gaps between occupational groups
- **Visualization:** Horizontal bar chart + detailed table
- **Key Insight:** Shows wage disparity from executives (48,293 THB) to agricultural workers (14,983 THB)
- **Use Case:** Policy makers can identify skill gaps requiring training programs

### Q2: Where is inequality? (ความเหลื่อมล้ำอยู่ที่ไหน?)
- **Purpose:** Identify geographic income disparities
- **Visualization:** Scatter plot (income vs inequality) + top 15 bar chart + detailed table
- **Key Metric:** Coefficient of Variation (CV) and P90-P10 gap
- **Use Case:** Target local economic development efforts to provinces with highest inequality

### Q3: What is the income structure? (รายได้มาจากไหน และใครพึ่งพาอะไร?)
- **Purpose:** Understand income source composition and economic vulnerability
- **Visualization:** Donut chart + stacked bar + breakdown table + risk analysis
- **Key Insight:** Shows employees rely 85-92% on salary, farmers only 74% on agriculture income
- **Use Case:** Design targeted support programs based on income dependencies

---

