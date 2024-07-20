# Hospital Waiting Time Analysis

## Tools Used
- **SQL:** Foundation for exploring and cleaning the database.
- **Jupyter Notebook:** Implemented Python code for data visualization & prediction.
- **Microsoft SQL Server Management Studio:** Familiar environment for SQL queries.
- **Power BI:** Used for waiting time visualizations.
- **Git and GitHub:** Version control and collaboration for scripts and analysis.

## Data Exploration

### Total Number of Patients and Their Average Wait Time
This query calculates the total number of patients and their average wait time:
```sql
SELECT SUM(Number_In) AS NumberOfPatients, AVG(Patient_WaitTime) AS AverageWaitTime
FROM WaitingTime
GROUP BY Number_In
ORDER BY AverageWaitTime ASC;
