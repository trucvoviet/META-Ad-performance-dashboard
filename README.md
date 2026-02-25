# Meta Ad Performance Power BI Dashboard 

## Overview
This tutorial demonstrates building a **real-time, industry-level Power BI dashboard** to analyze advertising performance across Meta platforms (Facebook and Instagram). The project uses advanced data modeling with multiple tables, complex DAX calculations, and interactive visualizations to track campaign effectiveness, audience engagement, and ROI.

## Problem Statement
**Business Objective:**
- Track advertising campaign performance on Facebook and Instagram
- Provide visibility into campaign reach, engagement, conversions, and budget utilization
- Enable marketing teams to identify the most effective platform
- Track campaign ROI and optimize budget allocations
- Understand audience engagement patterns

**Scope:**
- **In Scope:** Facebook and Instagram paid ads
- **Out of Scope:** Messenger, Audience Network, organic engagement

---

## DATASET STRUCTURE

### Source
- **Platform:** Kaggle
- **Period:** 3-4 months of real-time data (2025)
- **Format:** Multiple CSV/Excel files
- **Total Records:** 400,000+ rows

### Data Tables (5 Main Tables)

#### 1. Ad Events Table (Fact Table)
**Purpose:** Transaction table storing event-level logs

**Columns:**
- **Event ID** - Primary key (unique identifier)
- **Ad ID** - Foreign key to Ads table
- **User ID** - Foreign key to Users table
- **Timestamp** - Date and time of event
- **Day of Week** - Derived from timestamp
- **Time of Day** - Morning/afternoon/evening/night
- **Event Type** - Click, Comment, Impression, Like, Purchase, Share

**Records:** 400,000+ events

#### 2. Ads Table (Dimension Table)
**Purpose:** Details of each ad creative and targeting criteria

**Columns:**
- **Ad ID** - Primary key
- **Campaign ID** - Foreign key to Campaigns table
- **Ad Platform** - Facebook or Instagram
- **Ad Type** - Video, Story, Image, Carousel
- **Target Age Group** - 16-21, 22-30, 35-44, All
- **Target Gender** - Male, Female, All
- **Target Interest** - News, Technology, Health, Food, Finance, Lifestyle, Gaming, Fashion, Fitness, Sports, Travel, Education

**Records:** 200 ads

#### 3. Campaigns Table (Dimension Table)
**Purpose:** High-level campaign strategy and budget allocation

**Columns:**
- **Campaign ID** - Primary key
- **Campaign Name** - Campaign 1, Campaign 2, etc. (50 campaigns)
- **Start Date** - Campaign launch date
- **End Date** - Campaign completion date
- **Total Budget** - Dollar amount allocated

**Records:** 50 campaigns

#### 4. Users Table (Dimension Table)
**Purpose:** Demographic information of customers who interact with ads

**Columns:**
- **User ID** - Primary key (1,000 unique users)
- **Gender** - Male/Female
- **Age** - User age
- **Age Group** - Grouped age ranges
- **Country** - User location
- **Location** - City/region within country
- **Interest** - Food, Finance, Technology, etc.

**Records:** 1,000 users

**Important:** User interests are matched with ad interests through algorithms. What you search on Google, Amazon, Flipkart gets tracked and shown as ads on Meta platforms.

#### 5. Calendar Table (Date Dimension - Created)
**Purpose:** Date analysis and time intelligence

**Columns:**
- **Date** - Primary key (unique dates)
- **Month Name** - Jan, Feb, Mar, etc.
- **Day Name** - Mon, Tue, Wed, etc.
- **Day Number** - 1-31
- **Week Day** - 1-7 (Monday=1, Sunday=7)
- **Week Number** - Week of year

**Created from:** Event Date (extracted from Ad Events timestamp)

---

## DATA MODEL (Star Schema/Snowflake)

### Relationships

**Core Star Schema:**
```
Calendar Table (1) ----< Ad Events (Many)
Users (1) ----< Ad Events (Many)
Ads (1) ----< Ad Events (Many)
Campaigns (1) ----< Ads (Many)
```

**Relationship Details:**

1. **Calendar ↔ Ad Events**
   - Join: Date (Calendar) ↔ Event Date (Ad Events)
   - Cardinality: One-to-Many
   - Filter Direction: Single

2. **Users ↔ Ad Events**
   - Join: User ID (Users) ↔ User ID (Ad Events)
   - Cardinality: One-to-Many
   - Filter Direction: Single

3. **Ads ↔ Ad Events**
   - Join: Ad ID (Ads) ↔ Ad ID (Ad Events)
   - Cardinality: One-to-Many
   - Filter Direction: Single

4. **Campaigns ↔ Ads**
   - Join: Campaign ID (Campaigns) ↔ Campaign ID (Ads)
   - Cardinality: One-to-Many
   - Filter Direction: Single

**Schema Type:** Snowflake Schema (Fact table connected to dimensions, with one dimension connected to another)

**Why Data Modeling is Critical:**
- Incorrect joins = Incorrect numbers
- Wrong filter directions = Wrong insights
- Bad relationships = Business impact
- Always cross-check with SQL queries
- Data validation is mandatory

---

## DATA PREPARATION

### Step 1: Import Data

**Load CSV Files:**
```
Home > Get Data > Text/CSV
```

**Import order:**
1. Ad Events (fact table first)
2. Ads
3. Campaigns
4. Users

**Critical Import Setting:**
When opening Ad Events table, if prompted to convert:
- **ALWAYS click "Don't Convert"**
- Prevents ID column corruption
- Maintains data integrity
- Essential for proper relationships

### Step 2: Verify Data Types

**Power Query Editor:**
```
Home > Transform Data
```

**Check each table:**
- **Timestamp** → Date/Time
- **Start Date/End Date** → Date
- **IDs** → Text (not numbers)
- **Budget** → Decimal Number

### Step 3: Data Quality Check

**Enable Quality Indicators:**
```
View > Column Quality
View > Column Distribution
```

