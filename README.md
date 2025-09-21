# Employee-Engagement-Survey-Analysis

## Project Overview
This project involved a comprehensive analysis of an employee engagement survey for a large organization with 14,000 employees across 21 departments. Unlike a simple analysis of pre-aggregated scores, this project began with **raw, response-level data**, requiring extensive data cleaning, transformation, and aggregation. The final output is an interactive **Power BI dashboard** that provides leadership with clear, actionable insights into employee sentiment across various dimensions, including role levels and specific questions.

The analysis revealed a company-wide average sentiment score of **3.07/5**, indicating neutral but lukewarm engagement, with a significant opportunity to convert a large disengaged segment.

<img width="892" height="503" alt="image" src="https://github.com/user-attachments/assets/a5043825-b7ea-4ab4-9e32-4f52a3ecea50" />

## Data Source and Structure
The analysis began with a raw dataset extracted from the survey platform. The table structure was a **long-format** table where each row represented a single answer to a single survey question from one employee.

### Detailed Column Description
| Column Name | Data Type & Description |
| :--- | :--- |
| **`Response_ID`** | **Integer.** A unique identifier for each individual survey submission. Each employee has one unique `Response_ID` tied to their answers to all 11 questions. |
| **`Status`** | **Text.** The completion state of the survey (e.g., `'Complete'`, `'Incomplete'`). The analysis filtered to only `'Complete'` submissions. |
| **`Department`** | **Text.** The business unit the employee belongs to (e.g., "Human Resources," "IT"). There were 21 unique departments. |
| **`Director`** | **Boolean (1/0).** A flag indicating if the respondent's role was a Director (`1` = yes). |
| **`Manager`** | **Boolean (1/0).** A flag indicating if the respondent's role was a Manager (`1` = yes). |
| **`Supervisor`** | **Boolean (1/0).** A flag indicating if the respondent's role was a Supervisor (`1` = yes). |
| **`Staff`** | **Boolean (1/0).** A flag indicating if the respondent was an individual contributor (`1` = yes). |
| **`Question`** | **Text.** The full text of the survey question the respondent answered. |
| **`Response`** | **SmallInteger.** The numerical value of the answer on a standard **Likert scale** (e.g., 1=Strongly Disagree, 5=Strongly Agree). |
| **`Response_Text`** | **nVARCHARMAX.** Optional qualitative written feedback providing the "why" behind the numerical score. |

**Important Note on Structure:** Each employee (`Response_ID`) has **11 rows** in the raw data—one for each survey question. The role and department columns are repeated for each row. This long format is optimal for storage but requires transformation for analysis.

## Tools & Technologies Used
*   **Microsoft SQL Server:** Hosted the raw data and was used for the initial heavy lifting of data transformation.
*   **SQL:** The primary tool for cleaning, structuring, and aggregating the complex raw data into an analysis-ready dataset.
*   **Microsoft Power BI:** Connected to the processed SQL view to create an interactive dashboard for visualization and insight discovery.
*   **DAX:** Used within Power BI to create calculated measures and metrics.

## Project Methodology

### 1. Data Wrangling and cleaning with SQL
The most critical phase was transforming the raw response-level data. This was done entirely in SQL Server to ensure performance and accuracy before loading into Power BI.

**The Key SQL Transformation Query:**
I created a SQL view that served as the clean, foundation layer for the entire project. This query:
*   **Filtered** out incomplete and invalid responses.
*   **Consolidated** multiple binary role columns into a single, clear `Role` dimension using a `CASE` statement.
*   **Standardized** data types to prevent loading issues in Power BI.

