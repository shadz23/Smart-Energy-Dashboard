# Smart Energy Dashboard — Household Electricity Insights & Cost ⚡📊  
[![Releases](https://img.shields.io/badge/Releases-download-blue?logo=github)](https://github.com/shadz23/Smart-Energy-Dashboard/releases)  
https://github.com/shadz23/Smart-Energy-Dashboard/releases

An interactive Power BI dashboard that analyzes household electricity consumption to reveal usage patterns, peak hours, and estimated costs. It combines time-series analysis, load profiling, and cost modeling to help users manage energy use and spot savings opportunities.

---

Table of contents
- Overview
- Live demo & images
- Key features
- Dataset
- Power Query (M) — ETL steps
- DAX measures — examples
- Reports & pages
- Installation — download and run
- Usage tips
- Performance notes
- Contributing
- License
- Contact

---

Badges
- Topics: dashboard • data-analysis • data-visualization • dataset • dax • electricity • energy-consumption • power-bi • power-query • uci  
[![Topics](https://img.shields.io/badge/topics-dashboard%20%7C%20data--analysis%20%7C%20power--bi-lightgrey)](https://github.com/shadz23/Smart-Energy-Dashboard/releases)

Demo images
- Power BI report page (overview):  
  ![Overview](https://images.unsplash.com/photo-1504384308090-c894fdcc538d?auto=format&fit=crop&w=1200&q=60)
- Hourly load heatmap example:  
  ![Heatmap](https://upload.wikimedia.org/wikipedia/commons/8/87/Heatmap_example.png)
- Cost breakdown visual:  
  ![Cost Chart](https://cdn.pixabay.com/photo/2016/10/02/22/17/chart-1715268_1280.png)

Screenshots show example visuals, slicers, and measures applied to the UCI household power consumption dataset. Images illustrate layout and visual types used in the report.

---

Overview
- Purpose: Provide a clear view of when and how electricity is used in a household. The dashboard highlights peak hours, daily and weekly patterns, device-level estimates (where possible), and cost breakdowns.
- Audience: energy analysts, homeowners, facility managers, and students working with time-series electricity data.
- Tech: Power BI Desktop for report authoring, Power Query (M) for ETL, DAX for measures and time intelligence.

Key features
- Hourly and daily load visualizations (line charts, area charts).
- Heatmap that shows load by hour and day of week.
- Load profile clustering (basic segmentation inside Power BI).
- Estimated cost calculator based on tariff slabs or flat rates.
- Peak detection and rolling averages for smoothing spikes.
- Exportable PBIX with full data model, visuals, and measures.
- Reusable Power Query logic for different datasets.

Dataset
- Source used in examples: UCI Machine Learning Repository — "Individual household electric power consumption Data Set".
- Typical fields:
  - Date, Time
  - Global_active_power (kW)
  - Global_reactive_power (kW)
  - Voltage (V)
  - Sub_metering_1 (kWh)
  - Sub_metering_2 (kWh)
  - Sub_metering_3 (kWh)
- Frequency: 1-minute samples (original). The project uses aggregated views (minute → hourly, daily) to optimize visuals.
- Data quality steps: parse date/time, handle missing values, remove outliers beyond physical limits, aggregate to useful time grain.

Power Query (ETL) — recommended steps
1. Source
   - Import CSV or text file. For the UCI dataset use proper delimiter and header row.
2. Parse datetime
   - Merge Date and Time into a DateTime column.
3. Convert types
   - Set numeric columns to Decimal Number, DateTime to Date/Time.
4. Handle errors
   - Replace text-based missing values ("?") with null, then decide on interpolation or removal.
5. Aggregate
   - Use Table.Group to compute Hourly and Daily aggregates:
     - Hourly: Average or sum Global_active_power per hour.
     - Daily: Sum kWh per day.
6. Create time attributes
   - Add HourOfDay, DayOfWeek, WeekOfYear, Month, IsWeekend.
7. Load to model
   - Load raw minute-level only if needed. Prefer hourly aggregates for performance.

Example M snippet (Power Query)
```m
let
  Source = Csv.Document(File.Contents("household_power_consumption.txt"),[Delimiter=";", Columns=9, Encoding=65001, QuoteStyle=QuoteStyle.None]),
  Promoted = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
  Replaced = Table.ReplaceValue(Promoted,"? ",null,Replacer.ReplaceText,{"Global_active_power","Global_reactive_power","Voltage","Sub_metering_1","Sub_metering_2","Sub_metering_3"}),
  ChangedTypes = Table.TransformColumnTypes(Replaced,{
    {"Date", type date}, {"Time", type text}, {"Global_active_power", type number}, {"Voltage", type number},
    {"Sub_metering_1", type number}, {"Sub_metering_2", type number}, {"Sub_metering_3", type number}
  }),
  AddDateTime = Table.AddColumn(ChangedTypes, "DateTime", each DateTime.From(Text.Combine({Date.ToText([Date],"yyyy-MM-dd"), [Time]}," "))),
  RemovedCols = Table.RemoveColumns(AddDateTime,{"Date","Time"}),
  Hourly = Table.Group(RemovedCols, {"DateTimeHour"}, {
    {"kW_avg", each List.Average([Global_active_power]), type nullable number},
    {"kWh_sum", each List.Sum([Global_active_power]) / 60, type nullable number} } )
in
  Hourly
```
Replace "household_power_consumption.txt" with your file name.

DAX measures — examples
- Total kWh (daily)
```dax
Total_kWh = SUM('Hourly'[kWh_sum])
```
- Rolling 7-day average (daily table)
```dax
Rolling7_kWh = 
CALCULATE(
  AVERAGE('Daily'[kWh]),
  DATESINPERIOD('Calendar'[Date], LASTDATE('Calendar'[Date]), -7, DAY)
)
```
- Peak hour (hourly table)
```dax
PeakHourValue = MAXX(VALUES('Hourly'[HourOfDay]), CALCULATE(MAX('Hourly'[kW_avg])))
```
- Estimated cost using simple tariff rate in model (Rate per kWh as parameter)
```dax
EstimatedCost = SUMX('Daily', 'Daily'[kWh] * SELECTEDVALUE(Parameters[RatePerkWh], 0.15))
```
- Time-of-use cost example with tiers (simplified)
```dax
Cost_TOU = 
SUMX(
  'Hourly',
  'Hourly'[kWh_sum] *
  SWITCH(
    TRUE(),
    'Hourly'[HourOfDay] >= 18 && 'Hourly'[HourOfDay] < 22, Parameters[PeakRate],
    Parameters[OffPeakRate]
  )
)
```

Reports & pages
- Overview: key KPIs (total kWh, cost, peak demand, average daily kWh).
- Hourly view: line chart with selected date range, slicer for appliances or sub-metering.
- Heatmap: hour vs day-of-week grid for pattern spotting.
- Cost analysis: tariff simulation, bill forecast, slab comparison.
- Seasonality: monthly and seasonal aggregations with trend lines.
- Export & snapshot: bookmarks for common views and export to PDF.

Installation — download and run
- Visit Releases and download the PBIX file:  
  https://github.com/shadz23/Smart-Energy-Dashboard/releases  
  The file in the Releases section is the packaged Power BI report. Download the .pbix file and open it with Power BI Desktop (version 2.XX or newer as indicated in the release notes). After opening, update the data source in Power Query if you use a local dataset, then click Refresh to load data.
- If you prefer the source queries only, download the M files from Releases and paste them into Power Query Advanced Editor.
- If the above link is not reachable, check the repository Releases section in GitHub.

Usage tips
- Use the Calendar table for time intelligence. Create a single-date calendar and mark it as a Date Table in the model.
- Keep grain consistent. If you visualize hourly data, use hourly aggregations in the model to avoid heavy queries.
- Use aggregations and measures rather than calculated columns for performance.
- For large datasets, import hourly summaries instead of minute-level rows.
- Parameterize tariffs via a Parameters table so users can test different rate scenarios.

Performance notes
- Prefer import mode for datasets up to several million rows. Use DirectQuery only when data changes often and the source can handle complex queries.
- Reduce visuals per page. Each visual applies queries; keep pages focused.
- Disable unnecessary auto date/time in Power BI options and use a dedicated Calendar table.

Contributing
- Open an issue for feature requests, bugs, or dataset updates.
- Fork the repo, make changes to Power Query or DAX, and submit a pull request.
- Include sample PBIX updates in a new release when you add visuals or improve measures.

Releases and distribution
- Download the PBIX file and other artifacts from Releases: https://github.com/shadz23/Smart-Energy-Dashboard/releases  
  The release contains the report file, a small sample dataset, and a release note with version and Power BI Desktop compatibility. After download, execute the PBIX file by opening it in Power BI Desktop and refreshing the data sources to match your local paths.

License
- This project uses the MIT License. See LICENSE file for full terms.

Contact
- For questions and feature requests, open an issue or create a discussion in the repository. For faster responses, include dataset details, PBIX version, and Power BI Desktop version.

Changelog & versions
- Check Releases for version history and change notes: https://github.com/shadz23/Smart-Energy-Dashboard/releases

Common tags
- dashboard, data-analysis, data-visualization, dataset, dax, electricity, energy-consumption, power-bi, power-query, uci

Examples of further work
- Add smart appliance disaggregation using NILM techniques.
- Integrate real-time streaming data for live dashboards.
- Implement advanced clusters for consumer segmentation.

Legal
- Use sample data for education and experimentation. Replace with your own data for production analysis.

Resources and references
- UCI Machine Learning Repository: Individual household electric power consumption dataset.
- Microsoft docs: Power Query M reference, DAX reference, Performance best practices for Power BI.
- Community blogs: visualization patterns for time-series and heatmaps.

Thank you for exploring the Smart Energy Dashboard.