**Validation Rules:**
- **Valid:** Must be 100%
- **Error:** Must be 0%
- **Empty:** Acceptable (business dependent)

**Check Uniqueness:**
- Primary keys: Distinct = Unique
- Foreign keys: Distinct < Unique (repetitive values expected)

**Example:**
- Event ID: 1,000 distinct, 1,000 unique → Primary key ✓
- Ad ID: 200 distinct, 8 unique → Foreign key ✓

### Step 4: Create Calculated Columns

#### Extract Event Date (from Timestamp)

**Location:** Ad Events table

```DAX
Event Date = DATEVALUE('Ad Events'[Timestamp])
```

**Purpose:** Extract pure date (remove time component)

**Data Type:** Change to Date format

#### Extract Event Hour

```DAX
Event Hour = HOUR('Ad Events'[Timestamp])
```

**Result:** 0-23 (hour of day)

### Step 5: Create Calendar Table

**Purpose:** Microsoft recommends dedicated date tables for time intelligence

**DAX Formula:**

```DAX
Calendar Table = 
CALENDAR(
    MIN('Ad Events'[Event Date]),
    MAX('Ad Events'[Event Date])
)
```

**Result:** Creates continuous date range from minimum to maximum event date

**Example:**
- Min Date: May 7, 2025
- Max Date: August 6, 2025
- Table: All dates between these (no gaps)

#### Add Calendar Columns

**Month Name:**
```DAX
Month Name = FORMAT('Calendar Table'[Date], "MMM")
```
Result: Jan, Feb, Mar

**Day Name:**
```DAX
Day Name = FORMAT('Calendar Table'[Date], "DDD")
```
Result: Mon, Tue, Wed

**Day Number:**
```DAX
Day Number = FORMAT('Calendar Table'[Date], "D")
```
Result: 1-31

**Week Day:**
```DAX
Week Day = WEEKDAY('Calendar Table'[Date], 2)
```
- Parameter 2 = Monday as week start
- Result: 1=Monday, 7=Sunday

**Week Number:**
```DAX
Week Number = WEEKNUM('Calendar Table'[Date], 2)
```
Result: 1-52 (week of year)

**Sort Day Name by Week Day:**
```
Select Day Name column > Sort by Column > Week Day
```
Result: Displays Mon, Tue, Wed (not alphabetically)

---

## DAX MEASURES (KEY PERFORMANCE INDICATORS)

### Core Metrics

#### 1. Impressions

**Definition:** Number of times ads displayed to users

```DAX
Impressions = 
COUNTROWS(
    FILTER(
        'Ad Events',
        'Ad Events'[Event Type] = "Impression"
    )
)
```

**Logic:**
- Filter Ad Events for Event Type = "Impression"
- Count all matching rows

#### 2. Clicks

**Definition:** Number of times users clicked ads (engagement intent)

```DAX
Clicks = 
COUNTROWS(
    FILTER(
        'Ad Events',
        'Ad Events'[Event Type] = "Click"
    )
)
```

#### 3. Shares

**Definition:** Number of times ads shared (viral engagement)

```DAX
Shares = 
COUNTROWS(
    FILTER(
        'Ad Events',
        'Ad Events'[Event Type] = "Share"
    )
)
```

#### 4. Comments

**Definition:** User sentiment and feedback

```DAX
Comments = 
COUNTROWS(
    FILTER(
        'Ad Events',
        'Ad Events'[Event Type] = "Comment"
    )
)
```

#### 5. Purchases

**Definition:** Actual conversions (buying product from brand)

```DAX
Purchases = 
COUNTROWS(
    FILTER(
        'Ad Events',
        'Ad Events'[Event Type] = "Purchase"
    )
)
```

#### 6. Engagements

**Definition:** Total engagement volume (clicks + shares + comments)

```DAX
Engagements = 
[Clicks] + [Shares] + [Comments]
```

**Why important:** Shows user interest in ads

### Calculated Rates

#### 7. Click-Through Rate (CTR)

**Definition:** Percentage of impressions resulting in clicks (ad effectiveness)

```DAX
CTR = 
DIVIDE(
    [Clicks],
    [Impressions],
    0
)
```

**Format:** Percentage (Measure Tools > Format > Percentage)

**Example:**
- Clicks: 25,000
- Impressions: 216,000
- CTR: 11%

#### 8. Engagement Rate

**Definition:** Percentage of impressions resulting in engagement (ad appeal)

```DAX
Engagement Rate = 
DIVIDE(
    [Engagements],
    [Impressions],
    0
)
```

**Format:** Percentage

#### 9. Conversion Rate

**Definition:** Percentage of clicks resulting in purchases (funnel efficiency)

```DAX
Conversion Rate = 
DIVIDE(
    [Purchases],
    [Clicks],
    0
)
```

**Format:** Percentage

**Why based on clicks:** Measures post-click conversion

#### 10. Purchase Rate

**Definition:** Percentage of impressions resulting in purchases (conversion from reach)

```DAX
Purchase Rate = 
DIVIDE(
    [Purchases],
    [Impressions],
    0
)
```

**Format:** Percentage

**Difference from Conversion Rate:**
- Conversion Rate: Purchases / Clicks (higher)
- Purchase Rate: Purchases / Impressions (lower)

**Example:**
- Purchase Rate: 6%
- Conversion Rate: 5%
- Purchase Rate always lower (larger denominator)

### Budget Metrics

#### 11. Total Budget

**Definition:** Total spend allocated to campaigns (cost analysis)

```DAX
Total Budget = SUM(Campaigns[Total Budget])
```

**Format:** Currency (Dollar symbol, 1 decimal)

#### 12. Average Budget Per Campaign

**Definition:** Budget distribution across campaigns

```DAX
Average Budget Per Campaign = 
AVERAGE(Campaigns[Total Budget])
```

**Format:** Currency (Dollar symbol, 1 decimal)

### Dynamic Measures Parameter

**Purpose:** Allow users to switch between metrics dynamically

**Create Parameter:**
```
Modeling > New Parameter > Fields
```

