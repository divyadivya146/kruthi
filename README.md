# Uber Ride Bookings Dashboard – Power BI

## 1. Project overview

This project analyses Uber ride-booking data using **Microsoft Power BI**. The purpose of the dashboard is to turn a large booking dataset into clear visual information that helps users understand ride demand, booking outcomes, vehicle usage, revenue, cancellations, and customer/driver ratings.

The project follows a typical Power BI workflow:

1. Import the dataset.
2. Inspect and clean the data in Power Query Editor.
3. Load the prepared data into Power BI.
4. Create DAX measures for calculations.
5. Build the data model.
6. Create report pages and visuals.
7. Use filters and slicers to explore the results.

> **Files used**
> - `ncr_ride_bookings.csv` – source dataset.
> - `Uber.pbix` – Power BI report file.

## 2. Dataset description

The supplied CSV contains **150,000 rows and 21 columns**. Each row represents a ride-booking record. The data covers dates in 2024 and includes booking details, pickup and drop locations, vehicle type, status, timing, fare value, distance, ratings, and payment method.

### Column explanation

| Column | Meaning |
|---|---|
| Date | Date on which the ride was booked |
| Time | Booking time |
| Booking ID | Unique or identifying booking reference |
| Booking Status | Outcome/status of the booking |
| Customer ID | Customer reference |
| Vehicle Type | Type of vehicle selected |
| Pickup Location | Starting location |
| Drop Location | Destination |
| Avg VTAT | Average Vehicle Time to Arrive |
| Avg CTAT | Average Customer Time to Arrive, as named in the source |
| Cancelled Rides by Customer | Indicator/count field for customer-cancelled rides |
| Reason for cancelling by Customer | Reason supplied for customer cancellation |
| Cancelled Rides by Driver | Indicator/count field for driver-cancelled rides |
| Driver Cancellation Reason | Reason supplied for driver cancellation |
| Incomplete Rides | Indicator/count field for incomplete rides |
| Incomplete Rides Reason | Reason a ride was incomplete |
| Booking Value | Booking amount/value |
| Ride Distance | Distance travelled or recorded for the ride |
| Driver Ratings | Rating given to the driver |
| Customer Rating | Customer rating recorded for the booking |
| Payment Method | Method used for payment |

**Dataset observations:** the CSV contains five booking-status categories and seven vehicle types. Some columns are blank for many records because not every field applies to every booking. For example, cancellation reasons are expected to be populated only for cancelled bookings, while fare and rating information may be unavailable for unsuccessful bookings. Missing values should therefore be interpreted in context rather than automatically replaced with zero.

## 3. Importing the dataset into Power BI

1. Open **Power BI Desktop**.
2. Select **Home → Get data → Text/CSV**.
3. Browse to `ncr_ride_bookings.csv`.
4. Select the file and preview the data.
5. Choose **Transform Data** to open Power Query Editor. This allows the data to be checked and prepared before loading.
6. After completing the required transformations, select **Close & Apply**.

Power BI creates a query from the CSV. The imported table is the source for the report visuals and calculations.

## 4. Power Query Editor – data preparation

Power Query Editor is the data-cleaning and transformation area of Power BI. It is used to correct data types, remove unwanted records or columns, standardise values, and prepare the dataset for analysis. Transformations are saved as **Applied Steps** and are repeated when the data is refreshed.

### Recommended preparation steps for this dataset

The exact Applied Steps configured in the supplied PBIX could not be independently verified from the report package. The following describes the appropriate preparation workflow for the supplied CSV; it should not be read as a claim that every step was applied in the existing file.

#### Step 1: Check column names

- Confirm that all 21 columns have meaningful names.
- Make sure there are no accidental blank columns.
- Keep names consistent so they are easy to select in DAX.

#### Step 2: Set correct data types

Set the columns to suitable types:

| Columns | Recommended type |
|---|---|
| Date | Date |
| Time | Time |
| Booking ID, Customer ID | Text |
| Booking Status, Vehicle Type, locations, reasons, Payment Method | Text |
| Avg VTAT, Avg CTAT | Decimal number |
| Cancellation and Incomplete indicators | Whole number or decimal, depending on the source meaning |
| Booking Value, Ride Distance, ratings | Decimal number |

Correct data types are important. For example, a date stored as text may not sort or filter correctly in a date slicer, and a booking value stored as text cannot be summed reliably.

#### Step 3: Clean text fields

- Select text columns and use **Transform → Format → Trim** to remove unnecessary spaces at the beginning or end.
- Use **Clean** if unwanted non-printing characters are present.
- Check spelling and inconsistent labels in status, vehicle, payment, and reason columns.
- Preserve the original meaning of categories; do not merge different categories without a clear business rule.

#### Step 4: Handle missing values

