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
    LEFT JOIN job_postings_fact AS jpf ON sjd.job_id = jpf.job_id /*
    On big datasets left joins and then filtering with a WHERE clause
    is faster than inner joins.
    */
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