```sql
SELECT
    Response_ID,
    Department,
    -- Consolidates role flags into a single category
    CASE 
        WHEN Director = 1 THEN 'Director'
        WHEN Manager = 1 THEN 'Manager'
        WHEN Supervisor = 1 THEN 'Supervisor'
        WHEN Staff = 1 THEN 'Staff'
        ELSE 'Unknown'
    END AS Role,
    -- Ensures clean text data type for questions
    CAST(Question AS NVARCHAR(MAX)) AS Question,
    Response,
    Response_Text
FROM [dbo].[Employee Survey]
-- Filters for only valid, completed surveys
WHERE Status = 'Complete'
AND Response IS NOT NULL
AND Response != 0
### 2. Power BI Dashboard Development
I connected Power BI directly to the clean SQL view. The data was already structured and required minimal additional transformation within Power Query.

*   **Data Modeling:** The clean data from SQL was imported into Power BI. A simple star schema data model was established with the main fact table (survey responses) connected to dimension tables for `Date` and a manually created `Score Bucket` table for the response distribution. The `Department` and `Role` columns were treated as dimensions directly within the fact table.
*   **DAX Measures:** Created key measures to drive the dashboard's analytics:
    ```dax
    Average Response = AVERAGE('Survey Data'[Response])
    Total Responses = COUNT('Survey Data'[Response_ID])
    Agree or Strongly Agree = CALCULATE(
        [Total Responses],
        'Survey Data'[Response] IN {4, 5}
    )
    % Favorable = DIVIDE([Agree or Strongly Agree], [Total Responses])
    ```
*   **Visualization & Dashboard Assembly:** Built an interactive and visually coherent dashboard with the following components:
    *   **Key Metric Cards:** Displaying Total Staff (14K), Departments (21), Questions (11), and the overall Avg. Response (3.07).
    *   **Bar Charts:**
        *   *Average Score by Question:* To visually compare sentiment across all 11 survey questions and instantly identify strengths and weaknesses.
        *   *Average Score by Department:* To rank and compare performance across all 21 departments, highlighting top performers and those needing support.
    *   **Donut Chart:** Showing the *Distribution of All Responses* across the 5-point Likert scale, highlighting the large neutral segment (40.41%).
    *   **Horizontal Bar Chart:** Showing *Response by Department* to illustrate response volume and ensure data representation validity.
    *   **Interactivity:** Slicers and cross-filtering were enabled so that selecting a department or question would update all other visuals on the page to provide context-specific insights.

## Key Insights & Findings

### Answers to Project Questions

**Q1: What is the overall state of employee sentiment?**
**A1:** The overall sentiment is **neutral**, with a company-wide average score of **3.07/5**. A deep dive into the distribution revealed the core issue: **over 40% of all responses were neutral (score=3)**. This indicates a vast population of disengaged or apathetic employees, representing the largest opportunity for improving overall engagement.

**Q2: Which areas (questions) are the biggest strengths and weaknesses?**
**A2:** The analysis of average scores by question revealed a clear spread:
*   **Weaknesses:** The questions with the lowest average scores (closest to **2.6**) were identified as primary areas of dissatisfaction, requiring immediate root cause analysis and targeted action plans.
*   **Strengths:** The questions with the highest averages (closest to **3.5**) were identified as relative strengths to be celebrated and used as a model for other areas.

**Q3: How does sentiment vary across departments and roles?**
**A3:** Sentiment varied significantly across the organization:
*   **By Department:** Scores ranged from **2.6 to 3.48**, highlighting that departmental culture and local leadership are critical drivers of the employee experience. This variance helps prioritize support for underperforming teams.
*   **By Role (New Insight):** Because the raw data included role information, the analysis could be extended to show sentiment trends across different job levels (e.g., Directors vs. Staff), providing another layer of actionable insight for HR.

## Recommendations
Based on the analysis, here are data-driven recommendations:

1.  **Department-Specific Action Plans:** Leadership of departments scoring below the company average (3.07) must create targeted action plans. HR should facilitate best practice sharing from top-performing departments.
2.  **Engage the Neutral Majority:** Launch initiatives aimed at the 40% of neutral respondents. This includes manager training on recognition, clear career pathing programs, and improved communication to foster emotional investment.
3.  **Deep-Dive on Low Scores:** Conduct focused listening sessions (e.g., roundtables) on the specific low-scoring questions to understand the root cause of dissatisfaction.
4.  **Leverage Role-Based Insights:** Use the role-level data to tailor communication and development programs. For example, if "Staff" sentiment is low, investigate empowerment and voice mechanisms.

## Conclusion
This project exemplifies a complete data analysis pipeline. It started with complex, raw response-level data and used **advanced SQL wrangling** to clean, filter, and structure it into a meaningful dataset. This robust foundation allowed for the efficient creation of an insightful **Power BI dashboard**.

The final visualization moves beyond simple averages to provide a multi-dimensional view of organizational health, highlighting critical engagement opportunities within specific departments, role levels, and survey topics. The recommendations provide a clear, actionable roadmap for leadership to improve employee experience and drive business performance.