**Select Measures:**
1. Impressions
2. Engagements
3. Clicks
4. Comments
5. Purchases
6. Shares

**Parameter Name:** `Select Dynamic Measure`

**Result:** Creates new table with parameter field

**Usage:** Used in charts to make them dynamic (changes based on user selection)

---

## DASHBOARD DESIGN

### Canvas Setup

**Page Settings:**
```
Format > Page > Canvas Settings
```

**Dimensions:**
- Width: 1570 pixels
- Height: 900 pixels
- **Why these dimensions:** Close to standard PPT size (1600×900)

**Background Color:**
- Code: `#0D1AFF` (Meta blue)
- Transparency: 80%

**Alignment:** Middle (vertical centering)

### Title Design

**Component:** Text Box
```
Insert > Text Box
```

**Content:** "Meta Ad Performance Dashboard"

**Formatting:**
- Font: Calibri
- Size: 32pt
- Weight: Bold
- Color: Default

**Position:** Top-right of canvas

### Logo

**Component:** Image
```
Insert > Images
```

**Logo:** Meta logo (blue square with white text)

**Position:** Top-left of canvas

### Left Panel (Filter Panel)

**Component:** Rectangle Shape
```
Insert > Shape > Rectangle
```

**Dimensions:**
- Height: 800 pixels
- Width: 260 pixels

**Formatting:**
- Color: `#0081FF` (Meta logo color)
- Rounded Corners: 5 pixels

**Purpose:** Container for navigation buttons and slicers

---

## KPI CARDS

### Top KPI Row (6 Metrics)

**Component:** New Card Visual (multi-card)

**Dimensions:**
- Height: 120 pixels
- Width: 1265 pixels

**Metrics Displayed:**
1. Impressions
2. Clicks
3. Shares
4. Comments
5. Purchases
6. Engagements

**Configuration:**
```
Visual > Multicard > Maximum cards shown: 6
```

**Formatting:**

**Callout Values:**
- Font: Segoe UI Semibold
- Size: 25pt
- Color: White
- Decimal Places: 1

**Labels:**
- Font: Segoe UI
- Size: 11pt
- Color: White

**Spacing:**
- Between cards: 15 pixels
- Between label and value: 8 pixels

**Card Backgrounds (Individual Colors):**

1. **Impressions:** `#5946DC` (purple)
2. **Clicks:** `#7189FF` (light blue)
3. **Shares:** `#08CFDC` (cyan)
4. **Comments:** `#14D19D` (green)
5. **Purchases:** `#CD5395` (pink)
6. **Engagements:** `#7449AE` (dark purple)

**Card Shape:**
- Type: Rounded Rectangle
- Corner Radius: 15 pixels

**Effects:**
- Background: Enabled
- Color: Per metric (see above)

### Bottom KPI Row (6 Rates/Budget)

**Component:** New Card Visual (duplicate of top row)

**Metrics Displayed:**
1. Click-Through Rate (CTR)
2. Engagement Rate
3. Conversion Rate
4. Purchase Rate
5. Total Budget
6. Average Budget Per Campaign

**Same formatting as top row**

**Color Scheme:** Same as top row (recycled colors)

**Format Rates as Percentage:**
```
Select measure > Measure Tools > Format > Percentage
```

**Format Budget as Currency:**
```
Select measure > $ symbol > Dollar format > 1 decimal
```

---

## VISUALIZATIONS

### 1. Engagement by Gender (Donut Chart)

**Component:** Donut Chart

**Placeholder:**
- Shape: Rounded Rectangle (white)
- Height: 230 pixels
- Width: 280 pixels
- Corner Radius: 8 pixels
- Border: Gray

**Data:**
- Legend: Target Gender (from Ads table)
- Values: Select Dynamic Measure (parameter)

**Formatting:**

**Slice Settings:**
- Spacing: 70%

**Values Display:**
- Show: Category, Value, Percentage
- Rotation: 90° (rotated for readability)
- Font: Segoe UI Semibold, Black
- Decimals: 0 for percentage

**Legend:** Turned off

**Dynamic Title:**

```DAX
Gender Title = 
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " by Gender"
```

**Where Dynamic Title Column:**
```DAX
Dynamic Title = 
IF('Select Dynamic Measure'[Select Measure Order] = 0, "Impressions",
IF('Select Dynamic Measure'[Select Measure Order] = 1, "Engagements",
IF('Select Dynamic Measure'[Select Measure Order] = 2, "Clicks",
IF('Select Dynamic Measure'[Select Measure Order] = 3, "Shares",
IF('Select Dynamic Measure'[Select Measure Order] = 4, "Comments",
IF('Select Dynamic Measure'[Select Measure Order] = 5, "Purchases",
"Other"))))))
```

**Title Display:**
- Font: Segoe UI Semibold, Size 12
- No background
- Position: Above chart

**Insights:**
- 44% Female target
- 35% All (both genders)
- 21% Male target

### 2. Target Age (Column Chart)

**Component:** Clustered Column Chart

**Placeholder:**
- Height: 365 pixels
- Width: 475 pixels

**Data:**
- X-Axis: User Age (from Users table)
- Y-Axis: Select Dynamic Measure

**Filter:**
```
Filters on Visual > User Age > Less Than 51
```
**Reason:** After age 50, very few users (not meaningful)

**Formatting:**

**X-Axis:**
- Title: Off
- Values: Size 8, Gray
- Range: Start at 16 (minimum age in dataset)

**Y-Axis:**
- Title: Off
- Values: Size 8, Gray
- Start: 0

**Grid Lines:**
- Horizontal: Off
- Vertical: Off

**Columns:**
- Color: Purple (consistent with theme)
- Layout Spacing: 35%

**Title:** Off (using dynamic title instead)

**Dimensions:**
- Chart Height: 160 pixels
- Chart Width: 440 pixels

**Dynamic Title:**

```DAX
Age Title = 
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " by Age"
```

**Insights:**
- Peak at age 16 (highest engagement)
- Gradual increase from 16-26
- Decline after age 30
- Lowest at age 50

