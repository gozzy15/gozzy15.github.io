---
layout: default
title: Demographic Burden of Diseases
permalink: /projects/disease-demographics/
---

# Demographic Burden of Diseases: Age & Gender Analysis Using SQL

Problem: Disease burdens are not evenly distributed, children, the elderly, and specific genders often experience disproportionate impact.

Solution: Used MySQL 8.0 to analyze global disease demographics and identify which age and gender groups are most affected by major diseases. Results were visualized in Power BI.

## Approach

Data Cleaning: Removed duplicates, fixed missing values, normalized age-group percentages

Data Preparation: Created a clean, analysis-ready SQL view (2010–2020) with normalized age and gender distributions

Analysis: Ran demographic SQL queries across Malaria, Diabetes, HIV/AIDS, Tuberculosis, and COVID-19

Visualization: Exported aggregated results into Power BI dashboards

## Data Cleaning Steps

Duplicate Removal: Ensured unique rows using (country_id, disease_id, year, gender)

Missing Values: Replaced null percentages with disease-specific averages

Normalization: Scaled age-group percentages so each row sums to ~100%

Example SQL:

DELETE t1 
FROM disease_statistics t1
JOIN disease_statistics t2
  ON t1.stat_id > t2.stat_id
 AND t1.country_id = t2.country_id
 AND t1.disease_id = t2.disease_id
 AND t1.year = t2.year
 AND t1.gender = t2.gender;

UPDATE disease_statistics
SET total = ages_0_18_pct + ages_19_35_pct + ages_36_60_pct + ages_61_plus_pct,
    ages_0_18_pct = (ages_0_18_pct / total) * 100,
    ages_19_35_pct = (ages_19_35_pct / total) * 100,
    ages_36_60_pct = (ages_36_60_pct / total) * 100,
    ages_61_plus_pct = (ages_61_plus_pct / total) * 100;

## Data Preparation

Created a normalized SQL view for ages and gender between 2010–2020:

CREATE OR REPLACE VIEW vw_disease_age_normalized_2010_2020 AS
SELECT 
  ds.stat_id, d.disease_name, ds.gender, cys.country_id, cys.year, ds.pop_affected,
  COALESCE(ds.ages_0_18_pct,0) AS norm_0_18_pct,
  COALESCE(ds.ages_19_35_pct,0) AS norm_19_35_pct,
  COALESCE(ds.ages_36_60_pct,0) AS norm_36_60_pct,
  COALESCE(ds.ages_61_plus_pct,0) AS norm_61_plus_pct
FROM disease_statistics ds
JOIN country_year_stats cys ON ds.cy_id = cys.cy_id
JOIN diseases d USING(disease_id)
WHERE cys.year BETWEEN 2010 AND 2020;

## Insights

Malaria: Over 50% of cases occur in children (0–18)

Diabetes: Dominant in adults 36–60 and elderly 61+

HIV/AIDS: Concentrated in young adults (19–35), especially women

Tuberculosis: More common in men, largely working-age groups

COVID-19: Highest burden in populations aged 61+

Weighted analysis confirmed these demographic skews more strongly when considering population sizes.

## Results

Identified diseases disproportionately affecting children (Malaria, Respiratory Infections)

Confirmed chronic diseases (e.g., Diabetes, Cancer) impact older adults

Highlighted gender imbalances in HIV/AIDS and Tuberculosis prevalence

Showed gradual improvement in Malaria child cases post-2015

Produced Power BI visuals showcasing demographic disease burdens globally

## Code Highlights
-- Age distribution by disease and gender
SELECT d.disease_name, v.gender,
       ROUND(AVG(v.norm_0_18_pct),2) AS avg_0_18,
       ROUND(AVG(v.norm_19_35_pct),2) AS avg_19_35,
       ROUND(AVG(v.norm_36_60_pct),2) AS avg_36_60,
       ROUND(AVG(v.norm_61_plus_pct),2) AS avg_61_plus
FROM vw_disease_age_normalized_2010_2020 v
JOIN diseases d USING(disease_id)
WHERE d.disease_name IN ('Malaria','Diabetes','HIV/AIDS','Tuberculosis','COVID-19')
GROUP BY d.disease_name, v.gender;

## Disclaimer

The dataset used in this project is synthetic and created solely for educational and portfolio purposes. It does not represent real-world health statistics.
