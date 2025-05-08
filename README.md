# First Pass Yield Report

The **First Pass Yield (FPY) Report** offers a detailed view of manufacturing efficiency by quantifying the percentage of units that successfully pass quality inspection or testing on the first attempt, without requiring rework or reprocessing. This report helps identify areas of improvement, monitor quality trends over time, and support decision-making to enhance overall production performance.

## 🧮 The Data Model



## 🧮 Calculating the First Pass Yield- By Date (Measure)
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
ADD PHOTO

---

## 🧮 Calculating the First Pass Yield- By Part (Measure)

This measure follows the same logic as **FPYbyDate** but excludes the **TREATAS** function since both **JOBS_CLOSED** and **FAIL_LOG** are directly related to the **PARTMASTER** table. This ensures accurate FPY calculations while leveraging existing relationships in the data model
 ``` 
FPYbyPart =
VAR TotalJC = CALCULATE(SUM('JOBS_CLOSED'[quantity]))
VAR TotalFAIL=CALCULATE(SUM('FAIL_LOG'[quantity]))
VAR TotalPASS= TotalJC-TotalFAIL
RETURN
IF(
    TotalJC=0,BLANK(), TotalPASS/TotalJC)
 ``` 
REPLACE PHOTO
---


