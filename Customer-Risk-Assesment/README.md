# Supplier Risk Assessment

Analyzing supplier performance data ensures quality control, cost savings, and operational stability. By tracking defect rates, downtime, and material quality, companies identify underperforming suppliers, cut costs, and reduce risks. This leads to a more reliable supply chain, stronger supplier relationships, and higher operational efficiency.

## Importance of Supplier Performance Analysis

### 1. Quality Control
Tracking defects and their impact helps identify underperforming suppliers, ensuring that product quality standards are maintained.

### 2. Cost Management
By analyzing downtime caused by defective materials, companies can assess the financial impact on operations and take corrective actions to reduce costs.

### 3. Supplier Performance Evaluation
It enables businesses to evaluate suppliers' consistency in timely delivering high-quality materials, leading to better decision-making on future contracts.

### 4. Risk Mitigation
Understanding defect trends and downtime helps proactively manage risks, allowing businesses to develop contingency plans with reliable suppliers.

### 5. Operational Efficiency
Reducing downtime and improving supplier reliability enhances overall production efficiency, leading to smoother operations and higher customer satisfaction.

This analysis allows businesses to make data-driven decisions to optimize supplier selection, reduce operational risks, and ensure a stable supply chain.

## Real-World Examples

### Ensuring Product Quality
**Example:** An automotive manufacturer tracks defects in parts received from suppliers. If a specific supplier consistently delivers faulty brake components, this could lead to recalls or safety issues. By identifying this trend, the company can switch suppliers or demand improvements to maintain safety standards.

### Cost Reduction
**Example:** A consumer electronics company notices that a supplier’s defective microchips lead to frequent production stoppages. By analyzing downtime data, they discover this supplier is causing significant financial losses. Switching to a more reliable supplier reduces these costs, enhancing profitability.

### Improving Supplier Relationships
**Example:** A pharmaceutical company uses defect data to work collaboratively with a supplier of raw ingredients, helping them resolve quality issues. This strengthens the partnership, ensuring a consistent supply of high-quality inputs, crucial for compliance and patient safety.

### Minimizing Operational Downtime
**Example:** A food processing plant experiences production delays due to contaminated packaging materials. By analyzing downtime metrics, they identify the issue’s root cause and switch to a more reliable supplier, ensuring smooth operations and preventing further revenue loss.

### Mitigating Supply Chain Risks
**Example:** A tech company monitors supplier data to anticipate potential disruptions. When geopolitical tensions arise in a region where a key supplier is based, they proactively source alternatives to prevent shortages in critical components for their upcoming product launch.

By analyzing this data, businesses can make strategic, data-driven decisions, optimize their supplier base, and prevent costly disruptions in their supply chain.

## SQL Queries for Analysis

### Top Worst Category by Defect Rate
```sql
WITH TotalDefects AS (
    SELECT 
        Category,
        SUM([Total Defect Qty]) AS Defects
    FROM Supplier
    GROUP BY Category
),
Top1Defect AS (
    SELECT 
        Category,
        Defects,
        RANK() OVER (ORDER BY Defects DESC) AS Rank
    FROM TotalDefects
    WHERE Defects IS NOT NULL
),
MaxDefects AS (
    SELECT 
        MAX(Defects) AS MaxValue
    FROM Top1Defect
    WHERE Rank = 1
)
SELECT 
    Category
FROM TotalDefects
WHERE Defects = (SELECT MaxValue FROM MaxDefects);
```
OR
```PowerBI
-#1 Worst Defects Name by Category = 
Var vTable =
ADDCOLUMNS(
    SUMMARIZE(Category , Category[Category]),
    "Num_of_Defects", [Total_Defects])
   
Var _TopN =
TOPN(
    1,
    vTable,
    [Num_of_Defects],DESC)
   
Var _MaxValue =
MAXX(_TopN , [Num_of_Defects])
   
Var _Filter =
FILTER(vTable,
    [Num_of_Defects] = _MaxValue)

   Var Result =
CALCULATE(
    VALUES(Category[Category]),
    _Filter)
RETURN
Result

### Downtime Value
```sql
WITH TotalDowntime AS (
    SELECT 
        Category,
        SUM([Total Downtime (hrs)]) AS Downtime
    FROM Supplier
    GROUP BY Category
),
Top2Downtime AS (
    SELECT 
        Category,
        Downtime,
        RANK() OVER (ORDER BY Downtime DESC) AS Rank
    FROM TotalDowntime
    WHERE Downtime IS NOT NULL
)
SELECT 
    MIN(Downtime) AS Worst2ndDowntime
