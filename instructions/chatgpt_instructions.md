Nice — below is a complete, **actionable, step-by-step guide** you can follow inside Power BI Desktop to build the 6-page report exactly as described. I include:

- the **exact visuals** to add,
- the **DAX measures** you’ll need (copy/paste ready),
- precise **visual configuration / formatting steps**, and
- tips for interactivity (slicers, drillthrough, tooltips, bookmarks).

I assume you already applied the model we agreed on:
`GDP_Fact`, `CPI_Fact`, `PAT_Fact`, `Date`, `Sector`, `Company` and the basic measures (`Total GDP`, `Total PAT`, `CPI YoY %`, `GDP YoY %`).
If any measure below already exists with another name, use that instead.

---

# PREPARATION (global)

1. Open your `Nigeria_Economic_Analysis.pbix`.

2. In **Model** view confirm relationships:

   - Date[Year] → GDP_Fact[Year], CPI_Fact[Year], PAT_Fact[Year]
   - Sector[Sector] → GDP_Fact[Sector]
   - Company[Company] → PAT_Fact[Company]; Company[Sector] links to Sector

3. Create a report page size and theme:

   - View → Page View → **Actual Size**
   - Format pane (page) → Canvas settings:

     - Type: **Canvas**
     - Page size: **16:9** (or A4 Landscape if you’ll export PDF)
     - Background color: set a rectangle later for gradient; for now set page background to dark blue `#0F2B4C` at 100% (we’ll add gradient look with a shape).

4. Create a color palette in a small note for formatting:

   - Nigeria Green `#008751`
   - Professional Blue `#1B4F72`
   - Accent Gold `#F39C12`
   - Positive Green `#27AE60`
   - Negative Red `#E74C3C`
   - Neutral Gray `#95A5A6`

---

# KEY DAX MEASURES (create in \_Measures table)

Create a new measure table `_Measures` (Modeling → New Table → `_Measures = {1}` then add measures).

Copy / create these measures exactly:

```DAX
-- Total GDP (generic)
Total GDP := SUM( GDP_Fact[GDP_Value] )

-- Total GDP in a specific year
Total GDP Selected Year :=
CALCULATE( [Total GDP], ALL( Date ), Date[Year] = MAX( Date[Year] ) )

-- GDP YoY % (responds to context)
GDP YoY % :=
VAR Curr = [Total GDP Selected Year]
VAR Prev =
    CALCULATE( [Total GDP], ALL( Date ), Date[Year] = MAX( Date[Year] ) - 1 )
RETURN
IF( ISBLANK( Prev ), BLANK(), DIVIDE( Curr - Prev, Prev ) )

-- GDP CAGR (2015 -> selected year) — useful for "Average GDP Growth"
GDP CAGR (2015 to Selected) :=
VAR StartYear = 2015
VAR EndYear = MAX( Date[Year] )
VAR ValStart = CALCULATE( [Total GDP], ALL( Date ), Date[Year] = StartYear )
VAR ValEnd = CALCULATE( [Total GDP], ALL( Date ), Date[Year] = EndYear )
VAR Years = EndYear - StartYear
RETURN
IF( OR( ISBLANK( ValStart ), Years <= 0 ), BLANK(),
    ( POWER( DIVIDE( ValEnd, ValStart ), 1.0 / Years ) - 1 )
)

-- Total GDP for 2024 (explicit)
Total GDP 2024 :=
CALCULATE( [Total GDP], ALL( Date ), Date[Year] = 2024 )

-- Sector 10Y Growth % (per sector)
Sector GDP Growth 10Y % :=
VAR Start = CALCULATE( [Total GDP], ALL( Date ), Date[Year] = 2015 )
VAR End = CALCULATE( [Total GDP], ALL( Date ), Date[Year] = 2024 )
RETURN
IF( ISBLANK( Start ), BLANK(), DIVIDE( End - Start, Start ) )

-- Sector GDP Contribution % (for selected year)
Sector Contribution % :=
DIVIDE(
    [Total GDP],
    CALCULATE( [Total GDP], ALL( Sector ) )
)

-- 2024 Sector Contribution %
Sector Contribution 2024 % :=
DIVIDE(
    CALCULATE( [Total GDP], Date[Year] = 2024 ),
    CALCULATE( [Total GDP], ALL( Date ), Date[Year] = 2024 )
)

-- Best performing sector name by 10y growth (text)
Best Sector 10Y :=
VAR TopSector =
    TOPN( 1,
        ADDCOLUMNS( VALUES( Sector[Sector] ),
            "Growth", CALCULATE( [Sector GDP Growth 10Y % ] )
        ),
        [Growth], DESC
    )
RETURN
CONCATENATEX( TopSector, Sector[Sector], ", " )

-- CPI measures (per earlier guidance)
CPI YoY % :=
VAR Curr = AVERAGE( CPI_Fact[CPI_Index] )
VAR Prev = CALCULATE( AVERAGE( CPI_Fact[CPI_Index] ), ALL( Date ), Date[Year] = MAX( Date[Year] ) - 1 )
RETURN IF( ISBLANK( Prev ), BLANK(), DIVIDE( Curr - Prev, Prev ) )

-- Total PAT (generic)
Total PAT := SUM( PAT_Fact[PAT_Millions] )

-- PAT YoY %
PAT YoY % :=
VAR Curr = [Total PAT]
VAR Prev = CALCULATE( [Total PAT], ALL( Date ), Date[Year] = MAX( Date[Year] ) - 1 )
RETURN IF( ISBLANK( Prev ), BLANK(), DIVIDE( Curr - Prev, Prev ) )

-- Resilience score (example)
Sector Resilience Score :=
0.4 * [Sector GDP Growth 10Y %] +
0.3 * ( 1 - COALESCE( [CPI YoY %], 0 ) ) +
0.3 * COALESCE( [PAT YoY %], 0 )
```

