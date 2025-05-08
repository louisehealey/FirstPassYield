# First Pass Yield Report

The **First Pass Yield (FPY) Report** offers a detailed view of manufacturing efficiency by quantifying the percentage of units that successfully pass quality inspection or testing on the first attempt, without requiring rework or reprocessing. This report helps identify areas of improvement, monitor quality trends over time, and support decision-making to enhance overall production performance.

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

## 🔄 Dynamic Dashboard Title

The DashboardTitle measure generates a dynamic report title based on the selected year. It extracts the maximum date currently filtered in the report, retrieves its year and appends it to the static text label "Revenue Overview". This allows the report title to automatically update to reflect the year being analyzed, providing clear and context-sensitive labeling for the report.
 ``` 
DashboardTitle =
VAR SEL_YEAR= YEAR(MAX('Revenue Table'[FirstOfTheMonth]))
RETURN
"Revenue Overview" & ": " & SEL_YEAR
 ``` 
REPLACE PHOTO
---

## 📂 Quarter and Month Tabs

Using buttons and custom bookmarks, the report is organized into separate tabs, allowing for easier navigation between different time views—such as quarterly and monthly data.
<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/RevenueOverview/main/RevenueTrackerTabChange.gif" width="600"/>
</p>

## ℹ️ Tool Tips
Using both the default and custom tooltips provides more context to the report. In addition to the default tooltips (**Total Revenue %** and **Total Goal %**), exact figures for the **Total Revenue in Dollars** (`Total Revenue ($$)`) and the **Total Goal in Dollars** (`Total Goal ($$)`) are also included. These custom tooltips offer additional data points without overcrowding the visual, making the report easy and fast to digest.

<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/RevenueOverview/main/RevenueTrackerToolTip.gif" width="600"/>
</p>

