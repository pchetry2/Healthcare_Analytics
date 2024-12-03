**Hospital Length of Stay Analysis and Prediction in New York State Healthcare System**

1. **Executive Summary**
Developed comprehensive analysis and prediction model for patient length of stay using NY State SPARCS dataset, achieving 78.7% accuracy (R² score). Project combines SQL analysis, statistical testing, and machine learning to provide actionable healthcare insights.

2. **Project Overview**
2.1 **Problem Statement**
Healthcare facilities face challenges in:

Resource allocation efficiency
Bed capacity management
Cost optimization
Staff scheduling
Emergency preparedness

2.2 **Objectives**

Analyze cost patterns across facilities
Identify key LOS influencing factors
Predict patient length of stay
Generate data-driven operational insights

3. **Dataset**
3.1 **Source Information**

**Origin**: NY State Department of Health SPARCS
**Scope**: Hospital inpatient stays (2019)
**Size**: 1,048,575 records, 28 features
**Key Variables**: Demographics, Clinical Indicators, Costs, Facility data

3.2 **Features**
<img width="552" alt="Screenshot 2024-12-03 at 1 13 43 PM" src="https://github.com/user-attachments/assets/6bc3c968-257a-44d2-84c8-c847a5d0d55d">

4. **Data Pipeline and Infrastructure Setup**

Created AWS RDS PostgreSQL instance for scalable data storage
Established secure connection to database
Developed data loading pipeline to handle 1M+ records efficiently

5. **Exploratory Data Analysis and SQL Insights**

5.1 **Initial Data Assessment**

**Original dataset**: 1,048,575 rows and 28 columns
**Removed 5 columns with low fill rates**: CCSR_Procedure_Code, CCSR_Procedure_Description, Payment_Typology_2, Payment_Typology_3, Birth_Weight
Handled outliers in Length_of_Stay (clipped to 1-35 days)

5.2 **SQL Analysis Results**
5.2.1 **Top Diagnosis Cost Analysis**
WITH RankedDiagnoses AS (
    SELECT 
        Facility_Name,
        CCSR_Diagnosis_Description,
        SUM(Total_Costs) AS Total_Cost,
        RANK() OVER (PARTITION BY "Facility_Name" ORDER BY SUM(Total_Costs) DESC) as cost_rank
    FROM 
        hospital_Inpatient_data
    GROUP BY 
        Facility_Name, CCSR_Diagnosis_Description
)
SELECT 
    Facility_Name, 
    CCSR_Diagnosis_Description, 
    Total_Cost
FROM 
    RankedDiagnoses
WHERE 
    cost_rank = 1  
ORDER BY 
    Total_Cost DESC;
**Key Findings:**

**SEPTICEMIA** emerged as highest cost diagnosis
Significant cost variations between facilities
Emergency cases concentrated in specific diagnoses

5.2.2 **Average Length of Stay by Age Group**

WITH AvgLOSByAge AS (
    SELECT 
        Age_Group,
        AVG(Length_of_Stay) AS Avg_Length_of_Stay
    FROM 
        hospital_Inpatient_data
    GROUP BY 
        Age_Group
)
SELECT 
    Age_Group, 
    Avg_Length_of_Stay
FROM 
    AvgLOSByAge
ORDER BY 
    Avg_Length_of_Stay DESC
LIMIT 5;
**Key Insights:**

Elderly patients (70+) showed longest average stays (5.87 days)
Progressive increase in LOS with age
Age group 0-17 had shortest stays (3.66 days)

5.2.3 **Top 3 Payment Provider Analysis**
WITH RankedDiagnoses AS (
    SELECT 
        Payment_Typology_1,
        CCSR_Diagnosis_Description,
        SUM(Total_Charges) AS Total_Charge,
        RANK() OVER (PARTITION BY Payment_Typology_1 ORDER BY SUM(Total_Charges) DESC) as charge_rank
    FROM 
        hospital_Inpatient_data
    GROUP BY 
        Payment_Typology_1, CCSR_Diagnosis_Description
)
SELECT 
    Payment_Typology_1,
    CCSR_Diagnosis_Description,
    Total_Charge
