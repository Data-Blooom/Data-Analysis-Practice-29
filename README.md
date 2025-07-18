# Power BI: Programming Language Popularity Analysis

This repository contains a Power BI project focused on analyzing historical trends in the popularity of major programming languages. The dashboard provides a clear, data-driven view of the evolving developer landscape, making it a valuable tool for students, developers, and technology managers.

## 📝 Project Overview

The goal of this dashboard is to answer key questions about programming language relevance and growth:
* Which languages have shown the most significant growth over the past decade?
* Which languages have consistently remained at the top?
* How does the growth of a specific language like Python compare to the industry average?
* What is the overall distribution of popularity among leading languages?

The dashboard uses a combination of clear KPIs, trend lines, and comparative charts to tell a compelling story with data.

---
## 📊 Visualizations Breakdown

* **KPI Cards:** Display the all-time average popularity score for key languages: Python, Java, C/C++, and Visual Basic.
* **Line Chart (Average of Python by Year):** Tracks the historical popularity of Python, clearly illustrating its dramatic upward trend.
* **Donut Chart (Sum of Avg_Popularity by Attribute):** Shows the market share of each programming language, providing a high-level view of the popularity distribution.
* **Bar Chart (Count of Date by Attribute - Rank 1 to 5):** A unique visual that counts how many times each language has ranked in the top 5, acting as a measure of long-term consistency and relevance.
* **Combo Chart (Avg Python and Avg All Languages by Year):** Directly compares Python's popularity trend (line) against the average popularity of all languages combined (columns), highlighting Python's exceptional growth.

---
## 🛠️ Data Modeling: The Unpivot Transformation

A critical step in this project was transforming the data from a **wide format** to a **long format**. The original dataset likely had a separate column for each programming language.

**Why Unpivot?**
A wide format is difficult to analyze flexibly in Power BI. To create visuals that can be filtered or grouped by language name (e.g., in the donut chart), we need a single column containing all language names (`Attribute`) and another single column for their corresponding values (`Value`).

**How It Was Done (in Power Query):**
1.  Select all the columns representing programming languages (e.g., `Python`, `Java`, `C/C++`, etc.).
2.  Right-click on the header of any selected column.
3.  Choose the **"Unpivot Columns"** option.
4.  This action creates two new columns: "Attribute" (containing the language names) and "Value" (containing the popularity scores), making the data model efficient and scalable. The resulting table is named `unpivoted_dataset`.

---
## 💡 DAX Measures

The following DAX measures were created to power the visuals.

### **Simple Averages**
These measures calculate the average popularity for individual languages from the original wide table. Used for the KPI cards.

    Avg C/C++ = AVERAGE('dataset'[C/C++])
    Avg Java = AVERAGE('dataset'[Java])
    Avg Python = AVERAGE('dataset'[Python])
    Avg Visual Basic = AVERAGE('dataset'[Visual Basic])
    
### **Overall Average (From Unpivoted Data)**
This measure correctly calculates the average popularity across ALL languages for a given time period by iterating over the long-format table.

    Avg All Languages = AVERAGEX(VALUES(unpivoted_dataset[Attribute]), CALCULATE(AVERAGE(unpivoted_dataset[Value])))

### **Calculated Summary Table for Averages**
This DAX expression generates a summary table in memory, calculating the average popularity for each language. This is highly efficient for visuals like the donut chart.

    Language_Avg = 
    SUMMARIZECOLUMNS(
        'unpivoted_dataset'[Attribute],
        "Avg_Popularity", AVERAGE('unpivoted_dataset'[Value])
    )
    
### **Monthly Ranking of Languages**
This is the most complex measure. It calculates the popularity rank of each language for each specific month in the dataset. This is the engine behind the "Consistency Ranking" bar chart.

    Top_Language_Per_Month = 
    VAR Summary = 
        SUMMARIZE(
            'unpivoted_dataset',
            'unpivoted_dataset'[Date],
            'unpivoted_dataset'[Attribute],
            "Popularity", AVERAGE('unpivoted_dataset'[Value])
        )
    RETURN
        ADDCOLUMNS(
            Summary,
            "Rank", 
                RANKX(
                    FILTER(Summary, [Date] = EARLIER([Date])),
                    [Popularity],
                    ,
                    DESC
                )
        )

### **Utility Measure**
A simple measure for transparent backgrounds, used for aesthetic design purposes.

    BG Transparent = "rgba(255, 255, 255, 0)"