**Interpretation:** Young people (16-26) use Instagram/Facebook most

### 3. Engagement by Country (Map)

**Component:** Map Chart (traditional) or Azure Map

**Important Note:**
- Traditional Map visual being retired
- Recommended: Azure Map (requires Power BI Pro)
- Traditional map won't work on Power BI Service

**Placeholder:**
- Same styling as other charts

**Data:**
- Location: Country (from Users table)
- Bubble Size: Select Dynamic Measure

**Formatting:**

**Bubble Settings:**
- Size adjustment: +10 (increase visibility)

**Title:** Off (using dynamic title)

**Dynamic Title:**

```DAX
Map Title = 
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " by Country"
```

**Alternate: Azure Map**

If available:
```
Upgrade to Azure Map (automatic conversion)
```

**Publishing Issue:**
- Traditional map won't display on Power BI Service
- Must use Azure Map for online viewing

### 4. Analysis by Month (Calendar Heat Map)

**Component:** Matrix Visual

**Placeholder:**
- Dimensions maintained from other visuals

**Data:**
- Rows: Week Number (from Calendar table)
- Columns: Day Name (from Calendar table)
- Values: Day Number (from Calendar table)

**Sort Day Name:**
```
Select Day Name column > Sort by Column > Week Day
```
**Result:** Mon, Tue, Wed... (not alphabetical)

**Month Slicer:**

**Component:** Dropdown Slicer
- Field: Month Name (from Calendar table)
- Selection: Single Select
- Style: Dropdown

**Slicer Formatting:**
- Header: Size 10, Segoe UI Semibold, White
- Values: Size 10, Black
- Background: Blue (matching panel)
- Border: White, 2 pixels, Rounded 10

**Edit Interactions:**
```
Home > Edit Interactions
```
**Action:** Set month slicer to ONLY filter calendar heat map
- Click "None" on all other visuals
- Prevents month filter affecting other charts

**Matrix Formatting:**

**Hide Week Numbers:**
- Double-click "Week Number" header
- Add space: " " (single space)
- Result: Numbers hidden, column remains

**Grid:**
- Horizontal Grid Lines: On, White, Fine
- Vertical Grid Lines: On, White, Fine
- Border: Tender Blue

**Values:**
- Font: Segoe UI Semibold
- Size: 10
- Alternate Background: White

**Column Headers:**
- Font: Segoe UI Semibold
- Size: 8
- Background: Blue

**Row Headers:**
- Font: Size reduced (to minimize space)
- Banded Row Color: Off

**Conditional Formatting:**
```
Cell Elements > Background Color > Gradient
```
- Based on: Engagements
- Lowest: Light color
- Highest: Dark color
- Don't format zero: Checked

**Critical: Data Model Connection**

**Problem:** Conditional formatting won't work if Calendar table not connected

**Solution:**
```
Create relationship:
Calendar[Date] (1) ----< Ad Events[Event Date] (Many)
```

**Without relationship:** No color gradient
**With relationship:** Color gradient appears

**Dimensions:**
- Height: 185 pixels
- Width: 275 pixels

**Dynamic Title:**

```DAX
Calendar Title = "Analysis by Month"
```

**Note:** Not dynamic (shows all metrics in tooltip)

#### Tooltip Page

**Purpose:** Show all KPIs when hovering over dates

**Create New Page:**
```
Right-click page tab > Duplicate
```

**Page Settings:**
```
Format > Page > Canvas Settings > Type: Tooltip
```

**Dimensions:**
- Width: Custom
- Height: 430 pixels

**Content:**
- Copy top KPI card (6 metrics)
- Copy bottom KPI card (6 rates)
- Arrange vertically

**KPI Card Adjustments:**
```
General > Properties > Width: 100 pixels
Visual > Layout: Vertical
Callout Values: Size 15
Labels: Size 8
Space between cards: 5
Space between labels: 0
```

**Apply Tooltip:**
```
Select Calendar Matrix > Format > Tooltip
Turn On: Tooltip
Report Page: Calendar Tool Tip
```

**Hide Tooltip Page:**
```
Right-click page > Hide
```

**Result:**
- Hover over any date in calendar
- See all KPIs for that specific date
- Interactive and informative

**Example:**
- June 5: Shows impressions, clicks, purchases, etc. for that day
- June 12: Different values for that day

### 5. Weekly Engagement Trend (Stacked Column)

**Component:** Stacked Column Chart

**Placeholder:**
- Height: 365 pixels
- Width: 475 pixels

**Data:**
- X-Axis: Week Number (from Calendar table)
- Y-Axis: Select Dynamic Measure
- Legend: Ad Type (from Ads table)

**Ad Types:** Video, Story, Image, Carousel

**Formatting:**

**X-Axis:**
- Title: Off
- Values: Size 8, Gray
- Range: Start 19, End 32 (based on available data)

**Y-Axis:**
- Title: Off
- Values: Size 8, Gray
- Start: 0

**Grid Lines:** All off

**Columns:**
- Spacing: 35%

**Colors:**
- Video: Custom color 1
- Story: Custom color 2
- (Adjust as needed for visibility)

**Legend:** Off

**Chart Dimensions:**
- Height: 160 pixels
- Width: 440 pixels

**Dynamic Title:**

```DAX
Weekly Title = 
"Weekly " & SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " Trend"
```

**Example:** "Weekly Impressions Trend", "Weekly Engagements Trend"

**Insights:**
- Week 19-32 trend visible
- Compare ad type performance week-over-week
- Identify peaks and valleys

### 6. Hourly Engagement Trend (Area Chart)

**Component:** Area Chart

**Data:**
- X-Axis: Event Hour (from Ad Events - calculated column)
- Y-Axis: Select Dynamic Measure

**Event Hour Creation:**
```DAX
Event Hour = HOUR('Ad Events'[Timestamp])
```
Result: 0-23 (hours of day)

**Formatting:**

**X-Axis:**
- Title: Off
- Values: Size 8, Gray
- Range: 0 to 23

