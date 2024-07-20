# Waiting-time-of-patients-analysis
# Hospital Waiting Time Analysis

## Tools Used
- **SQL**: Foundation for exploring and cleaning the database.
- **Jupyter Notebook**: Implemented Python code for data visualization & prediction.
- **Microsoft SQL Server Management Studio**: Familiar environment for SQL queries.
- **Power BI**: Used for waiting time visualizations.
- **Git and GitHub**: Version control and collaboration for scripts and analysis.

## Data Exploration

### Total Number of Patients and Their Average Wait Time
This query calculates the total number of patients and their average wait time:
```sql
SELECT SUM(Number_In) AS NumberOfPatients, AVG(Patient_WaitTime) AS AverageWaitTime
FROM WaitingTime
GROUP BY Number_In
ORDER BY AverageWaitTime ASC;
Comparison of Total Patient Waiting Time vs Waiting Time in the Guichet
This query compares the total patient waiting time with the time spent waiting in the guichet:

sql
Copier le code
SELECT Patient_WaitTime, waitTime_guichet1, (waitTime_guichet1 / Patient_WaitTime) * 100 AS GuichetTimePercentage
FROM WaitingTime
WHERE Patient_WaitTime <> 0
ORDER BY GuichetTimePercentage DESC;
Correlation Between Patient Time and Number of Patients in the Hospital
This query calculates the correlation between the number of patients in the hospital and the patient wait time:

sql
Copier le code
SELECT
  (SUM(Number_In * Patient_WaitTime) - (SUM(Number_In) * SUM(Patient_WaitTime)) / COUNT(*))
  / SQRT(
    (SUM(POWER(Number_In, 2)) - POWER(SUM(Number_In), 2) / COUNT(*))
    * (SUM(POWER(Patient_WaitTime, 2)) - POWER(SUM(Patient_WaitTime), 2) / COUNT(*))
  ) AS correlation
FROM WaitingTime;
Data Cleaning
Standardizing the Date
The Patient-ArrivalTime column was standardized to a date format using the following query:

sql
Copier le code
ALTER TABLE WaitingTime
ADD ArrivalTimeConverted DATE;

UPDATE WaitingTime
SET ArrivalTimeConverted = CONVERT(DATE, [Patient-ArrivalTime]);
Populating Property Address Data
The PatientAddress column was split into PatientSplitAddress and PatientAddressCity using the following queries:

sql
Copier le code
SELECT PatientAddress,
  SUBSTRING(PatientAddress, 1, CHARINDEX(',', PatientAddress) - 1) AS address,
  SUBSTRING(PatientAddress, CHARINDEX(',', PatientAddress) + 1, LEN(PatientAddress)) AS city
FROM WaitingTime;

ALTER TABLE WaitingTime
ADD PatientSplitAddress NVARCHAR(255);

UPDATE WaitingTime
SET PatientSplitAddress = SUBSTRING(PatientAddress, 1, CHARINDEX(',', PatientAddress) - 1);

ALTER TABLE WaitingTime
ADD PatientAddressCity NVARCHAR(255);

UPDATE WaitingTime
SET PatientAddressCity = SUBSTRING(PatientAddress, CHARINDEX(',', PatientAddress) + 1, LEN(PatientAddress));
Removing Outliers
Rows with outlier values were removed using the following query:

sql
Copier le code
DELETE FROM WaitingTime
WHERE waitTime_guichet1 > Patient_WaitTime
  OR waitTime_Payment > Patient_WaitTime
  OR waitTime_Pneumo > Patient_WaitTime
  OR waitTime_Cardio > Patient_WaitTime
  OR waitTime_IECG > Patient_WaitTime;
Handling Null Values
Rows with null values in the PatientAddress column were removed using the following query:

sql
Copier le code
DELETE FROM WaitingTime
WHERE PatientAddress IS NULL;
Removing Duplicates
Duplicate rows were removed using the following query:

sql
Copier le code
WITH rowNumCte AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY Patient_WaitTime, PatientAddress, Number_In
      ORDER BY Patient_WaitTime
    ) AS Row_Num
  FROM WaitingTime
)
DELETE FROM rowNumCte
WHERE Row_Num > 1;
Removing Unused Columns
The unused columns PatientAddress1 and PatientAddress were removed using the following query:

sql
Copier le code
ALTER TABLE WaitingTime
DROP COLUMN PatientAddress1, PatientAddress;
Data Visualization
Distribution of Different Waiting Times
To gain a deeper understanding of the data, it is important to analyze the distribution of features. The following visualization, done with Power BI, showcases the main waiting times in the hospital: Pneumo, Cardio, IECG, Payment, and Guichet 1, showing an uneven distribution of observations.

Univariate Analysis
The Python code used to plot the different distributions:

python
Copier le code
import seaborn as sns
import matplotlib.pyplot as plt

features = [col for col in df.columns if df[col].dtype == 'float64' or col == 'Type']
num_features = [feature for feature in features if df[feature].dtype == 'float64']

# Histograms of numeric features
fig, axs = plt.subplots(nrows=4, ncols=7, figsize=(25, 15))
fig.suptitle('Numeric Features Histogram')

for j, feature in enumerate(num_features):
    sns.histplot(ax=axs[j // 7, j - 7 * (j // 7)], data=df, x=feature)

plt.show()
The resulting dataset was further analyzed to gain insights on the distribution of each feature. This figure presents the variation of parameters before normalization, showing a clear distribution pattern. However, certain variables such as NQ-room1 and NQ-room2 have almost identical distributions, indicating they can be considered as a single variable.

Data Normalization
To address the issue of differing distributions, normalization of all other features was performed to help the model learn the underlying patterns in the data more accurately, ultimately leading to better results. The following Python code was used to normalize the different distributions:

python
Copier le code
import pandas as pd
from sklearn import preprocessing

X = X.values  # Returns a numpy array
min_max_scaler = preprocessing.MinMaxScaler()
x_scaled = min_max_scaler.fit_transform(X)
X = pd.DataFrame(x_scaled)
