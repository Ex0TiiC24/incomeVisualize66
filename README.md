# Thai Household Income Dashboard

## Project summary

This project is a SvelteKit dashboard that visualizes Thailand’s average household income dataset from the National Statistical Office (NSO), published via Open Government Data of Thailand.

The dataset contains 7,700 records across 77 provinces and 10 occupation groups for year 2566 (2023). The project turns the raw CSV into a dashboard-friendly JSON file and presents the results through three policy questions:

### Q1: Who earns what?
Compares average monthly household income by occupational group and shows the gap between the highest- and lowest-earning groups.

### Q2: Where is inequality?
Explores geographic income inequality using average income, median income, P10/P90 spread, and coefficient of variation by province.

### Q3: What is the income structure?
Breaks down income sources by occupation group to show how heavily different households depend on wages, business income, agriculture, transfers, or property income.

## Current key findings

- The national average household income is about 25,327 THB per month.
- The national median is about 22,364 THB per month, which is lower than the mean and suggests a right-skewed income distribution.
- The highest-income occupational group is `ผู้จัดการนักวิชาการและผู้ปฏิบัติงานวิชาชีพ`.
- The lowest-income occupational group is `คนงานเกษตรป่าไม้และประมง`.
- Provincial inequality is strongest in provinces such as Narathiwat when measured by CV and P90-P10 gap.

## Suggested improvements

1. Keep both mean and median in the summary to reduce the risk of over-interpreting outliers.
2. Flag negative or zero income records in the UI so readers know the dataset contains edge cases.
3. Let the inequality chart stay sorted by inequality even when the table sort changes.
4. Add more explicit labels or tooltips for chart encodings to make the visuals easier to read on mobile.
5. Consider adding province or occupation filters for deeper exploration.
6. If the dashboard is used for policy discussion, add a short methodology note explaining how each metric is calculated.

## Data notes

- Source: NSO via Open Government Data of Thailand
- Unit: THB per month
- Time period: 2566
- The processing script keeps the published values intact and only cleans formatting, types, and duplicates.