FROM 
    RankedDiagnoses
WHERE 
    charge_rank = 1
ORDER BY 
    Total_Charge DESC
LIMIT 3;
**Key Findings:**

1. **Medicare**
Highest charges: SEPTICEMIA ($3.79B)
Dominated high-cost treatments
Largest payer in dataset

2. **Medicaid**
Highest charges: SEPTICEMIA ($1.06B)
Second largest payer
Significant cost difference from Medicare

3. **Private Health Insurance**
Different diagnosis pattern (LIVEBORN, $667.3M) 
Third highest in total charges

5.2.4 **NYU Langone Orthopedic Hospital Emergency Analysis**
SELECT 
    CCSR_Diagnosis_Description,
    Facility_Name,
    COUNT(*) AS Number_Of_Cases,
    Type_of_Admission
FROM 
    hospital_Inpatient_data
WHERE 
    Facility_Name = 'NYU Langone Orthopedic Hospital'
    AND Type_of_Admission = 'Emergency'
GROUP BY 
    CCSR_Diagnosis_Description, Facility_Name, Type_of_Admission
ORDER BY 
    Number_Of_Cases DESC
LIMIT 1;
**Key Findings:**
FRACTURE OF THE NECK OF THE FEMUR (HIP) is the most common case with 118 counts, reflects hospital's specialized emergency care

5.2.5 **Cost-LOS Correlation Analysis**
SELECT 
    Facility_Name,
    ROUND(AVG(Total_Costs)::numeric, 2) AS Avg_Total_Cost,
    ROUND(AVG(Length_of_Stay)::numeric, 2) AS Avg_Length_of_Stay,
    ROUND((AVG(Total_Costs) / NULLIF(AVG(Length_of_Stay), 0))::numeric, 2) AS Cost_Per_Day,
    COUNT(*) AS Total_Cases,
    ROUND(CORR(Total_Costs, Length_of_Stay)::numeric, 2) AS Cost_LOS_Correlation
FROM 
    hospital_Inpatient_data
GROUP BY 
    Facility_Name
HAVING 
    COUNT(*) > 10 
ORDER BY 
    Avg_Total_Cost DESC;
**Key Insights:**

**Specialized Care Costs**: Specialty hospitals show highest costs and longest stays

Henry J. Carter Hospital: Highest average cost - $89,339 avg cost, 22.42 days
Blythedale Children's Hospital: Longest average cost - $70,817 avg cost, 23.68 days


Cost Efficiency Variations: Daily cost varies significantly
Highest: Hospital for Special Surgery ($11,546/day)
Lowest: SJRH Park Care Pavilion ($650/day)
Shows wide range in service costs and resource utilization

6. **A/B Testing Analysis**
I conducted A/B testing to compare Length of Stay and Total Costs between different severity levels:
**Hypothesis Testing**
<img width="745" alt="Screenshot 2024-12-03 at 1 54 31 PM" src="https://github.com/user-attachments/assets/609bacc9-df70-4ead-bd34-66d1bc2dbc94">

**Results:**
Significant difference found in both LOS and costs (p < 0.05)
Demonstrated clear impact of severity on hospital stays

**Visualization:**
Created boxplots using seaborn to visualize:
LOS distribution by severity
Cost variations across severity levels

<img width="1192" alt="Screenshot 2024-12-03 at 1 59 40 PM" src="https://github.com/user-attachments/assets/094613a0-4057-4446-a3b4-8eb3cd0b3b5a">
<img width="1108" alt="Screenshot 2024-12-03 at 1 56 49 PM" src="https://github.com/user-attachments/assets/0d901689-aa92-4d3c-b2d5-75addaaf0878">




   
    





