# First Pass Yield Report

The **First Pass Yield (FPY) Report** offers a detailed view of manufacturing efficiency by quantifying the percentage of units that successfully pass quality inspection or testing on the first attempt, without requiring rework. This report helps identify areas of improvement, monitor quality trends over time, and support decision-making to enhance overall production performance.

![FPY Report](https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/FirstPassYieldReport.png)

## 🛠️ The Data Model

**Data Structure:** 
This data model follows a **star schema**, where multiple **fact tables** connect to several **dimension tables** through key relationships. The primary relationship type used is **One-to-Many (1:n)** to ensure efficient data structuring and aggregation. All relationships are configured with **single-directional filtering** to avoid ambiguity in filtering logic.


**Fact Tables (Transactional Data)**
These tables store transactional and operational data, serving as the foundation for analysis.

- **JOBS_CLOSED** – Logs details of completed production jobs and units.  
  - **Key Field:** `part_number`
- **FAIL_LOG** – Captures inspection failures, tracking defect occurrences and related metrics.  
  - **Key Field:** `part_number`

**Dimension Tables (Descriptive Attributes)**
These tables provide contextual information that enriches analysis and reporting.

- **ITEM_MASTER** – Contains metadata about parts, such as commodity type, description, and standard  unit cost.  
  - **Key Field:** `part_number`
- **STOCK_STATUS** – Stores inventory levels, on-hand quantities, stock locations, and safety stock thresholds.  
  - **Key Field:** `part_number`
- **Calendar** – Acts as the date dimension, enabling time-based analysis across various fact tables.  
  - **Key Field:** `inspection_date`

**Schema Design Highlights**
- **Optimized Relationships:**  
  - `part_number` serves as the primary link between ITEM_MASTER and transactional tables.  
  - `inspection_date` connects Calendar to JOBS_CLOSED for trend analysis.  

![FPY Report](https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/FPY_Data_Model.png)


## 📈 Calculating the First Pass Yield- By Date (Measure)
**(Total Units-Total Unit Failed)/Total Units = First Pass Yield**

This measure tracks FPY on a Month-over-Month basis by aligning **inspection dates** with **calendar dates** using the function **TREATAS**. Since there's no direct relationship between the `FAIL_LOG` and `Calendar` tables, `TREATAS` enables proper data mapping while preserving the **star schema** structure of the model.
``` 
RevenuePerformanceMeasure =
FPYbyDate =
  VAR TotalJC =
      CALCULATE(
          SUM(JOBS_CLOSED[quantity]),
          TREATAS(VALUES('Calendar'[Date]), JOBS_CLOSED[inspection_date]) )
  VAR TotalFAIL =
      CALCULATE(
          SUM('FAIL_LOG'[quantity]),
          TREATAS(VALUES('Calendar'[Date]), 'FAIL_LOG'[inspection_date]))
  VAR TotalPASS = TotalJC - TotalFAIL
RETURN
IF(TotalJC = 0, BLANK(), DIVIDE(TotalPASS, TotalJC))
 ``` 
<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/FirstPassYield.gif">
</p>

---

## 🧮 Total Cost of Failed Units- Matrix

This Matrix estimates the **total cost of failed units** by part on a **Month-to-Month** basis.  
By multiplying the **standard unit cost** by the **total failed units**, the measure provides a financial impact assessment of quality failures.
```
TotalSpend(FailedUnits) =

VAR TotalFAIL =
    CALCULATE(
        SUM('FAIL_LOG'[quantity]),
        TREATAS(VALUES('Calendar'[Date]), 'FAIL_LOG'[inspection_date]))
VAR STD_UD= MAX(ITEM_MASTER[standrd_unit_cost])
RETURN
IF(
    ISBLANK(TotalFAIL*STD_UD),0,TotalFAIL*STD_UD)

```
![FPY Report](https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/TotalFailedCostMatrix.png)

---
## 🧮 Other Key Measures

Total Failed Units

This measure calculates the **total failed units** while ensuring missing values are handled correctly using `IF(ISBLANK())`. The `TREATAS` function allows the measure to align **inspection dates** with **calendar dates**, enabling accurate aggregation without requiring direct relationships between tables.

```
TotalFail =
VAR TotalFAIL =
    CALCULATE(
        SUM('FAIL_LOG'[quantity]),
        TREATAS(VALUES('Calendar'[Date]), 'FAIL_LOG'[inspection_date]))
RETURN
IF(
  ISBLANK(TotalFAIL), 0, TotalFAIL)
```
The same logic is applied when calculating the **total units** produced.
```
TotalJobs =
VAR TotalJC =
    CALCULATE(
        SUM(JOBS_CLOSED[quantity]),
        TREATAS(VALUES('Calendar'[Date]), JOBS_CLOSED[inspection_date])
RETURN 
IF(
  ISBLANK(TotalJC ), 0, TotalJC )
```
