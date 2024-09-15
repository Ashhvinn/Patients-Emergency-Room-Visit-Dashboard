# Patients Emergency Room Visit Dashboard

This repository contains the DAX code and instructions to create a comprehensive dashboard for analyzing patient emergency room visits using Power BI. The dashboard provides valuable insights into patient waiting times, visit trends, departmental referrals, and patient satisfaction.

Author : Ashwin Baalaji R

## Project Overview

The Patients Emergency Room Visit Dashboard offers a detailed view of patient emergency visits. It helps healthcare professionals and hospital administrators make data-driven decisions to improve patient care, optimize resource allocation, and identify areas for improvement.

## Steps to Create the Dashboard

### 1. Prepare Data

- Import your dataset into Power BI.
- Ensure the data includes relevant columns like `patient_id`, `patient_age`, `patient_gender`, `patient_admin_flag`, `patient_waittime`, `patient_sat_score`, `department_referral`, and `Date`.

### 2. Create Date Table

Use the following DAX code to create a Date table that covers the range of your data:

```dax
Date = 
VAR MinYear = YEAR(MIN('Patient Dataset'[Date]))
VAR MaxYear = YEAR(MAX('Patient Dataset'[Date]))
RETURN
ADDCOLUMNS(
    FILTER(
        CALENDARAUTO(),
        YEAR([Date]) >= MinYear &&
        YEAR([Date]) <= MaxYear
    ),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date],"mmm"),
    "MonthNum", MONTH([Date]),
    "Day", DAY([Date]),
    "Weekday", FORMAT([Date],"ddd"),
    "Week_num", WEEKDAY([Date]),
    "Quarter", "Q-" & QUARTER([Date]),
    "WeekType", IF(WEEKDAY([Date])=1, "Weekend", IF(WEEKDAY([Date])=7, "Weekend", "Weekday")),
    "WK of Month",
        VAR FirstDayOfMonth = DATE(YEAR([Date]), MONTH([Date]), 1)
        VAR DaysInMonth = DAY(EOMONTH([Date], 0))
        VAR CurrentDate = [Date]
        VAR DaysFromStart = INT(CurrentDate - FirstDayOfMonth) + 1
        RETURN 
        "Week-" & CEILING(DaysFromStart / 7, 1)
)
```

### 3. Add Measures

Create the following DAX measures in Power BI:

```dax
% Administrative Scheduled = 
DIVIDE(
    COUNTROWS(FILTER('Patient Dataset', 'Patient Dataset'[patient_admin_flag] = TRUE())),
    COUNTROWS('Patient Dataset')
)

% None-Administrative Scheduled = 
DIVIDE(
    COUNTROWS(FILTER('Patient Dataset', 'Patient Dataset'[patient_admin_flag] = FALSE())),
    COUNTROWS('Patient Dataset')
)

% ReferredPatients = 
VAR _FilteredPatients = CALCULATE([Total Visit], 'Patient Dataset'[department_referral] <> "None")
VAR _TotalPatients = [Total Visit]
RETURN DIVIDE(_FilteredPatients, _TotalPatients, 0)

% Un-Refered Patients = 
VAR _FilteredPatients = CALCULATE([Total Visit], 'Patient Dataset'[department_referral] = "None")
VAR _TotalPatients = [Total Visit]
RETURN DIVIDE(_FilteredPatients, _TotalPatients, 0)

Average Patient Age = AVERAGE('Patient Dataset'[patient_age])

AverageSatisfactionScore = CALCULATE(
    AVERAGE('Patient Dataset'[patient_sat_score]),
    'Patient Dataset'[patient_sat_score] <> BLANK()
)

AverageWaitTime 2 = AVERAGE('Patient Dataset'[patient_waittime])

Caption = 
    VAR _SelectedMeasure = SELECTEDVALUE(Parameter[Parameter Order])
    RETURN IF(
        _SelectedMeasure = 0,
        "The darkest PURPLE on the SCALE denotes LOW Waiting TIME on the Age-Group",
        "Patients are most SATISFIED when the SCALE shows the darkest PURPLE on the Age-Group"
    )

CF Max Visit Month = 
VAR _Max = CALCULATETABLE(
    ADDCOLUMNS(
        SUMMARIZE('Date', 'Date'[Month]),
        "@Visit", [Total Visit]
    ),
    ALLSELECTED()
)
VAR MinVal = MINX(_Max, [@Visit])
VAR MaxVal = MAXX(_Max, [@Visit])
VAR _CurrentVal = [Total Visit]
VAR Result = SWITCH(
    TRUE(),
    _CurrentVal = MinVal, 1,
    _CurrentVal = MaxVal, 2
)
RETURN Result

CF Max Visit Year

```markdown
```dax
 = 
VAR _Max = CALCULATETABLE(
    ADDCOLUMNS(
        SUMMARIZE('Date', 'Date'[Year]),
        "@Visit", [Total Visit]
    ),
    ALLSELECTED()
)
VAR MinVal = MINX(_Max, [@Visit])
VAR MaxVal = MAXX(_Max, [@Visit])
VAR _CurrentVal = [Total Visit]
VAR Result = SWITCH(
    TRUE(),
    _CurrentVal = MinVal, 1,
    _CurrentVal = MaxVal, 2
)
RETURN Result

Female Visit = 
    VAR _FemaleGender = CALCULATE(
        [Total Visit],
        'Patient Dataset'[patient_gender] = "F"
    )
    RETURN DIVIDE(_FemaleGender, [Total Visit])

Male Visit = 
    VAR _MalesGender = CALCULATE(
        [Total Visit],
        'Patient Dataset'[patient_gender] = "M"
    )
    RETURN DIVIDE(_MalesGender, [Total Visit])

MultipleVisitsCount = 
CALCULATE(
    COUNTROWS('Patient Dataset'),
    ALLEXCEPT('Patient Dataset', 'Patient Dataset'[patient_id]),
    FILTER('Patient Dataset', COUNTROWS(FILTER('Patient Dataset', 'Patient Dataset'[patient_id] = EARLIER('Patient Dataset'[patient_id]))) > 1)
)

No Rating = 
    VAR _NoRatings = CALCULATE(
        [Total Visit],
        'Patient Dataset'[patient_sat_score] = BLANK()
    )
    RETURN DIVIDE(_NoRatings, [Total Visit])

Total Max Visit = 
VAR _Max = CALCULATETABLE(
    ADDCOLUMNS(
        SUMMARIZE('Date', 'Date'[Month]),
        "@Visit", [Total Visit]
    ),
    ALLSELECTED()
)
VAR MinVal = MINX(_Max, [@Visit])
VAR MaxVal = MAXX(_Max, [@Visit])
VAR _CurrentVal = [Total Visit]
VAR Result = SWITCH(
    TRUE(),
    _CurrentVal = MinVal, [Total Visit],
    _CurrentVal = MaxVal, [Total Visit]
)
RETURN Result

Total Visit = COUNTROWS('Patient Dataset')

Unspecified Gender Visit = 
    VAR _NC = CALCULATE(
        [Total Visit],
        'Patient Dataset'[patient_gender] = "NC"
    )
    RETURN DIVIDE(_NC, [Total Visit])
```

### 4. Build Visualizations

- **Average Waiting Time**: Create line charts or bar charts to visualize average waiting times over different periods.
- **Patient Visits by Month**: Use line or bar charts to show monthly visit trends.
- **Total Visits by Department Referral**: Use pie charts or bar charts to display the number of visits by referring department.
- **Breakdown of Patient Visits by Age Group**: Create bar charts or pie charts to show the distribution of visits across age groups.
- **Average Patient Satisfaction by Race and Age Group**: Use bar charts or scatter plots to analyze satisfaction scores by race and age group.
- **Average Wait Time by Race and Age Group**: Create scatter plots or bar charts to examine wait times by race and age group.
- **Patient Visit Trends**: Combine various visualizations to track overall trends in patient visits, referrals, wait times, and satisfaction.

### 5. Customize and Refine

- Add slicers and filters to allow users to interact with the data.
- Use conditional formatting and tooltips to enhance the dashboard's usability.
- Ensure the layout is intuitive and visually appealing.

## Snaps of Dashboard
![Screenshot 2024-09-15 214244](https://github.com/user-attachments/assets/47281c58-ccd5-4022-8062-91b743150a3c)
![Screenshot 2024-09-15 214315](https://github.com/user-attachments/assets/ec009524-de5a-4b99-96b9-186522b7ee5f)
![Screenshot 2024-09-15 214403](https://github.com/user-attachments/assets/8d06e882-ff27-4a49-8b00-3523a5fb3ff5)


## Conclusion

The **Patients Emergency Room Visit Dashboard** provides a comprehensive analysis of patient visits to the emergency room, offering valuable insights into various aspects of emergency healthcare. By following the steps outlined above, users can effectively visualize and analyze data related to patient waiting times, visit trends, departmental referrals, and patient satisfaction.

Implementing this dashboard in Power BI allows healthcare professionals and hospital administrators to make informed decisions, identify areas for improvement, and enhance overall patient care. The use of DAX measures and visualizations enables a deep understanding of patient flow and satisfaction, helping optimize resource allocation and improve the quality of emergency services.

This dashboard serves as a powerful tool for data-driven decision-making, fostering an environment where healthcare insights can lead to actionable outcomes and better patient experiences.