Notes:

- These measures are intentionally generic and context-aware. In visuals scoped to a Sector they compute per-sector; scoped to the whole model they compute global numbers.
- If your measures already exist, adapt names.

---

# PAGE-BY-PAGE: EXACT BUILD STEPS & FORMATTING

Use the same pattern for each page: add visuals, bind fields, configure axes/formatting, style.

---

## PAGE 1 — TITLE & EXECUTIVE SUMMARY

1. **Create new page** → rename `01_Executive Summary`.

2. **Canvas gradient background**

   - Insert → Shapes → Rectangle. Resize to full canvas.
   - Format shape → Fill → Gradient (if not available, use two overlapping rectangles with PNG gradient background you can import).

     - Gradient colors: from `#0F2B4C` (dark blue) to `#1B4F72` (lighter blue)

   - Send to back (right-click → Send to back)
   - Set transparency to ~5–10% on top rectangle to let text be readable.

3. **Title block (full width)**

   - Insert → Text box. Enter:

     - Line 1 (title): **Nigeria Economic Analysis 2015-2024** — font: Segoe UI Bold, 34pt, White
     - Line 2 (subtitle): _Performance Review & Strategic Recommendations_ — Segoe UI, 16pt, Light Gray
     - Line 3 (meta): _Prepared by: [Your Name] • Date: <today> • Tools: Power BI, Python/Excel_ — 11pt, Gray

   - Align left or center as desired. Add slight drop shadow: format → Effects → Shadow (offset 2, blur 4, transparency 30%).