**Y-Axis:**
- Title: Off
- Values: Size 8, Gray

**Grid Lines:** All off

**Line:**
- Width: 2 pixels
- Color: Dark purple
- Curve: Smooth (not linear)

**Shaded Area:**
- Transparency: 50%
- Color: Light purple (matching line)

**Markers:**
- Size: 3 pixels
- Enabled

**Chart Dimensions:**
- Height: 140 pixels
- Width: 450 pixels

**Dynamic Title:**

```DAX
Hourly Title = 
"Hourly " & SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " Trend"
```

**Important:** Change line color for each metric selection
- Clicks selected → Change to blue
- Purchases selected → Change to pink
- (Manual color adjustment needed)

**Insights:**
- Peak engagement hours: 6 PM - 10 PM (evening)
- Low engagement: 2 AM - 6 AM (night/early morning)
- Helps determine optimal ad scheduling times

### 7. Analysis by Ad Type (Matrix)

**Component:** Matrix Visual

**Placeholder:**
- Same styling as other charts

**Data:**
- Rows: Ad Type (from Ads table)
- Columns (Values):
  - Impressions (IM)
  - Clicks (CLK)
  - CTR
  - Purchase Rate (PR)
  - Engagement Rate (ER)
  - Conversion Rate (CR)

**Column Abbreviations:**
- Impressions → IM
- Clicks → CLK
- CTR → CTR
- Purchase Rate → PR
- Engagement Rate → ER
- Conversion Rate → CR

**Display Units:**
- Impressions: Thousands (K)
- Clicks: Thousands (K)
- Rates: Percentage

**Formatting:**

**Grid:**
- Horizontal Lines: White
- Vertical Lines: White, 3 pixels

**Column Headers:**
- Background: Blue
- Font: Segoe UI Semibold, Size 10
- Color: White

**Row Headers:**
- Font: Size 11
- Banded Row Colors: Off

**Values:**
- Font: Segoe UI Semibold, Size 10
- Alternate Background: White

**Borders:** White

**Conditional Formatting (Each Column):**

```
Specific Column > Conditional Formatting > Background Color > Gradient
```

**Color Schemes:**
1. **IM (Impressions):** Purple gradient
2. **CLK (Clicks):** Light blue gradient
3. **CTR:** Yellow gradient
4. **PR:** Pink gradient
5. **ER:** Yellow gradient
6. **CR:** Green gradient

**Adjust Columns:** Resize for proper horizontal spacing

**Dimensions:**
- Height: 130 pixels
- Width: 440 pixels

**Dynamic Title:**

```DAX
Matrix Title = 
"Analysis by Ad Type"
```

**Note:** Not dynamic (shows multiple metrics simultaneously)

**Subtotals:** All off

**Insights:**
- Compare ad type performance across metrics
- Identify best-performing format (Video, Story, Image, Carousel)
- Data-driven ad strategy decisions

---

## SLICERS (FILTERS)

### Location: Left Panel (Blue Rectangle)

### 1. Campaign Name Slicer

**Component:** Dropdown Slicer

**Data:** Campaign Name (from Campaigns table)

**Formatting:**
- Style: Dropdown
- Selection: Multi-select
- Background: Off
- Border: White, 2 pixels, Rounded 10

**Header:**
- Text: "Campaign Name"
- Font: Segoe UI Semibold, Size 10, White

**Values:**
- Font: Segoe UI Semibold, Size 10, White
- Background: Blue (matching panel)

### 2. Target Interest Slicer

**Component:** Dropdown Slicer

**Data:** Target Interest (from Ads table)

**Same formatting as Campaign Name slicer**

**Header:** "Target Interest"

**Values:** News, Technology, Health, Food, Finance, Lifestyle, Gaming, Fashion, Fitness, Sports, Travel, Education

### 3. Dynamic Measure Selector (Parameter)

**Component:** Dropdown Slicer (parameter-based)

**Data:** Select Dynamic Measure (created parameter)

**Same formatting as other slicers**

**Position:** Top of left panel

**Purpose:** Switch between metrics across all dynamic charts

---

## NAVIGATION & INTERACTIVITY

### Page Structure

**Two Separate Pages:**
1. **Facebook** - Shows Facebook ad data only
2. **Instagram** - Shows Instagram ad data only

**Why separate pages:** Better user experience than filtering on one page

### Page Filtering

**Facebook Page:**
```
Filters on Page > Ad Platform > Select: Facebook
```

**Instagram Page:**
```
Filters on Page > Ad Platform > Select: Instagram
```

**Result:** Each page automatically shows only relevant platform data

### Page Navigator

**Component:** Page Navigator Button
```
Insert > Buttons > Navigator > Page Navigator
```

**Configuration:**
- Layout: Vertical (stacked buttons)
- Shape: Rounded Rectangle, Radius 10

**Style Settings:**

**Default (Unselected):**
- Background: White
- Font: Segoe UI Semibold, Size 11, Black

**Selected:**
- Background: Black
- Font: Segoe UI Semibold, Size 11, White

**Position:** Top of left panel (above slicers)

**Buttons:**
- Facebook
- Instagram

**Interaction:**
- Click Facebook → Go to Facebook page (button turns black)
- Click Instagram → Go to Instagram page (button turns black)
- Current page button always highlighted

**Copy to Both Pages:**
```
Ctrl+C on Facebook page > Paste on Instagram page
```
Result: Navigation works on both pages

### Platform Logos

**Facebook Logo:**
- Position: Near Facebook button
- Image file provided

**Instagram Logo:**
- Position: Near Instagram button  
- Image file provided

**Copy to Both Pages:** Same as navigator

### Interactive Filters (Cross-Filtering)

**Purpose:** Click any visual element to filter entire dashboard

**Default Behavior:** Highlighting (not filtering)

**Change to Filtering:**
```
Format > Edit Interactions
```

**For Each Visual:**
1. Select source visual (e.g., Gender donut chart)
2. Click "Filter" icon on all other visuals
3. Repeat for all interactive visuals