FROM Top2Downtime
WHERE Rank <= 2;
```
OR
``` PowerBi
Worst2ndDowntime = 
VAR TotalDowntime = 
    SUMMARIZE(
        Supplier,
        Supplier[Category],
        "Downtime", SUM(Supplier[Total Downtime Hours])
    )

VAR RankedDowntime = 
    ADDCOLUMNS(
        TotalDowntime,
        "Rank", RANKX(
            TotalDowntime,
            [Downtime],
            ,
            DESC,
            Skip
        )
    )

VAR Top2Downtime = 
    FILTER(
        RankedDowntime,
        [Rank] <= 2
    )

RETURN
    MINX(
        Top2Downtime,
        [Downtime]
    )

### High-Risk Vendors by Downtime (Over 400 Hours)
```sql
WITH TotalDowntime AS (
    SELECT 
        Vendor,
        SUM([Total Downtime (Hrs) related to Supplier]) AS Downtime
    FROM SupplierQuality
    GROUP BY Vendor
),
HighRiskVendors AS (
    SELECT 
        Vendor,
        Downtime
    FROM TotalDowntime
    WHERE Downtime > 400
)
SELECT 
    SUM(Downtime) AS TotalHighRiskDowntime
FROM HighRiskVendors;
```

### Downtime Cost by Impact and Rejected Defect Types
```sql
SELECT 
    SUM([Total Downtime Cost]) AS TotalDowntimeCost
FROM 
    DefectType
WHERE 
    [Defect Type] IN ('Impact', 'Rejected');
```
---PowerBi 
Total_Downtime_Costs = CALCULATE([Total Downtime]*24,Supplier[Defect Type]<>"No Impact")
### Moving Average for Downtime Costs
```sql
WITH FilteredData AS (
    SELECT 
        c.Date,
        COALESCE(d.[Total Downtime Cost], 0) AS TotalDowntimeCost
    FROM Calendar c
    LEFT JOIN Downtime d 
        ON c.Date = d.Date
    WHERE c.Date <= GETDATE() -- Ensures only past dates are considered
),
MovingAverage AS (
    SELECT 
        Date,
        TotalDowntimeCost,
        ROUND(
            AVG(TotalDowntimeCost) OVER (
                ORDER BY Date 
                ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
            ), 2
        ) AS MovingAvgCost
    FROM FilteredData
)
SELECT 
    Date, 
    TotalDowntimeCost, 
    MovingAvgCost
FROM MovingAverage
ORDER BY Date;
```

## Key Insights

- **High Total Defects:** The overall total defects are extremely high at 2.59 billion. The month of October and specific segments have significantly higher defect rates than others.
- **Medium Risk Vendors by Downtime:** Medium-risk vendors have accumulated 41,738 hours of downtime. There are spikes in Q4 2019 and other segments, indicating potential issues with certain vendors during this period.
- **Recent Moving Average Trends:**
  - Downtime costs declined sharply on November 23, 2019, dropping by 28.88% over approximately 1.27 months.
  - A steep increase in downtime costs occurred between January 1, 2018, and January 19, 2018, where costs surged from 14,592 to 141,168.
  - The longest growth trend in downtime costs was observed between January 21, 2018, and May 7, 2018, increasing by 70,512.

## Recommendations

### Investigate and Address High Defect Rates
- Conduct root cause analysis for months and segments with the highest defect rates, particularly October.
- Implement Quality Assurance (QA) programs with suppliers, including tighter quality controls, regular audits, and stricter KPIs.

### Optimize Medium-Risk Vendors by Downtime
- Focus on performance reviews and improvement plans for medium-risk vendors contributing to 41,738 hours of downtime.
- Analyze root causes of spikes in Q4 2019 and consider renegotiating contracts or implementing Service Level Agreements (SLAs).

### Leverage the Moving Average Trend Insights
- Investigate actions leading to the reduction in downtime costs post-November 23, 2019, to replicate across other areas.
- Analyze spikes in costs during January 2018 to understand driving factors and implement preventive measures.
- Assess prolonged growth periods for better resource allocation during high-risk times.

## Conclusion
The supplier assessment reveals significant areas for improvement, especially regarding defect management and downtime reduction. By taking a proactive approach to quality control, optimizing supplier performance, and learning from historical trends, the company can reduce operational risks, enhance supply chain efficiency, and lower overall costs.