- Review nulls in timing, cancellation reasons, booking value, distance, and ratings.
- Do not replace every blank with zero. A blank rating means “not recorded”, not necessarily a rating of zero.
- For cancellation-reason analysis, filter to the relevant cancelled bookings.
- If a calculation requires a value, define the treatment explicitly and document it.

#### Step 5: Check duplicate booking IDs

The CSV has 150,000 rows and 148,767 distinct Booking ID values, so the ID column is not unique in the raw file. Before removing duplicates, investigate why IDs repeat. They may be repeated records or multiple records associated with one booking. Do not remove rows solely because the ID repeats; confirm the intended grain of the dataset first.

#### Step 6: Validate numerical values

Check that:
- Booking Value is not negative unless refunds/adjustments are intentionally represented.
- Ride Distance is within a sensible range.
- Ratings are within the expected rating scale.
- VTAT and CTAT values are plausible.
- Indicator columns contain the expected values.

#### Step 7: Apply changes

Select **Home → Close & Apply**. Power BI loads the transformed data into the model.

### What happens after transformation?

Power Query records the transformation instructions. When the source data is refreshed, Power BI runs those steps again. This makes the data preparation process repeatable and reduces the need to clean the file manually each time.

## 5. Data modelling

Data modelling is the process of organising tables and defining relationships so that report calculations and filters work correctly.

### Model structure

The supplied CSV is a single flat booking table. If the PBIX uses only this imported table, the model is a **single-table model**: each booking record is stored with its date, customer, vehicle, locations, status, payment, fare, distance, and ratings.

A single-table model can be sufficient for a small project. For a more scalable model, the booking table can be treated as a fact table and connected to separate dimension tables.

### Recommended star-schema design

- **FactBookings** – one row per booking at the confirmed booking grain. Contains booking identifiers and numeric values such as booking value, distance, and timing.
- **DimDate** – date, year, quarter, month, month number, and day.
- **DimVehicle** – vehicle type.
- **DimCustomer** – customer identifier.
- **DimLocation** – pickup/drop location attributes, if the analysis requires them.
- **DimPayment** – payment method.
- **DimBookingStatus** – booking status and status grouping.

In a star schema, dimension tables filter the fact table through one-to-many relationships. The dimension-side key should be unique. Use single-direction filtering in most cases unless a specific analysis requires otherwise.

### Creating a relationship

1. Open **Model view** in Power BI Desktop.
2. Add or create the dimension tables if using a star schema.
3. Ensure the key column on the dimension side contains unique values.
4. Drag the dimension key to the matching foreign key in the fact table, or use **Manage relationships → New**.
5. Confirm the cardinality and filter direction.
6. Test the relationship using a table visual and slicers.

For a date table, mark it as the date table using **Table tools → Mark as date table** after ensuring it contains a continuous, unique date column.

> **Important:** The actual relationship diagram and cardinalities in the supplied PBIX were not fully recoverable from the available report metadata. The design above is a recommended modelling explanation, not a verified diagram of every relationship in the existing file.

## 6. Measures and DAX

A **measure** is a calculation created using Data Analysis Expressions (DAX). Unlike a calculated column, a measure is evaluated when a visual is queried and responds to filters and slicers. Measures are useful for totals, averages, percentages, and comparisons.

To create a measure:

1. Select the table in the Fields/Data pane.
2. Select **Table tools or Modeling → New measure**.
3. Enter the DAX formula in the formula bar.
4. Press Enter.
5. Give the measure a meaningful name.
6. Set its format (whole number, decimal, currency, or percentage).
7. Add it to a visual.

### Example measures for this dataset

The following are example measures that can be used or adapted. They are **not confirmed copies of the exact DAX formulas stored in the supplied PBIX**. Adjust table and column names if your Power BI model uses different names.

Assume the imported table is named `ncr_ride_bookings`.

```DAX
Total Bookings =
COUNTROWS('ncr_ride_bookings')
```

Counts the rows currently visible after filters are applied. Use this as the total record count only if one row represents one booking.

```DAX
Successful Bookings =
CALCULATE(
    COUNTROWS('ncr_ride_bookings'),
    'ncr_ride_bookings'[Booking Status] = "Completed"
)
```

Counts records whose status is Completed.

```DAX
Cancelled Bookings =
CALCULATE(
    COUNTROWS('ncr_ride_bookings'),
    FILTER(
        'ncr_ride_bookings',
        CONTAINSSTRING(
            'ncr_ride_bookings'[Booking Status],
            "Cancel"
        )
    )
)
```

Counts rows whose status text contains “Cancel”. Confirm the actual status labels before relying on this definition.

```DAX
Cancellation Rate =
DIVIDE(
    [Cancelled Bookings],
    [Total Bookings],
    0
)
```