**Visuals with Filtering:**
- Engagement by Gender
- Target Age
- Engagement by Country
- Weekly Trend
- Hourly Trend
- Analysis by Ad Type

**Example Interactions:**
- Click "Female" on gender chart → All visuals filter for female only
- Click age "16" → All visuals show age 16 data only
- Click "India" on map → All visuals show India data only
- Click "Video" in matrix → Filter for video ads only

**Month Slicer Exception:**
```
Edit Interactions > Select Month Slicer > Click "None" on all visuals EXCEPT Calendar Heat Map
```
**Reason:** Month should only control calendar, not affect other time-based trends

---

## DASHBOARD INSIGHTS

### Facebook Performance (Example Data)

**Overall Metrics:**
- **Impressions:** 216,000 (ads shown)
- **Clicks:** 25,000 (users clicked)
- **Shares:** 1,300 (viral engagement)
- **Comments:** 2,600 (user sentiment)
- **Purchases:** 1,300 (conversions)
- **Engagements:** 28,900 (total interactions)

**Calculated Rates:**
- **CTR:** 11% (click-through rate)
- **Engagement Rate:** 13%
- **Conversion Rate:** 5% (purchases / clicks)
- **Purchase Rate:** 6% (purchases / impressions)

**Budget:**
- **Total Budget:** $2.54 million
- **Average per Campaign:** Calculated value

**Key Insight:** Only 11% click, 5% convert after clicking

### Target Demographics

**Gender Distribution:**
- Female: 44% (highest target)
- All Genders: 35%
- Male: 21% (lowest target)

**Age Distribution:**
- Peak: Age 16 (highest engagement)
- Strong: Ages 16-26
- Declining: After age 30
- Minimal: Age 50+

**Interpretation:** Young users dominate platform usage

### Engagement Patterns

**Hourly Trends:**
- **Peak Hours:** 6 PM - 10 PM (evening)
- **Low Hours:** 2 AM - 6 AM (night)
- **Insight:** Schedule ads for evening hours

**Weekly Trends:**
- Consistent across most weeks
- Incomplete weeks show lower values (expected)

**Country Performance:**
- Visualized via bubble size on map
- Larger bubbles = higher engagement

### Ad Type Performance

**Matrix Analysis:**
- Compare Video, Story, Image, Carousel
- Metrics: Impressions, Clicks, CTR, Rates
- Identifies best-performing format

**Example Insights:**
- Video may have higher CTR
- Stories may have better engagement rate
- Data-driven format selection

---

## BUSINESS VALUE

### Marketing Team Benefits

**1. Platform Optimization:**
- Compare Facebook vs Instagram performance
- Identify most effective platform
- Allocate budget accordingly

**2. Campaign ROI Tracking:**
- Monitor spend vs conversions
- Calculate return on investment
- Optimize budget allocation

**3. Audience Understanding:**
- Demographic patterns (age, gender, country)
- Interest alignment
- Target the right users

**4. Timing Optimization:**
- Best hours for ad display
- Weekly patterns
- Seasonal trends

**5. Content Strategy:**
- Best-performing ad types
- Format preferences by audience
- Creative optimization

### Real-World Application

**Ad Spend Decisions:**
- Brand spends millions on ads
- Dashboard shows effectiveness
- Justifies marketing budget

**ROI Calculation:**
- $2.54M budget
- 1,300 purchases
- Average customer value determines profitability

**Initial Loss Acceptance:**
- Brands often lose money initially
- Goal: Brand recognition
- Long-term customer acquisition

**Example Scenario:**
- Company invests $3M in Meta ads
- Gets 2,000 customers
- Lifetime value of customers > $3M = Profitable

---

## PUBLISHING TO POWER BI SERVICE

### Prerequisites

**Microsoft Account:**
- Must be signed in
- Power BI Pro or Premium license (for full features)

### Publish Dashboard

**Step 1: Save File**
```
File > Save
```

**Step 2: Publish**
```
Home > Publish
```

**Step 3: Select Workspace**
- Choose: "My Workspace" (or team workspace)
- Click: Select

**Step 4: Wait for Upload**
- Progress bar appears
- Uploads all data and visuals
- May take 1-3 minutes

**Step 5: Open in Browser**
- Click: "Open in Power BI"
- Browser opens automatically
- Dashboard fully interactive online

### Share Dashboard

**Embed in Portfolio/LinkedIn:**
```
File > Embed Report > Publish to Web
```

**Important:**
- Creates public link
- Anyone with link can view
- Fully interactive online
- Update tutorial video for enabling this feature (recent 10-min video)

**Use Cases:**
- Portfolio website
- LinkedIn profile
- Resume link
- Interview presentations

**Benefit:** Interviewers see live, working dashboard (not just screenshots)

---

## ADVANCED FEATURES

### Dynamic Titles

**Concept:** Title changes based on selected measure

**Implementation:**

**Step 1: Create Helper Column**
```DAX
Dynamic Title = 
IF([Select Measure Order] = 0, "Impressions",
IF([Select Measure Order] = 1, "Engagements",
IF([Select Measure Order] = 2, "Clicks",
IF([Select Measure Order] = 3, "Shares",
IF([Select Measure Order] = 4, "Comments",
IF([Select Measure Order] = 5, "Purchases",
"Other"))))))
```

**Step 2: Unhide Parameter Fields**
- Select Measure Order (hidden by default)
- Right-click > Unhide

**Step 3: Create Title Measures**

Gender Title:
```DAX
Gender Title = 
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " by Gender"
```

Age Title:
```DAX
Age Title = 
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " by Age"
```

Map Title:
```DAX
Map Title = 
SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " by Country"
```

Weekly Title:
```DAX
Weekly Title = 
"Weekly " & SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " Trend"
```

Hourly Title:
```DAX
Hourly Title = 
"Hourly " & SELECTEDVALUE('Select Dynamic Measure'[Dynamic Title]) & " Trend"
```

Calendar Title:
```DAX
Calendar Title = "Analysis by Month"
```
(Not dynamic - shows all metrics in tooltip)

