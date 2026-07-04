# Data Analyst Job Market Analysis

This project involves analyzing a job posting dataset for the **Data Analyst** role. My goal was to identify key market trends to help make informed career development decisions.

## 1. High-demand skills with above-average salaries (top skills to learn)
*Analysis that defines the most valuable skill for career growth by filtering
for technologies that exceed market averages in both job demand and compensation.*

* **Business Goal:** Identify the "golden mean" of technologies that combine high job demand with above-average compensation. This helps prioritize skills that offer the highest return on investment.
* **Technology:** Uses CTEs and subqueries to dynamically filter for skills that outperform market averages in both job posting frequency and salary.
* **Key Insight:** This analysis isolates top-tier professional requirements from general market noise, providing a clear roadmap for data-driven career growth.

```sql
 WITH skill_demand AS (
    SELECT
        sd.skills,
        AVG(jpf.salary_year_avg) AS average_salary,
        count(jpf.job_id) AS job_offers_count
    FROM
        skills_job_dim AS sjd
    LEFT JOIN skills_dim AS sd ON sjd.skill_id = sd.skill_id
    LEFT JOIN job_postings_fact AS jpf ON sjd.job_id = jpf.job_id 
    WHERE
        jpf.salary_year_avg IS NOT NULL
        AND jpf.job_title LIKE '%Data Analyst%'
    GROUP BY sd.skills
)
SELECT
    skills,
    average_salary,
    job_offers_count
FROM
    skill_demand
WHERE
    job_offers_count  > (SELECT AVG(job_offers_count) FROM skill_demand)
    AND average_salary > (SELECT AVG(average_salary) FROM skill_demand)
ORDER BY job_offers_count DESC, average_salary DESC;
```
## 2. Market Gap Analysis (Identifying Undervalued Opportunities)
*Advanced analysis that uncovers high-demand skills currently associated with lower-than-average salaries.*

* **Business Goal:** Identify skills with high market demand (at least 50 job postings) that are currently undervalued in terms of salary. This helps uncover "hidden gems" or potential entry-level advantages in the job market.
* **Technology:** Utilizes advanced CTEs to establish a market salary benchmark, combined with complex filtering to isolate specific skills against that benchmark.
* **Key Insight:** This analysis highlights skills that offer a high volume of opportunities, allowing professionals to strategically enter high-demand fields by identifying competitive, yet accessible, market niches.

```sql
WITH offers_count AS (
    SELECT 
        AVG(salary_year_avg) AS avg_offer_salary
    FROM
        job_postings_fact
    WHERE
        job_title_short = 'Data Analyst'
),
skill_stats AS (
    SELECT 
        skills_dim.skills,
        COUNT(job_postings_fact.job_id) AS skill_count,
        AVG(salary_year_avg) AS avg_skill_salary
    FROM
        skills_job_dim
    LEFT JOIN job_postings_fact ON skills_job_dim.job_id = job_postings_fact.job_id
    LEFT JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_postings_fact.job_title_short = 'Data Analyst'
    GROUP BY skills_dim.skills
)
SELECT 
    *
FROM 
    skill_stats
WHERE 
    avg_skill_salary < (SELECT avg_offer_salary FROM offers_count) 
    AND skill_count > 50
ORDER BY 
    avg_skill_salary DESC;