4. **Executive Summary Cards (4 across)**

   - Insert four **Card** visuals (or use KPI/Smart Narrative for styling).
   - Place in a single row across page.
   - Fields & formatting:

     - Card 1: Value = `[Total GDP 2024]`

       - Display units: show as **Trillions** or set label to "Naira (T)".
       - Format → Data label: Precision 1, Display units: None if you pre-convert to trillions in model OR use `Format` for `X,###` and include a subtitle text box with "Total GDP (Naira Trillions)".

     - Card 2: Value = `GDP CAGR (2015 to Selected)` — Format as `Percent`, 1 decimal.

       - If you want the 3.2% style, use the CAGR measure and format as percent with 1 decimal.

     - Card 3: Value = `[CPI YoY %]` — Format percentage with 1 decimal (e.g., 33.24%).
     - Card 4: Value = `[Best Sector 10Y]` — Use a **multi-row card** or **Card with category label**. If the measure is text, drag it into a Card (text will show; if it doesn't, use a small table with measure as a value).

   - Card visual style:

     - Background: semi-transparent white (10%).
     - Border radius via formatting: turned on if available, otherwise use a rounded shape behind the card.
     - Add small icons on left: insert image for targets (download a small icon) or use unicode (✓).

5. **Key Insights (3 bullet boxes)**

   - Insert → Text box. Create 3 narrow text boxes beneath cards:

     - Box 1: "Nigeria's economy grew from 69.2T to 79.3T (14.6%) despite significant challenges"

       - If you prefer dynamic numbers, create measures for `GDP_2015` and `GDP_2024` and use a **Card + text**. But plain text is fine if these numbers are final.

     - Box 2: "Financial services and ICT emerged as star sectors with 132% and 82% growth respectively"
     - Box 3: "Oil & Gas sector declined 34%, signaling urgent need for economic diversification"

   - Formatting: white text, medium weight, small icon bullets (use small circles or PNG icons). Keep the text concise.

6. **Small Area Chart: GDP trend 2015–2024**

   - Visual: **Area chart**
   - Axis: `Date[Year]`
   - Values: `Total GDP`
   - Configuration:

     - X-axis: categorical, show labels 2015–2024
     - Y-axis: display units = Billions/Trillions depending on your data (set in visual Format → Y axis → Display units)
     - Data colors: `#27AE60` (green) for area fill; outline `#1B4F72`.
     - Effects: turn on markers if desired.

   - Place to the right or below the key insights.

7. **Nigeria flag / watermark**

   - Insert → Image → small Nigerian flag PNG, place subtly in corner, set transparency to ~30%.

8. **Interactions**

   - No slicers needed on this page, but you can add year slicer if you want dynamic executive numbers.

---

## PAGE 2 — GDP PERFORMANCE DEEP DIVE

1. Rename page `02_GDP Performance`.

2. **Top KPI row (5 cards)**

   - Add five Cards across top:

     1. Current GDP — use `[Total GDP Selected Year]`
     2. YoY Growth % — use `[GDP YoY %]`
     3. Best Growth Sector — create measure `Best Sector 10Y` (already added)
     4. Worst Growth Sector — create `Worst Sector 10Y` similarly:

        ```DAX
        Worst Sector 10Y :=
        VAR Bottom = TOPN(1, ADDCOLUMNS(VALUES(Sector[Sector]),"Growth",CALCULATE([Sector GDP Growth 10Y %])), [Growth], ASC)
        RETURN CONCATENATEX(Bottom, Sector[Sector], ", ")
        ```

     5. Avg Sector Growth — use `AVERAGEX( VALUES( Sector[Sector] ), [Sector GDP Growth 10Y %] )` measure:

        ```DAX
        Avg Sector 10Y Growth % := AVERAGEX( VALUES( Sector[Sector] ), [Sector GDP Growth 10Y %] )
        ```

   - Format cards uniformly: white text for titles, large numbers, unit labels in subtitle.

3. **Main Visual 1 — Stacked Area: GDP by Major Sector Over Time**

   - Visual: **Stacked area chart**
   - Axis: `Date[Year]`
   - Legend: `Sector[Sector]`
   - Values: `Total GDP` (it will aggregate per sector because Sector in legend)
   - Filter: limit sectors to top 6 by contribution (use visual level filter or create a Top N filter: Sector[Sector] Top N 6 by `Total GDP` for 2024). Steps:

     - Visual → Filters pane → add Sector[Sector] → Advanced filtering → Top N → show items: Top 6 by measure `Total GDP 2024`.

   - Formatting:

     - Data colors: assign distinct colors (use the palette defined).
     - X-axis: continuous years, tick marks every year.
     - Y-axis: Display units = Billions; Title = "GDP (Naira, Billions)"
     - Tooltip: show Sector, Year, Value, Contribution%.

4. **Main Visual 2 — Horizontal Bar: 2024 GDP Contribution by Sector**

   - Visual: **Stacked bar/Clustered bar** but set orientation to horizontal: use **Bar chart** and sort descending.

   - Axis: `Sector[Sector]`

   - Value: `Total GDP` but add a measure `Sector Contribution 2024 %` as second value or use data label to show percent.

   - Format:

     - Sort descending by `Total GDP` for 2024.
     - Data labels on: Display units as percentage for `Contribution` or show both value and percentage in tooltip.
     - Conditional coloring by performance (growth vs decline). Create a measure `Sector Growth Positive = IF( [Sector GDP Growth 10Y %] > 0, 1, 0 )` and place in Format → Data colors → show rules: map 1 → `#27AE60`, 0 → `#E74C3C`.
     - If Power BI doesn't allow dynamic color rules on bar chart directly, use the field `Sector Growth Positive` in the 'Color saturation' or use the `Switch` measure approach returning HEX color and use the **Field formatting** (or use Charticulator/custom visual). Simpler: color all positive green, negatives red using two measures (Positive Value, Negative Value).

   - Example two‐measure approach:

     - Create:

       ```DAX
       Sector GDP Positive 2024 := IF( [Sector GDP Growth 10Y %] >= 0, CALCULATE([Total GDP], Date[Year]=2024), BLANK() )
       Sector GDP Negative 2024 := IF( [Sector GDP Growth 10Y %] < 0, CALCULATE([Total GDP], Date[Year]=2024), BLANK() )
       ```

     - Plot both as stacked bars (so each sector shows only one color).

5. **Bottom Section — Line Chart: YoY Growth Rate by Top 5 Sectors**

   - Visual: **Line chart**
   - Axis: `Date[Year]`
   - Legend: `Sector[Sector]` (limit to Top 5 by 10Y growth or contribution)
   - Values: measure `[GDP YoY %]` — this will evaluate in context of sector × year
   - Add **constant line** at 0%: Format → Analytics → Add constant line 0, style dashed and color gray.
   - Markers: On; Data labels off for readability, tooltip on shows exact percent.

6. **Text Box (Bottom Right)**

   - Insert → Text box with the insight line; style font white, italic, small.

---

## PAGE 3 — SECTOR PERFORMANCE MATRIX

1. Rename page `03_Sector Matrix`.

2. **Left: Scatter Plot (Big)**

   - Visual: **Scatter chart**
   - X axis: `Sector Contribution 2024 %` (measure)
   - Y axis: `Sector GDP Growth 10Y %` (measure)
   - Details: `Sector[Sector]`
   - Size: `Total GDP` (or `CALCULATE([Total GDP], Date[Year]=2024)`)
   - Color saturation: `Sector` (use categorical color palette)
   - Quadrants:

     - Insert 2 shapes (lines) crossing the scatter: vertical line at median or specific pivot (e.g., x = 0.10 → 10% contribution), horizontal at 0% growth. If you need interactive quadrant shading, use background shape images. Simpler: add two reference lines:

       - Format → Analytics → add constant line for X (not available on scatter). Alternative is to add axis constant labels via textbox overlays.

   - Tooltip: show Sector, 2015 GDP, 2024 GDP, 10y Growth, Contribution%.

3. **Bottom: Table / Matrix**

   - Visual: **Matrix**
   - Rows: `Sector[Sector]`
   - Columns: static columns (you’ll use measures):

     - 2015 GDP: measure:

       ```DAX
       GDP 2015 :=
       CALCULATE( [Total GDP], ALL( Date ), Date[Year] = 2015 )
       ```

     - 2024 GDP: `Total GDP 2024` but per sector context (it will naturally filter)
     - Total Growth %: `Sector GDP Growth 10Y %`
     - 2024 Contribution %: `Sector Contribution 2024 %`
     - Avg YoY %: measure:

       ```DAX
       Sector Avg YoY % := AVERAGEX( VALUES( Date[Year] ), [GDP YoY %] )
       ```

   - Conditional formatting: Matrix → format → Conditional formatting for each measure → Background color scale:

     - Growth %: red→white→green; set min and max values (e.g., -1 to 2 or percentages accordingly).
     - Contribution %: grayscale.

4. **Left Slicer Panel**

   - Add vertical slicers:

     - Year range: use `Date[Year]` as a **Between** slicer (set slicer type to between).
     - Sector: `Sector[Sector]` multi-select list.

   - Format slicers as dropdown or tile style with white background and rounded corners.
   - Sync slicers if you want them across pages: View → Sync slicers → select pages.

---

## PAGE 4 — INFLATION & CPI ANALYSIS

1. Rename page `04_CPI`.

2. **Header KPIs (4 cards)**

   - Card 1: `CPI YoY %` (current year)
   - Card 2: Highest inflation component — create measure:

     ```DAX
     Highest CPI Component :=
     TOPN(1, ADDCOLUMNS( VALUES(CPI_Fact[Component]), "CompInfl", CALCULATE([CPI YoY %]) ), [CompInfl], DESC )
     ```

     Use `CONCATENATEX` similar to best sector to show the name.

   - Card 3: Lowest inflation component — similar measure with ASC sorting.
   - Card 4: Food inflation rate — filter `CPI_Fact[Component] = "Food"` and compute YoY:

     ```DAX
     Food CPI YoY % := CALCULATE( [CPI YoY %], CPI_Fact[Component] = "Food" )
     ```

3. **Main Visual 1 — Combo Chart: Overall Inflation vs Food Inflation**

   - Visual: **Line and clustered column chart** (Combo chart)
   - Shared axis: `Date[Year]`
   - Column values: `AVERAGE( CPI_Fact[CPI_Index] )` or use a measure `CPI Index Selected`
   - Line values: `CPI YoY %` and `Food CPI YoY %` (two lines possible; choose separate series)
   - Secondary axis: set inflation % to secondary axis so the index columns are on primary and inflation on secondary.
   - Highlight 2023-2024: Format → Data colors → set data point color override for those years, or add a shape overlay.

4. **Main Visual 2 — Heat Map Matrix: CPI Component Growth Over Time**

   - Visual: **Matrix** where:

     - Rows: `CPI_Fact[Component]`
     - Columns: `Date[Year]`
     - Values: `CPI Yearly Index` but we want YoY % growth; create measure:

       ```DAX
       CPI Component YoY % :=
       CALCULATE( [CPI YoY %], ALLEXCEPT(CPI_Fact, CPI_Fact[Component]) )
       ```

     - Format cells with conditional formatting: color scale red(high) → green(low). Matrix supports background color by measure.

   - Alternatively use a **Heatmap custom visual** from marketplace (e.g., "HTML Viewer" trick) but matrix conditional formatting works.

5. **Bottom Left — Waterfall: Contributors to 2024 Inflation**

   - Visual: **Waterfall chart**
   - Category: `CPI_Fact[Component]`
   - Values: contribution amount — compute each component's % change from previous year × weight? Simpler: compute index change from previous year and scale by weight if you have CPI weights. If not available, show index change:

     - Measure for change:

       ```DAX
       CPI Index Change :=
       VAR Curr = CALCULATE( AVERAGE( CPI_Fact[CPI_Index] ), Date[Year] = MAX( Date[Year] ) )
       VAR Prev = CALCULATE( AVERAGE( CPI_Fact[CPI_Index] ), Date[Year] = MAX( Date[Year] ) - 1 )
       RETURN Curr - Prev
       ```

   - Use that as value; add "Total" at the end automatically shown by Waterfall.

6. **Bottom Right Text Box**

   - Insert insight text.

7. **Timeline small multiples**

   - Visual: Small multiples line charts (Power BI has Small multiples):

     - Visual: Line chart, drag `CPI_Fact[Component]` into Small multiples field and filter the components to Food, Transport, Housing, Healthcare.
     - Axis: Date[Year], Values: CPI Index or CPI YoY %.

---

## PAGE 5 — GDP-CPI-PAT CORRELATION ANALYSIS

1. Rename `05_Correlation`.

2. **Top Left — Dual-Axis: GDP Growth (sector) vs CPI (component)**

   - Visual: Line chart with dual axis support (use combo if needed).

   - Axis: Date[Year]

   - Values:

     - Left axis: `GDP YoY %` (filtered to selected sector via slicer)
     - Right axis: `CPI YoY %` (filtered to matching CPI component via another slicer or a mapping table)

   - Add slicers:

     - Sector (to select sector)
     - CPI component (user selects corresponding CPI component to compare)

   - Add measure `Correlation` if you want to compute correlation in DAX (approximate):

     ```DAX
     Correlation GDP vs CPI :=
     VAR tbl =
       SUMMARIZE(
         FILTER( ALL( Date ), Date[Year] >= 2015 && Date[Year] <= 2024 ),
         Date[Year],
         "g", CALCULATE( [GDP YoY % ] ),
         "c", CALCULATE( [CPI YoY % ] )
       )
     VAR cov =
       SUMX( tbl, ( [g] - AVERAGEX(tbl, [g]) ) * ( [c] - AVERAGEX(tbl, [c]) ) ) / ( COUNTROWS(tbl) - 1 )
     VAR sdg = SQRT( SUMX(tbl, ( [g] - AVERAGEX(tbl, [g]) ) ^ 2 ) / ( COUNTROWS(tbl) - 1 ) )
     VAR sdc = SQRT( SUMX(tbl, ( [c] - AVERAGEX(tbl, [c]) ) ^ 2 ) / ( COUNTROWS(tbl) - 1 ) )
     RETURN IF( sdg * sdc = 0, BLANK(), DIVIDE( cov, sdg * sdc ) )
     ```

   - Show Correlation score in a card nearby.

   > Note: DAX correlation is compute-heavy but OK for 10 years. If slow, use R/Python visuals.

3. **Top Right — Sector KPI Cards**

   - Cards using measures in context of selected sector:

     - Sector GDP Growth %
     - Related CPI YoY %
     - Correlation measure
     - Resilience rating (`Sector Resilience Score`)

4. **Bottom Left — Clustered Column: PAT Growth by Company**

   - Axis: `Date[Year]`
   - Legend: `Company[Company]`
   - Values: `Total PAT`
   - Format: Show negative values clearly (reference line at 0). Data labels off for many companies.

5. **Bottom Right — Combo: Company PAT vs Sector GDP**

   - Visual: Combo chart (line + clustered column)
   - Columns: `Sector Total GDP` (value computed for the sector the company belongs to)
   - Line: `Company PAT` (the selected company from slicer)
   - Add company slicer to select company; make sure the Company slicer syncs to the PAT visual only.
   - Configure axis: Company PAT (right axis), Sector GDP left axis.

---

## PAGE 6 — INVESTMENT RECOMMENDATIONS & CONCLUSIONS

1. Rename `06_Investment`.

2. **Top Left — Investment Scorecard Table**

   - Create measures per company:

     - `Company 10Y Growth %` (same as sector but filtered by company)
     - `Inflation Resilience` (derive using PAT growth vs CPI)
     - `Risk Level` (a calculated column or measure mapping conditions)
     - `Recommendation` (calculated measure returning text "Invest","Caution","Avoid" based on thresholds)

   - Visual: Table — Columns: Company[Company], Company 10Y Growth %, Inflation Resilience, Risk Level, Recommendation.
   - Conditional formatting: Add icons using field formatting (Unicode or images) mapped by Recommendation.

   Example Recommendation measure:

   ```DAX
   Recommendation :=
   VAR g = [Company 10Y Growth %]
   VAR r = [Company Resilience Score] -- you create it similarly
   RETURN
   SWITCH(TRUE(),
       g >= 0.5 && r >= 0.6, "Invest",
       g >= 0 && r >= 0.4, "Caution",
       "Avoid"
   )
   ```

3. **Middle — Gauge Charts for Top 3 Companies**

   - Use Gauge visual for each company with measure `Company Resilience Score` scaled 0–100 (multiply by 100 if needed).
   - Formatting: set zones: Red 0–40, Yellow 40–70, Green 70–100.

4. **Right — Donut Charts**

   - Donut 1: Recommended sectors by GDP contribution — Category: Sector[Sector], Value: `IF( [Recommendation] = "Invest", [Total GDP 2024], BLANK() )`.
   - Donut 2: Risk-adjusted sector allocation — create weights via measure and plot similarly.

5. **Bottom — Policy Cards**

   - Insert three text boxes styled as cards, color-coded:

     - Urgent Priority (red border): Oil & Gas revival...
     - Leverage Strengths (green border): ICT, Financial...
     - Inflation Control (orange border): Food supply-chain...

   - Add small icons (insert image).

6. **Final Summary Text Box**

   - Insert a large text box with the final summary statement.

---

# INTERACTIVITY, TOOLTIP & NAVIGATION

1. **Slicers**

   - Add Year slicer (between) and Sector slicer to pages that need them.
   - Synchronize slicers:

     - View → Sync slicers → choose which pages to sync.

2. **Drillthrough**

   - Create a `Sector Details` page and add drillthrough field `Sector[Sector]`.
   - On any sector visual, right-click → Drillthrough → Sector Details.

3. **Tooltips**

   - Create a tooltip page (size tooltip) showing small sparkline, 3 KPIs; set the page’s Tooltip property On.
   - For visuals, Format → Tooltip → Report page → choose your tooltip page.

4. **Bookmarks & Buttons**

   - Create bookmarks for main views (e.g., Executive Summary, GDP Deep Dive, CPI Deep Dive).
   - Add buttons with actions to navigate between pages:

     - Insert → Buttons → Blank. Format → Action → Page navigation → select page.

5. **Export settings**

   - File → Export → PDF. Adjust page order and set export quality.

---

# FORMATTING / POLISH DETAILS (follow for all pages)

- Fonts: Segoe UI for headers and body; bold headers.
- Card shadows: emulate by inserting slightly larger rectangle behind card with 5% black transparency offset.
- Align visuals: use View → Snap to grid and align tools.
- Use consistent number formatting:

  - GDP numbers: show value in Trillions or Billions with `Display units`.
  - Percentages: 1 decimal place.

- Use white/light grey text on dark header backgrounds; dark text on white cards.

---

# PERFORMANCE TIPS

- Avoid heavy DAX in visuals repeated across many sectors; pre-filter top lists in the filter pane.
- If correlation DAX is slow, compute correlation in Power Query (once) or use an R visual.
- When using Top N filters (Top 5/6), set them at visual level, not at the query level.

---

# Quick checklist to finish & verify

- [ ] All measures created and validated (compare manually for a couple of sectors)
- [ ] Date table has years 2015–2024 and is linked
- [ ] Sector and Company dimensions filled and linked
- [ ] All visuals placed and formatted on each page
- [ ] Slicers working and synced as intended
- [ ] Tooltips & Drillthrough configured
- [ ] Bookmarks + Navigation buttons created

---

If you want I can now:

- Produce the **exact JSON color and format settings** for each visual, or
- Export a ready-to-copy **list of all measures** in one block, or
- Build the **Correlation** measure optimized for performance (faster DAX).

Which of those next?