Matrix Title:
```DAX
Matrix Title = "Analysis by Ad Type"
```
(Not dynamic - shows multiple columns)

**Step 4: Display Titles**
- Use Card visual
- Add title measure to Values
- Format: Size 12, Segoe UI Semibold
- Remove background and border

**Result:**
- Select "Impressions" → All titles update
- Select "Engagements" → Titles change to "Engagements"
- Consistent across entire dashboard

### Conditional Formatting

**KPI Cards:**
- Individual background colors per metric
- Rounded corners for modern look
- White text for contrast

**Matrix:**
- Gradient backgrounds per column
- Visual hierarchy
- Easy comparison

**Charts:**
- Custom color schemes
- Brand-aligned palette (Meta blue, Instagram purple/pink)

### Tooltip Pages

**Benefit:** Hover over data point, see detailed breakdown

**Calendar Heat Map Example:**
- Hover over June 5
- See all 12 KPIs for that date
- No need to filter entire dashboard

**Implementation:**
- Create duplicate page
- Set as Tooltip type
- Arrange KPIs vertically
- Apply to visual

---

## BEST PRACTICES

### Data Modeling

**Golden Rules:**
1. **Always create date tables** (Microsoft recommendation)
2. **Use star schema** (or snowflake for complex data)
3. **Verify relationships** (cardinality, filter direction)
4. **One fact table, multiple dimensions** (clear structure)
5. **Primary keys must be unique** (no duplicates)
6. **Foreign keys can repeat** (many-to-one relationships)

**Validation:**
- Cross-check with SQL queries
- Verify numbers match source data
- Test all filter interactions

**Impact of Bad Modeling:**
- Wrong relationships = Wrong numbers
- Incorrect filters = Wrong insights
- Bad joins = Business decisions impacted

### Dashboard Design

**Visual Guidelines:**
1. **Consistent color scheme** (aligned with brand)
2. **Adequate white space** (not cluttered)
3. **Logical grouping** (related metrics together)
4. **Clear titles** (descriptive and dynamic)
5. **Minimal borders** (clean look)