Calculates the share of bookings classified as cancelled. Format the measure as Percentage. The definition of cancellation should match the business rule used in the report.

```DAX
Total Revenue =
CALCULATE(
    SUM('ncr_ride_bookings'[Booking Value]),
    'ncr_ride_bookings'[Booking Status] = "Completed"
)
```

Sums booking value for completed bookings. This is a suggested revenue definition; confirm whether the project treats all booking values or only completed rides as revenue.

```DAX
Average Ride Distance =
AVERAGE('ncr_ride_bookings'[Ride Distance])
```

Calculates average recorded ride distance.

```DAX
Average Driver Rating =
AVERAGE('ncr_ride_bookings'[Driver Ratings])
```

Calculates the average available driver rating.

```DAX
Average Customer Rating =
AVERAGE('ncr_ride_bookings'[Customer Rating])
```

Calculates the average available customer rating.

```DAX
Average VTAT =
AVERAGE('ncr_ride_bookings'[Avg VTAT])
```

Calculates the average recorded vehicle-arrival time.

```DAX
Average CTAT =
AVERAGE('ncr_ride_bookings'[Avg CTAT])
```

Calculates the average recorded CTAT value.

**Why use `DIVIDE`?** It safely handles a denominator of zero and can return a specified alternate result. Measures also respond to filter context: selecting a month, vehicle, or payment method changes the calculation to reflect that selection.

## 7. Deep explanation of the dashboard

A dashboard/report is a visual way of presenting important information from a dataset. Instead of reading thousands of rows, a user can understand performance through cards, charts, tables, and slicers.

The supplied Power BI report contains six report pages. The descriptions below are based on the page names and visual titles/types visible in the PBIX report layout.

### 7.1 Home Page

The Home Page is the starting or navigation page of the report. It contains action buttons. These buttons can be used to move to the detailed report sections.

**Purpose**
- Give the report a clear starting point.
- Help users navigate to the required analysis page.
- Make the report easier to use.

### 7.2 Overall

This page gives a high-level view of booking performance.

Visible report elements include:
- **Total Bookings** card – displays the overall booking count according to the measure used.
- **Ride Volume Over Time** line chart – shows how ride volume changes over dates. Peaks can indicate high-demand days, while drops can indicate lower demand.
- **Booking Status Breakdown** pie chart – compares the distribution of booking statuses, helping users see the proportion of completed, cancelled, incomplete, or other booking outcomes.
- **Date slicer** – filters the page to a selected date or date range.

**How to interpret it:** Start with the total-bookings card to understand the size of the selected data. Then use the line chart to identify demand patterns and the status chart to understand booking outcomes. Select a date range to focus the analysis on a particular period.

### 7.3 Vehicle Type

This page focuses on the different vehicle categories used for bookings. The PBIX layout shows multiple card visuals and a date slicer on this page.

**Purpose**
- Compare the performance of vehicle categories.
- Understand which vehicle types receive more bookings.
- Help identify differences in demand between vehicle types.

**How to interpret it:** Compare the values shown for each vehicle category. A higher booking count indicates greater booking volume for that category, while other card metrics should be interpreted according to their displayed labels. Use the date filter, where enabled, to compare the same categories over a selected period.

### 7.4 Revenue

This page focuses on the financial and distance-related aspects of ride bookings.

Visible report elements include:
- **Revenue by Payment Method** column chart – compares revenue or booking value across payment methods.
- **Ride Distance Distribution** column chart – shows how recorded ride distances are distributed across the categories/bins configured in the visual.
- **Top 5 Customers** table – lists the five leading customers according to the field and ranking measure configured in the report.
- **Date slicer** – allows the user to focus on a selected date range.

**How to interpret it:** Use the payment-method chart to compare the contribution of different payment methods. Use the distance chart to understand the distribution of ride lengths. The customer table helps identify customers with high values under the ranking metric. Check the visual's field/measure definition to confirm whether “Top 5” means revenue, booking count, or another measure.

### 7.5 Cancellation

This page examines unsuccessful bookings and cancellation behaviour.

Visible report elements include:
- **Cancelled Rides by Customers** pie chart – shows customer cancellation categories/reasons as configured.
- **Cancelled Rides by Drivers** pie chart – shows driver cancellation categories/reasons as configured.
- **Total Bookings** card.
- **Success Bookings** card.
- **Cancelled Bookings** card.
- **Cancellation Rate** card.
- **Date slicer**.

**How to interpret it:** Compare customer and driver cancellation reasons to identify common causes. Review successful and cancelled booking totals together rather than relying only on the percentage. The cancellation rate shows the proportion of bookings classified as cancelled under the report's DAX definition. Use the date slicer to see whether cancellation patterns change over time.

**Business value:** This analysis can help identify operational issues such as driver availability, customer changes, incorrect pickup details, or other recorded cancellation reasons. The report can support investigation, but it does not by itself prove why a cancellation occurred.

