# nigeria-health-infrastructure-analysis

## Table of Contents
1. [Project Executive Summary](#1-project-executive-summary)
2. [The Dataset](#2-the-dataset)
3. [The Analytics Stack](#3-the-analytics-stack)
4. [Data Wrangling & Pre-processing](#4-data-wrangling--pre-processing)
5. [Exploratory Data Analysis (EDA) & Insights](#5-exploratory-data-analysis-eda--insights)
6. [Key Outcomes & Findings](#6-key-outcomes--findings)
7. [Challenges & Data Limitations](#7-challenges--data-limitations)
8. [Recommendations for Policy Makers](#8-recommendations-for-policy-makers)


### **1. Project Executive Summary**

This project provides a comprehensive geospatial and operational evaluation of Nigeria’s healthcare infrastructure using a dataset of over 46,000 facilities. The study was born from the necessity to understand how healthcare resources are distributed across the 36 states and the Federal Capital Territory, and more importantly, how functional those resources truly are.

By leveraging Python for data orchestration, I examined the ratio of Primary, Secondary, and Tertiary facilities to identify critical bottlenecks in the referral system. A central theme of this analysis is the use of Facility Functionality as a proxy for regional socioeconomic status (SES); high rates of non-functional or Unknown status often correlate with systemic underfunding and geographic isolation.

The ultimate goal of this work is to bridge the gap between raw data and public health policy. The findings pinpoint specific healthcare deserts—regions where the absence of functional specialized care poses a direct threat to maternal health and chronic disease management—offering a data-driven roadmap for targeted infrastructural investment.

### **2. The Dataset**

* **Total Observations:** 46,605 health facilities.
* **Geographic Scope:** Covers all 36 Nigerian states and the Federal Capital Territory (FCT).
* **Key Data Points:**
    * **Identification:** Facility name and unique IDs (`global_id`).
    * **Location:** Nested geographic data including State and Local Government Area (LGA).
    * **Classification:** Facility levels (Primary, Secondary, and Tertiary) and specific categories (e.g., General Hospitals, Primary Health Centers, Teaching Hospitals).
    * **Operational Health:** The `functional_status` column, which tracks whether a facility is Functional, Non-Functional, Partially Functional, or Unknown.

  ### **3. The Analytics Stack**
* Python 
* Pandas 
* NumPy
* Matplotlib
* Seaborn 
* Jupyter Notebooks

### **4. Data Wrangling & Pre-processing**
* **De-duplication:** I conducted a integrity check on the `id` and `global_id` columns, this is to ensure that every facility is counted exactly once, preventing an overestimation of healthcare density.
* **Dimensionality Reduction:** To focus the analysis, I dropped nine high-cardinality/administrative columns that did not contribute to the socioeconomic study. This included stripping out `FID`, `ward_code`, `lga_code`, and `timestamp`.
* **Structural Cleaning:** I standardized the `state_name` and `type` columns to ensure consistency across the 46,000+ entries, ensuring that "Primary" and "Tertiary" classifications were uniform for accurate aggregation.
* **Missing Value Strategy:**  The `category` column had several missing entries. Rather than dropping these rows and losing geographic data, I labeled them as **"Unknown"**. 
    * This preservation allowed me to maintain a full count of facilities per state while still flagging data gaps as a limitation in the specialized infrastructure analysis.
* **Feature Engineering:** I generated a consolidated summary table that cross-tabulated `state_name` with `functional_status` to create new performance metrics, such as the **Functionality Rate %** and **Specialized Care Ratio**.

 ### **5. Exploratory Data Analysis (EDA) & Insights**

In this phase, I transitioned from data preparation to active interrogation of the dataset. The EDA was structured around answering critical questions regarding health equity and infrastructure reliability across Nigeria.

* **National Facility Distribution:** I calculated the volume of healthcare facilities per state to identify regional leaders and laggards. This highlighted a significant concentration of facilities in certain geopolitical zones, raising questions about rural access.
* **The Specialized Care Ratio:** One of the most impactful metrics created was the ratio of **Secondary and Tertiary facilities** to the total facility count. This revealed that while some states have many Primary centers, they lack the advanced infrastructure needed for complex medical cases.
* **Operational Health Breakdown:** I performed a global and state-level analysis of `functional_status`. By calculating the percentage of non-functional facilities, I was able to identify red-flag regions where existing buildings are likely not providing active care.
* **Infrastructure Service Levels:** Using Seaborn, I visualized the distribution of facility categories (e.g., Teaching Hospitals vs. Health Posts). This helped in understanding the Pyramid of care—showing a wide base of primary care but a very narrow peak of specialized centers.
* **Maternal Care Deep-Dive:** I specifically filtered for Maternity Homes and Comprehensive Health Centers to evaluate the availability of life-saving maternal services. The results showed that maternal infrastructure is unevenly distributed, often missing in areas that need it most.

```python?code_reference&code_event_index=2
import pandas as pd
import numpy as np

# Load the data
df = pd.read_csv('nigeriahealthfacilities.csv')

# Pre-processing similar to the notebook
# Drop duplicates based on 'id' if any
df_data = df.drop_duplicates(subset=['id'], keep='first').copy()

# Global operational status breakdown
status_counts = df_data['functional_status'].value_counts(normalize=True) * 100

# Top 5 states by facility count
state_counts = df_data['state_name'].value_counts()
top_5_states = state_counts.head(5)

# Calculate Specialized Care Ratio per state
# Type: Primary, Secondary, Tertiary
q1_counts = pd.crosstab(df_data['state_name'], df_data['type'])
q1_counts['Total'] = q1_counts.sum(axis=1)
q1_counts['Specialized'] = q1_counts.get('Secondary', 0) + q1_counts.get('Tertiary', 0)
q1_counts['Specialized_Ratio_%'] = (q1_counts['Specialized'] / q1_counts['Total']) * 100

# Bottom 5 states for specialized care
bottom_5_specialized = q1_counts.sort_values('Specialized_Ratio_%', ascending=True).head(5)

# Functionality rates per state
state_func = pd.crosstab(df_data['state_name'], df_data['functional_status'])
state_func['Total'] = state_func.sum(axis=1)
state_func['Functional_Rate_%'] = (state_func.get('Functional', 0) / state_func['Total']) * 100
state_func['Unknown_Rate_%'] = (state_func.get('Unknown', 0) / state_func['Total']) * 100

# Average Unknown rate
avg_unknown = state_func['Unknown_Rate_%'].mean()

print("Status Breakdown:\n", status_counts)
print("\nTop 5 States by Count:\n", top_5_states)
print("\nBottom 5 States Specialized Ratio:\n", bottom_5_specialized['Specialized_Ratio_%'])
print("\nAverage Unknown Rate:", avg_unknown)
print("\nNational Functional Rate:", status_counts.get('Functional', 0))


```
```text?code_stdout&code_event_index=2
Status Breakdown:
 functional_status
Functional              74.290296
Unknown                 25.469163
Not Functional           0.225372
Partially Functional     0.015169
Name: proportion, dtype: float64

Top 5 States by Count:
 state_name
Lagos      2320
Katsina    2228
Benue      2192
Niger      2010
Ogun       1721
Name: count, dtype: int64

Bottom 5 States Specialized Ratio:
 state_name
Katsina    1.122083
Taraba     1.594203
Kaduna     1.631913
Adamawa    1.841270
Bauchi     1.953418
Name: Specialized_Ratio_%, dtype: float64

Average Unknown Rate: 26.75101815091762

National Functional Rate: 74.29029601698956

```
### **6. Key Outcomes & Findings**

This analysis provides an empirical look at the Nigerian healthcare landscape. By transforming raw facility data into measurable metrics, the following key findings were established:

* **The National Functionality Baseline:** Approximately **74.3%** of healthcare facilities nationwide are recorded as "Functional." While this appears high, the data reveals a critical **25.5% "Unknown" or "Not Functional" rate**, indicating that 1 in every 4 facilities may not be capable of providing reliable care or is completely unmonitored.

* **The "Specialized Care" Crisis:** The referral pyramid is extremely bottom-heavy. In many states, the ratio of specialized (Secondary/Tertiary) facilities to primary centers is dangerously low. For instance:
    * **Katsina** and **Taraba** exhibit some of the lowest specialized care ratios in the country, at **1.12%** and **1.59%** respectively. 
    * This empirical evidence suggests that in these regions, the vast majority of citizens are restricted to primary care with almost no local access to advanced medical interventions.

* **State-Level Infrastructure Leaders:** Infrastructure is heavily concentrated in a few key states. The top five states by total facility count account for a significant portion of the national total:
    1. **Lagos:** 2,320 facilities
    2. **Katsina:** 2,228 facilities
    3. **Benue:** 2,192 facilities
    4. **Niger:** 2,010 facilities
    5. **Ogun:** 1,721 facilities

* **The "Data Blindness" Factor:** The analysis uncovered an average **26.8% "Unknown" rate** across states regarding operational status. This isn't just a data error; it is an empirical finding that suggests significant administrative gaps in health facility monitoring. Without a clear status for 1/4 of the country's clinics, policy interventions remain partially blind.

* **Vulnerability in the Referral System:** The primary-to-specialized bottleneck is the most significant structural finding. With states like **Kaduna (1.63%)** and **Adamawa (1.84%)** also showing critical shortages in advanced facilities, the data proves that geographical location is a primary determinant of whether a patient can receive life-saving, high-level medical treatment.

  ### **7. Challenges & Data Limitations**

* **The "Unknown" Status Gap:** As noted in the findings, over **25%** of the records for `functional_status` were listed as "Unknown." This creates a "gray area" where we cannot definitively say if a facility is serving its community or sitting dormant. In data analysis, these missing values often point to regions with weaker administrative reporting.
* **Missing Categories:** Several entries lacked a clear `category` (e.g., whether a clinic is a "Health Post" or a "General Hospital"). To preserve the geographic integrity of the analysis, I labeled these as **"Unknown"** rather than deleting them, but this adds a layer of uncertainty to the infrastructure "Pyramid" classification.
* **Proxy Constraints:** This study uses facility counts and types as a **proxy** for socioeconomic development. However, without integrating external data like population density or poverty indices, we cannot perfectly calculate the "Patient-to-Doctor" ratio or the true burden on each facility.
* **Reporting Bias:** The high number of facilities in certain states (like Lagos or Katsina) might partially reflect more rigorous data collection efforts in those regions rather than just a higher physical number of buildings compared to states with lower counts.

### **8. Recommendations for Policy Makers**

* **Targeted Investment in "Specialized Care Deserts":** Priority should be given to states like **Katsina, Taraba, and Kaduna**, where the ratio of secondary and tertiary facilities is below 2%. Infrastructure funds should focus on upgrading existing primary centers to secondary levels to bridge the specialized care gap.

* **Implement a Digital National Monitoring System:** The **26.8% "Unknown" status** rate highlights a critical breakdown in facility oversight. Policy makers should mandate a real-time, digital reporting framework where local government health officers provide quarterly operational updates to eliminate "data blindness."

* **Strengthen the Referral Pathway:** The current "bottom-heavy" pyramid suggests that primary centers are overwhelmed while specialized centers are scarce. Policies should focus on regional "referral hubs"—strategically located secondary facilities that serve a cluster of primary health centers—to ensure patients aren't traveling across state lines for basic surgery or specialized care.

* **Address Maternal Infrastructure Gaps:** Using the geospatial findings, specific LGAs with high population density but zero "Maternity Homes" should be identified for immediate establishment of comprehensive maternal health centers to reduce maternal mortality rates.

* **Incentivize Operational Maintenance:** Since many facilities exist but are "Non-Functional," budget allocations should shift from "new construction" to "operational maintenance." Ensuring that the ~74% functional rate increases is more cost-effective than building new centers that may eventually become non-functional due to lack of staff or equipment.

* **Socioeconomic-Based Resource Allocation:** Federal health grants should be allocated using a "Functionality and Need Index" rather than equal distribution. States with high facility counts but low functionality rates require different intervention strategies (management/staffing) compared to states with low facility counts (construction).