**Usability:**
- **Important metrics top** (KPIs most visible)
- **Interactive elements clear** (obvious what's clickable)
- **Filters accessible** (left panel standard)
- **Navigation intuitive** (clear page switching)

**Performance:**
- **Limit visuals per page** (8-10 maximum)
- **Optimize DAX** (avoid complex, nested calculations)
- **Use measures, not calculated columns** (when possible)
- **Minimize relationships** (only necessary ones)

### Professional Touches

**Company Branding:**
1. Visit client website
2. Extract color palette
3. Use same colors in dashboard
4. Add company logo
5. Match font styles

**Consistency:**
- Same fonts across all visuals
- Same border styles
- Same spacing
- Same rounded corners

**Documentation:**
- Business requirements document
- Domain knowledge document
- Dashboard insights document
- Project explanation guide

**Storytelling:**
- Lead with key insights
- Use numbers in explanations
- Show trends over time
- Highlight actionable items


---

## PROVIDED DOCUMENTS

### 1. Business Requirements Document (BRD)

**Contents:**
- Business objectives
- Scope (in/out)
- KPI definitions (12 metrics)
- Chart requirements (7 visualizations)
- Data sources
- Acceptance criteria

**Use:** Foundation for building dashboard

### 2. Domain Knowledge Document

**Contents:**
- Data explanation
- Table descriptions (5 tables)
- Column meanings
- Relationship logic
- Business context
- Digital marketing terminology

**Use:** Understanding data before building

### 3. Dashboard Insights Document

**Contents:**
- Key findings from data
- Metric interpretations
- Trend analysis
- Actionable recommendations
- Performance benchmarks

**Use:** Explaining dashboard to stakeholders

### 4. Project Explanation Guide

**Contents:**
- Interview talking points
- Technical details to emphasize
- Business value statements
- Numbers to memorize
- Common questions & answers

**Use:** Interview preparation

### 5. Data Files

**Provided:**
- Ad Events (CSV/Excel)
- Ads (CSV/Excel)
- Campaigns (CSV/Excel)
- Users (CSV/Excel)

**Also included:**
- Images (Meta logo, Facebook logo, Instagram logo)
- Color codes (documented)

---

## KEY TAKEAWAYS

### Technical Skills Demonstrated

1. **Advanced Data Modeling**
   - Multi-table relationships
   - Star/snowflake schema
   - One-to-many cardinality
   - Filter direction control

2. **Complex DAX**
   - Calculated columns
   - Measures with logic
   - Parameters for dynamic behavior
   - SELECTEDVALUE for context
   - DIVIDE for safe division
   - COUNTROWS with FILTER
   - Date/time functions

3. **Professional Visualization**
   - 7 chart types used correctly
   - Interactive cross-filtering
   - Dynamic titles
   - Custom tooltips
   - Consistent formatting
   - Brand-aligned colors

4. **User Experience**
   - Intuitive navigation
   - Clear filters
   - Responsive interactions
   - Drill-through capability
   - Mobile-ready design

### Business Analysis Skills

1. **Digital Marketing Metrics**
   - Impressions, Clicks, CTR
   - Engagement rate
   - Conversion tracking
   - Purchase funnel
   - Budget allocation

2. **Audience Segmentation**
   - Demographics (age, gender, country)
   - Behavioral patterns (time, day)
   - Interest targeting
   - Platform preferences

3. **Performance Optimization**
   - ROI calculation
   - Cost per acquisition
   - A/B testing (ad types)
   - Timing optimization
   - Budget efficiency

### Industry Relevance

**This project demonstrates:**
- **Real-time data analysis** (2025 data)
- **Large-scale datasets** (400K+ rows)
- **Multi-platform comparison** (Facebook/Instagram)
- **Campaign tracking** (50 campaigns)
- **Executive reporting** (KPI-focused)

**Transferable to:**
- Google Ads performance
- LinkedIn Ads tracking
- E-commerce analytics
- Marketing attribution
- Sales funnel analysis

---

## COMMON ISSUES & SOLUTIONS

### Issue 1: Relationship Not Creating

**Symptom:** Can't drag to create relationship

**Causes:**
- Data types don't match
- One column has blanks
- Values don't align

**Solution:**
```
1. Check data types (both must be Text or both Number)
2. Remove blanks: Power Query > Replace null with "Unknown"
3. Verify matching values exist
```

### Issue 2: Conditional Formatting Not Working

**Symptom:** Colors don't appear in matrix/chart

**Cause:** Calendar table not related to fact table

**Solution:**
```
Create relationship:
Calendar[Date] ----< Ad Events[Event Date]
```

**Result:** Conditional formatting immediately works

### Issue 3: Dynamic Title Shows Error

**Symptom:** #ERROR in title card

**Cause:** Parameter field hidden

**Solution:**
```
1. Go to Select Dynamic Measure table
2. Right-click "Select Measure Order"
3. Unhide in report view
4. Use in DAX formula
```

### Issue 4: Map Not Publishing to Service

**Symptom:** Map visual blank online

**Cause:** Traditional map being retired

**Solution:**
```
Upgrade to Azure Map (if available)
Or: Use different visualization (table, matrix)
```

### Issue 5: Slicers Filtering Wrong Visuals

**Symptom:** Month slicer affects all charts

**Cause:** Default edit interactions

**Solution:**
```
1. Select month slicer
2. Format > Edit Interactions
3. Click "None" on all visuals except calendar heat map
```

### Issue 6: KPI Cards Not Showing All Values

**Symptom:** Only 1-2 KPIs visible

**Cause:** Maximum cards not set

**Solution:**
```
Visual > Multicard > Maximum cards shown: 6
```

### Issue 7: Week Numbers Don't Align

**Symptom:** Calendar heat map looks wrong

**Cause:** WEEKNUM parameter incorrect

**Solution:**
```DAX
Week Number = WEEKNUM('Calendar Table'[Date], 2)
```
Parameter 2 = Monday as week start

### Issue 8: Percentage Shows Decimal

**Symptom:** CTR shows 0.11 instead of 11%

**Cause:** Not formatted as percentage

**Solution:**
```
Select measure > Measure Tools > Format > Percentage
```

---

## ENHANCEMENTS & EXTENSIONS

### Additional Features to Add

**1. Year-over-Year Comparison:**
```DAX
YoY Growth = 
DIVIDE(
    [Impressions] - CALCULATE([Impressions], SAMEPERIODLASTYEAR('Calendar Table'[Date])),
    CALCULATE([Impressions], SAMEPERIODLASTYEAR('Calendar Table'[Date])),
    0
)
```

**2. Moving Average:**
```DAX
7-Day Moving Avg = 
AVERAGEX(
    DATESINPERIOD('Calendar Table'[Date], LASTDATE('Calendar Table'[Date]), -7, DAY),
    [Engagements]
)
```

**3. Benchmark Comparison:**
- Add industry benchmark data
- Compare actual vs target
- Show variance

**4. Cost Metrics:**
```DAX
Cost Per Click = DIVIDE([Total Budget], [Clicks], 0)
Cost Per Acquisition = DIVIDE([Total Budget], [Purchases], 0)
ROAS = DIVIDE([Revenue], [Total Budget], 0)
```
(Note: Would need revenue data)

**5. Predictive Analytics:**
- Forecast next month's performance
- Trend lines
- Confidence intervals

**6. Drill-Through Pages:**
- Campaign details page
- User demographics deep-dive
- Ad performance details

**7. Mobile Layout:**
- Create mobile-optimized page
- Vertical card arrangement
- Simplified visuals

**8. Alerts & Notifications:**
- Set up alerts in Power BI Service
- Email when CTR < 5%
- Notify when budget exceeds threshold

---

## CONCLUSION

### What Was Accomplished

**Technical:**
- ✅ Multi-table data model (snowflake schema)
- ✅ 5 data sources integrated
- ✅ 12+ DAX measures created
- ✅ 7 interactive visualizations
- ✅ Dynamic parameters implemented
- ✅ Custom tooltips designed
- ✅ Separate platform pages
- ✅ Published to Power BI Service

**Business:**
- ✅ Campaign ROI tracking
- ✅ Platform comparison (Facebook/Instagram)
- ✅ Audience segmentation insights
- ✅ Timing optimization
- ✅ Budget allocation guidance
- ✅ Ad format performance

### Key Learnings

1. **Data modeling is foundational** - Everything depends on correct relationships
2. **DAX enables advanced analytics** - Beyond simple sums and counts
3. **Dynamic features improve UX** - Parameters, titles, tooltips
4. **Cross-filtering enhances discovery** - Interactive exploration
5. **Design consistency matters** - Professional appearance builds trust
6. **Business context is essential** - Technical skills + domain knowledge
7. **Validation is non-negotiable** - Always verify numbers

### Real-World Application

**This dashboard enables:**
- **Marketing teams:** Optimize campaigns, allocate budget
- **Executives:** Track ROI, make strategic decisions
- **Agencies:** Report to clients, demonstrate value
- **Analysts:** Identify trends, provide recommendations

**Similar projects:**
- Google Ads performance
- E-commerce sales funnel
- Social media analytics
- Email campaign tracking
- Website traffic analysis

### Portfolio Value

**Why this project stands out:**
1. **Real-time data** (2025)
2. **Large-scale** (400K+ rows)
3. **Multi-table** (advanced modeling)
4. **Industry-relevant** (digital marketing)
5. **Fully interactive** (published online)
6. **Well-documented** (4 support documents)
7. **Polished design** (professional appearance)

**For job seekers:**
- Add to resume (with link to live dashboard)
- Explain in interviews (use provided guide)
- Showcase in portfolio (embed on website)
- Demonstrate on LinkedIn (publish link)

---

*This comprehensive dashboard demonstrates enterprise-level Power BI skills including advanced data modeling, complex DAX calculations, interactive visualizations, and business storytelling - all essential for modern data analyst roles.*