### 7.6 Ratings

This page contains multiple card visuals for rating-related information. The page name and card visual types are visible in the report layout, but the individual card titles were not reliably available in the extracted metadata.

The source dataset includes both **Driver Ratings** and **Customer Rating**.

**Purpose**
- Summarise rating information.
- Monitor customer experience and driver performance.
- Help identify whether recorded ratings are high or low.

**How to interpret it:** Read each card's title and value to identify the metric being summarised. Compare driver and customer ratings only after confirming that the cards use the intended columns and aggregation (for example, average rating rather than count). Ratings may be blank for bookings where no rating was recorded, so the average is based on available values.

## 8. Understanding report visuals and interactions

### Cards
A card displays one important number, such as total bookings, successful bookings, or cancellation rate. Cards are useful for quick performance summaries.

### Line chart
A line chart displays changes over time. The horizontal axis normally represents date, and the vertical axis represents booking volume. It helps identify trends, peaks, and drops.

### Pie chart
A pie chart shows how a total is divided among categories. It is useful for a small number of booking statuses or cancellation reasons. Very large numbers of categories can make a pie chart difficult to read.

### Column chart
A column chart compares values across categories, such as payment methods or distance groups. Taller columns represent larger values.

### Table
A table presents detailed records or rankings, such as the Top 5 Customers list. It is useful when the user needs to read names and exact values rather than only see a trend.

### Slicers and cross-filtering
A slicer is an interactive filter. Selecting a date range can update the visuals on a page. Selecting a data point in a chart may also filter or highlight related values in other visuals, depending on the visual interactions configured in Power BI.

## 9. How to use the dashboard

1. Open `Uber.pbix` in Power BI Desktop.
2. Start at **Home Page**.
3. Select a navigation button to open a report section.
4. On a page with a Date slicer, choose the date or range you want to analyse.
5. Read the KPI cards first to understand the main totals.
6. Use charts to compare categories and identify patterns.
7. Select a chart category to cross-filter other visuals, if interactions are enabled.
8. Clear selections or reset slicers to return to the full view.
9. Use **File → Save** after making report changes.

## 10. Key business questions this dashboard can help answer

- How many booking records are present in the selected period?
- How does ride volume change over time?
- What proportion of bookings are completed, cancelled, or incomplete?
- Which vehicle types have higher booking volume?
- How does booking value vary by payment method?
- What is the distribution of ride distances?
- Which customers rank highest under the Top 5 metric?
- What reasons are most frequently recorded for customer and driver cancellations?
- What are the average recorded driver and customer ratings?

The answers depend on the selected filters, the definitions used in the DAX measures, and the quality of the source data.

## 11. Refreshing the report

If the CSV is updated:

1. Keep the source file in the same location, or update the file path in the query's **Source** step.
2. Open the PBIX file.
3. Select **Home → Refresh**.
4. Check that the refresh completes without errors.
5. Review key totals and visuals to confirm the new data has loaded.
6. Save the PBIX file.

If the report is published to Power BI Service, configure the appropriate data source credentials and refresh settings there.

## 12. Limitations and points to verify

- The CSV has repeated Booking ID values. Confirm the intended row grain before treating row count as unique bookings or removing duplicates.
- Many fields contain nulls. A missing value is not automatically zero.
- Revenue should be defined consistently, for example as booking value from completed rides if that is the project's business rule.
- The exact DAX expressions and all Power Query Applied Steps in the supplied PBIX were not independently recoverable from the report layout metadata. The formulas in this README are examples, not a verified inventory of measures in the file.
- The report layout confirms page names and several visual titles/types, but it does not establish every visual's field assignment, formatting, interaction, or card title.
- Confirm the meaning and unit of VTAT and CTAT with the dataset documentation or project owner before presenting them as operational definitions.

## 13. Conclusion

The Uber Ride Bookings Dashboard converts a 150,000-row ride-booking dataset into an interactive report. Power Query Editor prepares the data, the data model organises it for analysis, and DAX measures calculate important business indicators. The six report pages provide views of overall booking activity, vehicle types, revenue, cancellations, and ratings.

Together, these features make it easier to explore ride-booking performance, identify patterns, and support data-informed decisions.

---

## Project summary

| Component | Description |
|---|---|
| Tool | Microsoft Power BI Desktop |
| Source | `ncr_ride_bookings.csv` |
| Report | `Uber.pbix` |
| Dataset size | 150,000 rows × 21 columns |
| Main workflow | Import → Power Query → Data model → DAX measures → Visualisation |
| Report pages | Home Page, Overall, Vechical Type, Renvenue, Cancellation, Ratings |
| Analysis areas | Bookings, vehicle types, revenue, distance, cancellations, ratings |
